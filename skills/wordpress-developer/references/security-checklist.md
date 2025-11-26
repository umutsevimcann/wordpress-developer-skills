# Security Checklist

Official documentation: https://developer.wordpress.org/apis/security/

## Input Sanitization

### Text & String

```php
$text      = sanitize_text_field($_POST['text']);         // Single line text
$textarea  = sanitize_textarea_field($_POST['textarea']); // Multiline text
$email     = sanitize_email($_POST['email']);             // Email address
$url       = esc_url_raw($_POST['url']);                  // URL for database
$key       = sanitize_key($_POST['key']);                 // Lowercase alphanumeric
$slug      = sanitize_title($_POST['title']);             // URL slug
$filename  = sanitize_file_name($_POST['filename']);      // File name
$html      = wp_kses_post($_POST['content']);             // HTML (post content)
$class     = sanitize_html_class($_POST['class']);        // HTML class name
```

### Numbers

```php
$int       = absint($_POST['number']);                    // Absolute integer
$int       = intval($_POST['number']);                    // Integer
$float     = floatval($_POST['decimal']);                 // Float
```

### Arrays

```php
$array = array_map('sanitize_text_field', $_POST['items']);
$ids   = array_map('absint', $_POST['ids']);
```

### Custom Sanitization

```php
// Sanitize specific values
function sanitize_status($input) {
    $allowed = ['pending', 'approved', 'rejected'];
    return in_array($input, $allowed, true) ? $input : 'pending';
}
```

## Output Escaping

### HTML Context

```php
// Plain text in HTML
echo esc_html($text);

// HTML attributes
echo '<input value="' . esc_attr($value) . '">';

// URLs
echo '<a href="' . esc_url($url) . '">';

// JavaScript strings
echo '<script>var name = "' . esc_js($name) . '";</script>';

// Textarea content
echo '<textarea>' . esc_textarea($text) . '</textarea>';

// JSON for data attributes
echo '<div data-config="' . esc_attr(wp_json_encode($config)) . '">';

// HTML content (allows safe HTML)
echo wp_kses_post($content);

// Custom allowed HTML
$allowed = [
    'a'      => ['href' => [], 'title' => [], 'target' => []],
    'strong' => [],
    'em'     => [],
];
echo wp_kses($content, $allowed);
```

### Translation with Escaping

```php
echo esc_html__('Text to translate', 'text-domain');
echo esc_attr__('Attribute text', 'text-domain');
esc_html_e('Echoed text', 'text-domain');

// With placeholders
printf(
    esc_html__('Hello %s, you have %d messages.', 'text-domain'),
    esc_html($name),
    absint($count)
);
```

## Nonce Verification

### Forms

```php
// Create nonce field
<form method="post">
    <?php wp_nonce_field('my_action', 'my_nonce'); ?>
    <input type="text" name="data" />
    <button type="submit">Submit</button>
</form>

// Verify nonce
function process_form() {
    if (!isset($_POST['my_nonce']) || 
        !wp_verify_nonce($_POST['my_nonce'], 'my_action')) {
        wp_die(__('Security check failed.', 'my-plugin'));
    }
    
    // Process form...
}
```

### AJAX

```php
// JavaScript
jQuery.ajax({
    url: ajaxurl,
    data: {
        action: 'my_action',
        nonce: myPlugin.nonce,
        // other data...
    }
});

// PHP Handler
add_action('wp_ajax_my_action', 'handle_ajax');

function handle_ajax() {
    // Verify nonce
    check_ajax_referer('my_ajax_nonce', 'nonce');
    
    // Process request...
    wp_send_json_success(['message' => 'Success']);
}

// Localize script with nonce
wp_localize_script('my-script', 'myPlugin', [
    'ajaxurl' => admin_url('admin-ajax.php'),
    'nonce'   => wp_create_nonce('my_ajax_nonce'),
]);
```

### URLs

```php
// Create nonced URL
$url = wp_nonce_url(
    admin_url('admin.php?page=my-page&action=delete&id=' . $id),
    'delete_item_' . $id
);

// Verify
if (!isset($_GET['_wpnonce']) || 
    !wp_verify_nonce($_GET['_wpnonce'], 'delete_item_' . $id)) {
    wp_die(__('Invalid request.', 'my-plugin'));
}
```

## Capability Checks

### Basic Checks

```php
// Check capability before action
if (!current_user_can('edit_posts')) {
    wp_die(__('You do not have permission to perform this action.', 'my-plugin'));
}

// Check specific post
if (!current_user_can('edit_post', $post_id)) {
    wp_die(__('You cannot edit this post.', 'my-plugin'));
}
```

### Common Capabilities

