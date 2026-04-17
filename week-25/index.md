---
layout: page
title: "Project: JSON-Powered Quiz App"
week_number: "25"
release_date: "2026-10-06"
phase_label: "Phase 5"
phase_name: "Intermediate JavaScript"
phase_color: "#06d6a0"
permalink: /week-25/
---

> Week 25. Phase 5 project week.
> 
> You have now covered: HTML, CSS, JavaScript, the DOM, the Fetch API, localStorage, and Node.js. This week you build a quiz app that pulls questions from a real external API and tracks scores locally.
> 
> This project uses everything from the last 5 weeks. It is your choice how it looks and what features to add beyond the requirements.

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

# JavaScript: Fetch API, JSON, and Async/Await

## What You Will Be Able to Do

By the end of this guide you will be able to:

- Fetch data from a URL using the `fetch()` API
- Parse and use JSON data
- Write async functions using `async`/`await`
- Handle errors from network requests
- Display real API data on a web page

---

## What is an API?

An API (Application Programming Interface) is a URL that returns data instead of a web page. You send a request to the URL and it responds with structured data (usually JSON).

Examples:
- `https://api.github.com/users/octocat` - returns data about a GitHub user
- `https://wttr.in/Sydney?format=j1` - returns weather data for Sydney
- `https://api.coindesk.com/v1/bpi/currentprice.json` - returns Bitcoin price

APIs are how apps talk to each other. Your JavaScript fetches data from an API. Your page renders it.

---

## JSON: The Data Format

JSON (JavaScript Object Notation) is how APIs send data. It looks like a JavaScript object.

```json
{
  "name": "Braeden",
  "age": 20,
  "skills": ["html", "javascript", "python"],
  "location": {
    "city": "Brisbane",
    "country": "Australia"
  }
}
```

Rules:
- Keys must be in double quotes
- Strings must be in double quotes (not single)
- Valid values: string, number, boolean, null, array, object

Convert between JSON and JavaScript objects:

```javascript
// String (what you receive from an API) -> JavaScript object
const obj = JSON.parse('{"name":"Braeden","age":20}');
console.log(obj.name); // "Braeden"

// JavaScript object -> String (what you send to an API)
const str = JSON.stringify({ name: "Braeden", age: 20 });
console.log(str); // '{"name":"Braeden","age":20}'
```

---

## Callbacks, Promises, and Async/Await

JavaScript is single-threaded. Network requests take time. JavaScript uses asynchronous code so the page does not freeze while waiting.

Three ways to handle async code:

**1. Callbacks (old, avoid)**
```javascript
// Nested callbacks become unreadable ("callback hell")
fetchData(function(data) {
  processData(data, function(result) {
    displayResult(result, function() { ... });
  });
});
```

**2. Promises (better)**
```javascript
fetch("https://api.github.com/users/octocat")
  .then(response => response.json())
  .then(data => console.log(data.name))
  .catch(error => console.error(error));
```

**3. Async/Await (best - use this)**
```javascript
async function getUser() {
  const response = await fetch("https://api.github.com/users/octocat");
  const data = await response.json();
  console.log(data.name);
}

getUser();
```

`async`/`await` makes async code read like regular synchronous code. Always prefer it.

---

## The Fetch API

`fetch()` returns a Promise that resolves to a Response object.

```javascript
const response = await fetch("https://api.example.com/data");
```

The Response object has:
- `response.ok` - true if status is 200-299
- `response.status` - the HTTP status code (200, 404, etc.)
- `response.json()` - parse body as JSON (also a Promise)
- `response.text()` - parse body as plain text
- `response.headers` - response headers

**Important:** `fetch()` only throws an error for network failures (no connection). A 404 or 500 response does NOT throw. You must check `response.ok`:

```javascript
async function fetchData(url) {
  const response = await fetch(url);

  if (!response.ok) {
    throw new Error(`HTTP error: ${response.status}`);
  }

  return response.json();
}
```

---

## Full Pattern: Fetch and Display

HTML:

```html
<button id="load-btn">Load User</button>
<div id="output"></div>
```

JavaScript:

```javascript
const btn = document.getElementById("load-btn");
const output = document.getElementById("output");

async function loadUser() {
  output.textContent = "Loading...";

  try {
    const response = await fetch("https://api.github.com/users/octocat");

    if (!response.ok) {
      throw new Error(`Request failed: ${response.status}`);
    }

    const user = await response.json();

    output.innerHTML = `
      <h2>${user.name}</h2>
      <p>${user.bio}</p>
      <p>Public repos: ${user.public_repos}</p>
      <img src="${user.avatar_url}" alt="${user.login}" width="100">
    `;
  } catch (error) {
    output.textContent = `Error: ${error.message}`;
  }
}

btn.addEventListener("click", loadUser);
```

The `try/catch` block catches both network errors and errors you throw manually.

---

## Sending Data (POST Request)

To send data to an API, pass options to `fetch()`:

