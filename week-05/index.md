---
layout: page
title: "JavaScript and the DOM"
week_number: "5"
release_date: "2026-05-19"
phase_label: "Phase 1"
phase_name: "Web Foundations"
phase_color: "#4361ee"
permalink: /week-05/
---

> Week 5. This week: connecting JavaScript to your HTML.
> 
> Last week JavaScript ran in isolation. This week it talks to the page. You will select elements, change their content, and respond to user actions.
> 
> This is how every interactive website works -- a click triggers a JS function that reads the DOM and updates it.

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

## Activity: Interactive Counter + Live Form Validation

1. Build the counter from the DOM guide: three buttons (+1, -1, Reset), a display showing the count
1. Add one extra rule: if the count goes below 0, the number turns red. If it returns to 0 or above, it turns black
1. Then add live form validation to the contact form from Week 2: the email field should show an error message as the user types if the input does not contain @ and a dot
1. The error message should disappear as soon as the input becomes valid
1. You know you are done when: the counter changes colour correctly, the form shows an error in real time without needing to click submit

---

## Check-in Questions

Write your answers down before your next session.

1. What is the difference between textContent and innerHTML? When would you never use innerHTML?
2. What does event.preventDefault() do on a form submit event?
3. What does classList.toggle() do?
4. Why does querySelector return null if no element matches, and why does that cause an error?