| Capability | Description |
|------------|-------------|
| `read` | View content |
| `edit_posts` | Create/edit own posts |
| `edit_others_posts` | Edit others' posts |
| `publish_posts` | Publish posts |
| `delete_posts` | Delete own posts |
| `manage_options` | Access settings |
| `manage_categories` | Manage categories |
| `upload_files` | Upload media |
| `edit_users` | Edit users |
| `activate_plugins` | Activate plugins |

### Custom Capabilities

```php
// Register custom capability
add_action('admin_init', 'add_custom_caps');

function add_custom_caps() {
    $role = get_role('administrator');
    $role->add_cap('manage_questions');
}

// Use in code
if (current_user_can('manage_questions')) {
    // Allow access
}
```

## SQL Injection Prevention

### Using $wpdb->prepare()

```php
global $wpdb;

// Single placeholder
$results = $wpdb->get_results(
    $wpdb->prepare(
        "SELECT * FROM {$wpdb->prefix}my_table WHERE id = %d",
        $id
    )
);

// Multiple placeholders
$results = $wpdb->get_results(
    $wpdb->prepare(
        "SELECT * FROM {$wpdb->prefix}my_table WHERE status = %s AND user_id = %d",
        $status,
        $user_id
    )
);

// LIKE queries
$search = '%' . $wpdb->esc_like($search_term) . '%';
$results = $wpdb->get_results(
    $wpdb->prepare(
        "SELECT * FROM {$wpdb->prefix}my_table WHERE title LIKE %s",
        $search
    )
);

// IN clause
$ids = [1, 2, 3, 4, 5];
$placeholders = implode(',', array_fill(0, count($ids), '%d'));
$results = $wpdb->get_results(
    $wpdb->prepare(
        "SELECT * FROM {$wpdb->prefix}my_table WHERE id IN ({$placeholders})",
        ...$ids
    )
);
```

## XSS Prevention

### Stored XSS

```php
// Always sanitize on input
$title = sanitize_text_field($_POST['title']);
update_post_meta($post_id, '_my_title', $title);

// Always escape on output
echo esc_html(get_post_meta($post_id, '_my_title', true));
```

### Reflected XSS

```php
// Never trust URL parameters
$page = isset($_GET['page']) ? sanitize_key($_GET['page']) : '';
$search = isset($_GET['s']) ? sanitize_text_field($_GET['s']) : '';

// Escape in output
echo '<input type="text" value="' . esc_attr($search) . '">';
```

### DOM XSS

```javascript
// Bad: Direct HTML insertion
element.innerHTML = userInput;

// Good: Use text content or sanitize
element.textContent = userInput;

// Or use DOM APIs
const link = document.createElement('a');
link.href = sanitizedUrl;
link.textContent = linkText;
parent.appendChild(link);
```

## File Security

### Direct Access Prevention

```php
// At the top of PHP files
if (!defined('ABSPATH')) {
    exit; // Exit if accessed directly
}

// Alternative
defined('ABSPATH') || exit;
```

### .htaccess Protection

```apache
# Deny access to PHP files in uploads
<Directory "/path/to/wp-content/uploads">
    <FilesMatch "\.php$">
        Deny from all
    </FilesMatch>
</Directory>
```

### Empty Index Files

```php
<?php
// Silence is golden.
```

## Security Headers

```php
add_action('send_headers', 'add_security_headers');

function add_security_headers() {
    header('X-Content-Type-Options: nosniff');
    header('X-Frame-Options: SAMEORIGIN');
    header('X-XSS-Protection: 1; mode=block');
    header('Referrer-Policy: strict-origin-when-cross-origin');
}
```

## Checklists

### Form Processing

- [ ] Verify nonce with `wp_verify_nonce()` or `check_admin_referer()`
- [ ] Check user capability with `current_user_can()`
- [ ] Sanitize all input data
- [ ] Validate data format and values
- [ ] Escape all output

### AJAX Handler

- [ ] Use `check_ajax_referer()` for nonce verification
- [ ] Check user capability
- [ ] Sanitize input data
- [ ] Validate request parameters
- [ ] Return proper JSON response with `wp_send_json_success/error`

### Database Operations

- [ ] Always use `$wpdb->prepare()` for queries with variables
- [ ] Use `esc_like()` for LIKE queries
- [ ] Validate table and column names
- [ ] Use appropriate placeholders (%d, %s, %f)

### File Uploads

- [ ] Verify file type with `wp_check_filetype()`
- [ ] Use `wp_handle_upload()` for processing
- [ ] Check file size limits
- [ ] Sanitize file names
- [ ] Store outside web root if sensitive
- [ ] Check user capability for uploads
