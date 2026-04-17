---
layout: page
title: "JavaScript: Fetch API and Working with Real Data"
week_number: "21"
release_date: "2026-09-08"
phase_label: "Phase 5"
phase_name: "Intermediate JavaScript"
phase_color: "#06d6a0"
permalink: /week-21/
---

> Week 21. Phase 5 starts: intermediate JavaScript.
> 
> You know how to build pages and make them interactive. Now you will connect them to real data from the internet.
> 
> Every app you use -- weather apps, dashboards, social feeds -- uses exactly what you are learning this week: fetch a URL, get JSON back, display it on the page.

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

## Activity: GitHub Profile Viewer

1. Build a page with a text input and a Search button
1. When the button is clicked, fetch https://api.github.com/users/{username} where {username} is the value from the input
1. Display: the user's avatar, name, bio, public repo count, and follower count
1. If the username does not exist (404), show a friendly error message: 'User not found'
1. While the request is in progress, show 'Loading...' in the output area
1. You know you are done when: it shows real data for a valid username, shows an error for an invalid one, and shows a loading state while waiting

---

## Check-in Questions

Write your answers down before your next session.

1. What does await do and why do you need the async keyword to use it?
2. Why does a 404 response NOT throw an error in fetch()? How do you detect it?
3. What is JSON.parse() and when do you need it?
4. What does Promise.all() do and when is it faster than sequential awaits?
