# Resume — Static Site for GitHub Pages

## Overview

A single-page resume website that renders dynamically from a JSON data file. No build step, no frameworks, no dependencies. Hosted via GitHub Pages.

## File Structure

```
resume/
├── index.html    — Shell page; fetches data.json and renders sections via type-based renderers
├── detail.html   — Detail page; reads details.json based on ?id= query param
├── style.css     — All styling; responsive + print-friendly
├── data.json     — Resume content; sections array drives the entire layout
├── details.json  — Expanded detail content for each experience highlight
└── README.md     — This file
```

## Architecture

- `index.html` contains zero hardcoded resume content. On load, it fetches `data.json` and iterates over the `sections` array, dispatching each section to a renderer based on its `type`.
- `style.css` is a standalone stylesheet with no preprocessor. Uses a blue (`#2563eb`) accent color throughout.
- `detail.html` reads the `?id=` query parameter, looks up the matching entry in `details.json`, and renders the expanded view.
- `data.json` is the only file to edit for resume content. The `sections` array controls what appears and in what order.
- `details.json` holds expanded content for each experience highlight (title, description, detail bullets, technologies).

## data.json Schema

```json
{
  "name": "string",
  "title": "string — job title / headline",
  "contact": {
    "email": "string (optional)",
    "github": "string — full URL (optional)",
    "linkedin": "string — full URL (optional)",
    "location": "string (optional)"
  },
  "sections": [
    { "type": "text",       "title": "string", "content": "string — paragraph text" },
    { "type": "experience", "title": "string", "items": [
        {
          "role": "string",
          "company": "string",
          "period": "string",
          "highlights": [
            { "id": "string (optional) — links to details.json", "text": "string" }
          ]
        }
      ]
    },
    { "type": "tags",       "title": "string", "items": ["string"] },
    { "type": "education",  "title": "string", "items": [
        { "degree": "string", "institution": "string", "period": "string" }
      ]
    },
    { "type": "projects",   "title": "string", "items": [
        { "name": "string", "url": "string (optional)", "description": "string" }
      ]
    },
    { "type": "list",       "title": "string", "items": ["string — one bullet per entry"] }
  ]
}
```

## Section Types

| Type         | Data Shape                          | Renders As                                      |
|--------------|-------------------------------------|-------------------------------------------------|
| `text`       | `content: "string"`                 | A paragraph                                     |
| `experience` | `items: [{role, company, period, highlights}]` | Entry blocks with clickable highlight bullets |
| `tags`       | `items: ["string"]`                 | Pill-shaped badges in a flex-wrap row            |
| `education`  | `items: [{degree, institution, period}]` | Entry blocks with title and date             |
| `projects`   | `items: [{name, url?, description}]` | Entry blocks with optional linked title         |
| `list`       | `items: ["string"]`                 | A simple `<ul>` bullet list                     |

To add a new section, just append an object to the `sections` array in `data.json` using one of the types above. No code changes needed.

## How Rendering Works

1. `fetch('data.json')` loads the JSON.
2. The script iterates over `d.sections`, looks up a renderer function by `s.type`, and calls it.
3. Each renderer returns an HTML string. Unknown types fall back to the `text` renderer.
4. The result is set via `innerHTML` on `#resume`.

There is no virtual DOM, no framework, no templating engine. It's plain JS in a single `<script>` block inside `index.html`.

## CSS Details

| Class / Element   | Purpose                                      |
|-------------------|----------------------------------------------|
| `.container`      | Centered card (max-width 800px, white bg)    |
| `header`          | Name, title, contact links — centered        |
| `.subtitle`       | Job title under the name                     |
| `.contact`        | Nav with pipe-separated links                |
| `h2`              | Section headings — uppercase, blue, underlined |
| `.entry`          | A single experience/education/project block  |
| `.entry-header`   | Flexbox row: title left, date right          |
| `.skills`         | Flex-wrap container for skill tags           |
| `.tag`            | Pill-shaped skill badge (blue on light blue) |
| `.back-link`      | Return link on detail page                   |

Accent color: `#2563eb` (used in headings, links, tags, header border).

### Responsive Behavior

- `@media (max-width: 600px)`: removes card border-radius, stacks entry headers vertically.
- `@media print`: removes background color and box shadow for clean printing.

## Editing Guide for LLMs

### To update resume content
Edit `data.json` only. Do not touch `index.html` unless adding a new section type.

### To add a new section (using existing types)
Append to the `sections` array in `data.json`. No code changes needed.

### To add a new section type
1. Add a new renderer function in the `renderers` object in `index.html`.
2. Use the new type name in `data.json` sections.
3. Add any needed CSS classes to `style.css`.

### To change styling
Edit `style.css`. The accent color `#2563eb` appears in: `header` border-bottom, `h2` color, `.contact a` color, `.entry-header a` color, `.tag` color and background (`#eff6ff`).

### To add a new contact field
Add the field to `data.json` → `contact`, then add a corresponding entry in the contact array builder in `index.html`.

### Optional fields
- `contact.*` — any field can be omitted; missing fields are excluded from the header
- `highlights[].id` — omit to render as plain text instead of a detail link
- `projects[].url` — omit to render project name without a link

## Deployment

Push to GitHub → Settings → Pages → Source: `main` branch, root `/`. No build step required.

## Local Preview

Requires an HTTP server (fetch won't work over `file://`):

```bash
npx serve .
# or
python3 -m http.server
```
