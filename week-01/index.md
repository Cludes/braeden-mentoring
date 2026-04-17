---
layout: page
title: "Welcome + HTML: Structure of a Web Page"
week_number: "1"
release_date: "2026-04-21"
phase_label: "Phase 1"
phase_name: "Web Foundations"
phase_color: "#4361ee"
permalink: /week-01/
---

> Welcome Braeden! This is Week 1 of your learning plan.
> 
> This week we are starting with HTML -- the language that every web page is built on.
> 
> By the end of this week you will be able to write a basic HTML page from scratch.
> 
> Goal for this week: Read the guide below, then build your own 'About Me' page using only a text editor and your browser. You do not need to install anything.
> 
> If you get stuck on anything, write down exactly what you tried and what happened. That is the right way to debug.

---

# HTML Basics: Structure of a Web Page

## What You Will Be Able to Do

By the end of this guide you will be able to:

- Explain what HTML is and why browsers need it
- Write a valid HTML document from memory
- Use headings, paragraphs, lists, and images correctly
- Explain the difference between an element, a tag, and an attribute

---

## What is HTML?

HTML stands for **HyperText Markup Language**. It is not a programming language. It is a set of instructions that tells the browser what is on a page -- headings, paragraphs, images, links, etc.

Every webpage you have ever visited is built on HTML.

Think of it like this:
- HTML = the walls, floors, and rooms of a house (structure)
- CSS = the paint, furniture, and decorations (appearance)
- JavaScript = the lights, appliances, and doors (behaviour)

---

## The Anatomy of an HTML Element

An element has three parts:

```
<tagname attribute="value">Content goes here</tagname>
   ^                              ^                 ^
opening tag                   content           closing tag
```

Example:
```html
<p class="intro">This is a paragraph.</p>
```

- `<p>` = opening tag (p stands for paragraph)
- `class="intro"` = an attribute (adds extra info to the element)
- `This is a paragraph.` = the content
- `</p>` = closing tag

Some elements have no content and no closing tag. These are called **void elements**:

```html
<img src="photo.jpg" alt="A cat sitting on a keyboard">
<br>
<hr>
```

---

## A Valid HTML Document

Every HTML page must follow this skeleton:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My First Page</title>
  </head>
  <body>
    <h1>Hello, World!</h1>
    <p>This is my first web page.</p>
  </body>
</html>
```

What each part does:

| Part | Purpose |
|------|---------|
| `<!DOCTYPE html>` | Tells the browser this is modern HTML5 |
| `<html lang="en">` | Root element; `lang` helps screen readers |
| `<head>` | Invisible metadata (title, charset, CSS links) |
| `<meta charset="UTF-8">` | Allows all characters including non-English |
| `<title>` | Text shown in the browser tab |
| `<body>` | Everything visible on the page |

---

## Headings

There are six heading levels. Use them in order -- do not skip levels.

```html
<h1>Main Title (only one per page)</h1>
<h2>Section Title</h2>
<h3>Sub-section Title</h3>
<h4>Smaller sub-section</h4>
<h5>Rarely used</h5>
<h6>Smallest heading</h6>
```

Rule: there should be **exactly one `<h1>`** per page. It describes what the whole page is about.

---

## Paragraphs and Text Formatting

```html
<p>This is a paragraph. Browsers ignore extra spaces and line breaks in HTML.</p>

<p>This is a second paragraph. Each <p> creates a new block of text.</p>

<strong>Bold text -- used for important content</strong>
<em>Italic text -- used for emphasis</em>
<code>Inline code -- used for code snippets</code>
```

To force a line break inside a paragraph:

```html
<p>Line one.<br>Line two is on a new line.</p>
```

---

## Lists

**Unordered list** (bullet points -- order does not matter):

```html
<ul>
  <li>Apples</li>
  <li>Bananas</li>
  <li>Oranges</li>
</ul>
```

**Ordered list** (numbered -- order matters):

```html
<ol>
  <li>Open the terminal</li>
  <li>Navigate to the project folder</li>
  <li>Run the script</li>
