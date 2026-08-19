# Article Taxonomy Guidelines

These rules apply when creating or updating Markdown articles in `source/_posts`.

## Scope

- Treat only first-level `source/_posts/*.md` files as articles. Subdirectories contain article assets.
- Preserve existing titles, dates, comments settings, filenames, and article bodies when changing taxonomy metadata.
- If a title and the existing metadata do not make the primary subject clear, read the relevant article content before classifying it.

## Categories

- Assign every article exactly one ordered, two-level category path.
- Choose the category from the article's primary subject and intended outcome, not from technologies mentioned incidentally.
- Use the following category paths:
  - `Software Development` → `Algorithms`, `Java`, `Systems Programming`, or `Version Control`
  - `Artificial Intelligence` → `Machine Learning` or `Agent Engineering`
  - `Data Engineering` → `PostgreSQL` or `In-Memory Databases`
  - `Systems and Operations` → `Linux`, `Android`, or `Networking`
  - `Digital Tools` → `E-books` or `Browser Automation`
- Reuse an existing path whenever it accurately describes the article. Add a new category only for a durable subject that does not fit the existing taxonomy.

Use this YAML shape:

```yaml
categories:
  - Top Level Category
  - Topic Category
```

## Tags

- Assign 3–5 precise English tags per article.
- Prefer reusable tags describing the core language, product, platform, protocol, tool, or technical concept.
- Use official names and capitalization, such as `Cloudflare`, `Java`, `PostgreSQL`, `Google Play`, `AZW3`, and `MOBI`.
- Avoid vague labels, incidental technologies, duplicate tags, synonyms, and case-only variants.
- A technology may appear as both a category and a tag when the tag remains useful for cross-category discovery.

Use this YAML shape:

```yaml
tags:
  - Primary Technology
  - Product or Platform
  - Key Concept
```
