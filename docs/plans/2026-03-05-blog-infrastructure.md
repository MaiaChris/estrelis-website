# Blog Infrastructure Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add a blog to the Estrelis marketing website using Astro Content Collections, with 5 starter articles targeting SEO and thought leadership.

**Architecture:** Astro 5.x Content Collections with `.md` files. Blog posts live in `src/content/blog/`. A shared `BlogPost.astro` layout renders them with Article JSON-LD. Blog listing at `/blog/` and individual posts at `/blog/[slug]/`. Footer gets a "Resources" column with blog link.

**Tech Stack:** Astro 5.17 Content Collections, Markdown, existing CSS variables/patterns.

---

### Task 1: Content Collection Config

**Files:**
- Create: `src/content.config.ts`
- Create: `src/content/blog/` (directory)

**Step 1: Create the content collection config**

Create `src/content.config.ts` (Astro 5.x puts this at `src/content.config.ts`):

```ts
import { defineCollection, z } from 'astro:content';

const blog = defineCollection({
  type: 'content',
  schema: z.object({
    title: z.string(),
    description: z.string(),
    author: z.string(),
    authorTitle: z.string().optional(),
    date: z.date(),
    tags: z.array(z.string()).optional(),
  }),
});

export const collections = { blog };
```

**Step 2: Create the blog content directory**

```bash
mkdir -p src/content/blog
```

**Step 3: Create a test post to verify the collection works**

Create `src/content/blog/test-post.md`:

```markdown
---
title: "Test Post"
description: "Testing content collections"
author: "Estrelis Team"
date: 2026-03-05
---

This is a test post.
```

**Step 4: Verify build succeeds**

Run: `npx astro build`
Expected: Build completes with no errors about content collections.

**Step 5: Delete test post and commit**

```bash
rm src/content/blog/test-post.md
git add src/content.config.ts
git commit -m "feat(blog): add content collection config for blog posts"
```

---

### Task 2: Blog Post Layout

**Files:**
- Create: `src/layouts/BlogPost.astro`

**Step 1: Create the blog post layout**

Create `src/layouts/BlogPost.astro`. This wraps `BaseLayout` and adds:
- Article header (title, date, author byline)
- Article JSON-LD schema
- Content slot for rendered Markdown
- Bottom CTA section

```astro
---
import BaseLayout from './BaseLayout.astro';
import '../styles/content.css';
import '../styles/blog.css';

interface Props {
  title: string;
  description: string;
  author: string;
  authorTitle?: string;
  date: Date;
  tags?: string[];
}

const { title, description, author, authorTitle, date, tags } = Astro.props;
const formattedDate = date.toLocaleDateString('en-US', { year: 'numeric', month: 'long', day: 'numeric' });
const isoDate = date.toISOString();
const canonicalUrl = `https://www.estrelis.ai${Astro.url.pathname}`;
---
<BaseLayout title={`${title} | Estrelis.ai Blog`} description={description} breadcrumbs={[{name: "Blog", url: "/blog/"}, {name: title, url: Astro.url.pathname}]}>
  <script type="application/ld+json" set:html={JSON.stringify({
    "@context": "https://schema.org",
    "@type": "Article",
    "headline": title,
    "description": description,
    "datePublished": isoDate,
    "dateModified": isoDate,
    "author": {
      "@type": author === "Estrelis Team" ? "Organization" : "Person",
      "name": author,
      ...(authorTitle && author !== "Estrelis Team" ? { "jobTitle": authorTitle } : {})
    },
    "publisher": {
      "@id": "https://www.estrelis.ai/#organization"
    },
    "mainEntityOfPage": {
      "@type": "WebPage",
      "@id": canonicalUrl
    }
  })} />

  <article class="blog-post">
    <header class="blog-header">
      <div class="blog-meta">
        <time datetime={isoDate}>{formattedDate}</time>
        <span class="blog-meta-sep">·</span>
        <span class="blog-author">{author}{authorTitle ? `, ${authorTitle}` : ''}</span>
      </div>
      <h1>{title}</h1>
      <p class="blog-description">{description}</p>
      {tags && tags.length > 0 && (
        <div class="blog-tags">
          {tags.map(tag => <span class="blog-tag">{tag}</span>)}
        </div>
      )}
    </header>

    <div class="blog-content content-section">
      <slot />
    </div>
  </article>

  <section class="cta">
    <h2>Ready to modernize your equipment planning?</h2>
    <p>See how Estrelis can transform your next healthcare project.</p>
    <a href="/contact" class="btn-primary">Request a Demo</a>
  </section>
</BaseLayout>
```

**Step 2: Commit**

```bash
git add src/layouts/BlogPost.astro
git commit -m "feat(blog): add BlogPost layout with Article JSON-LD"
```

---

### Task 3: Blog CSS

**Files:**
- Create: `src/styles/blog.css`

**Step 1: Create blog-specific styles**

Create `src/styles/blog.css`:

```css
/* Blog listing page */
.blog-listing {
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
}

