# /doc-to-html

Convert a markdown document to a styled, self-contained HTML page using the CCAF Starter Kit design system.

## When to Use

When you have a markdown document (checklist, guide, pattern catalog, audit report) and need a polished HTML version that:
- Renders on GitHub Pages without external CSS dependencies
- Works on mobile
- Matches the Apple-clean dashboard aesthetic
- Can be opened in any browser or printed to PDF

## Inputs

- `source`: Path to the markdown file to convert
- `output`: Path for the HTML output (defaults to same name with .html extension)
- `title`: Page title (defaults to the first h1 in the markdown)
- `back-link`: URL for the back/home link at top (defaults to `../index.html`)
- `mermaid`: Include Mermaid JS CDN for diagram support (true/false, defaults to false)

## Design Tokens

All CSS is inlined. The only external dependency is Google Fonts (Inter).

```
Backgrounds:   --bg: #FFFFFF  --bg-page: #F2F2F7  --bg-card: #F9F9FB
Text:           --text: #1D1D1F  --text2: #86868B  --text3: #AEAEB2
Accent:         --accent: #FF1D58 (eyebrows and dividers only)
Status:         --green: #34C759  --red: #FF3B30  --blue: #007AFF  --teal: #57D7BA
Border:         --border: rgba(0,0,0,0.06)
Font:           Inter, -apple-system, BlinkMacSystemFont, sans-serif
Container:      max-width 720px, margin auto, padding 40px 24px 60px
```

## Steps

### Step 1: Read the Source Markdown

Read the full markdown file. Identify:
- The document title (first h1)
- Section structure (h2 = major sections, h3 = subsections)
- Special content types: checklists, tables, code blocks, mermaid diagrams
- Any cross-references to other documents

### Step 2: Build the HTML Structure

Create a self-contained HTML file with this skeleton:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{title}</title>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
  <!-- If mermaid=true: -->
  <script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
  <style>
    /* Inline all CSS using the design tokens above */
  </style>
</head>
<body>
  <div class="container">
    <!-- Back link -->
    <!-- Content sections -->
    <!-- Footer -->
  </div>
  <!-- If mermaid=true: -->
  <script>mermaid.initialize({startOnLoad:true, theme:'neutral'});</script>
</body>
</html>
```

### Step 3: Convert Content Elements

Apply these conversion rules:

| Markdown Element | HTML Rendering |
|---|---|
| h1 | Page title in hero section |
| h2 | Section card with uppercase eyebrow-style heading |
| h3 | Subsection heading inside a card |
| Paragraph | `<p>` with 14px, line-height 1.55 |
| Checklist `- [ ]` | Visual card/row with checkbox-style indicator |
| Table | Styled table with alternating row backgrounds |
| Code block | `<pre>` on #F2F2F7 background, monospace font, border-radius 8px |
| Mermaid block | `<div class="mermaid">` (only if mermaid=true) |
| Bold text | `<strong>` |
| Links | Apple blue `#007AFF` with underline |

Special rendering rules:
- Items labeled "RED LINE" or "automatic fail" get a red left border and light red background
- "Anti-pattern" sections get an orange/red left border callout style
- "Areas to improve" or "Key rules" get a blue left border callout style
- PASS/FAIL status text becomes colored pills (green/red)
- Domain references (D1, D2, etc.) become small colored badges

### Step 4: Add Navigation and Footer

Top of page:
```html
<a class="backlink" href="{back-link}">Back to Starter Kit</a>
```

Bottom of page:
```html
<div class="footer">
  <a href="{source-filename}">Download markdown version</a>
  <br>
  Part of the <a href="{back-link}">CCAF Architecture Starter Kit</a>
</div>
```

### Step 5: Responsive Check

Verify the output includes a mobile breakpoint:

```css
@media(max-width:600px){
  .container{padding:24px 16px 40px;}
  h1{font-size:24px;}
  .section{padding:18px 20px;}
  pre{font-size:12px;}
}
```

### Step 6: Write Output

Write the self-contained HTML file to the output path.

## Output

A single `.html` file with:
- All CSS inlined (no external stylesheets except Google Fonts)
- Mobile-responsive layout
- Navigation back to parent page
- Download link to the source markdown
- Footer attribution

## Constraints

- NEVER include external CSS file references (except Google Fonts CDN)
- NEVER use em dashes. Use periods, commas, colons, or rewrite.
- NEVER include proprietary links or internal system references
- All images must be inline (base64) or emoji-based. No external image URLs.
- Keep the HTML under 50KB for fast GitHub Pages loading
- Test that all internal links resolve correctly (relative paths)
