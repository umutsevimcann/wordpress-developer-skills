---
name: wordpress-developer
description: WordPress theme and plugin development expert. This skill is triggered when users mention WordPress, theme, plugin, Gutenberg, WooCommerce, hook, shortcode, wp_enqueue, add_action, add_filter, or related terms.
---

# WordPress Developer Skill

Expert guidance for WordPress 6.5+ theme and plugin development.

## Official Resources

| Resource | URL |
|----------|-----|
| Plugin Handbook | https://developer.wordpress.org/plugins/ |
| Theme Handbook | https://developer.wordpress.org/themes/ |
| Block Editor Handbook | https://developer.wordpress.org/block-editor/ |
| REST API Handbook | https://developer.wordpress.org/rest-api/ |
| Common APIs | https://developer.wordpress.org/apis/ |
| Coding Standards | https://developer.wordpress.org/coding-standards/ |

## Core Principles

1. **Follow WordPress Coding Standards (WPCS)**
2. **Security First**: Always sanitize input, escape output, verify nonces
3. **Use Hooks**: Extend functionality via actions and filters
4. **Internationalization**: Use `__()`, `_e()`, `esc_html__()` functions
5. **Prefix Everything**: Use unique prefixes to avoid conflicts

## Plugin Development

### Minimum Plugin Structure

```
my-plugin/
├── my-plugin.php          # Main plugin file
├── includes/
│   ├── class-main.php
│   └── class-admin.php
├── admin/
│   ├── css/
│   └── js/
├── public/
│   ├── css/
│   └── js/
├── languages/
└── readme.txt
```

### Main Plugin File Header

```php
<?php
/**
 * Plugin Name:       My Plugin
 * Plugin URI:        https://example.com/my-plugin
 * Description:       Short description of the plugin.
 * Version:           1.0.0
 * Requires at least: 6.0
 * Requires PHP:      8.0
 * Author:            Author Name
 * Author URI:        https://example.com
 * License:           GPL v2 or later
 * License URI:       https://www.gnu.org/licenses/gpl-2.0.html
 * Text Domain:       my-plugin
 * Domain Path:       /languages
 */

// Prevent direct access
if (!defined('ABSPATH')) {
    exit;
}

// Plugin constants
define('MY_PLUGIN_VERSION', '1.0.0');
define('MY_PLUGIN_PATH', plugin_dir_path(__FILE__));
define('MY_PLUGIN_URL', plugin_dir_url(__FILE__));

// Autoloader
spl_autoload_register(function ($class) {
    $prefix = 'MyPlugin\\';
    $base_dir = MY_PLUGIN_PATH . 'includes/';
    
    $len = strlen($prefix);
    if (strncmp($prefix, $class, $len) !== 0) {
        return;
    }
    
    $relative_class = substr($class, $len);
    $file = $base_dir . 'class-' . strtolower(str_replace('\\', '-', $relative_class)) . '.php';
    
    if (file_exists($file)) {
        require $file;
    }
});

// Initialize plugin
add_action('plugins_loaded', function() {
    load_plugin_textdomain('my-plugin', false, dirname(plugin_basename(__FILE__)) . '/languages');
});
```

## Custom Post Type

```php
add_action('init', 'myplugin_register_post_types');

function myplugin_register_post_types() {
    register_post_type('question', [
        'labels' => [
            'name'               => __('Questions', 'my-plugin'),
            'singular_name'      => __('Question', 'my-plugin'),
            'add_new'            => __('Add New', 'my-plugin'),
            'add_new_item'       => __('Add New Question', 'my-plugin'),
            'edit_item'          => __('Edit Question', 'my-plugin'),
            'view_item'          => __('View Question', 'my-plugin'),
            'search_items'       => __('Search Questions', 'my-plugin'),
            'not_found'          => __('No questions found', 'my-plugin'),
        ],
        'public'              => true,
        'has_archive'         => true,
        'show_in_rest'        => true,  // Enable Gutenberg & REST API
        'supports'            => ['title', 'editor', 'author', 'thumbnail', 'comments', 'custom-fields'],
        'menu_icon'           => 'dashicons-editor-help',
        'rewrite'             => ['slug' => 'questions'],
        'template'            => [
            ['core/paragraph', ['placeholder' => __('Write your question here...', 'my-plugin')]]
        ],
    ]);
}
```

