---
layout: page
title: "CSS: Styling Your Pages"
week_number: "3"
release_date: "2026-05-05"
phase_label: "Phase 1"
phase_name: "Web Foundations"
phase_color: "#4361ee"
permalink: /week-03/
---

> Week 3. This week: CSS -- how to make your HTML look good.
> 
> Last week you built a form. Right now it looks like plain text on a white page. By the end of this week you will be able to style it with colours, fonts, spacing, and layout.
> 
> Goal: Add a style.css file to your About Me page and make it look like something you would actually show someone.

---

# CSS Basics: Styling Your Web Pages

## What You Will Be Able to Do

By the end of this guide you will be able to:

- Link a CSS file to an HTML page
- Use element, class, and ID selectors
- Change colours, fonts, sizes, and spacing
- Understand the box model
- Use Flexbox to lay out elements side by side or in a column

---

## What is CSS?

CSS stands for **Cascading Style Sheets**. It controls how HTML elements look.

Without CSS, every page is plain black text on a white background. CSS adds colour, layout, spacing, fonts, and animations.

HTML answers "what is on the page?"
CSS answers "how does it look?"

The "cascading" part means styles flow downward from parent to child, and more specific rules override less specific ones.

---

## How to Link CSS to HTML

**Option 1: External stylesheet (recommended)**

Create a file called `style.css`. Link it in the `<head>` of your HTML:

```html
<head>
  <link rel="stylesheet" href="style.css">
</head>
```

**Option 2: Inline styles (avoid except for testing)**

```html
<p style="color: red; font-size: 18px;">This is red text.</p>
```

Avoid inline styles in real projects -- they are hard to maintain and override.

---

## Basic Syntax

```css
selector {
  property: value;
  property: value;
}
```

Example:

```css
p {
  color: #333333;
  font-size: 16px;
  line-height: 1.6;
}
```

This sets all `<p>` elements to dark grey, 16px, with 1.6x line spacing.

Comments in CSS:

```css
/* This is a comment -- it is ignored by the browser */
```

---

## Selectors

| Selector | CSS syntax | Selects |
|----------|-----------|---------|
| Element | `p` | All `<p>` elements |
| Class | `.intro` | All elements with `class="intro"` |
| ID | `#header` | The element with `id="header"` |
| Descendant | `nav a` | All `<a>` elements inside a `<nav>` |
| Direct child | `ul > li` | `<li>` elements that are direct children of `<ul>` |
| Multiple | `h1, h2, h3` | All h1, h2, and h3 elements |

```css
/* All paragraphs */
p { color: #333; }

/* Only elements with class="warning" */
.warning { color: red; font-weight: bold; }

/* Only the element with id="logo" */
#logo { width: 200px; }

/* All links inside nav */
nav a { text-decoration: none; color: white; }
```

**Class vs ID:**
- A class can be used on many elements: `<p class="intro">`, `<div class="intro">`
- An ID should be used on exactly one element per page: `<header id="main-header">`
- Use classes for styling. Use IDs for JavaScript targeting or anchor links.

---

## Colours

CSS accepts colours in several formats:

```css
color: red;                  /* named colour */
color: #ff0000;              /* hex (most common) */
color: rgb(255, 0, 0);       /* red, green, blue (0-255) */
color: rgba(255, 0, 0, 0.5); /* red, green, blue, alpha (0-1 transparency) */
color: hsl(0, 100%, 50%);    /* hue, saturation, lightness */
```

For picking colours: https://coolors.co or just Google "colour picker".

---

## Typography

```css
body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  font-size: 16px;
  font-weight: 400;      /* normal */
  line-height: 1.6;      /* 1.6 times the font size */
  color: #1a1a1a;
}

h1 {
  font-size: 2rem;       /* rem = relative to root font size (2 * 16px = 32px) */
  font-weight: 700;      /* bold */
  letter-spacing: -0.5px;
}

.highlight {
  font-style: italic;
  text-decoration: underline;
  text-transform: uppercase;
}
```

**px vs rem:**
- `px` is a fixed pixel size
- `rem` is relative to the root font size (usually 16px) - better for accessibility because it respects the user's browser font size settings

---

## The Box Model

Every HTML element is a rectangle. The box model describes the layers around its content:

```
+---------------------------+
|         margin            |
|  +---------------------+  |
|  |      border         |  |
|  |  +---------------+  |  |
|  |  |    padding    |  |  |
|  |  |  +---------+  |  |  |
|  |  |  | content |  |  |  |
|  |  |  +---------+  |  |  |
|  |  +---------------+  |  |
|  +---------------------+  |
+---------------------------+
```

- **content** - the text, image, or child elements
- **padding** - space between content and border (inside the background colour)
- **border** - a line around the padding
- **margin** - space outside the border (between this element and others)

```css
.box {
  width: 300px;
  height: 200px;

  padding: 20px;                 /* all sides */
  padding: 10px 20px;            /* top/bottom | left/right */
  padding: 10px 20px 15px 5px;   /* top | right | bottom | left */

  border: 2px solid #333;
  border-radius: 8px;            /* rounded corners */

  margin: 0 auto;                /* 0 top/bottom, auto left/right = centred */
}
```

**Important:** By default, `width` does not include padding and border. This causes confusion. Fix it globally:

```css
*, *::before, *::after {
  box-sizing: border-box;
}
```

With `border-box`, `width: 300px` means the whole box is 300px including padding and border. Always add this at the top of your CSS.

---

## Display and Layout

Every element has a default `display` value:

