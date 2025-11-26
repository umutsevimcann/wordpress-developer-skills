# Block Development

## Getting Started

```bash
# Create a new block
npx @wordpress/create-block my-block
cd my-block
npm start
```

## Block Structure

```
my-block/
├── build/                 # Compiled files
├── src/
│   ├── block.json        # Block metadata
│   ├── edit.js           # Editor component
│   ├── save.js           # Frontend save
│   ├── index.js          # Block registration
│   ├── editor.scss       # Editor styles
│   ├── style.scss        # Frontend styles
│   ├── render.php        # Dynamic render (optional)
│   └── view.js           # Frontend JS (optional)
├── my-block.php          # Plugin file
└── package.json
```

## block.json (API v3)

```json
{
    "$schema": "https://schemas.wp.org/trunk/block.json",
    "apiVersion": 3,
    "name": "myplugin/custom-card",
    "version": "1.0.0",
    "title": "Custom Card",
    "category": "widgets",
    "icon": "card",
    "description": "A custom card block with image and text.",
    "keywords": ["card", "box", "container"],
    "textdomain": "my-plugin",
    "attributes": {
        "title": {
            "type": "string",
            "default": ""
        },
        "content": {
            "type": "string",
            "default": ""
        },
        "mediaId": {
            "type": "number"
        },
        "mediaUrl": {
            "type": "string"
        },
        "backgroundColor": {
            "type": "string",
            "default": "#ffffff"
        },
        "align": {
            "type": "string",
            "default": "none"
        }
    },
    "supports": {
        "html": false,
        "align": ["wide", "full"],
        "color": {
            "background": true,
            "text": true,
            "gradients": true
        },
        "spacing": {
            "margin": true,
            "padding": true
        },
        "typography": {
            "fontSize": true,
            "lineHeight": true
        },
        "__experimentalBorder": {
            "radius": true,
            "color": true,
            "width": true
        }
    },
    "styles": [
        { "name": "default", "label": "Default", "isDefault": true },
        { "name": "outlined", "label": "Outlined" },
        { "name": "shadow", "label": "Shadow" }
    ],
    "example": {
        "attributes": {
            "title": "Card Title",
            "content": "This is an example card with some content."
        }
    },
    "editorScript": "file:./index.js",
    "editorStyle": "file:./index.css",
    "style": "file:./style-index.css",
    "render": "file:./render.php",
    "viewScript": "file:./view.js"
}
```

## Edit Component (edit.js)

```jsx
import { __ } from '@wordpress/i18n';
import {
    useBlockProps,
    RichText,
    InspectorControls,
    MediaUpload,
    MediaUploadCheck,
    BlockControls,
    AlignmentToolbar,
} from '@wordpress/block-editor';
import {
    PanelBody,
    Button,
    ToggleControl,
    RangeControl,
    SelectControl,
} from '@wordpress/components';
import './editor.scss';

export default function Edit({ attributes, setAttributes }) {
    const { title, content, mediaId, mediaUrl, showButton, buttonText } = attributes;
    
    const blockProps = useBlockProps({
        className: 'custom-card-block',
    });
    
    const onSelectMedia = (media) => {
        setAttributes({
            mediaId: media.id,
            mediaUrl: media.url,
        });
    };
    
    const removeMedia = () => {
        setAttributes({
            mediaId: 0,
            mediaUrl: '',
        });
    };
    
    return (
        <>
            <InspectorControls>
                <PanelBody title={__('Card Settings', 'my-plugin')}>
                    <ToggleControl
                        label={__('Show Button', 'my-plugin')}
                        checked={showButton}
                        onChange={(value) => setAttributes({ showButton: value })}
                    />
                    {showButton && (
                        <RichText
                            tagName="span"
                            placeholder={__('Button text...', 'my-plugin')}
                            value={buttonText}
                            onChange={(value) => setAttributes({ buttonText: value })}
                        />
                    )}
                </PanelBody>
                
                <PanelBody title={__('Image', 'my-plugin')}>
                    <MediaUploadCheck>
                        <MediaUpload
                            onSelect={onSelectMedia}
                            allowedTypes={['image']}
                            value={mediaId}
                            render={({ open }) => (
                                <Button
                                    onClick={open}
                                    variant={mediaId ? 'secondary' : 'primary'}
                                >
                                    {mediaId
                                        ? __('Replace Image', 'my-plugin')
                                        : __('Select Image', 'my-plugin')}
                                </Button>
                            )}
                        />
                    </MediaUploadCheck>
                    {mediaId && (
                        <Button onClick={removeMedia} isDestructive>
                            {__('Remove Image', 'my-plugin')}
                        </Button>
                    )}
                </PanelBody>
            </InspectorControls>
            
            <BlockControls>
                <AlignmentToolbar
                    value={attributes.textAlign}
                    onChange={(value) => setAttributes({ textAlign: value })}
                />
            </BlockControls>
            
            <div {...blockProps}>
                {mediaUrl && (
                    <img src={mediaUrl} alt="" className="card-image" />
                )}
                <RichText
                    tagName="h3"
                    placeholder={__('Card Title...', 'my-plugin')}
                    value={title}
                    onChange={(value) => setAttributes({ title: value })}
                    className="card-title"
                />
                <RichText
                    tagName="p"
                    placeholder={__('Card content...', 'my-plugin')}
                    value={content}
                    onChange={(value) => setAttributes({ content: value })}
                    className="card-content"
                />
            </div>
        </>
    );
}
```