## Custom Taxonomy

```php
add_action('init', 'myplugin_register_taxonomies');

function myplugin_register_taxonomies() {
    register_taxonomy('question_category', 'question', [
        'labels' => [
            'name'              => __('Categories', 'my-plugin'),
            'singular_name'     => __('Category', 'my-plugin'),
            'search_items'      => __('Search Categories', 'my-plugin'),
            'all_items'         => __('All Categories', 'my-plugin'),
            'edit_item'         => __('Edit Category', 'my-plugin'),
            'add_new_item'      => __('Add New Category', 'my-plugin'),
        ],
        'hierarchical'      => true,
        'public'            => true,
        'show_in_rest'      => true,  // Enable Gutenberg & REST API
        'show_admin_column' => true,
        'rewrite'           => ['slug' => 'question-category'],
    ]);
}
```

## Hooks System

### Actions

```php
// Add action with priority and arguments
add_action('hook_name', 'callback_function', 10, 2);

// Common actions
add_action('init', 'myplugin_init');                      // After WordPress loads
add_action('admin_init', 'myplugin_admin_init');          // Admin area loads
add_action('wp_enqueue_scripts', 'myplugin_scripts');     // Frontend scripts
add_action('admin_enqueue_scripts', 'myplugin_admin_scripts'); // Admin scripts
add_action('save_post', 'myplugin_save_post', 10, 3);     // Post save
add_action('wp_ajax_my_action', 'myplugin_ajax_handler'); // AJAX (logged in)
add_action('wp_ajax_nopriv_my_action', 'myplugin_ajax_handler'); // AJAX (not logged in)

// Custom action
do_action('myplugin_after_save', $post_id, $data);
```

### Filters

```php
// Add filter
add_filter('hook_name', 'callback_function', 10, 2);

// Common filters
add_filter('the_content', 'myplugin_modify_content');
add_filter('the_title', 'myplugin_modify_title', 10, 2);
add_filter('wp_insert_post_data', 'myplugin_modify_post_data', 10, 2);
add_filter('manage_question_posts_columns', 'myplugin_custom_columns');

// Filter example
function myplugin_modify_content($content) {
    if (is_singular('question') && in_the_loop()) {
        $content .= '<div class="question-meta">' . esc_html__('Asked by:', 'my-plugin') . ' ' . get_the_author() . '</div>';
    }
    return $content;
}

// Custom filter
$value = apply_filters('myplugin_custom_filter', $default_value, $arg1);
```

## Security

### Input Sanitization

```php
$text    = sanitize_text_field($_POST['text']);
$email   = sanitize_email($_POST['email']);
$url     = esc_url_raw($_POST['url']);
$key     = sanitize_key($_POST['key']);
$html    = wp_kses_post($_POST['content']);
$int     = absint($_POST['number']);
$array   = array_map('sanitize_text_field', $_POST['items']);
```

### Output Escaping

```php
echo esc_html($text);           // Plain text
echo esc_attr($attribute);      // HTML attributes
echo esc_url($url);             // URLs
echo wp_kses_post($html);       // HTML content
echo esc_textarea($text);       // Textarea content
```

### Nonce Verification

```php
// Create nonce field in form
wp_nonce_field('myplugin_save_action', 'myplugin_nonce');

// Verify nonce
if (!isset($_POST['myplugin_nonce']) || 
    !wp_verify_nonce($_POST['myplugin_nonce'], 'myplugin_save_action')) {
    wp_die(__('Security check failed', 'my-plugin'));
}

// Capability check
if (!current_user_can('edit_posts')) {
    wp_die(__('You do not have permission to perform this action', 'my-plugin'));
}
```

## Database Operations

```php
global $wpdb;

// SELECT with prepare
$results = $wpdb->get_results(
    $wpdb->prepare(
        "SELECT * FROM {$wpdb->prefix}myplugin_table WHERE user_id = %d AND status = %s",
        $user_id,
        $status
    )
);

// INSERT
$wpdb->insert(
    $wpdb->prefix . 'myplugin_table',
    [
        'user_id' => $user_id,
        'content' => $content,
        'created_at' => current_time('mysql')
    ],
    ['%d', '%s', '%s']
);
$insert_id = $wpdb->insert_id;

// UPDATE
$wpdb->update(
    $wpdb->prefix . 'myplugin_table',
    ['status' => 'published'],
    ['id' => $id],
    ['%s'],
    ['%d']
);

// DELETE
$wpdb->delete(
    $wpdb->prefix . 'myplugin_table',
    ['id' => $id],
    ['%d']
);
```

