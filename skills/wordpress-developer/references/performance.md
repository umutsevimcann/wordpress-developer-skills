# Performance Optimization

## Script & Style Optimization

### Conditional Loading

```php
// Load scripts only where needed
add_action('wp_enqueue_scripts', 'conditional_scripts');

function conditional_scripts() {
    // Only on single posts
    if (is_singular('post')) {
        wp_enqueue_script('post-scripts', ...);
    }
    
    // Only on specific page template
    if (is_page_template('template-contact.php')) {
        wp_enqueue_script('contact-form', ...);
    }
    
    // Only if shortcode is present
    global $post;
    if (is_a($post, 'WP_Post') && has_shortcode($post->post_content, 'my_shortcode')) {
        wp_enqueue_script('shortcode-scripts', ...);
    }
}
```

### Defer & Async

```php
add_filter('script_loader_tag', 'add_defer_async', 10, 3);

function add_defer_async($tag, $handle, $src) {
    $defer_scripts = ['my-analytics', 'social-share'];
    $async_scripts = ['tracking-script'];
    
    if (in_array($handle, $defer_scripts)) {
        return str_replace(' src', ' defer src', $tag);
    }
    
    if (in_array($handle, $async_scripts)) {
        return str_replace(' src', ' async src', $tag);
    }
    
    return $tag;
}
```

### Remove Unnecessary Assets

```php
add_action('wp_enqueue_scripts', 'remove_unnecessary_assets', 100);

function remove_unnecessary_assets() {
    // Remove block library CSS if not using blocks
    if (!is_admin()) {
        wp_dequeue_style('wp-block-library');
        wp_dequeue_style('wp-block-library-theme');
        wp_dequeue_style('wc-blocks-style'); // WooCommerce blocks
    }
    
    // Remove emoji scripts
    remove_action('wp_head', 'print_emoji_detection_script', 7);
    remove_action('wp_print_styles', 'print_emoji_styles');
    
    // Remove jQuery migrate if not needed
    if (!is_admin()) {
        wp_deregister_script('jquery');
        wp_register_script('jquery', includes_url('/js/jquery/jquery.min.js'), [], null, true);
    }
}
```

### Script Modules (WordPress 6.5+)

```php
// Register ES Module
add_action('wp_enqueue_scripts', 'enqueue_script_modules');

function enqueue_script_modules() {
    wp_enqueue_script_module(
        'my-module',
        plugin_dir_url(__FILE__) . 'js/my-module.js',
        [],
        '1.0.0'
    );
    
    // With dependencies
    wp_enqueue_script_module(
        'my-app',
        plugin_dir_url(__FILE__) . 'js/app.js',
        ['@wordpress/interactivity'],
        '1.0.0'
    );
}
```

## Database Optimization

### Efficient Queries

```php
// Bad: Gets all post data
$posts = get_posts(['posts_per_page' => -1]);

// Good: Only get IDs
$post_ids = get_posts([
    'posts_per_page' => -1,
    'fields'         => 'ids',
]);

// Good: Only get specific fields
$posts = get_posts([
    'posts_per_page' => 100,
    'no_found_rows'  => true, // Skip pagination count
    'fields'         => 'ids',
]);
```

### Meta Query vs Taxonomy

```php
// Slow: Meta query
$query = new WP_Query([
    'meta_key'   => 'color',
    'meta_value' => 'red',
]);

// Fast: Taxonomy query
$query = new WP_Query([
    'tax_query' => [
        [
            'taxonomy' => 'color',
            'field'    => 'slug',
            'terms'    => 'red',
        ],
    ],
]);
```

### Avoid N+1 Problem

```php
// Bad: N+1 queries
$posts = get_posts(['posts_per_page' => 10]);
foreach ($posts as $post) {
    $author = get_userdata($post->post_author); // Query per post
}

// Good: Single query with cache priming
$posts = get_posts([
    'posts_per_page'      => 10,
    'update_post_meta_cache' => true,
    'update_post_term_cache' => true,
]);

// Pre-fetch author data
$author_ids = wp_list_pluck($posts, 'post_author');
$authors = get_users(['include' => array_unique($author_ids)]);
```

### Custom Table Index

```php
// Add index to custom table
global $wpdb;
$table = $wpdb->prefix . 'my_table';

$wpdb->query("ALTER TABLE {$table} ADD INDEX idx_user_status (user_id, status)");
```

## Caching

### Transient API

```php
function get_popular_posts() {
    $cache_key = 'popular_posts_' . get_locale();
    $posts = get_transient($cache_key);
    
    if ($posts === false) {
        $posts = new WP_Query([
            'posts_per_page' => 10,
            'meta_key'       => 'views',
            'orderby'        => 'meta_value_num',
            'order'          => 'DESC',
        ]);
        
        set_transient($cache_key, $posts, HOUR_IN_SECONDS);
    }
    
    return $posts;
}

// Invalidate cache when post is updated
add_action('save_post', 'invalidate_popular_posts_cache');

function invalidate_popular_posts_cache($post_id) {
    delete_transient('popular_posts_' . get_locale());
}
```

