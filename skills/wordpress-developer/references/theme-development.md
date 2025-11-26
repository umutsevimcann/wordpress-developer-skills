# Theme Development

## Block Theme vs Classic Theme

| Feature | Block Theme | Classic Theme |
|---------|-------------|---------------|
| Template System | HTML files + blocks | PHP files |
| Customization | Site Editor (FSE) | Customizer |
| Required File | `templates/index.html` | `index.php` + `style.css` |
| Global Styles | `theme.json` | CSS + Customizer |
| Template Parts | `parts/` directory | `get_template_part()` |

## Block Theme Structure

```
my-theme/
├── style.css              # Theme metadata
├── theme.json             # Global styles and settings
├── templates/             # Page templates
│   ├── index.html
│   ├── single.html
│   ├── page.html
│   ├── archive.html
│   ├── 404.html
│   └── search.html
├── parts/                 # Reusable template parts
│   ├── header.html
│   ├── footer.html
│   └── sidebar.html
├── patterns/              # Block patterns
│   └── hero.php
├── styles/                # Style variations
│   └── dark.json
└── assets/
    ├── fonts/
    └── images/
```

## theme.json v3

```json
{
    "$schema": "https://schemas.wp.org/wp/6.5/theme.json",
    "version": 3,
    "settings": {
        "appearanceTools": true,
        "color": {
            "palette": [
                {
                    "slug": "primary",
                    "color": "#0073aa",
                    "name": "Primary"
                },
                {
                    "slug": "secondary",
                    "color": "#23282d",
                    "name": "Secondary"
                },
                {
                    "slug": "background",
                    "color": "#ffffff",
                    "name": "Background"
                },
                {
                    "slug": "foreground",
                    "color": "#1e1e1e",
                    "name": "Foreground"
                }
            ],
            "gradients": [
                {
                    "slug": "primary-to-secondary",
                    "gradient": "linear-gradient(135deg, #0073aa 0%, #23282d 100%)",
                    "name": "Primary to Secondary"
                }
            ],
            "defaultPalette": false,
            "defaultGradients": false
        },
        "typography": {
            "fontFamilies": [
                {
                    "fontFamily": "-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif",
                    "slug": "system",
                    "name": "System"
                },
                {
                    "fontFamily": "'Inter', sans-serif",
                    "slug": "inter",
                    "name": "Inter",
                    "fontFace": [
                        {
                            "fontFamily": "Inter",
                            "fontWeight": "400",
                            "fontStyle": "normal",
                            "src": ["file:./assets/fonts/inter-regular.woff2"]
                        },
                        {
                            "fontFamily": "Inter",
                            "fontWeight": "700",
                            "fontStyle": "normal",
                            "src": ["file:./assets/fonts/inter-bold.woff2"]
                        }
                    ]
                }
            ],
            "fontSizes": [
                { "slug": "small", "size": "0.875rem", "name": "Small" },
                { "slug": "medium", "size": "1rem", "name": "Medium" },
                { "slug": "large", "size": "1.25rem", "name": "Large" },
                { "slug": "x-large", "size": "1.5rem", "name": "Extra Large" },
                { "slug": "xx-large", "size": "2.5rem", "name": "Huge" }
            ],
            "fluid": true
        },
        "spacing": {
            "units": ["px", "em", "rem", "%", "vw", "vh"],
            "spacingSizes": [
                { "slug": "10", "size": "0.5rem", "name": "1" },
                { "slug": "20", "size": "1rem", "name": "2" },
                { "slug": "30", "size": "1.5rem", "name": "3" },
                { "slug": "40", "size": "2rem", "name": "4" },
                { "slug": "50", "size": "3rem", "name": "5" }
            ]
        },
        "shadow": {
            "presets": [
                {
                    "slug": "sm",
                    "shadow": "0 1px 2px 0 rgb(0 0 0 / 0.05)",
                    "name": "Small"
                },
                {
                    "slug": "md",
                    "shadow": "0 4px 6px -1px rgb(0 0 0 / 0.1)",
                    "name": "Medium"
                }
            ]
        },
        "layout": {
            "contentSize": "800px",
            "wideSize": "1200px"
        },
        "useRootPaddingAwareAlignments": true
    },
    "styles": {
        "color": {
            "background": "var(--wp--preset--color--background)",
            "text": "var(--wp--preset--color--foreground)"
        },
        "typography": {
            "fontFamily": "var(--wp--preset--font-family--system)",
            "fontSize": "var(--wp--preset--font-size--medium)",
            "lineHeight": "1.6"
        },
        "spacing": {
            "blockGap": "var(--wp--preset--spacing--20)",
            "padding": {
                "left": "var(--wp--preset--spacing--20)",
                "right": "var(--wp--preset--spacing--20)"
            }
        },
        "elements": {
            "link": {
                "color": {
                    "text": "var(--wp--preset--color--primary)"
                },
                ":hover": {
                    "color": {
                        "text": "var(--wp--preset--color--secondary)"
                    }
                }
            },
            "button": {
                "color": {
                    "background": "var(--wp--preset--color--primary)",
                    "text": "#ffffff"
                },
                "border": {
                    "radius": "4px"
                },
                ":hover": {
                    "color": {
                        "background": "var(--wp--preset--color--secondary)"
                    }
                }
            },
            "heading": {
                "typography": {
                    "fontWeight": "700",
                    "lineHeight": "1.2"
                }
            }
        },
        "blocks": {
            "core/navigation": {
                "typography": {
                    "fontSize": "var(--wp--preset--font-size--small)"
                }
            },
            "core/post-title": {
                "typography": {
                    "fontSize": "var(--wp--preset--font-size--xx-large)"
                }
            }
        }
    },
    "templateParts": [
        { "name": "header", "title": "Header", "area": "header" },
        { "name": "footer", "title": "Footer", "area": "footer" }
    ],
    "customTemplates": [
        { "name": "page-no-title", "title": "Page (No Title)", "postTypes": ["page"] }
    ]
}
```