## REST API Custom Endpoint

```php
add_action('rest_api_init', 'myplugin_register_routes');

function myplugin_register_routes() {
    register_rest_route('myplugin/v1', '/questions', [
        'methods'             => 'GET',
        'callback'            => 'myplugin_get_questions',
        'permission_callback' => '__return_true',
        'args'                => [
            'per_page' => [
                'default'           => 10,
                'sanitize_callback' => 'absint',
            ],
        ],
    ]);
    
    register_rest_route('myplugin/v1', '/questions/(?P<id>\d+)', [
        'methods'             => 'GET',
        'callback'            => 'myplugin_get_question',
        'permission_callback' => '__return_true',
        'args'                => [
            'id' => [
                'validate_callback' => function($param) {
                    return is_numeric($param);
                }
            ],
        ],
    ]);
}

function myplugin_get_questions(WP_REST_Request $request) {
    $per_page = $request->get_param('per_page');
    
    $questions = get_posts([
        'post_type'      => 'question',
        'posts_per_page' => $per_page,
        'post_status'    => 'publish',
    ]);
    
    return new WP_REST_Response($questions, 200);
}
```

## AJAX Handler

```php
// PHP Handler
add_action('wp_ajax_myplugin_vote', 'myplugin_handle_vote');
add_action('wp_ajax_nopriv_myplugin_vote', 'myplugin_handle_vote');

function myplugin_handle_vote() {
    // Verify nonce
    check_ajax_referer('myplugin_vote_nonce', 'nonce');
    
    $post_id = absint($_POST['post_id']);
    $vote_type = sanitize_key($_POST['vote_type']);
    
    if (!$post_id) {
        wp_send_json_error(['message' => __('Invalid post ID', 'my-plugin')]);
    }
    
    // Process vote
    $current_votes = (int) get_post_meta($post_id, '_vote_count', true);
    $new_votes = $vote_type === 'up' ? $current_votes + 1 : $current_votes - 1;
    update_post_meta($post_id, '_vote_count', $new_votes);
    
    wp_send_json_success([
        'votes' => $new_votes,
        'message' => __('Vote recorded', 'my-plugin')
    ]);
}

// Enqueue script with AJAX data
add_action('wp_enqueue_scripts', 'myplugin_enqueue_scripts');

function myplugin_enqueue_scripts() {
    wp_enqueue_script(
        'myplugin-voting',
        MY_PLUGIN_URL . 'public/js/voting.js',
        ['jquery'],
        MY_PLUGIN_VERSION,
        true
    );
    
    wp_localize_script('myplugin-voting', 'myPluginAjax', [
        'ajaxurl' => admin_url('admin-ajax.php'),
        'nonce'   => wp_create_nonce('myplugin_vote_nonce'),
    ]);
}
```

```javascript
// JavaScript (voting.js)
jQuery(document).ready(function($) {
    $('.vote-button').on('click', function(e) {
        e.preventDefault();
        
        var $button = $(this);
        var postId = $button.data('post-id');
        var voteType = $button.data('vote-type');
        
        $.ajax({
            url: myPluginAjax.ajaxurl,
            type: 'POST',
            data: {
                action: 'myplugin_vote',
                nonce: myPluginAjax.nonce,
                post_id: postId,
                vote_type: voteType
            },
            success: function(response) {
                if (response.success) {
                    $button.siblings('.vote-count').text(response.data.votes);
                } else {
                    alert(response.data.message);
                }
            }
        });
    });
});
```

## Block Registration (Interactivity API)

```php
add_action('init', 'myplugin_register_blocks');

function myplugin_register_blocks() {
    register_block_type(__DIR__ . '/blocks/interactive-counter');
}
```

For detailed block development with Interactivity API, see `references/block-development.md`.

## Additional References

- Theme Development: `references/theme-development.md`
- Block Development: `references/block-development.md`
- WooCommerce: `references/woocommerce.md`
- Performance: `references/performance.md`
- Security Checklist: `references/security-checklist.md`
- Coding Standards: `references/coding-standards.md`
