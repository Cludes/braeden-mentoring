---
layout: page
title: "Build a Project: Personal Portfolio Page"
week_number: "6"
release_date: "2026-05-26"
phase_label: "Phase 1"
phase_name: "Web Foundations"
phase_color: "#4361ee"
permalink: /week-06/
---

> Week 6. This week is a project week.
> 
> No new theory. You are building something real using everything from the last 5 weeks.
> 
> This is intentional. The best way to find out what you actually understand is to build something without step-by-step instructions.
> 
> If you get stuck, that is normal and expected. Write down exactly what you tried. That is the skill.

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

# JavaScript and the DOM: Changing What's on the Page

## What You Will Be Able to Do

By the end of this guide you will be able to:

- Explain what the DOM is
- Select elements on the page using JavaScript
- Change text, styles, and attributes of elements
- React to user actions (click, input, submit) with event listeners
- Create and remove elements dynamically

---

## What is the DOM?

When a browser loads an HTML page it builds a tree of objects called the **DOM** (Document Object Model). Each HTML element becomes a node in this tree.

```
document
  └── html
        ├── head
        │     └── title
        └── body
              ├── h1 ("Hello")
              ├── p  ("First paragraph")
              └── ul
                    ├── li ("Item 1")
                    └── li ("Item 2")
```

JavaScript can read and modify this tree. When you change a DOM node, the browser immediately updates the page. No reload needed.

---

## Selecting Elements

```javascript
// By ID (returns one element or null)
const btn = document.getElementById("submit-btn");

// By CSS selector -- returns the FIRST match
const heading = document.querySelector("h1");
const intro = document.querySelector(".intro");      // class
const email = document.querySelector("#email");      // id
const input = document.querySelector("form input");  // nested selector

// By CSS selector -- returns ALL matches (as a NodeList)
const items = document.querySelectorAll("li");
const buttons = document.querySelectorAll(".btn");
```

`querySelector` uses the same syntax as CSS. If you know CSS selectors, you already know how to select elements in JS.

**Check if an element exists before using it:**

```javascript
const btn = document.querySelector("#submit-btn");
if (btn) {
  // safe to use btn here
}
```

---

## Reading and Changing Content

```javascript
const heading = document.querySelector("h1");

// Read current text
console.log(heading.textContent); // "Hello"

// Change text (safe -- treats value as plain text)
heading.textContent = "Welcome, Braeden!";

// Change HTML (use with caution -- can create XSS vulnerabilities if content comes from user input)
heading.innerHTML = "Welcome, <strong>Braeden</strong>!";
```

**Security rule:** Never use `innerHTML` with data that came from a user or external source. Use `textContent` for plain text.

---

## Reading and Changing Attributes

```javascript
const link = document.querySelector("a");

// Read
link.getAttribute("href");   // "/about"

// Set
link.setAttribute("href", "/contact");

// Remove
link.removeAttribute("target");

// Special attributes have direct properties (faster):
link.href    // read
link.href = "/contact";  // write

const img = document.querySelector("img");
img.src = "new-photo.jpg";
img.alt = "New description";

const input = document.querySelector("input");
input.value           // current text in the field
input.value = "";     // clear the field
input.disabled = true;  // disable the input
```

---

## Reading and Changing CSS

```javascript
const box = document.querySelector(".box");

// Inline styles (camelCase property names)
box.style.color = "red";
box.style.backgroundColor = "black";  // background-color -> backgroundColor
box.style.fontSize = "24px";

// Better: toggle CSS classes
box.classList.add("active");       // add a class
box.classList.remove("hidden");    // remove a class
box.classList.toggle("selected");  // add if absent, remove if present
box.classList.contains("active");  // true/false check
```

Using CSS classes is cleaner than inline styles because:
- Styles stay in your CSS file (separation of concerns)
- One JS call can change multiple CSS properties at once
- Easier to maintain

---

## Responding to Events

An event listener waits for something to happen (a click, a keypress, a form submit) and then runs a function.

```javascript
const btn = document.querySelector("#my-btn");

btn.addEventListener("click", function() {
  console.log("Button clicked!");
});

// Arrow function version (same result):
btn.addEventListener("click", () => {
  console.log("Button clicked!");
});
```

The structure is always:
```
element.addEventListener("event-name", functionToRun);
```

---

## Common Events

```javascript
// Click
btn.addEventListener("click", () => { ... });

// Input changes (fires on every keystroke)
input.addEventListener("input", () => {
  console.log("Current value:", input.value);
});

// Form submit
form.addEventListener("submit", (event) => {
  event.preventDefault(); // stop the page from reloading
  console.log("Form submitted");
});

// Mouse enter/leave
box.addEventListener("mouseenter", () => { box.style.background = "blue"; });
box.addEventListener("mouseleave", () => { box.style.background = ""; });

// Key press
document.addEventListener("keydown", (event) => {
  console.log("Key pressed:", event.key);
  if (event.key === "Enter") { ... }
});
```

