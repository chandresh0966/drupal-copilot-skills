---
name: drupal-search-api
description: Search API configuration, boosting strategies, and processor patterns for Drupal. Covers index configuration, field types, custom boost processors, number field boosting, engagement metrics, and reindexing workflows.
applyTo: "**"
---

# Drupal Search API Patterns

This skill documents Search API configuration patterns, boosting strategies, and custom
processor development for Drupal sites.

## Index Configuration

### Field Type Requirements

**Boolean Fields for Boosting** — Always configure as `type: boolean`:
```yaml
field_settings:
  field_featured:
    label: Featured
    datasource_id: 'entity:node'
    property_path: field_featured
    type: boolean  # NOT integer
    boost: 8.0
```

**Numeric Fields for Engagement Boosting**:
```yaml
field_settings:
  flag_bookmark_count:
    label: 'Bookmark count'
    property_path: flag_bookmark_count
    type: integer
  flag_favorite_count:
    label: 'Favorite count'
    property_path: flag_favorite_count
    type: integer
```

## Custom Boolean Boost Processor

Use for boolean field boosting (featured flags, promoted content):

```php
<?php
namespace Drupal\custom_search\Plugin\search_api\processor;

use Drupal\search_api\Item\ItemInterface;
use Drupal\search_api\Processor\ProcessorPluginBase;

/**
 * @SearchApiProcessor(
 *   id = "featured_content_boost",
 *   label = @Translation("Featured content boost"),
 *   description = @Translation("Adds a boost to indexed items marked as featured."),
 *   stages = { "preprocess_index" = 0 },
 *   locked = false,
 *   hidden = false,
 * )
 */
class FeaturedContentBoost extends ProcessorPluginBase {

  public function preprocessIndexItems(array $items) {
    foreach ($items as $item) {
      try {
        $entity = $item->getOriginalObject()->getValue();
        if ($entity->hasField('field_featured') &&
            !$entity->get('field_featured')->isEmpty() &&
            $entity->get('field_featured')->value == 1) {
          $old_boost = $item->getBoost();
          // Apply 2x boost to featured content (multiplicative)
          $item->setBoost($old_boost * 2.0);
        }
      } catch (\Exception $e) {
        continue;
      }
    }
  }
}
```

**Key Points**:
- Use multiplicative boost: `$item->setBoost($old_boost * boost_factor)`
- Boost at index time via `preprocess_index` stage
- Always get old boost first to preserve other boosts
- Recommended boost factors: Featured `2.0x`, Premium `1.5x–2.0x`

## Number Field Boost Processor

For engagement metrics (likes, bookmarks, views):

```yaml
processor_settings:
  number_field_boost:
    weights:
      preprocess_index: 0
    boosts:
      flag_bookmark_count:
        boost_factor: 0.01
        aggregation: max
      flag_favorite_count:
        boost_factor: 0.1
        aggregation: max
```

**Recommended Boost Factors**:
- Bookmark count: `0.01–0.05` (for counts in 10–1000 range)
- Favorite count: `0.1–0.5` (for counts in 1–50 range)
- View count: `0.001–0.01` (for counts in 100–10000 range)

## Configuration Management

### Direct Config Updates with PHP

When `drush config:set` fails or causes side effects:

```bash
ddev drush php:eval "
\$config = \Drupal::configFactory()->getEditable('search_api.index.{index_name}');
\$config->set('field_settings.field_featured.type', 'boolean');
\$config->set('processor_settings.number_field_boost.boosts.flag_bookmark_count.boost_factor', 0.01);
\$config->save();
echo \"Configuration updated\n\";
"
```

**After PHP config updates**:
1. Verify: `ddev drush config:get search_api.index.{index_name} field_settings.field_featured`
2. Export: `ddev drush config:export -y`
3. Review: `git diff config/default/`
4. Revert any unintended changes

### Config Export Side Effects

When modifying Search API server config, Drupal may update index configs:
```yaml
# Unintended change in search_api.index.{index_name}.yml
server: null  # Changed from {server_name}
```

**Prevention**: Review `git diff config/default/` before committing.
**Recovery**: `git checkout HEAD -- config/default/search_api.index.*.yml`

## Reindexing After Changes

Always reindex after changing field type, boost factors, adding/removing processors:

```bash
ddev drush cr
ddev drush search-api:clear {index_name}
ddev drush search-api:index {index_name}
ddev drush search-api:status {index_name}
```

## Boost Stacking Example

```
Node: "Getting Started" (featured, 138 bookmarks)
- Base score: 1.0
- Featured: 1.0 x 2.0 = 2.0
- 138 bookmarks: 2.0 + (138 x 0.01) = 3.38
- Final boost: 3.38
```

## Performance: Index-Time vs Query-Time Boosting

**Index-Time Boosting (Recommended)**:
- Better performance (computed once during indexing)
- Scales with high query volume
- Requires reindex when boost logic changes

## Troubleshooting

### Boost Not Applied — Checklist

1. Processor enabled in index config
2. Field type correct (boolean/integer)
3. Full reindex completed
4. No conflicting processors
5. Cache cleared after config changes

```bash
ddev drush config:get search_api.index.{index_name} processor_settings
ddev drush config:get search_api.index.{index_name} field_settings
ddev drush search-api:status {index_name}
```

### Processor Conflicts

If multiple processors target the same field:
```bash
# Remove field from number_field_boost if using custom processor
ddev drush config:set search_api.index.{index_name} \
  processor_settings.number_field_boost.boosts.field_featured null
```

**Best Practice**: Use custom processor for boolean/complex logic, `number_field_boost` for simple numeric boosts. Don't configure the same field in multiple processors.
