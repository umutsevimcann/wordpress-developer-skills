# WooCommerce Development

Official documentation: https://developer.woocommerce.com/docs/

## Essential Hooks

### Product Page Hooks

```php
// Before/after product summary
add_action('woocommerce_before_single_product_summary', 'custom_before_summary', 5);
add_action('woocommerce_after_single_product_summary', 'custom_after_summary', 15);

// Product meta (after add to cart)
add_action('woocommerce_product_meta_start', 'custom_meta_start');
add_action('woocommerce_product_meta_end', 'custom_meta_end');

// Product tabs
add_filter('woocommerce_product_tabs', 'custom_product_tabs');
```

### Cart Hooks

```php
add_action('woocommerce_before_cart', 'custom_before_cart');
add_action('woocommerce_after_cart', 'custom_after_cart');
add_action('woocommerce_cart_contents', 'custom_cart_contents');
add_action('woocommerce_cart_totals_before_order_total', 'custom_before_total');
add_filter('woocommerce_cart_item_price', 'custom_item_price', 10, 3);
```

### Checkout Hooks

```php
add_action('woocommerce_before_checkout_form', 'custom_before_checkout');
add_action('woocommerce_checkout_before_customer_details', 'custom_before_customer');
add_action('woocommerce_checkout_after_customer_details', 'custom_after_customer');
add_action('woocommerce_review_order_before_payment', 'custom_before_payment');
add_action('woocommerce_checkout_order_processed', 'custom_order_processed', 10, 3);
```

### Order Hooks

```php
add_action('woocommerce_order_status_changed', 'custom_status_changed', 10, 4);
add_action('woocommerce_payment_complete', 'custom_payment_complete');
add_action('woocommerce_thankyou', 'custom_thankyou_page');
add_action('woocommerce_new_order', 'custom_new_order');
```

## Customize Checkout Fields

```php
// Add custom field
add_filter('woocommerce_checkout_fields', 'add_custom_checkout_field');

function add_custom_checkout_field($fields) {
    $fields['billing']['billing_company_id'] = [
        'type'        => 'text',
        'label'       => __('Company ID', 'my-plugin'),
        'placeholder' => __('Enter your company ID', 'my-plugin'),
        'required'    => false,
        'class'       => ['form-row-wide'],
        'priority'    => 25,
    ];
    
    return $fields;
}

// Validate field
add_action('woocommerce_checkout_process', 'validate_custom_field');

function validate_custom_field() {
    if (!empty($_POST['billing_company_id'])) {
        $company_id = sanitize_text_field($_POST['billing_company_id']);
        if (strlen($company_id) < 10) {
            wc_add_notice(__('Company ID must be at least 10 characters.', 'my-plugin'), 'error');
        }
    }
}

// Save field
add_action('woocommerce_checkout_update_order_meta', 'save_custom_field');

function save_custom_field($order_id) {
    if (!empty($_POST['billing_company_id'])) {
        update_post_meta(
            $order_id,
            '_billing_company_id',
            sanitize_text_field($_POST['billing_company_id'])
        );
    }
}

// Display in admin order
add_action('woocommerce_admin_order_data_after_billing_address', 'display_custom_field_admin');

function display_custom_field_admin($order) {
    $company_id = get_post_meta($order->get_id(), '_billing_company_id', true);
    if ($company_id) {
        echo '<p><strong>' . __('Company ID', 'my-plugin') . ':</strong> ' . esc_html($company_id) . '</p>';
    }
}
```

## Price Filters

```php
// Modify displayed price
add_filter('woocommerce_get_price_html', 'custom_price_html', 10, 2);

function custom_price_html($price, $product) {
    if ($product->is_on_sale()) {
        $price .= ' <span class="sale-badge">' . __('Sale!', 'my-plugin') . '</span>';
    }
    return $price;
}

// Modify cart item price
add_filter('woocommerce_cart_item_price', 'custom_cart_price', 10, 3);

function custom_cart_price($price, $cart_item, $cart_item_key) {
    // Add custom logic
    return $price;
}
```

## Custom Product Tab

```php
add_filter('woocommerce_product_tabs', 'add_custom_product_tab');

function add_custom_product_tab($tabs) {
    $tabs['custom_tab'] = [
        'title'    => __('Specifications', 'my-plugin'),
        'priority' => 50,
        'callback' => 'custom_tab_content',
    ];
    
    return $tabs;
}

function custom_tab_content() {
    global $product;
    
    echo '<h2>' . esc_html__('Product Specifications', 'my-plugin') . '</h2>';
    
    $specs = get_post_meta($product->get_id(), '_product_specs', true);
    if ($specs) {
        echo '<div class="product-specs">' . wp_kses_post($specs) . '</div>';
    }
}
```