`event.preventDefault()` on a form submit is one of the most important patterns. Without it, the browser reloads the page and you lose your JS changes.

---

## The `event` Object

The function passed to `addEventListener` receives an event object with information about what happened.

```javascript
btn.addEventListener("click", (event) => {
  console.log(event.type);     // "click"
  console.log(event.target);   // the element that was clicked
});

form.addEventListener("submit", (event) => {
  event.preventDefault();
  const data = new FormData(event.target);
  console.log(data.get("email")); // value of the "email" input
});
```

---

## Creating and Removing Elements

```javascript
// Create a new element
const newItem = document.createElement("li");
newItem.textContent = "New skill: JavaScript";

// Add it to the page
const list = document.querySelector("ul");
list.appendChild(newItem);     // adds at the end
list.prepend(newItem);         // adds at the start

// Remove an element
const oldItem = document.querySelector(".old");
oldItem.remove();              // removes itself from the DOM
```

---

## Full Example: Interactive Counter

HTML:

```html
<div id="counter-app">
  <p id="count">0</p>
  <button id="increment">+1</button>
  <button id="decrement">-1</button>
  <button id="reset">Reset</button>
</div>
```

JavaScript:

```javascript
let count = 0;

const display = document.getElementById("count");
const incrementBtn = document.getElementById("increment");
const decrementBtn = document.getElementById("decrement");
const resetBtn = document.getElementById("reset");

function updateDisplay() {
  display.textContent = count;
  display.style.color = count < 0 ? "red" : "black";
}

incrementBtn.addEventListener("click", () => {
  count++;
  updateDisplay();
});

decrementBtn.addEventListener("click", () => {
  count--;
  updateDisplay();
});

resetBtn.addEventListener("click", () => {
  count = 0;
  updateDisplay();
});
```

This is the core pattern for interactive JS:
1. Store state in a variable (`count`)
2. Update the variable when events happen
3. Call a function to re-render the display

---

## Full Example: Live Form Validation

HTML:

```html
<form id="signup-form">
  <input type="text" id="username" placeholder="Username">
  <span id="username-error" class="error"></span>
  <button type="submit">Sign Up</button>
</form>
```

JavaScript:

```javascript
const form = document.getElementById("signup-form");
const usernameInput = document.getElementById("username");
const errorSpan = document.getElementById("username-error");

usernameInput.addEventListener("input", () => {
  const value = usernameInput.value.trim();
  if (value.length < 3) {
    errorSpan.textContent = "Username must be at least 3 characters.";
  } else if (value.includes(" ")) {
    errorSpan.textContent = "Username cannot contain spaces.";
  } else {
    errorSpan.textContent = "";
  }
});

form.addEventListener("submit", (event) => {
  event.preventDefault();
  if (errorSpan.textContent === "") {
    console.log("Form is valid. Submitting:", usernameInput.value);
  }
});
```

---

## Common Mistakes

1. **Selecting an element that doesn't exist yet.** If your `<script>` tag is in `<head>` and runs before the HTML loads, `querySelector` returns null. Put scripts at the bottom of `<body>` or use `defer`.

2. **Forgetting `event.preventDefault()` on form submit.** The page reloads and you lose your JS state.

3. **Using `innerHTML` to insert untrusted content.** Use `textContent` for user input. `innerHTML` with user data is an XSS vulnerability.

4. **Calling `addEventListener` on null.** If the selector finds nothing, calling `.addEventListener()` on `null` throws a TypeError. Always check the element exists.

5. **Modifying `style` directly when a class is cleaner.** Add/remove CSS classes for behaviour changes. Use `style` only for dynamic values you cannot know in advance (e.g., a position that depends on a calculation).

---

## What to Read Next

- Build your first interactive page by combining `04_html-basics.md`, `05_html-forms-links.md`, `06_javascript-intro.md`, and this guide
- Cybersecurity learning hub for why `innerHTML` + user input is dangerous: `learning/03_cybersecurity/beginner/02_owasp-top10.md`
- MDN DOM introduction: https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Introduction

---

## Activity: Personal Portfolio Page

1. Build a single HTML file called portfolio.html with an external style.css and script.js
1. The page must include: your name in an h1, a short bio paragraph, a skills list, a projects section (at least one project card), and a contact form
1. Style it with CSS: choose a colour scheme, use Flexbox for the layout of the project cards, make it look good on both desktop and a narrow window
1. Add JavaScript: the contact form should validate before submitting (name and email required, email must be valid format). Show an inline success message when the form passes validation
1. Optional: add a dark/light mode toggle button
1. You know you are done when: the page looks intentional (not like default browser styling), the form validates, the layout does not break when you make the window narrow

---

## Check-in Questions

Write your answers down before your next session.

1. What was the hardest part of this project? Describe the problem and how you solved it.
2. How did you centre the project cards horizontally?
3. What does the Flexbox property justify-content: space-between do?
4. If you had to add a second page (e.g. a blog page), what files would you create and how would you link to it?