## Dynamic Block (render.php)

```php
<?php
/**
 * @var array    $attributes Block attributes.
 * @var string   $content    Block content.
 * @var WP_Block $block      Block instance.
 */

$wrapper_attributes = get_block_wrapper_attributes([
    'class' => 'custom-card-block',
]);

$title = isset($attributes['title']) ? $attributes['title'] : '';
$content = isset($attributes['content']) ? $attributes['content'] : '';
$media_url = isset($attributes['mediaUrl']) ? $attributes['mediaUrl'] : '';
?>

<div <?php echo $wrapper_attributes; ?>>
    <?php if ($media_url) : ?>
        <img src="<?php echo esc_url($media_url); ?>" alt="" class="card-image" />
    <?php endif; ?>
    
    <?php if ($title) : ?>
        <h3 class="card-title"><?php echo wp_kses_post($title); ?></h3>
    <?php endif; ?>
    
    <?php if ($content) : ?>
        <p class="card-content"><?php echo wp_kses_post($content); ?></p>
    <?php endif; ?>
</div>
```

## Interactivity API (WordPress 6.5+)

The Interactivity API provides a standard way to add client-side interactivity to blocks.

### block.json with Interactivity

```json
{
    "apiVersion": 3,
    "name": "myplugin/interactive-counter",
    "title": "Interactive Counter",
    "supports": {
        "interactivity": true
    },
    "render": "file:./render.php",
    "viewScriptModule": "file:./view.js"
}
```

### render.php with Directives

```php
<?php
$counter_id = wp_unique_id('counter-');
$initial_count = isset($attributes['initialCount']) ? (int) $attributes['initialCount'] : 0;

wp_interactivity_state('myplugin/counter', [
    'count' => $initial_count,
]);
?>

<div
    <?php echo get_block_wrapper_attributes(); ?>
    data-wp-interactive="myplugin/counter"
    data-wp-context='<?php echo wp_json_encode(['counterId' => $counter_id]); ?>'
>
    <button
        data-wp-on--click="actions.decrement"
        data-wp-bind--disabled="state.isMinimum"
    >
        -
    </button>
    
    <span
        data-wp-text="state.count"
        data-wp-class--highlight="state.isHighlighted"
    >
        <?php echo esc_html($initial_count); ?>
    </span>
    
    <button
        data-wp-on--click="actions.increment"
        data-wp-bind--disabled="state.isMaximum"
    >
        +
    </button>
    
    <button data-wp-on--click="actions.reset">
        <?php esc_html_e('Reset', 'my-plugin'); ?>
    </button>
</div>
```

### view.js with Store

```javascript
import { store, getContext } from '@wordpress/interactivity';

const { state, actions } = store('myplugin/counter', {
    state: {
        count: 0,
        get isMinimum() {
            return state.count <= 0;
        },
        get isMaximum() {
            return state.count >= 100;
        },
        get isHighlighted() {
            return state.count > 10;
        },
    },
    actions: {
        increment() {
            if (state.count < 100) {
                state.count += 1;
            }
        },
        decrement() {
            if (state.count > 0) {
                state.count -= 1;
            }
        },
        reset() {
            state.count = 0;
        },
    },
    callbacks: {
        logCount() {
            console.log('Current count:', state.count);
        },
    },
});
```

### Interactivity API Directives

