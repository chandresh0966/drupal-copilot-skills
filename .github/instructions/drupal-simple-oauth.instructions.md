---
name: drupal-simple-oauth
description: OAuth2 authentication patterns for Drupal using simple_oauth module. Covers TokenAuthUser permission logic, scope/role matching, mobile app token flows, field_permissions integration, CSRF bypass, and debugging token issues.
applyTo: "**"
---

# Drupal Simple OAuth Patterns

Comprehensive patterns for working with the `simple_oauth` module for OAuth2 authentication in
Drupal. Use when working with API authentication, mobile app tokens, or OAuth2 implementation.

## Critical OAuth Token Concepts

### TokenAuthUser: The Core Authentication Wrapper

When a request is authenticated with an OAuth token, Drupal wraps the user in a
`TokenAuthUser` decorator that enforces BOTH token AND user permissions.

**Permission Check Logic** — BOTH the token AND the user must have the permission (AND condition):
```php
public function hasPermission($permission) {
  if ((int) $this->id() === 1) {
    return TRUE;  // User #1 bypasses all checks
  }
  return $this->token->hasPermission($permission) && $this->subject->hasPermission($permission);
}
```

**Role Intersection Logic** — Only roles in BOTH the token AND the user are granted:
```php
$token_roles = array_unique(array_merge($this->token->getRoles(...), $default_roles));
$user_roles  = $this->subject->getRoles(...);
return array_intersect($token_roles, $user_roles);
```

### The Scope/Role Matching Requirement

For an OAuth token to grant a permission:

1. The user MUST have a role with that permission
2. The token request MUST include a scope matching that role
3. An OAuth2 scope entity MUST exist with that name

**If any condition fails, permission is DENIED.**

## Common Pitfalls

### Pitfall 1: Scope/Role Mismatch

**Problem:**
```php
// User has: administrator role
// Token requested with: scope=api_consumer
// Result: Only 'authenticated' role granted (intersection)
$token_roles = ['authenticated', 'api_consumer'];
$user_roles  = ['authenticated', 'administrator'];
$granted = array_intersect($token_roles, $user_roles); // ['authenticated'] only!
```

**Solution:** Request token with the correct scope matching the user's actual role:
```javascript
formData.append('scope', 'administrator');
```

### Pitfall 2: Non-existent Scope Entity

**Problem:** Mobile app requests `scope=subscriber` but no "subscriber" OAuth2 scope entity exists.
**Result:** Token has NO scopes, NO roles, NO permissions.

**Check existing scopes:**
```bash
ddev drush sqlq "SELECT id FROM consumer_scopes"
```

### Pitfall 3: Testing with uid=1

Admin users (uid=1) bypass all permission checks. **Always test OAuth with regular users.**

## Debugging OAuth Permission Issues

### Step 1: Verify Scope Entity Exists
```bash
ddev drush sqlq "SELECT id, description FROM consumer_scopes"
```

### Step 2: Check User Roles
```bash
ddev drush user:role:list username@example.com
```

### Step 3: Verify Role Permissions
```bash
ddev drush role:perm:list ROLE_NAME | grep "your_permission"
```

### Step 4: Test API Request
```bash
# Get OAuth token
TOKEN_RESPONSE=$(curl -s -X POST "https://yoursite.ddev.site/oauth/token" \
  -d "grant_type=password" \
  -d "client_id=YOUR_CLIENT_ID" \
  -d "client_secret=YOUR_CLIENT_SECRET" \
  -d "username=test@example.com" \
  -d "password=password123" \
  -d "scope=SCOPE_NAME")

ACCESS_TOKEN=$(echo "$TOKEN_RESPONSE" | python3 -c "import sys, json; print(json.load(sys.stdin).get('access_token', ''))")

# Test API request
curl -s -X GET "https://yoursite.ddev.site/jsonapi/node/article/1" \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/vnd.api+json"
```

## OAuth Client Configuration

```javascript
const formData = new FormData();
formData.append('client_id', clientId);
formData.append('client_secret', clientSecret);
formData.append('scope', scope);  // MUST match user's role!
formData.append('grant_type', 'password');
formData.append('username', username);
formData.append('password', password);
```

### Creating OAuth Clients

```bash
ddev drush simple-oauth:create-client \
  --label="Mobile App" \
  --secret="your-secret" \
  --confidential \
  --user-id=1
```

## Scope Entity Management

```yaml
# Via config: config/install/consumer.oauth2_scope.subscriber.yml
uuid: YOUR-UUID
langcode: en
status: true
id: subscriber
description: 'Subscriber role access'
grant_user_permissions: true
umbrella: false
granularity: role
```

## Bearer CSRF Bypass Module

When mobile/React Native apps send a Bearer token AND a session cookie, Drupal's CSRF
validation incorrectly triggers. Fix by creating a custom module that decorates the
`session_configuration` service:

```php
// src/Session/BearerSessionConfiguration.php
public function hasSession(Request $request): bool {
  $auth_header = $request->headers->get('Authorization', '');
  if (str_starts_with($auth_header, 'Bearer ') && $this->isValidBearerToken($request)) {
    return FALSE;  // No session = no CSRF check
  }
  return $this->inner->hasSession($request);
}
```

```yaml
# {module_name}.services.yml
services:
  {module_name}.session_configuration:
    class: Drupal\{module_name}\Session\BearerSessionConfiguration
    decorates: session_configuration
    decoration_priority: 10
    arguments:
      - '@{module_name}.session_configuration.inner'
      - '@simple_oauth.server.resource_server.factory'
      - '@psr7.http_message_factory'
```

## Best Practices

1. **Match scopes to user roles** — Check user's roles, request matching scope
2. **Create scope entities for all API roles**
3. **Request multiple scopes if needed**: `formData.append('scope', 'premium_user api_consumer')`
4. **Use specific permissions** — Granular field control over broad permissions
5. **Never test with uid=1** — Admin bypass skews results

## Troubleshooting Checklist

- [ ] Does the OAuth2 scope entity exist? `ddev drush sqlq "SELECT id FROM consumer_scopes WHERE id='SCOPE_NAME'"`
- [ ] Does the user have the required role? `ddev drush user:role:list username@example.com`
- [ ] Does the role have the required permission? `ddev drush role:perm:list ROLE_NAME | grep PERMISSION`
- [ ] Does the token request include the correct scope?
- [ ] Is the scope matching the user's role? (Most common issue!)
- [ ] Are you testing with uid=1? (Don't — use regular user)
- [ ] Is scope provider set to "dynamic"? `ddev drush config:get simple_oauth.settings scope_provider`