.blog-list {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.blog-card {
  display: block;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 2rem;
  text-decoration: none;
  transition: all 0.2s;
}
.blog-card:hover {
  border-color: var(--primary);
  transform: translateY(-2px);
}

.blog-card-meta {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.8rem;
  color: var(--text-muted);
  margin-bottom: 0.75rem;
}

.blog-card h2 {
  font-size: 1.35rem;
  font-weight: 700;
  letter-spacing: -0.02em;
  margin-bottom: 0.5rem;
  color: var(--text);
}

.blog-card p {
  color: var(--text-muted);
  font-size: 0.95rem;
  line-height: 1.6;
}

/* Blog post page */
.blog-post {
  max-width: 800px;
  margin: 0 auto;
}

.blog-header {
  text-align: center;
  padding: 6rem 2rem 2rem;
}

.blog-header h1 {
  font-size: 2.5rem;
  font-weight: 800;
  letter-spacing: -0.03em;
  line-height: 1.15;
  margin-bottom: 1rem;
}

.blog-description {
  color: var(--text-muted);
  font-size: 1.15rem;
  line-height: 1.7;
  max-width: 600px;
  margin: 0 auto;
}

.blog-meta {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  font-size: 0.85rem;
  color: var(--text-muted);
  margin-bottom: 1.5rem;
}

.blog-meta-sep {
  color: var(--border);
}

.blog-tags {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin-top: 1.5rem;
}

.blog-tag {
  display: inline-block;
  padding: 0.25rem 0.75rem;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 100px;
  font-size: 0.75rem;
  color: var(--text-muted);
}

/* Markdown content styling */
.blog-content h2 {
  margin-top: 2.5rem;
}

.blog-content h3 {
  font-size: 1.25rem;
  font-weight: 600;
  margin-top: 2rem;
  margin-bottom: 0.75rem;
}

.blog-content a {
  color: var(--primary);
  text-decoration: underline;
  text-underline-offset: 2px;
}
.blog-content a:hover {
  color: var(--accent);
}

.blog-content strong {
  color: var(--text);
  font-weight: 600;
}

.blog-content blockquote {
  border-left: 3px solid var(--primary);
  padding-left: 1.5rem;
  margin: 1.5rem 0;
  color: var(--text-muted);
  font-style: italic;
}

.blog-content ol {
  list-style: decimal;
  padding-left: 1.5rem;
  margin: 1.5rem 0;
}
.blog-content ol li {
  color: var(--text-muted);
  padding: 0.35rem 0;
  line-height: 1.6;
}

@media (max-width: 768px) {
  .blog-header { padding: 4rem 1.5rem 1.5rem; }
  .blog-header h1 { font-size: 1.85rem; }
  .blog-listing { padding: 1.5rem; }
}

@media (max-width: 480px) {
  .blog-header h1 { font-size: 1.5rem; }
  .blog-card { padding: 1.5rem; }
  .blog-card h2 { font-size: 1.15rem; }
}
```

**Step 2: Commit**

```bash
git add src/styles/blog.css
git commit -m "feat(blog): add blog CSS styles"
```

---

### Task 4: Blog Listing Page

**Files:**
- Create: `src/pages/blog/index.astro`

**Step 1: Create the blog listing page**

Create `src/pages/blog/index.astro`:

```astro
---
import { getCollection } from 'astro:content';
import BaseLayout from '../../layouts/BaseLayout.astro';
import '../../styles/content.css';
import '../../styles/blog.css';

const posts = (await getCollection('blog'))
  .sort((a, b) => b.data.date.valueOf() - a.data.date.valueOf());
---
<BaseLayout title="Blog — Healthcare Equipment Planning Insights | Estrelis.ai" description="Insights on medical equipment planning, hospital construction, AI automation, and healthcare facility management from the Estrelis team." breadcrumbs={[{name: "Blog", url: "/blog/"}]}>

  <section class="content-hero">
    <h1>Blog</h1>
    <p>Insights on medical equipment planning, healthcare construction, and the technology transforming how hospitals get built.</p>
  </section>

  <section class="blog-listing">
    <div class="blog-list">
      {posts.map(post => (
        <a href={`/blog/${post.id}/`} class="blog-card">
          <div class="blog-card-meta">
            <time datetime={post.data.date.toISOString()}>
              {post.data.date.toLocaleDateString('en-US', { year: 'numeric', month: 'long', day: 'numeric' })}
            </time>
            <span class="blog-meta-sep">·</span>
            <span>{post.data.author}</span>
          </div>
          <h2>{post.data.title}</h2>
          <p>{post.data.description}</p>
        </a>
      ))}
    </div>
  </section>

  <section class="cta">
    <h2>Ready to modernize your equipment planning?</h2>
    <p>See how Estrelis can transform your next healthcare project.</p>
    <a href="/contact" class="btn-primary">Request a Demo</a>
  </section>
</BaseLayout>
```

**Step 2: Commit**

```bash
git add src/pages/blog/index.astro
git commit -m "feat(blog): add blog listing page"
```

---

### Task 5: Dynamic Post Page

**Files:**
- Create: `src/pages/blog/[...slug].astro`

**Step 1: Create the dynamic post renderer**

Create `src/pages/blog/[...slug].astro`:

```astro
---
import { getCollection, render } from 'astro:content';
import BlogPost from '../../layouts/BlogPost.astro';

export async function getStaticPaths() {
  const posts = await getCollection('blog');
  return posts.map(post => ({
    params: { slug: post.id },
    props: { post },
  }));
}

const { post } = Astro.props;
const { Content } = await render(post);
---
<BlogPost
  title={post.data.title}
  description={post.data.description}
  author={post.data.author}
  authorTitle={post.data.authorTitle}
  date={post.data.date}
  tags={post.data.tags}
>
  <Content />
</BlogPost>
```

**Step 2: Commit**

```bash
git add src/pages/blog/[...slug].astro
git commit -m "feat(blog): add dynamic blog post page"
```

---

### Task 6: Add Footer Blog Link

**Files:**
- Modify: `src/layouts/BaseLayout.astro` (footer section, around line 177)

**Step 1: Add a "Resources" column to the footer**

In `BaseLayout.astro`, after the "Compare" footer column (line 181), add:

```html
<div class="footer-col">
  <h4>Resources</h4>
  <a href="/blog/">Blog</a>
</div>
```

**Step 2: Update footer grid to accommodate 5 columns**

In the `<style>` section, change `.footer-top` grid from `2fr 1fr 1fr 1fr` to `2fr 1fr 1fr 1fr 1fr`.

**Step 3: Verify build**

Run: `npx astro build`
Expected: Builds with no errors.

**Step 4: Commit**

```bash
git add src/layouts/BaseLayout.astro
git commit -m "feat(blog): add Resources column with blog link to footer"
```

---

### Task 7: Write Starter Article 1 — Hospital Construction Equipment Planning

**Files:**
- Create: `src/content/blog/how-to-plan-medical-equipment-hospital-construction.md`

**Step 1: Write the article**

Target keyword: "how to plan medical equipment hospital construction"

Write a 600-800 word article covering:
- Introduction: scope of equipment planning in new hospitals
- Step-by-step process: needs assessment, catalog selection, room-by-room planning, budget tracking, procurement, installation coordination
- Common pitfalls: disconnected spreadsheets, stale catalogs, manual reconciliation
- How AI/software helps at each step
- Internal links to: `/product/`, `/use-cases/new-construction/`, `/compare/spreadsheets/`
- CTA at end linking to `/contact`

Frontmatter:
```yaml
---
title: "How to Plan Medical Equipment for New Hospital Construction"
description: "A step-by-step guide to medical equipment planning for new hospital construction — from needs assessment through procurement and installation."
author: "Chris Vandivere"
authorTitle: "Founder, Estrelis.ai"
date: 2026-03-05
tags: ["equipment planning", "hospital construction", "new construction"]
---
```

**Step 2: Verify build**

Run: `npx astro build`
Expected: 16+ pages built (15 existing + blog index + this post).

**Step 3: Commit**

```bash
git add src/content/blog/how-to-plan-medical-equipment-hospital-construction.md
git commit -m "content(blog): add article — how to plan medical equipment for hospital construction"
```

---

### Task 8: Write Starter Article 2 — Spreadsheets vs Software

**Files:**
- Create: `src/content/blog/ai-equipment-planning-vs-spreadsheets.md`

**Step 1: Write the article**

Target keyword: "medical equipment planning spreadsheet vs software"

Write a 600-800 word article covering:
- Why spreadsheets are the default in MEQ planning
- Where they break down: version control, stale data, no real-time budgets, collaboration friction
- What purpose-built software solves: single source of truth, AI catalog matching, real-time budgets, audit trails
- Decision framework: when to stay with spreadsheets vs when to switch
- Internal links to: `/compare/spreadsheets/`, `/product/`, `/use-cases/renovation/`

Frontmatter:
```yaml
---
title: "Why Healthcare Teams Are Moving Beyond Spreadsheets for Equipment Planning"
description: "Spreadsheets served equipment planners well for decades. Here's why healthcare teams are switching to purpose-built planning software."
author: "Chris Vandivere"
authorTitle: "Founder, Estrelis.ai"
date: 2026-03-03
tags: ["equipment planning", "spreadsheets", "software"]
---
```

**Step 2: Commit**

```bash
git add src/content/blog/ai-equipment-planning-vs-spreadsheets.md
git commit -m "content(blog): add article — spreadsheets vs equipment planning software"
```

---

### Task 9: Write Starter Article 3 — Field Inventory Best Practices

**Files:**
- Create: `src/content/blog/field-inventory-best-practices-hospital-renovation.md`

**Step 1: Write the article**

Target keyword: "hospital renovation equipment inventory best practices"

Write a 600-800 word article covering:
- Why field inventory is critical for renovations (reuse vs. replace decisions)
- Best practices: systematic room-by-room approach, photo documentation, barcode/asset tag capture, condition assessment
- Common mistakes: incomplete surveys, no photos, inconsistent naming
- Technology: iPad capture, 360-degree photos, AI asset identification, barcode scanning
- Internal links to: `/use-cases/field-inventory/`, `/use-cases/renovation/`, `/product/`

Frontmatter:
```yaml
---
title: "Field Inventory Best Practices for Hospital Renovation Projects"
description: "How to conduct thorough equipment field inventories for hospital renovations — best practices for documentation, technology, and avoiding common mistakes."
author: "Estrelis Team"
date: 2026-03-01
tags: ["field inventory", "renovation", "best practices"]
---
```

**Step 2: Commit**

```bash
git add src/content/blog/field-inventory-best-practices-hospital-renovation.md
git commit -m "content(blog): add article — field inventory best practices for hospital renovation"
```

---

### Task 10: Write Starter Article 4 — Revit BIM Integration

**Files:**
- Create: `src/content/blog/revit-medical-equipment-bim-integration.md`

**Step 1: Write the article**

Target keyword: "revit medical equipment bim integration"

Write a 600-800 word article covering:
- The disconnect between BIM design and equipment planning
- What Revit integration means for equipment planners: room/level hierarchy sync, two-way data flow
- Benefits: architects and planners stay coordinated, room changes auto-reflect, no manual cross-referencing
- How it works at a high level (no screenshots per design rules)
- Internal links to: `/integrations/revit/`, `/product/`, `/use-cases/new-construction/`

Frontmatter:
```yaml
---
title: "How Revit Integration Transforms Medical Equipment Coordination"
description: "Two-way BIM synchronization between Revit models and equipment plans — how it works and why it matters for healthcare construction."
author: "Chris Vandivere"
authorTitle: "Founder, Estrelis.ai"
date: 2026-02-28
tags: ["revit", "BIM", "integration", "coordination"]
---
```

**Step 2: Commit**

```bash
git add src/content/blog/revit-medical-equipment-bim-integration.md
git commit -m "content(blog): add article — Revit BIM integration for medical equipment"
```

---

### Task 11: Write Starter Article 5 — Future of AI in Healthcare Equipment Planning

**Files:**
- Create: `src/content/blog/future-of-healthcare-equipment-planning.md`

**Step 1: Write the article**

This is a thought leadership piece — more opinionated, forward-looking.

Write a 600-800 word article covering:
- Where healthcare equipment planning is today (mostly manual, spreadsheet-driven)
- AI capabilities emerging now: catalog matching, spec comparison, discontinuation detection
- Near-future possibilities: predictive budgeting, automated procurement, AI-driven room templates from historical data
- The shift from reactive to proactive planning
- Why purpose-built AI matters (vs generic tools)
- Internal links to: `/product/`, `/about/`, `/compare/traditional-planning/`

Frontmatter:
```yaml
---
title: "The Future of AI in Healthcare Equipment Planning"
description: "How artificial intelligence is reshaping medical equipment planning — from catalog management to predictive budgeting and automated procurement."
author: "Chris Vandivere"
authorTitle: "Founder, Estrelis.ai"
date: 2026-02-25
tags: ["AI", "healthcare", "future", "thought leadership"]
---
```

**Step 2: Commit**

```bash
git add src/content/blog/future-of-healthcare-equipment-planning.md
git commit -m "content(blog): add article — future of AI in healthcare equipment planning"
```

---

### Task 12: Final Build Verification and Push

**Step 1: Full build**

Run: `npx astro build`
Expected: 20+ pages built (15 existing + 1 blog index + 5 blog posts). No errors.

**Step 2: Local preview**

Run: `npx astro dev --port 4322`
Check:
- `/blog/` — shows 5 posts, newest first
- Click each post — renders with title, author, date, content, CTA
- Footer shows "Resources" column with "Blog" link
- Check mobile responsive (narrow browser)

**Step 3: Push to deploy**

```bash
git push
```

Expected: GitHub Actions deploys to Azure Static Web App within ~1 minute.

**Step 4: Verify on production**

Navigate to `https://www.estrelis.ai/blog/` and spot-check 1-2 posts.
