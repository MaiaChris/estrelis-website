# Blog Infrastructure Design

**Date:** 2026-03-05
**Status:** Approved

## Goal

Add a blog to the Estrelis marketing website targeting both long-tail SEO keywords and thought leadership content. 4-5 starter articles.

## Approach

Astro Content Collections with Markdown files. Each post is a `.md` file with typed frontmatter. One shared layout renders all posts.

## File Structure

```
src/
  content/
    blog/
      how-to-plan-medical-equipment-hospital-construction.md
      ai-equipment-planning-vs-spreadsheets.md
      field-inventory-best-practices-hospital-renovation.md
      revit-medical-equipment-bim-integration.md
      future-of-healthcare-equipment-planning.md
    config.ts
  pages/
    blog/
      index.astro
      [...slug].astro
  layouts/
    BlogPost.astro
  styles/
    blog.css
```

## Frontmatter Schema

```yaml
title: string (required)
description: string (required)
author: string (required) — "Chris Vandivere" or "Estrelis Team"
authorTitle: string (optional) — e.g. "Founder, Estrelis.ai"
date: date (required)
tags: string[] (optional)
```

## Pages

- `/blog/` — listing page, newest first, title + date + author + excerpt
- `/blog/[slug]/` — individual post with Article JSON-LD, author byline, CTA
- Blog listing linked from footer ("Resources" column), not in main nav
- Uses `content.css` base + `blog.css` for blog-specific styles

## Starter Articles

| # | Type | Target Keyword | Title |
|---|------|---------------|-------|
| 1 | SEO | how to plan medical equipment hospital construction | How to Plan Medical Equipment for New Hospital Construction |
| 2 | SEO | medical equipment planning spreadsheet vs software | Why Healthcare Teams Are Moving Beyond Spreadsheets for Equipment Planning |
| 3 | SEO | hospital renovation equipment inventory best practices | Field Inventory Best Practices for Hospital Renovation Projects |
| 4 | SEO | revit medical equipment bim integration | How Revit Integration Transforms Medical Equipment Coordination |
| 5 | Thought leadership | future of AI in healthcare construction | The Future of AI in Healthcare Equipment Planning |

## SEO

- Article JSON-LD schema (author, datePublished, publisher)
- Posts auto-included in sitemap
- OG tags with post-specific title/description
- Internal cross-links to product/use-case pages