## Product Meta Box (Admin)

```php
add_action('add_meta_boxes', 'add_product_meta_box');

function add_product_meta_box() {
    add_meta_box(
        'product_specs_box',
        __('Product Specifications', 'my-plugin'),
        'product_specs_callback',
        'product',
        'normal',
        'high'
    );
}

function product_specs_callback($post) {
    wp_nonce_field('save_product_specs', 'product_specs_nonce');
    $specs = get_post_meta($post->ID, '_product_specs', true);
    
    wp_editor($specs, 'product_specs', [
        'textarea_name' => 'product_specs',
        'textarea_rows' => 10,
    ]);
}

add_action('save_post_product', 'save_product_specs');

function save_product_specs($post_id) {
    if (!isset($_POST['product_specs_nonce']) ||
        !wp_verify_nonce($_POST['product_specs_nonce'], 'save_product_specs')) {
        return;
    }
    
    if (defined('DOING_AUTOSAVE') && DOING_AUTOSAVE) {
        return;
    }
    
    if (!current_user_can('edit_post', $post_id)) {
        return;
    }
    
    if (isset($_POST['product_specs'])) {
        update_post_meta($post_id, '_product_specs', wp_kses_post($_POST['product_specs']));
    }
}
```

## Custom Shipping Method

```php
add_action('woocommerce_shipping_init', 'init_custom_shipping');

function init_custom_shipping() {
    class WC_Custom_Shipping extends WC_Shipping_Method {
        public function __construct($instance_id = 0) {
            $this->id                 = 'custom_shipping';
            $this->instance_id        = absint($instance_id);
            $this->method_title       = __('Custom Shipping', 'my-plugin');
            $this->method_description = __('Custom shipping method with special rates.', 'my-plugin');
            $this->supports           = ['shipping-zones', 'instance-settings'];
            
            $this->init();
        }
        
        public function init() {
            $this->init_form_fields();
            $this->init_settings();
            
            $this->title = $this->get_option('title');
            $this->cost  = $this->get_option('cost');
            
            add_action('woocommerce_update_options_shipping_' . $this->id, [$this, 'process_admin_options']);
        }
        
        public function init_form_fields() {
            $this->instance_form_fields = [
                'title' => [
                    'title'   => __('Method Title', 'my-plugin'),
                    'type'    => 'text',
                    'default' => __('Custom Shipping', 'my-plugin'),
                ],
                'cost' => [
                    'title'   => __('Cost', 'my-plugin'),
                    'type'    => 'price',
                    'default' => '10',
                ],
            ];
        }
        
        public function calculate_shipping($package = []) {
            $cost = $this->cost;
            
            // Custom calculation logic
            $cart_total = WC()->cart->get_cart_contents_total();
            if ($cart_total > 100) {
                $cost = 0; // Free shipping over $100
            }
            
            $this->add_rate([
                'id'    => $this->id,
                'label' => $this->title,
                'cost'  => $cost,
            ]);
        }
    }
}

add_filter('woocommerce_shipping_methods', 'add_custom_shipping_method');

function add_custom_shipping_method($methods) {
    $methods['custom_shipping'] = 'WC_Custom_Shipping';
    return $methods;
}
```

## Custom Payment Gateway