### Object Cache

```php
// Using wp_cache (requires object cache plugin like Redis)
function get_expensive_data($key) {
    $cache_key = 'expensive_' . md5($key);
    $cache_group = 'my_plugin';
    
    $data = wp_cache_get($cache_key, $cache_group);
    
    if ($data === false) {
        $data = expensive_operation($key);
        wp_cache_set($cache_key, $data, $cache_group, 3600);
    }
    
    return $data;
}
```

### Fragment Caching

```php
function render_sidebar_widget() {
    $cache_key = 'sidebar_widget_' . get_the_ID();
    $output = get_transient($cache_key);
    
    if ($output === false) {
        ob_start();
        // Expensive widget rendering
        include 'widget-template.php';
        $output = ob_get_clean();
        
        set_transient($cache_key, $output, 12 * HOUR_IN_SECONDS);
    }
    
    echo $output;
}
```

## Image Optimization

### Native Lazy Loading

```php
// WordPress 5.5+ adds lazy loading automatically
// Disable for above-the-fold images
add_filter('wp_img_tag_add_loading_attr', 'disable_lazy_load_hero', 10, 3);

function disable_lazy_load_hero($value, $image, $context) {
    if ($context === 'the_content') {
        // Check if it's the first image
        static $first_image = true;
        if ($first_image) {
            $first_image = false;
            return false; // Don't add loading="lazy"
        }
    }
    return $value;
}

// Add fetchpriority="high" to hero images
add_filter('wp_img_tag_add_decoding_attr', 'add_fetchpriority', 10, 3);

function add_fetchpriority($value, $image, $context) {
    if (strpos($image, 'hero-image') !== false) {
        return str_replace('<img', '<img fetchpriority="high"', $image);
    }
    return $value;
}
```

### Responsive Images

```php
// Custom image sizes
add_action('after_setup_theme', 'custom_image_sizes');

function custom_image_sizes() {
    add_image_size('card-thumbnail', 400, 300, true);
    add_image_size('hero-large', 1920, 800, true);
}

// Add to size selector
add_filter('image_size_names_choose', 'custom_image_size_names');

function custom_image_size_names($sizes) {
    return array_merge($sizes, [
        'card-thumbnail' => __('Card Thumbnail', 'my-plugin'),
        'hero-large'     => __('Hero Large', 'my-plugin'),
    ]);
}
```

### WebP Support

```php
// Enable WebP uploads
add_filter('upload_mimes', 'allow_webp_upload');

function allow_webp_upload($mimes) {
    $mimes['webp'] = 'image/webp';
    return $mimes;
}
```

## AJAX & Heartbeat

### Reduce Heartbeat Frequency

```php
add_filter('heartbeat_settings', 'optimize_heartbeat');

function optimize_heartbeat($settings) {
    // Increase interval to 60 seconds
    $settings['interval'] = 60;
    return $settings;
}

// Disable heartbeat on non-essential pages
add_action('init', 'disable_heartbeat');

function disable_heartbeat() {
    global $pagenow;
    
    if ($pagenow !== 'post.php' && $pagenow !== 'post-new.php') {
        wp_deregister_script('heartbeat');
    }
}
```

## Preloading & Resource Hints

```php
add_action('wp_head', 'add_resource_hints', 1);

function add_resource_hints() {
    // DNS Prefetch for external resources
    echo '<link rel="dns-prefetch" href="//fonts.googleapis.com">' . "\n";
    echo '<link rel="dns-prefetch" href="//www.google-analytics.com">' . "\n";
    
    // Preconnect for critical third-party origins
    echo '<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>' . "\n";
    
    // Preload critical assets
    echo '<link rel="preload" href="' . get_template_directory_uri() . '/fonts/main.woff2" as="font" type="font/woff2" crossorigin>' . "\n";
    
    // Prefetch non-critical resources
    echo '<link rel="prefetch" href="' . get_template_directory_uri() . '/js/deferred.js">' . "\n";
}
```

## Autoload Optimization

```php
// Don't autoload large options
update_option('my_large_option', $data, false); // false = no autoload

// Check autoloaded data size
global $wpdb;
$autoload_size = $wpdb->get_var(
    "SELECT SUM(LENGTH(option_value)) 
     FROM {$wpdb->options} 
     WHERE autoload = 'yes'"
);
```

## Debug & Profiling

```php
// Enable query logging (development only)
define('SAVEQUERIES', true);

// After page load
global $wpdb;
echo '<pre>';
print_r($wpdb->queries);
echo '</pre>';

// Log slow queries
add_filter('log_query_custom_data', 'log_slow_queries', 10, 5);

function log_slow_queries($query_data, $query, $query_time, $query_callstack, $query_start) {
    if ($query_time > 0.05) { // 50ms threshold
        error_log("Slow query ({$query_time}s): {$query}");
    }
    return $query_data;
}
```
