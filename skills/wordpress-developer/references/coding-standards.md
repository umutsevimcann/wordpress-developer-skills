# WordPress Coding Standards

Official documentation: https://developer.wordpress.org/coding-standards/

## PHP Standards

### Naming Conventions

```php
// Functions: lowercase with underscores, prefixed
function myplugin_get_user_data($user_id) {}
function myplugin_process_form() {}

// Classes: Capitalized words, prefixed
class MyPlugin_User_Handler {}
class MyPlugin_Admin_Settings {}

// Methods: lowercase with underscores
public function get_user_name() {}
private function process_data() {}

// Constants: uppercase with underscores
define('MYPLUGIN_VERSION', '1.0.0');
const MYPLUGIN_MAX_ITEMS = 100;

// Variables: lowercase with underscores
$user_name = 'John';
$post_count = 10;

// Files: lowercase with hyphens
class-user-handler.php
admin-settings.php
```

### Prefix Usage

```php
// All public functions, classes, and global variables must be prefixed
function myplugin_init() {}           // Good
function init() {}                     // Bad - too generic

class MyPlugin_Settings {}            // Good
class Settings {}                      // Bad - conflicts possible

$myplugin_options = [];               // Good
$options = [];                         // Bad - too generic

// Hooks should also be prefixed
do_action('myplugin_after_save', $data);
apply_filters('myplugin_output', $content);
```

### Indentation & Spacing

```php
// Use TABS for indentation, not spaces
function myplugin_example() {
	$foo = 'bar';        // Tab indented
	
	if ($foo === 'bar') {
		echo 'Hello';
	}
}

// Spaces around operators
$result = $a + $b;
$name = $first . ' ' . $last;
$check = ($a === $b);

// Spaces after commas
my_function($arg1, $arg2, $arg3);

// No space before opening parenthesis in function calls
my_function();          // Good
my_function ();         // Bad

// Space before opening brace
if ($condition) {       // Good
if ($condition){        // Bad
```

### Yoda Conditions

```php
// Put constant/literal on left side
if ('active' === $status) {}      // Good - Yoda
if ($status === 'active') {}      // Acceptable but not preferred

if (true === $is_valid) {}        // Good
if (null !== $value) {}           // Good
```

### Strict Comparisons

```php
// Always use strict comparisons
if ($a === $b) {}                 // Good
if ($a == $b) {}                  // Bad

if ($value !== false) {}          // Good
if ($value != false) {}           // Bad

// Type checking
if (is_array($data)) {}
if (is_string($input)) {}
```

### String Formatting

```php
// Single quotes for simple strings
$name = 'John';

// Double quotes when interpolation needed
$greeting = "Hello, {$name}!";

// Concatenation with spaces around dot
$full_name = $first . ' ' . $last;

// Use sprintf for complex formatting
$message = sprintf(
    __('User %s has %d posts.', 'my-plugin'),
    $user_name,
    $post_count
);
```

### DocBlocks

```php
/**
 * Calculate the total price with tax.
 *
 * Calculates the final price by adding the specified
 * tax rate to the base price.
 *
 * @since 1.0.0
 * @since 1.2.0 Added $include_shipping parameter.
 *
 * @param float  $price           Base price.
 * @param float  $tax_rate        Tax rate as decimal (e.g., 0.20 for 20%).
 * @param bool   $include_shipping Optional. Whether to include shipping. Default false.
 * @return float Total price with tax.
 */
function myplugin_calculate_total($price, $tax_rate, $include_shipping = false) {
    $total = $price * (1 + $tax_rate);
    
    if ($include_shipping) {
        $total += myplugin_get_shipping_cost();
    }
    
    return $total;
}

/**
 * Class for handling user operations.
 *
 * @since 1.0.0
 */
class MyPlugin_User_Handler {
    
    /**
     * User ID.
     *
     * @since 1.0.0
     * @var int
     */
    private $user_id;
    
    /**
     * Constructor.
     *
     * @since 1.0.0
     *
     * @param int $user_id User ID.
     */
    public function __construct($user_id) {
        $this->user_id = $user_id;
    }
}
```

### Hooks Documentation

```php
/**
 * Fires after a question is saved.
 *
 * @since 1.0.0
 *
 * @param int   $question_id Question post ID.
 * @param array $data        Submitted form data.
 */
do_action('myplugin_after_question_save', $question_id, $data);

/**
 * Filters the question content before display.
 *
 * @since 1.0.0
 *
 * @param string $content     Question content.
 * @param int    $question_id Question post ID.
 * @return string Modified content.
 */
$content = apply_filters('myplugin_question_content', $content, $question_id);
```

## JavaScript Standards

### Naming Conventions

```javascript
// Variables and functions: camelCase
const userName = 'John';
function getUserData() {}

// Classes and constructors: PascalCase
class UserHandler {}
function UserModel() {}

// Constants: UPPER_SNAKE_CASE
const MAX_ITEMS = 100;
const API_ENDPOINT = '/api/v1/';

// jQuery objects: prefix with $
const $container = jQuery('.container');
const $buttons = jQuery('button');
```

