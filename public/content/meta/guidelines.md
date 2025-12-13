---
title: Editing Guidelines
description: Style and formatting guidelines for wiki contributors
tags: [meta, guidelines, style]
category: Meta
date: 2025-12-12
---

# Editing Guidelines

This page outlines best practices for creating and editing content on the Slayer Legend Wiki.

## Page Structure

Every page should include:

### Frontmatter
```yaml
---
title: Your Page Title
description: Brief description for search and previews
tags: [relevant, tags]
category: Section Name
date: 2025-12-12
---
```

### Content Organization

1. **Main heading** (`#`) - Page title
2. **Introduction** - Brief overview of the topic
3. **Sections** (`##`) - Major topics
4. **Subsections** (`###`) - Detailed breakdown

## Writing Style

### Clarity
- Use simple, clear language
- Avoid jargon unless necessary
- Define terms on first use
- Keep sentences concise

### Formatting
- Use **bold** for important terms
- Use *italic* for emphasis
- Use `code blocks` for game terms/UI elements
- Use lists for multiple items

### Examples

**Good:**
> The **Promotion System** allows you to upgrade your character to higher tiers, increasing their base stats.

**Avoid:**
> The promotion system thing lets you make your guy stronger by doing promotions which makes stats go up.

## Images

When adding images:

1. Place images in `/public/images/` in appropriate subdirectories
2. Use descriptive filenames: `promotion-mithril-tier.jpg`
3. Add alt text: `![Mithril tier promotion](/images/promotions/mithril-tier.jpg)`
4. Optimize file sizes (keep under 500KB)

## Tables

Use tables for structured data:

```markdown
| Tier | Gold Cost | Stats Bonus |
|------|-----------|-------------|
| Bronze | 10,000 | +5% |
| Silver | 50,000 | +10% |
| Gold | 100,000 | +15% |
```

## Links

### Internal Links
```markdown
[See Promotion Guide](/characters/promotions)
```

### External Links
```markdown
[Official Discord](https://discord.gg/example)
```

## Validation

Before submitting:

1. **Preview your changes** - Use the preview mode
2. **Check metadata** - Ensure all required fields are filled
3. **Verify links** - Make sure all links work
4. **Test formatting** - Check tables, lists, and code blocks

## Common Mistakes

- ❌ Missing frontmatter
- ❌ Broken image links
- ❌ Inconsistent heading levels
- ❌ Malformed tables
- ❌ Unclosed code blocks

## Metadata Requirements

The page editor validates metadata to ensure quality:

- **Title**: Required, max 200 characters
- **Description**: Max 500 characters
- **Tags**: Max 20 tags, 50 characters each
- **Category**: Max 100 characters
- **Date**: Valid date format (YYYY-MM-DD)

## Questions?

If you need help or have questions about these guidelines, visit [How to Contribute](/meta/contributing) or open an issue on GitHub.
