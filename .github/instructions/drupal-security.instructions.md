---
name: drupal-security
description: Drupal development and security patterns from Ivan Grynenko's cursor rules. Covers OWASP Top 10, authentication, access control, injection prevention, cryptography, configuration, database standards, file permissions, and more. Use for security review, access control, SQL injection prevention, XSS prevention, authentication.
applyTo: "**/*.php,**/*.module,**/*.inc,**/*.install,**/*.theme"
---

# Drupal Security Patterns

**Source**: [Ivan Grynenko - Cursor Rules](https://github.com/ivangrynenko/cursorrules)

Security patterns for Drupal development covering OWASP Top 10 vulnerabilities.

## Authentication Security (OWASP A07:2021)

### Session Management

```php
// ✅ CORRECT: Regenerate session ID after privilege change
\Drupal::service('session')->migrate(TRUE);

// ✅ CORRECT: Invalidate session on logout
\Drupal::service('session')->invalidate();

// ❌ WRONG: Never trust session data without validation
$user_id = $_SESSION['user_id']; // Direct access
```

### Password Security

```php
// ✅ CORRECT: Use Drupal's password service
$password_hasher = \Drupal::service('password');
$hashed = $password_hasher->hash($plaintext_password);
$valid = $password_hasher->check($plaintext_password, $hashed_password);

// ❌ WRONG: Never use weak hashing
$hash = md5($password); // NEVER
$hash = sha1($password); // NEVER
```

## Access Control (OWASP A01:2021)

```php
// ✅ CORRECT: Always check permissions
if (!\Drupal::currentUser()->hasPermission('administer content')) {
  throw new AccessDeniedException();
}

// ✅ CORRECT: Use entity access checks
$access = $entity->access('view', $account, TRUE);
if (!$access->isAllowed()) {
  throw new AccessDeniedException();
}

// ❌ WRONG: Never bypass access checks
// Don't use \Drupal::entityTypeManager()->getStorage()->load() without checking access
```

## Injection Prevention (OWASP A03:2021)

### SQL Injection

```php
// ✅ CORRECT: Use query builder with placeholders
$result = \Drupal::database()->select('node', 'n')
  ->fields('n', ['nid', 'title'])
  ->condition('n.type', $type)  // Automatically parameterized
  ->execute();

// ✅ CORRECT: Named placeholders
$result = \Drupal::database()->query(
  "SELECT * FROM {node} WHERE type = :type AND status = :status",
  [':type' => $type, ':status' => 1]
);

// ❌ WRONG: Never concatenate user input
$query = "SELECT * FROM node WHERE type = '" . $type . "'"; // SQL INJECTION!
```

### XSS Prevention

```php
// ✅ CORRECT: Use t() for user-facing strings
$output = t('Hello @name', ['@name' => $user_name]); // Auto-escaped with @

// ✅ CORRECT: Render arrays (auto-escaped)
$build = ['#markup' => '<strong>' . Html::escape($user_input) . '</strong>'];

// ✅ CORRECT: Explicit filtering
$safe = Xss::filter($user_input);           // Allows safe HTML tags
$safe = Html::escape($user_input);          // Escapes all HTML
$safe = check_plain($user_input);           // Deprecated, use Html::escape

// ❌ WRONG: Never output raw user input
echo $user_input;                          // XSS!
print '<p>' . $user_input . '</p>';        // XSS!
$build = ['#markup' => $user_input];       // XSS!
```

### Command Injection

```php
// ✅ CORRECT: Use Symfony Process
use Symfony\Component\Process\Process;
$process = new Process(['ls', '-la', $safe_path]);
$process->run();

// ❌ WRONG: Never pass user input to shell
exec('ls ' . $user_input);  // Command injection!
shell_exec($user_input);    // Command injection!
```

## Cryptographic Security (OWASP A02:2021)

```php
// ✅ CORRECT: Generate secure random bytes
$token = \Drupal::csrfToken()->get($value);
$random = \Drupal::service('uuid')->generate();
$bytes = random_bytes(32);

// ✅ CORRECT: Use Drupal's crypto service
$encrypted = \Drupal::service('encryption')->encrypt($data, $encryption_profile);

// ❌ WRONG: Weak random number generation
$token = rand(1000, 9999);  // Predictable!
$token = mt_rand();          // Not cryptographically secure!
```

## Security Configuration (OWASP A05:2021)

```php
// ✅ CORRECT: Store secrets in environment variables or settings.php (not in config)
$config['system.mail']['interface']['default'] = getenv('MAIL_SYSTEM');

// ✅ CORRECT: Verify CSRF tokens on form submissions
$form['#token'] = 'my_form_token';

// ❌ WRONG: Never hardcode credentials
define('API_KEY', 'my-secret-key-here');  // Never commit secrets!
```

## File Security

```php
// ✅ CORRECT: Validate file extensions and MIME types
$validators = [
  'file_validate_extensions' => ['jpg jpeg png gif'],
  'file_validate_size' => [25 * 1024 * 1024],
];
$file = file_save_upload('file_field', $validators);

// ✅ CORRECT: Use managed files (Drupal handles access)
$file = File::create([
  'uri' => 'public://uploads/' . $filename,
  'status' => 1,
]);

// ❌ WRONG: Never trust file extensions alone
if (pathinfo($filename, PATHINFO_EXTENSION) === 'jpg') {  // Insufficient!
  // Process file...
}
```

## Database Standards

```php
// ✅ CORRECT: Use table prefixes with curly braces
$query = \Drupal::database()->select('{node}', 'n');

// ✅ CORRECT: Always use transactions for multi-step operations
$transaction = \Drupal::database()->startTransaction();
try {
  // Multiple database operations...
  // Transaction commits on $transaction going out of scope
} catch (\Exception $e) {
  $transaction->rollBack();
  throw $e;
}
```

## Security Review Checklist

Before committing Drupal code, verify:

- [ ] All user input is sanitized before output (XSS)
- [ ] All database queries use placeholders (SQL injection)
- [ ] All access-controlled routes check permissions
- [ ] No hardcoded secrets or API keys
- [ ] File uploads validate extension AND MIME type
- [ ] CSRF protection on all state-changing forms
- [ ] Sessions are properly invalidated on logout
- [ ] Error messages don't expose internal details
- [ ] No use of `eval()`, `exec()`, or `shell_exec()` with user input
- [ ] Sensitive data is encrypted at rest where appropriate
