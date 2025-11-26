# WordPress Developer Skills

Professional WordPress development skills for Claude Code. Comprehensive guide for theme and plugin development with WordPress 6.5+.

## Quick Install

```bash
# Add marketplace
/plugin marketplace add umutsevimcann/wordpress-developer-skills

# Install skill
/plugin install wordpress-developer@wordpress-developer-skills
```

## What's Included

### Main SKILL.md
- Plugin development basics and structure
- Custom Post Types & Taxonomies (REST API enabled)
- WordPress Hooks system (Actions & Filters)
- Security best practices (sanitization, escaping, nonces)
- Database operations with $wpdb
- REST API custom endpoints
- AJAX handling (PHP + JavaScript)
- Block registration

### Reference Files

| File | Content |
|------|---------|
| `theme-development.md` | Block Theme, FSE, theme.json v3, patterns |
| `block-development.md` | Interactivity API (6.5+), Gutenberg blocks, directives |
| `woocommerce.md` | Hooks, checkout customization, shipping/payment gateways |
| `performance.md` | Script optimization, lazy loading, caching, preloading |
| `security-checklist.md` | XSS, CSRF, SQL Injection protection |
| `coding-standards.md` | WPCS, naming conventions, PHPCS, ESLint |

## Features

- WordPress 6.5+ compatible
- Interactivity API support
- Block Theme (FSE) development
- WooCommerce integration
- Security best practices
- Performance optimization
- WordPress Coding Standards

## Usage Examples

After installation, just ask Claude Code:

```
"Create a question custom post type with REST API support"
"Build an interactive toggle block with Interactivity API"
"Add a custom payment gateway to WooCommerce"
"Set up theme.json for a block theme"
"Create a secure AJAX handler"
```

## Official WordPress Resources

This skill references official WordPress documentation:
- [Plugin Handbook](https://developer.wordpress.org/plugins/)
- [Theme Handbook](https://developer.wordpress.org/themes/)
- [Block Editor Handbook](https://developer.wordpress.org/block-editor/)
- [REST API Handbook](https://developer.wordpress.org/rest-api/)
- [Security APIs](https://developer.wordpress.org/apis/security/)

## Manual Installation

If you prefer manual installation:

1. Clone this repository
2. Copy `skills/wordpress-developer` to `~/.claude/skills/`
3. Restart Claude Code

## License

MIT License

## Author

**Umut Sevimcan**

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
