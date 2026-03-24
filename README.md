# Resume — Static Site for GitHub Pages

## Overview

A single-page resume website that renders dynamically from a JSON data file. No build step, no frameworks, no dependencies. Hosted via GitHub Pages.

## File Structure

```
resume/
├── index.html    — Shell page; fetches data.json and renders all sections via template literals
├── detail.html   — Detail page; reads details.json based on ?id= query param
├── style.css     — All styling; responsive + print-friendly
├── data.json     — Resume content; highlights have IDs linking to detail pages
├── details.json  — Expanded detail content for each experience highlight
└── README.md     — This file
```

## Architecture

- `index.html` contains zero hardcoded resume content. On load, it fetches `data.json` and injects HTML into `<div id="resume">` using JavaScript template literals.
- `style.css` is a standalone stylesheet with no preprocessor. Uses a blue (`#2563eb`) accent color throughout.
- `detail.html` reads the `?id=` query parameter, looks up the matching entry in `details.json`, and renders the expanded view.
- `data.json` is the file to edit for resume content. Each experience highlight has an `id` that maps to a key in `details.json`.
- `details.json` holds expanded content for each highlight (title, description, detail bullets, technologies).

## data.json Schema

```json
{
  "name": "string",
  "title": "string — job title / headline",
  "contact": {
    "email": "string",
    "github": "string — full URL",
    "linkedin": "string — full URL",
    "location": "string"
  },
  "summary": "string — paragraph text",
  "experience": [
    {
      "role": "string",
      "company": "string",
      "period": "string — e.g. 'Jan 2023 – Present'",
      "highlights": [
        {
          "id": "string — unique key matching a details.json entry",
          "text": "string — the bullet point text"
        }
      ]
    }
  ],
  "skills": ["string — one per tag"],
  "education": [
    {
      "degree": "string",
      "institution": "string",
      "period": "string"
    }
  ],
  "projects": [
    {
      "name": "string",
      "url": "string — full URL",
      "description": "string"
    }
  ]
}
```

All arrays (`experience`, `skills`, `education`, `projects`) support any number of entries. The renderer maps over each array, so adding/removing items in the JSON is all that's needed.

## How Rendering Works

1. `fetch('data.json')` loads the JSON.
2. The script builds an HTML string using template literals and `.map().join('')` for arrays.
3. The result is set via `innerHTML` on `#resume`.

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

Accent color: `#2563eb` (used in headings, links, tags, header border).

### Responsive Behavior

- `@media (max-width: 600px)`: removes card border-radius, stacks entry headers vertically.
- `@media print`: removes background color and box shadow for clean printing.

## Editing Guide for LLMs

### To update resume content
Edit `data.json` only. Do not touch `index.html` unless changing the layout structure.

### To add a new section
1. Add the data to `data.json` under a new top-level key.
2. In `index.html`, add a new `<section>` block inside the template literal that maps over the new key.
3. Add any needed CSS classes to `style.css`.

### To change styling
Edit `style.css`. The accent color `#2563eb` appears in: `header` border-bottom, `h2` color, `.contact a` color, `.entry-header a` color, `.tag` color and background (`#eff6ff`).

### To add a new contact field
Add the field to `data.json` → `contact`, then add a corresponding `<a>` or `<span>` in the `<nav class="contact">` template in `index.html`.

## Deployment

Push to GitHub → Settings → Pages → Source: `main` branch, root `/`. No build step required.

## Local Preview

Requires an HTTP server (fetch won't work over `file://`):

```bash
npx serve .
# or
python3 -m http.server
```