| Display | Default elements | Behaviour |
|---------|-----------------|-----------|
| `block` | `div`, `p`, `h1`-`h6`, `ul`, `li` | Takes full width, stacks vertically |
| `inline` | `span`, `a`, `strong`, `em` | Flows with text, width/height ignored |
| `inline-block` | `img`, `button` | Flows with text but respects width/height |

Change it:

```css
a { display: block; }    /* link takes full width */
div { display: inline; } /* div flows with text */
```

---

## Flexbox

Flexbox is the easiest way to lay out items in a row or column. Apply it to the **container**, and it controls the **children**.

```css
.container {
  display: flex;
  flex-direction: row;        /* row (default) or column */
  justify-content: center;    /* horizontal alignment: flex-start | center | flex-end | space-between | space-around */
  align-items: center;        /* vertical alignment: flex-start | center | flex-end | stretch */
  gap: 16px;                  /* space between items */
  flex-wrap: wrap;            /* allow items to wrap to next line */
}
```

Common pattern - centred hero section:

```css
.hero {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  min-height: 100vh;           /* full viewport height */
  text-align: center;
}
```

Common pattern - navigation bar:

```css
nav {
  display: flex;
  justify-content: space-between;  /* logo on left, links on right */
  align-items: center;
  padding: 1rem 2rem;
}
```

Control individual flex children:

```css
.child {
  flex: 1;          /* grow to fill available space equally */
  flex: 0 0 200px;  /* fixed 200px width, don't grow or shrink */
}
```

---

## Pseudo-classes

Pseudo-classes target elements in a specific state:

```css
/* Hover state */
a:hover {
  color: blue;
  text-decoration: underline;
}

button:hover {
  background-color: darkblue;
  cursor: pointer;
}

/* Focus (when element is selected by keyboard or click) */
input:focus {
  outline: 2px solid blue;
  border-color: blue;
}

/* First and last child */
li:first-child { font-weight: bold; }
li:last-child  { border-bottom: none; }

/* nth child */
tr:nth-child(even) { background: #f5f5f5; }
```

---

## Responsive Design: Media Queries

Make your page look good on different screen sizes:

```css
/* Default styles: mobile first */
.container {
  padding: 1rem;
  flex-direction: column;
}

/* Tablet and up (768px+) */
@media (min-width: 768px) {
  .container {
    padding: 2rem;
    flex-direction: row;
  }
}

/* Desktop and up (1024px+) */
@media (min-width: 1024px) {
  .container {
    max-width: 1200px;
    margin: 0 auto;
  }
}
```

---

## CSS Variables

CSS variables (custom properties) let you define values once and reuse them:

```css
:root {
  --primary: #4361ee;
  --text: #1a1a2e;
  --bg: #f8f9fa;
  --radius: 8px;
}

button {
  background: var(--primary);
  border-radius: var(--radius);
  color: white;
}

a {
  color: var(--primary);
}
```

Change `--primary` in one place and every element using it updates.

---

## Quick Reference: Common Properties

```css
/* Colour and background */
color: #333;
background-color: white;
background-image: url("photo.jpg");

/* Text */
font-size: 16px;
font-weight: bold;
text-align: center;
text-decoration: none;
line-height: 1.6;

/* Spacing */
margin: 0 auto;
padding: 1rem 2rem;

/* Box */
width: 100%;
max-width: 760px;
border: 1px solid #ddd;
border-radius: 6px;
box-shadow: 0 2px 8px rgba(0,0,0,0.1);

/* Display and layout */
display: flex;
justify-content: center;
align-items: center;
gap: 16px;

/* Position */
position: relative;
position: absolute;   /* positioned relative to nearest positioned ancestor */
top: 0; right: 0;
```

---

## Common Mistakes

1. **Typos in colour values.** `#ff000` (5 hex chars) silently fails. Always 3 or 6 chars.

2. **Forgetting `box-sizing: border-box`.** Add it at the top of every stylesheet.

3. **Using `margin` to centre text.** Use `text-align: center` for text, `margin: 0 auto` for block elements with a fixed width.

4. **Too specific selectors.** `nav ul li a.active` is hard to override. Prefer a single class: `.nav-link-active`.

5. **Not adding `:hover` styles.** Every interactive element should have a visible hover state.

---

## What to Read Next

- `06_javascript-intro.md` + `07_javascript-dom.md` - Add interactivity to your styled page
- MDN CSS reference: https://developer.mozilla.org/en-US/docs/Web/CSS
- CSS Tricks Flexbox guide: https://css-tricks.com/snippets/css/a-guide-to-flexbox/

---

## Activity: Style Your About Me Page

1. Create a file called style.css in the same folder as your about.html
1. Link it to your HTML with <link rel="stylesheet" href="style.css"> inside <head>
1. Add box-sizing: border-box to * at the top of your CSS file
1. Choose a colour scheme (2-3 colours maximum). Apply a background colour to body and a different colour to headings
1. Style your navigation links: remove the underline, add a hover colour change
1. Add padding and max-width: 760px to your main content area, centred with margin: 0 auto
1. Make your form look clean: labels on their own line, inputs full width, a styled submit button
1. You know you are done when: the page has visible colour, the form looks tidy, and links change colour on hover

---

## Check-in Questions

Write your answers down before your next session.

1. What is the difference between a class selector (.intro) and an ID selector (#header)?
2. What does box-sizing: border-box change about how width is calculated?
3. What are the four layers of the CSS box model from inside to outside?
4. What is the difference between padding and margin?