```php
add_action('plugins_loaded', 'init_custom_gateway');

function init_custom_gateway() {
    class WC_Custom_Gateway extends WC_Payment_Gateway {
        public function __construct() {
            $this->id                 = 'custom_gateway';
            $this->icon               = '';
            $this->has_fields         = true;
            $this->method_title       = __('Custom Gateway', 'my-plugin');
            $this->method_description = __('Custom payment gateway integration.', 'my-plugin');
            $this->supports           = ['products', 'refunds'];
            
            $this->init_form_fields();
            $this->init_settings();
            
            $this->title       = $this->get_option('title');
            $this->description = $this->get_option('description');
            $this->enabled     = $this->get_option('enabled');
            $this->api_key     = $this->get_option('api_key');
            
            add_action('woocommerce_update_options_payment_gateways_' . $this->id, [$this, 'process_admin_options']);
        }
        
        public function init_form_fields() {
            $this->form_fields = [
                'enabled' => [
                    'title'   => __('Enable/Disable', 'my-plugin'),
                    'type'    => 'checkbox',
                    'label'   => __('Enable Custom Gateway', 'my-plugin'),
                    'default' => 'no',
                ],
                'title' => [
                    'title'       => __('Title', 'my-plugin'),
                    'type'        => 'text',
                    'description' => __('Title shown to customers.', 'my-plugin'),
                    'default'     => __('Custom Payment', 'my-plugin'),
                ],
                'description' => [
                    'title'   => __('Description', 'my-plugin'),
                    'type'    => 'textarea',
                    'default' => __('Pay securely using our custom gateway.', 'my-plugin'),
                ],
                'api_key' => [
                    'title' => __('API Key', 'my-plugin'),
                    'type'  => 'password',
                ],
            ];
        }
        
        public function payment_fields() {
            if ($this->description) {
                echo wpautop(wp_kses_post($this->description));
            }
            
            // Custom payment fields
            ?>
            <fieldset>
                <p class="form-row form-row-wide">
                    <label for="card_number"><?php esc_html_e('Card Number', 'my-plugin'); ?> <span class="required">*</span></label>
                    <input id="card_number" name="card_number" type="text" autocomplete="off" />
                </p>
            </fieldset>
            <?php
        }
        
        public function validate_fields() {
            if (empty($_POST['card_number'])) {
                wc_add_notice(__('Card number is required.', 'my-plugin'), 'error');
                return false;
            }
            return true;
        }
        
        public function process_payment($order_id) {
            $order = wc_get_order($order_id);
            
            // Process payment with external API
            $response = $this->call_payment_api($order);
            
            if ($response['success']) {
                $order->payment_complete($response['transaction_id']);
                $order->add_order_note(__('Payment completed via Custom Gateway.', 'my-plugin'));
                
                WC()->cart->empty_cart();
                
                return [
                    'result'   => 'success',
                    'redirect' => $this->get_return_url($order),
                ];
            } else {
                wc_add_notice(__('Payment failed: ', 'my-plugin') . $response['message'], 'error');
                return ['result' => 'fail'];
            }
        }
        
        public function process_refund($order_id, $amount = null, $reason = '') {
            $order = wc_get_order($order_id);
            $transaction_id = $order->get_transaction_id();
            
            // Process refund with external API
            $response = $this->call_refund_api($transaction_id, $amount);
            
            if ($response['success']) {
                $order->add_order_note(
                    sprintf(__('Refunded %s via Custom Gateway.', 'my-plugin'), wc_price($amount))
                );
                return true;
            }
            
            return new WP_Error('refund_failed', $response['message']);
        }
        
        private function call_payment_api($order) {
            // Implement your API call here
            return ['success' => true, 'transaction_id' => 'TXN123456'];
        }
        
        private function call_refund_api($transaction_id, $amount) {
            // Implement your refund API call here
            return ['success' => true];
        }
    }
}

add_filter('woocommerce_payment_gateways', 'add_custom_gateway');

function add_custom_gateway($gateways) {
    $gateways[] = 'WC_Custom_Gateway';
    return $gateways;
}
```

## Extending WooCommerce REST API

```php
add_action('rest_api_init', 'register_custom_wc_routes');

function register_custom_wc_routes() {
    register_rest_route('wc/v3', '/products/(?P<id>\d+)/specs', [
        'methods'             => 'GET',
        'callback'            => 'get_product_specs',
        'permission_callback' => function() {
            return current_user_can('read');
        },
    ]);
}

function get_product_specs($request) {
    $product_id = $request['id'];
    $specs = get_post_meta($product_id, '_product_specs', true);
    
    return new WP_REST_Response([
        'product_id' => $product_id,
        'specs'      => $specs,
    ], 200);
}
```

## Useful Functions

```php
// Create order programmatically
$order = wc_create_order([
    'customer_id' => get_current_user_id(),
    'status'      => 'pending',
]);

$order->add_product(wc_get_product($product_id), $quantity);
$order->set_address($address, 'billing');
$order->calculate_totals();
$order->save();

// Get cart
$cart = WC()->cart;
$cart_total = $cart->get_cart_contents_total();
$cart_items = $cart->get_cart();

// Format price
echo wc_price(99.99);

// Get product
$product = wc_get_product($product_id);
$price = $product->get_price();
$name = $product->get_name();
$sku = $product->get_sku();
```