```javascript
async function createPost(title, body) {
  const response = await fetch("https://jsonplaceholder.typicode.com/posts", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({ title, body, userId: 1 }),
  });

  if (!response.ok) {
    throw new Error(`Failed: ${response.status}`);
  }

  const newPost = await response.json();
  console.log("Created post with ID:", newPost.id);
}
```

Always set `Content-Type: application/json` when sending JSON.

---

## Query Parameters

Many APIs accept parameters in the URL to filter or customise results:

```javascript
// Manual URL construction
const city = "Sydney";
const url = `https://wttr.in/${encodeURIComponent(city)}?format=j1`;

// Using URLSearchParams (cleaner for multiple params)
const params = new URLSearchParams({
  format: "j1",
  lang: "en",
});
const url2 = `https://wttr.in/Sydney?${params.toString()}`;
```

Always use `encodeURIComponent()` for values that come from user input. Spaces and special characters break URLs.

---

## Fetching Multiple Things in Parallel

`Promise.all` runs multiple fetches at the same time instead of one after another:

```javascript
// Sequential (slow - waits for each)
const user = await fetch("/api/user").then(r => r.json());
const posts = await fetch("/api/posts").then(r => r.json());

// Parallel (fast - both start at once)
const [user, posts] = await Promise.all([
  fetch("/api/user").then(r => r.json()),
  fetch("/api/posts").then(r => r.json()),
]);
```

---

## Loading States and Error States

Good UI always shows three states:

```javascript
async function loadData() {
  // 1. Loading state
  showLoading();

  try {
    const data = await fetchSomething();

    // 2. Success state
    showData(data);
  } catch (error) {
    // 3. Error state
    showError(error.message);
  }
}

function showLoading() {
  document.getElementById("output").textContent = "Loading...";
}

function showData(data) {
  const el = document.getElementById("output");
  el.textContent = data.name;
}

function showError(message) {
  const el = document.getElementById("output");
  el.textContent = `Something went wrong: ${message}`;
  el.style.color = "red";
}
```

---

## Free Public APIs for Practice

These APIs require no account or API key:

| API | URL | What it returns |
|-----|-----|----------------|
| GitHub Users | `https://api.github.com/users/{username}` | GitHub profile data |
| JSONPlaceholder | `https://jsonplaceholder.typicode.com/posts` | Fake blog post data |
| Open Library | `https://openlibrary.org/search.json?q=harry+potter` | Book search |
| IP Geolocation | `https://ipapi.co/json/` | Your IP location |
| Joke | `https://official-joke-api.appspot.com/random_joke` | Random programming joke |
| PokeAPI | `https://pokeapi.co/api/v2/pokemon/pikachu` | Pokemon data |
| Dog CEO | `https://dog.ceo/api/breeds/image/random` | Random dog image URL |
| Open-Meteo | `https://api.open-meteo.com/v1/forecast?latitude=-27.47&longitude=153.02&current_weather=true` | Brisbane weather (no key) |

---

## Security: Never Expose API Keys in Frontend JavaScript

If an API requires a key:
- Never put it directly in your `.js` file that gets sent to the browser
- Anyone can open DevTools and read it
- Solution: put the key in a backend (Node.js, Python Flask) that proxies the request

```javascript
// WRONG -- key visible in browser DevTools
const response = await fetch(`https://api.example.com/data?key=MY_SECRET_KEY`);

// RIGHT -- key is on the server, not the browser
const response = await fetch("/api/proxy-endpoint");
```

This is an OWASP vulnerability (A02 - Cryptographic Failures / exposed secrets).

---

## Common Mistakes

1. **Not checking `response.ok`.** A 404 does not throw. Always check.

2. **Forgetting `await` before `fetch()` or `.json()`.** Without it you get a Promise object, not the data.

3. **Trying to use `await` outside an `async` function.** Only works inside `async` functions (or at the top level of a module).

4. **No error handling.** Network requests can always fail. Always `try/catch`.

5. **Building URLs with user input without `encodeURIComponent`.** Spaces and special characters break the URL.

---

## What to Read Next

- `04_python-automation.md` - Using the `requests` library to do the same thing in Python
- Build Week 22 project: a weather app using Open-Meteo

---

## Activity: Quiz App using Open Trivia DB

1. Use the Open Trivia Database API: https://opentdb.com/api.php?amount=10&category=18&type=multiple (category 18 = computers)
1. Fetch 10 questions when the app loads. Display them one at a time with 4 answer options shuffled randomly
1. Track and display the score. At the end show: X/10 correct, a review of wrong answers with the correct answer shown
1. Save the high score to localStorage. Show 'New high score!' when it is beaten
1. Add a 'New Quiz' button that fetches a fresh set of questions
1. Style it properly -- it should look like an actual quiz app
1. You know you are done when: questions are randomised each time, score is tracked, high score persists after refresh

---

## Check-in Questions

Write your answers down before your next session.

1. How did you shuffle the answer options so the correct answer is not always in the same position?
2. What happens if the API is unavailable? How does your app handle it?
3. How did you save and compare the high score?
4. If you were going to add a timer to each question, what JavaScript feature would you use?