### Spacing

```javascript
// Spaces around operators
const result = a + b;
const check = (a === b);

// Spaces after keywords
if (condition) {}
for (let i = 0; i < 10; i++) {}
while (condition) {}

// Spaces in objects and arrays
const obj = { key: 'value', another: 'value' };
const arr = [ 1, 2, 3 ];

// No space before function parenthesis
function myFunction() {}      // Good
function myFunction () {}     // Bad
```

### Strict Equality

```javascript
// Always use strict equality
if (a === b) {}               // Good
if (a == b) {}                // Bad

if (value !== undefined) {}   // Good
if (value != undefined) {}    // Bad
```

### Modern JavaScript

```javascript
// Use const and let, not var
const immutableValue = 'constant';
let mutableValue = 'can change';

// Arrow functions for callbacks
items.forEach((item) => {
    console.log(item);
});

// Template literals
const greeting = `Hello, ${name}!`;

// Destructuring
const { id, name } = user;
const [first, second] = items;

// Async/await for promises
async function fetchData() {
    try {
        const response = await fetch(url);
        const data = await response.json();
        return data;
    } catch (error) {
        console.error('Error:', error);
    }
}
```

### jQuery IIFE Pattern

```javascript
(function($) {
    'use strict';
    
    $(document).ready(function() {
        // Code here
        $('.my-button').on('click', function(e) {
            e.preventDefault();
            // Handle click
        });
    });
})(jQuery);
```

## CSS Standards

### Naming Conventions

```css
/* Lowercase with hyphens */
.my-plugin-container {}
.my-plugin-button-primary {}
.my-plugin-form-field {}

/* BEM methodology */
.block {}
.block__element {}
.block--modifier {}

/* Example */
.myplugin-card {}
.myplugin-card__title {}
.myplugin-card__content {}
.myplugin-card--featured {}
```

### Property Order

```css
.my-element {
    /* Positioning */
    position: absolute;
    top: 0;
    right: 0;
    z-index: 10;
    
    /* Display & Box Model */
    display: flex;
    width: 100%;
    padding: 20px;
    margin: 10px;
    border: 1px solid #ccc;
    
    /* Typography */
    font-family: sans-serif;
    font-size: 16px;
    line-height: 1.5;
    color: #333;
    
    /* Visual */
    background-color: #fff;
    border-radius: 4px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    
    /* Animation */
    transition: all 0.3s ease;
}
```

## Tools Configuration

### PHPCS (phpcs.xml.dist)

```xml
<?xml version="1.0"?>
<ruleset name="My Plugin Coding Standards">
    <description>PHPCS ruleset for My Plugin</description>
    
    <!-- Scan these files -->
    <file>.</file>
    
    <!-- Exclude paths -->
    <exclude-pattern>/vendor/*</exclude-pattern>
    <exclude-pattern>/node_modules/*</exclude-pattern>
    <exclude-pattern>/build/*</exclude-pattern>
    
    <!-- Use WordPress Coding Standards -->
    <rule ref="WordPress">
        <exclude name="WordPress.Files.FileName.InvalidClassFileName"/>
    </rule>
    
    <!-- Set text domain -->
    <rule ref="WordPress.WP.I18n">
        <properties>
            <property name="text_domain" type="array">
                <element value="my-plugin"/>
            </property>
        </properties>
    </rule>
    
    <!-- Set minimum PHP version -->
    <config name="testVersion" value="8.0-"/>
    
    <!-- Prefixes -->
    <rule ref="WordPress.NamingConventions.PrefixAllGlobals">
        <properties>
            <property name="prefixes" type="array">
                <element value="myplugin"/>
                <element value="MyPlugin"/>
            </property>
        </properties>
    </rule>
</ruleset>
```

### PHPStan (phpstan.neon)

```neon
parameters:
    level: 6
    paths:
        - includes
        - admin
    excludePaths:
        - vendor
        - node_modules
    scanDirectories:
        - vendor/wordpress/wordpress/src
    checkMissingIterableValueType: false
```

### ESLint (.eslintrc.json)

```json
{
    "root": true,
    "extends": [
        "plugin:@wordpress/eslint-plugin/recommended"
    ],
    "env": {
        "browser": true,
        "jquery": true
    },
    "globals": {
        "wp": "readonly",
        "myPluginData": "readonly"
    },
    "rules": {
        "no-console": "warn",
        "prefer-const": "error"
    }
}
```

### EditorConfig (.editorconfig)

```ini
# EditorConfig helps maintain consistent coding styles

root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
indent_style = tab
indent_size = 4

[*.{json,yml,yaml}]
indent_style = space
indent_size = 2

[*.md]
trim_trailing_whitespace = false
```