## Block Pattern

```php
<?php
// patterns/hero.php
/**
 * Title: Hero Section
 * Slug: mytheme/hero
 * Categories: featured
 * Keywords: hero, banner, header
 */
?>
<!-- wp:cover {"overlayColor":"primary","minHeight":500,"align":"full"} -->
<div class="wp-block-cover alignfull" style="min-height:500px">
    <span class="wp-block-cover__background has-primary-background-color"></span>
    <div class="wp-block-cover__inner-container">
        <!-- wp:heading {"textAlign":"center","level":1,"textColor":"white"} -->
        <h1 class="wp-block-heading has-text-align-center has-white-color has-text-color">
            <?php echo esc_html__('Welcome to Our Site', 'mytheme'); ?>
        </h1>
        <!-- /wp:heading -->
        
        <!-- wp:paragraph {"align":"center","textColor":"white"} -->
        <p class="has-text-align-center has-white-color has-text-color">
            <?php echo esc_html__('Discover amazing content and resources.', 'mytheme'); ?>
        </p>
        <!-- /wp:paragraph -->
        
        <!-- wp:buttons {"layout":{"type":"flex","justifyContent":"center"}} -->
        <div class="wp-block-buttons">
            <!-- wp:button {"backgroundColor":"white","textColor":"primary"} -->
            <div class="wp-block-button">
                <a class="wp-block-button__link has-primary-color has-white-background-color has-text-color has-background">
                    <?php echo esc_html__('Get Started', 'mytheme'); ?>
                </a>
            </div>
            <!-- /wp:button -->
        </div>
        <!-- /wp:buttons -->
    </div>
</div>
<!-- /wp:cover -->
```

## Template Example (single.html)

```html
<!-- wp:template-part {"slug":"header","tagName":"header"} /-->

<!-- wp:group {"tagName":"main","layout":{"type":"constrained"}} -->
<main class="wp-block-group">
    <!-- wp:post-title {"level":1} /-->
    
    <!-- wp:post-featured-image {"align":"wide"} /-->
    
    <!-- wp:group {"layout":{"type":"flex","flexWrap":"nowrap"},"fontSize":"small"} -->
    <div class="wp-block-group">
        <!-- wp:post-date /-->
        <!-- wp:post-author {"showAvatar":false} /-->
        <!-- wp:post-terms {"term":"category"} /-->
    </div>
    <!-- /wp:group -->
    
    <!-- wp:post-content {"layout":{"type":"constrained"}} /-->
    
    <!-- wp:post-terms {"term":"post_tag"} /-->
    
    <!-- wp:comments /-->
</main>
<!-- /wp:group -->

<!-- wp:template-part {"slug":"footer","tagName":"footer"} /-->
```

## Template Hierarchy

```
Front Page:    front-page.html → home.html → index.html
Single Post:   single-{post-type}-{slug}.html → single-{post-type}.html → single.html → singular.html → index.html
Page:          page-{slug}.html → page-{id}.html → page.html → singular.html → index.html
Category:      category-{slug}.html → category-{id}.html → category.html → archive.html → index.html
Archive:       archive-{post-type}.html → archive.html → index.html
Taxonomy:      taxonomy-{taxonomy}-{term}.html → taxonomy-{taxonomy}.html → taxonomy.html → archive.html → index.html
```

## Child Theme

```css
/* style.css */
/*
Theme Name:   My Child Theme
Theme URI:    https://example.com
Description:  Child theme for Parent Theme
Author:       Author Name
Version:      1.0.0
Template:     parent-theme-folder-name
Text Domain:  my-child-theme
*/
```

```php
<?php
// functions.php
add_action('wp_enqueue_scripts', 'mytheme_child_enqueue_styles');

function mytheme_child_enqueue_styles() {
    wp_enqueue_style(
        'parent-style',
        get_template_directory_uri() . '/style.css'
    );
    
    wp_enqueue_style(
        'child-style',
        get_stylesheet_uri(),
        ['parent-style'],
        wp_get_theme()->get('Version')
    );
}
```

## Using Block Features in Classic Themes

```php
// functions.php - Add block support to classic theme
add_action('after_setup_theme', 'mytheme_setup');

function mytheme_setup() {
    // Add support for block styles
    add_theme_support('wp-block-styles');
    
    // Add support for full and wide align images
    add_theme_support('align-wide');
    
    // Add support for responsive embeds
    add_theme_support('responsive-embeds');
    
    // Add support for editor styles
    add_theme_support('editor-styles');
    add_editor_style('editor-style.css');
    
    // Add support for custom line heights
    add_theme_support('custom-line-height');
    
    // Add support for custom spacing
    add_theme_support('custom-spacing');
    
    // Add support for appearance tools
    add_theme_support('appearance-tools');
}
```