</ol>
```

Lists can be nested:

```html
<ul>
  <li>Fruits
    <ul>
      <li>Apples</li>
      <li>Bananas</li>
    </ul>
  </li>
  <li>Vegetables</li>
</ul>
```

---

## Images

```html
<img src="cat.jpg" alt="A tabby cat sitting on a desk" width="400">
```

Attributes:

| Attribute | Required? | Purpose |
|-----------|-----------|---------|
| `src` | Yes | Path to the image file |
| `alt` | Yes | Description for screen readers and when image fails to load |
| `width` | Optional | Width in pixels (height scales automatically if only width is set) |

The `alt` attribute is not optional in practice. Without it the page fails accessibility standards.

---

## Links

```html
<!-- Link to another website -->
<a href="https://developer.mozilla.org">MDN Web Docs</a>

<!-- Link to another page on the same site -->
<a href="about.html">About Us</a>

<!-- Link that opens in a new tab -->
<a href="https://github.com" target="_blank" rel="noopener noreferrer">GitHub</a>

<!-- Link to a section on the same page (anchor) -->
<a href="#contact">Jump to Contact Section</a>

<section id="contact">
  <h2>Contact</h2>
</section>
```

---

## Divisions and Semantic Elements

`<div>` is a generic container with no meaning. Use it when no semantic element fits.

Semantic elements have meaning built in:

```html
<header>   -- site header or article header
<nav>      -- navigation links
<main>     -- main content of the page (only one)
<section>  -- a thematic group of content
<article>  -- self-contained content (blog post, card)
<aside>    -- sidebar or related content
<footer>   -- footer of page or section
```

Semantic elements are preferred over bare `<div>` tags because they help search engines and screen readers understand your page.

Example page structure:

```html
<body>
  <header>
    <h1>My Portfolio</h1>
    <nav>
      <a href="#about">About</a>
      <a href="#projects">Projects</a>
    </nav>
  </header>

  <main>
    <section id="about">
      <h2>About Me</h2>
      <p>I am learning web development.</p>
    </section>

    <section id="projects">
      <h2>Projects</h2>
    </section>
  </main>

  <footer>
    <p>Built by Braeden</p>
  </footer>
</body>
```

---

## Quick Reference

| Element | Purpose |
|---------|---------|
| `<h1>` - `<h6>` | Headings (h1 = most important) |
| `<p>` | Paragraph |
| `<a href="...">` | Link |
| `<img src="..." alt="...">` | Image |
| `<ul>` + `<li>` | Unordered list |
| `<ol>` + `<li>` | Ordered list |
| `<strong>` | Bold / important |
| `<em>` | Italic / emphasis |
| `<div>` | Generic container |
| `<br>` | Line break (void element) |
| `<header>`, `<main>`, `<footer>` | Semantic layout |

---

## Common Mistakes

1. **Forgetting the closing tag.** Most elements need one. If you open `<p>`, close it with `</p>`.

2. **Nesting elements incorrectly.** Wrong: `<p><strong>text</p></strong>`. Right: `<p><strong>text</strong></p>`.

3. **Missing the `alt` attribute on images.** Always include it.

4. **Using `<br>` to add vertical space.** Use CSS margin instead. `<br>` is only for meaningful line breaks (like lines in an address or poem).

5. **Multiple `<h1>` tags.** One per page.

---

## What to Read Next

- `05_html-forms-links.md` - Forms, inputs, buttons
- `06_javascript-intro.md` - Making pages interactive
- MDN HTML reference: https://developer.mozilla.org/en-US/docs/Web/HTML/Element

---

## Activity: Build Your About Me Page

1. Open a plain text editor (Notepad on Windows, TextEdit on Mac, or VS Code if you have it)
1. Create a new file called 'about.html'
1. Build a page that includes: your name in an h1, a short paragraph about yourself, a list of 3 things you want to learn, and at least one link
1. Open the file in your browser by double-clicking it
1. You know you are done when: the page shows your name, the list displays as bullet points, and the link goes somewhere when clicked

---

## Check-in Questions

Write your answers down before your next session.

1. What is the difference between <head> and <body>?
2. What does the alt attribute on an image do?
3. Why should there only be one <h1> per page?
4. What is a void element? Name one example.