| Directive | Purpose | Example |
|-----------|---------|---------|
| `data-wp-interactive` | Define interactive region | `data-wp-interactive="namespace"` |
| `data-wp-context` | Local context data | `data-wp-context='{"id": 1}'` |
| `data-wp-bind--{attr}` | Bind attribute | `data-wp-bind--disabled="state.isDisabled"` |
| `data-wp-class--{class}` | Toggle class | `data-wp-class--active="state.isActive"` |
| `data-wp-style--{prop}` | Set style | `data-wp-style--color="state.textColor"` |
| `data-wp-text` | Set text content | `data-wp-text="state.message"` |
| `data-wp-on--{event}` | Event handler | `data-wp-on--click="actions.handleClick"` |
| `data-wp-watch` | Side effects | `data-wp-watch="callbacks.onChange"` |
| `data-wp-init` | Initialize | `data-wp-init="callbacks.init"` |
| `data-wp-each` | Loop/iteration | `data-wp-each="state.items"` |

### Toggle Example with Interactivity API

```php
<!-- render.php -->
<?php
wp_interactivity_state('myplugin/toggle', [
    'isOpen' => false,
]);
?>

<div
    <?php echo get_block_wrapper_attributes(); ?>
    data-wp-interactive="myplugin/toggle"
>
    <button
        data-wp-on--click="actions.toggle"
        data-wp-bind--aria-expanded="state.isOpen"
    >
        <span data-wp-text="state.buttonText">Show Content</span>
    </button>
    
    <div
        class="toggle-content"
        data-wp-class--is-open="state.isOpen"
        data-wp-bind--hidden="!state.isOpen"
    >
        <?php echo wp_kses_post($content); ?>
    </div>
</div>
```

```javascript
// view.js
import { store } from '@wordpress/interactivity';

store('myplugin/toggle', {
    state: {
        isOpen: false,
        get buttonText() {
            return state.isOpen ? 'Hide Content' : 'Show Content';
        },
    },
    actions: {
        toggle() {
            state.isOpen = !state.isOpen;
        },
    },
});
```

## Inner Blocks

```jsx
import { useBlockProps, InnerBlocks } from '@wordpress/block-editor';

const ALLOWED_BLOCKS = ['core/paragraph', 'core/heading', 'core/image'];
const TEMPLATE = [
    ['core/heading', { placeholder: 'Enter heading...' }],
    ['core/paragraph', { placeholder: 'Enter content...' }],
];

export default function Edit() {
    const blockProps = useBlockProps();
    
    return (
        <div {...blockProps}>
            <InnerBlocks
                allowedBlocks={ALLOWED_BLOCKS}
                template={TEMPLATE}
                templateLock={false}
            />
        </div>
    );
}

export function Save() {
    const blockProps = useBlockProps.save();
    
    return (
        <div {...blockProps}>
            <InnerBlocks.Content />
        </div>
    );
}
```

## Block Variations

```javascript
import { registerBlockVariation } from '@wordpress/blocks';

registerBlockVariation('core/group', {
    name: 'card-group',
    title: 'Card',
    description: 'A card-style group block',
    icon: 'card',
    attributes: {
        className: 'is-style-card',
        style: {
            border: {
                radius: '8px',
            },
            spacing: {
                padding: '24px',
            },
        },
    },
    innerBlocks: [
        ['core/heading', { level: 3, placeholder: 'Card Title' }],
        ['core/paragraph', { placeholder: 'Card content...' }],
    ],
    scope: ['inserter'],
});
```

## Block Transforms

```javascript
import { createBlock } from '@wordpress/blocks';

const transforms = {
    from: [
        {
            type: 'block',
            blocks: ['core/paragraph'],
            transform: ({ content }) => {
                return createBlock('myplugin/custom-card', {
                    content,
                });
            },
        },
        {
            type: 'enter',
            regExp: /^card$/,
            transform: () => {
                return createBlock('myplugin/custom-card');
            },
        },
    ],
    to: [
        {
            type: 'block',
            blocks: ['core/paragraph'],
            transform: ({ content }) => {
                return createBlock('core/paragraph', {
                    content,
                });
            },
        },
    ],
};

registerBlockType('myplugin/custom-card', {
    // ... other settings
    transforms,
});
```

## Useful Hooks

```javascript
import { useSelect, useDispatch } from '@wordpress/data';
import { useEntityProp } from '@wordpress/core-data';

// Get current post data
const { postType, postId } = useSelect((select) => {
    const { getCurrentPostType, getCurrentPostId } = select('core/editor');
    return {
        postType: getCurrentPostType(),
        postId: getCurrentPostId(),
    };
});

// Get and set post meta
const [meta, setMeta] = useEntityProp('postType', postType, 'meta');
const myMetaValue = meta?.my_meta_key;
const updateMeta = (value) => setMeta({ ...meta, my_meta_key: value });

// Save post
const { savePost } = useDispatch('core/editor');
```
