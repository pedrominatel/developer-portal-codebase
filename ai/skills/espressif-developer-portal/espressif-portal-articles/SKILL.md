---
name: article-writing-skills
description: Skills related to writing articles, including content creation, editing, and publishing.
compatibility: Applies to all agents working in this repository.
---

# Article Writing Skills

## Front matter

Use `archetypes/blog.md` as the template. Required fields:

```yaml
---
title: "Your Article Title"
date: "YYYY-MM-DD"
lastmod: "YYYY-MM-DD"  # only required when the article has been updated after initial publication
showAuthor: false      # set to true to display the default Espressif author instead of listing authors
summary: "One paragraph: what the article is about and what value it brings to the reader. No links or formatting."
authors:
  - "author-name"   # kebab-case, matches file paths under content/authors/ and data/authors/
tags:
  - tag1
  - tag2
---
```

## Author Setup

Before referencing an author in front matter, check whether their entry already exists under `content/authors/`. If it does not, create the following three files:

1. **Author page** — `content/authors/<author-name>/_index.md`
   ```markdown
   ---
   title: Author Name
   ---
   <!-- (optional) A few words about yourself -->
   ```

2. **Author data** — `data/authors/<author-name>.json`
   ```json
   {
       "name": "Author Name",
       "type": "espressif",
       "image": "img/authors/<author-name>.webp",
       "bio": "(optional) Your role at Espressif",
       "social": [
           { "linkedin": "https://www.linkedin.com/..." },
           { "github": "https://github.com/..." }
       ]
   }
   ```
   `type` must be one of: `espressif`, `partner`, `community`.

3. **Author photo** *(optional)* — `assets/img/authors/<author-name>.webp`
   If omitted, use the default image `img/authors/espressif.webp` in the JSON above.

For anonymous, AI-generated, or script-generated articles, skip author file creation and set `showAuthor: true` in front matter to use the default Espressif author instead.

## Tags

- Lowercase, singular form
- **Capitalize** proper nouns and established terms: `ESP32-P4`, `ESP-IDF`, `Wi-Fi`, `SoC`
- Avoid umbrella tags (`IoT`, `Espressif`) and overly narrow tags (single-article tags)
- Use spaces to separate words; hyphens only in established compound terms
- Full guidelines: [Tagging content](content/pages/contribution-guide/tagging-content/index.md)

## File and Folder Naming

- **No underscores** in file names or folder names — use kebab-case only (e.g., `my-article-title/`, not `my_article_title/`)
- The **folder name** should be a short form of the article title (e.g., `ulp-lp-core-get-started`)
- Blog articles must be placed under the path matching their publication date:
  ```
  content/blog/YYYY/MM/<article-folder-name>/index.md
  ```
- The `date` field in front matter must be the **publication date or a future date** — never a past date that predates the actual publication

## Writing Style

Follow the [Espressif Manual of Style](https://mos.espressif.com/) and the *Chicago Manual of Style* for writing and formatting conventions.

### Images

Avoid using standard Markdown image syntax (`![alt](path)`) in articles. Use the `{{< figure >}}` shortcode instead — it gives more control over rendering and avoids grey backgrounds added by Blowfish image processing:

```markdown
{{</* figure
    src="img/screenshot.webp"
    alt="Description of the image"
    caption="Optional caption"
*/>}}
```

Add `default="true"` when the image has a transparent background to skip Blowfish processing.

### Summary

The `summary` front matter field should cover in one paragraph:

1. **What the article is about** (1–2 sentences)
2. **What value it brings to the reader** (1–2 sentences)

No links or formatting in summaries — they appear in article cards and are not rendered as Markdown.

## Contribution Workflow

- Work in a feature branch; PR targets `main` of [espressif/developer-portal](https://github.com/espressif/developer-portal) on GitHub
- **External contributors**: fork the public repo, open a PR from the fork
- **Espressif internal**: prepare in the private GitLab mirror, then push the branch to the public GitHub repo and open a PR
- Ask for review before merging — invite Espressif reviewers via a GitHub discussion or directly on the PR
- Full workflow details: [Contribution workflow](content/pages/contribution-guide/contrib-workflow/index.md)
- Writing guide: [Writing content](content/pages/contribution-guide/writing-content/index.md)
