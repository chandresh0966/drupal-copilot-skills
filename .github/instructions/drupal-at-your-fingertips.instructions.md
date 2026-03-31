---
name: drupal-at-your-fingertips
description: Comprehensive Drupal patterns from "Drupal at Your Fingertips" by Selwyn Polit. Covers 50+ topics including services, hooks, events, plugins, entities, forms, routing, theming, caching, testing, Batch API, Queue API, cron, AJAX, and JavaScript.
applyTo: "**/*.php,**/*.module,**/*.inc,**/*.install,**/*.theme,**/templates/**/*.twig"
---

# Drupal at Your Fingertips

**Source**: [drupalatyourfingertips.com](https://drupalatyourfingertips.com)
**Author**: Selwyn Polit

Comprehensive Drupal development reference covering 50+ topics.

## Dependency Injection & Services

```php
// Define a service in *.services.yml
services:
  mymodule.my_service:
    class: Drupal\mymodule\MyService
    arguments: ['@entity_type.manager', '@logger.factory']

// Inject via constructor
class MyService {
  public function __construct(
    protected EntityTypeManagerInterface $entityTypeManager,
    protected LoggerChannelFactoryInterface $loggerFactory,
  ) {}

  public static function create(ContainerInterface $container) {
    return new static(
      $container->get('entity_type.manager'),
      $container->get('logger.factory'),
    );
  }
}
```

## Hooks

```php
// hook_entity_presave — alter entities before save
function mymodule_entity_presave(EntityInterface $entity) {
  if ($entity instanceof NodeInterface && $entity->getType() === 'article') {
    $entity->set('field_last_modified', \Drupal::time()->getRequestTime());
  }
}

// hook_form_alter — alter forms
function mymodule_form_alter(&$form, FormStateInterface $form_state, $form_id) {
  if ($form_id === 'node_article_form') {
    $form['#validate'][] = 'mymodule_article_validate';
  }
}

// hook_theme — define Twig templates
function mymodule_theme() {
  return [
    'my_template' => [
      'variables' => ['title' => NULL, 'items' => []],
    ],
  ];
}
```

## Event Subscribers

```php
// Register in *.services.yml
services:
  mymodule.event_subscriber:
    class: Drupal\mymodule\EventSubscriber\MyEventSubscriber
    tags:
      - { name: event_subscriber }

// Event subscriber class
class MyEventSubscriber implements EventSubscriberInterface {
  public static function getSubscribedEvents(): array {
    return [
      KernelEvents::REQUEST => ['onRequest', 10],
    ];
  }

  public function onRequest(RequestEvent $event): void {
    // Handle the event
  }
}
```

## Plugins

```php
// Plugin annotation
/**
 * @Block(
 *   id = "my_block",
 *   admin_label = @Translation("My Block"),
 *   category = @Translation("Custom"),
 * )
 */
class MyBlock extends BlockBase {
  public function build(): array {
    return ['#markup' => $this->t('Hello World')];
  }
}
```

## Entity API & Custom Entities

```php
// Load entities
$node = \Drupal::entityTypeManager()->getStorage('node')->load($nid);
$nodes = \Drupal::entityTypeManager()->getStorage('node')->loadMultiple($nids);

// Entity query
$nids = \Drupal::entityQuery('node')
  ->condition('type', 'article')
  ->condition('status', 1)
  ->sort('created', 'DESC')
  ->range(0, 10)
  ->accessCheck(TRUE)
  ->execute();

// Access field values
$title = $node->get('title')->value;
$body = $node->get('body')->value;
$image_fid = $node->get('field_image')->target_id;
```

## Form API

```php
class MyForm extends FormBase {
  public function getFormId(): string {
    return 'mymodule_my_form';
  }

  public function buildForm(array $form, FormStateInterface $form_state): array {
    $form['name'] = [
      '#type' => 'textfield',
      '#title' => $this->t('Name'),
      '#required' => TRUE,
    ];
    $form['submit'] = [
      '#type' => 'submit',
      '#value' => $this->t('Submit'),
    ];
    return $form;
  }

  public function validateForm(array &$form, FormStateInterface $form_state): void {
    if (strlen($form_state->getValue('name')) < 3) {
      $form_state->setErrorByName('name', $this->t('Name must be at least 3 characters.'));
    }
  }

  public function submitForm(array &$form, FormStateInterface $form_state): void {
    \Drupal::messenger()->addStatus($this->t('Form submitted successfully.'));
  }
}
```

## Routing & Controllers

```yaml
# mymodule.routing.yml
mymodule.my_page:
  path: '/my-page/{node}'
  defaults:
    _controller: '\Drupal\mymodule\Controller\MyController::myPage'
    _title: 'My Page'
  requirements:
    _permission: 'access content'
    node: \d+
```

```php
class MyController extends ControllerBase {
  public function myPage(NodeInterface $node): array {
    return [
      '#theme' => 'my_template',
      '#title' => $node->getTitle(),
    ];
  }
}
```

## Twig Theming

```twig
{# templates/node--article.html.twig #}
{% extends "node.html.twig" %}

{% block content %}
  <article{{ attributes }}>
    <h1>{{ label }}</h1>
    {% if content.field_image %}
      {{ content.field_image }}
    {% endif %}
    <div class="body">
      {{ content.body }}
    </div>
  </article>
{% endblock %}
```

```php
// hook_preprocess_node — add variables to templates
function mymodule_preprocess_node(&$variables) {
  $node = $variables['node'];
  $variables['custom_var'] = $node->get('field_custom')->value;
}
```

## Caching

```php
// Render array cache tags and contexts
$build = [
  '#markup' => $content,
  '#cache' => [
    'tags' => ['node:' . $node->id(), 'node_list'],
    'contexts' => ['user.roles', 'url.query_args'],
    'max-age' => Cache::PERMANENT,
  ],
];

// Invalidate cache tags programmatically
Cache::invalidateTags(['node:' . $nid]);
\Drupal::service('cache_tags.invalidator')->invalidateTags(['node_list']);
```

## Batch API

```php
function mymodule_start_batch(array $nids): void {
  $operations = [];
  foreach (array_chunk($nids, 50) as $chunk) {
    $operations[] = ['\Drupal\mymodule\Batch\MyBatch::process', [$chunk]];
  }

  batch_set([
    'title' => t('Processing items...'),
    'operations' => $operations,
    'finished' => '\Drupal\mymodule\Batch\MyBatch::finished',
  ]);
}

class MyBatch {
  public static function process(array $nids, array &$context): void {
    foreach ($nids as $nid) {
      // Process each node
    }
    $context['results'][] = count($nids);
  }
}
```

## Queue API

```php
// Add items to queue
$queue = \Drupal::queue('mymodule_my_queue');
$queue->createItem(['nid' => $nid, 'action' => 'update']);

// Queue worker plugin
/**
 * @QueueWorker(
 *   id = "mymodule_my_queue",
 *   title = @Translation("My Queue Worker"),
 *   cron = {"time" = 60}
 * )
 */
class MyQueueWorker extends QueueWorkerBase {
  public function processItem($data): void {
    $node = Node::load($data['nid']);
    // Process the item
  }
}
```

## Testing (PHPUnit / DTT)

```php
// ExistingSite test with Drupal Test Traits
use weitzman\DrupalTestTraits\ExistingSiteBase;

#[Group('custom')]
class MyFeatureTest extends ExistingSiteBase {

  public function testNodeCreation(): void {
    $node = $this->createNode([
      'type' => 'article',
      'title' => 'Test Article',
    ]);
    $this->assertNotNull($node->id());
    $this->assertEquals('Test Article', $node->getTitle());
  }
}
```

```bash
# Run specific test
vendor/bin/phpunit --filter MyFeatureTest::testNodeCreation

# Run full suite
vendor/bin/phpunit --testsuite custom
```

## Key Drush Commands

```bash
drush cr                          # Clear caches
drush updb -y                     # Run database updates
drush cim / ddev drush cex        # Config import/export
drush watchdog:show --severity=Error  # View error logs
drush php:eval "..."              # Evaluate PHP
drush generate                    # Generate scaffolding
```
