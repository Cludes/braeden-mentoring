---
layout: page
title: "Introduction to Node.js"
week_number: "24"
release_date: "2026-09-29"
phase_label: "Phase 5"
phase_name: "Intermediate JavaScript"
phase_color: "#06d6a0"
permalink: /week-24/
---

> Week 24. Node.js -- JavaScript outside the browser.
> 
> Browser JS is sandboxed: it cannot read your files, run shell commands, or listen on network ports. Node.js removes those restrictions. It is how you build web servers, CLI tools, and automation scripts in JavaScript.
> 
> Install Node.js from https://nodejs.org (LTS version). Verify with: node --version && npm --version

---

# JavaScript Intro: Making Pages Interactive

## What You Will Be Able to Do

By the end of this guide you will be able to:

- Explain what JavaScript does and how it runs in the browser
- Use variables, data types, and basic operators
- Write and call a function
- Use `if`, `else`, and loops
- Log output to the browser console

---

## What is JavaScript?

JavaScript is a programming language that runs in the browser. It makes pages interactive.

Examples of what JavaScript does:
- When you click a button and something changes without the page reloading
- When a form shows an error before you even submit it
- When a chat message appears in real time
- When a game runs in the browser

JavaScript is different from HTML and CSS:
- HTML describes **what is on the page** (structure)
- CSS describes **how it looks** (style)
- JavaScript describes **what it does** (behaviour)

---

## How to Add JavaScript to a Page

**Option 1: External file (recommended for anything larger than a few lines)**

```html
<!-- At the bottom of <body>, before </body> -->
<script src="script.js"></script>
```

**Option 2: Inline in the HTML file**

```html
<script>
  console.log("Hello from inline JS");
</script>
```

Place script tags at the **bottom of `<body>`**, not in `<head>`, unless you use `defer`. This ensures the HTML is loaded before the script runs.

---

## The Console

The browser console is your best tool for learning JavaScript. Open it with `F12` (Windows) or `Cmd+Option+J` (Mac), then click "Console".

```javascript
console.log("Hello, World!");     // prints: Hello, World!
console.log(42);                  // prints: 42
console.log(2 + 2);               // prints: 4
console.error("Something broke"); // prints red error message
```

`console.log()` does not change anything on the page. It is for you to see what is happening.

---

## Variables

A variable is a named container that holds a value.

```javascript
let name = "Braeden";   // can be reassigned
const age = 20;          // cannot be reassigned
var old = "avoid this";  // old style, avoid in new code
```

Rules for `let` vs `const`:
- Use `const` by default
- Use `let` only if you need to change the value later
- Never use `var`

Variable names:
- Can contain letters, numbers, `_`, `$`
- Cannot start with a number
- Are case-sensitive (`name` and `Name` are different)
- Use camelCase: `firstName`, `userScore`, `isLoggedIn`

---

## Data Types

JavaScript has 7 primitive types:

```javascript
// String -- text, wrapped in quotes
let greeting = "Hello";
let name = 'Alice';           // single quotes also work
let message = `Hi, ${name}`; // template literal -- embeds variables

// Number -- integers and decimals (no separate int/float)
let score = 95;
let price = 19.99;

// Boolean -- true or false only
let isActive = true;
let hasError = false;

// Null -- intentionally empty
let result = null;

// Undefined -- variable declared but no value assigned
let x;
console.log(x); // undefined

// BigInt -- for very large integers (rare)
let big = 9007199254740991n;

// Symbol -- unique identifier (advanced, rare for beginners)
```

Check the type of a value:

```javascript
typeof "hello"   // "string"
typeof 42        // "number"
typeof true      // "boolean"
typeof null      // "object" (this is a known bug in JS, null is not an object)
typeof undefined // "undefined"
```

---

## Operators

```javascript
// Arithmetic
5 + 3    // 8
10 - 4   // 6
3 * 7    // 21
10 / 4   // 2.5
10 % 3   // 1  (remainder/modulo)
2 ** 8   // 256 (exponentiation)

// String concatenation
"Hello" + " " + "World"  // "Hello World"
`Score: ${95}`            // "Score: 95"

// Comparison (always returns true or false)
5 === 5     // true  (strict equality -- checks value AND type)
5 === "5"   // false (number vs string)
5 !== 3     // true
5 > 3       // true
5 >= 5      // true
5 < 3       // false

// Logical
true && false   // false (AND -- both must be true)
true || false   // true  (OR -- at least one must be true)
!true           // false (NOT -- flips the value)
```

**Always use `===` not `==`**. The `==` operator does type coercion which causes unpredictable results:

```javascript
5 == "5"    // true  (== coerces types -- confusing)
5 === "5"   // false (=== does not coerce -- correct)
```

---

## Conditionals

```javascript
let score = 85;

if (score >= 90) {
  console.log("A");
} else if (score >= 80) {
  console.log("B");
} else if (score >= 70) {
  console.log("C");
} else {
  console.log("Below C");
}
// prints: B
```

Shorthand for simple true/false: the ternary operator

```javascript
let status = score >= 50 ? "Pass" : "Fail";
// same as:
// if (score >= 50) { status = "Pass"; } else { status = "Fail"; }
```

---

## Loops

**`for` loop -- when you know how many times to repeat:**

```javascript
for (let i = 0; i < 5; i++) {
  console.log(i); // prints 0, 1, 2, 3, 4
}
```

The three parts: `let i = 0` (start), `i < 5` (condition), `i++` (step)

**`while` loop -- when you repeat until a condition is false:**

```javascript
let count = 0;
while (count < 3) {
  console.log("Count:", count);
  count++;
}
// prints: Count: 0, Count: 1, Count: 2
```

**`for...of` -- loop over every item in an array:**

```javascript
let fruits = ["apple", "banana", "cherry"];
for (let fruit of fruits) {
  console.log(fruit);
}
// prints: apple, banana, cherry
```

---

## Functions

A function is a reusable block of code. Give it a name, give it inputs (parameters), it does work and returns a result.

```javascript
function greet(name) {
  return "Hello, " + name + "!";
}

let message = greet("Braeden");
console.log(message); // Hello, Braeden!
```

**Arrow functions** (modern shorthand -- same behaviour for most cases):

```javascript
const greet = (name) => {
  return "Hello, " + name + "!";
};

// One-liner: if body is a single expression, you can omit { } and return
const greet = (name) => "Hello, " + name + "!";
```

**Default parameters:**

```javascript
function connect(host, port = 443) {
  return `${host}:${port}`;
}

connect("example.com");        // "example.com:443"
connect("example.com", 8080);  // "example.com:8080"
```

---

## Arrays

An array holds an ordered list of values.

```javascript
let skills = ["html", "css", "javascript"];

// Access by index (starts at 0)
skills[0]  // "html"
skills[2]  // "javascript"

// Length
skills.length  // 3

// Add to end
skills.push("python");  // ["html", "css", "javascript", "python"]

// Remove from end
skills.pop();  // removes "python"

// Check if value exists
skills.includes("css")  // true

// Loop over
for (let skill of skills) {
  console.log(skill);
}
```

---

## Objects

An object holds key-value pairs. Use it to group related data.

```javascript
let user = {
  name: "Braeden",
  age: 20,
  skills: ["html", "javascript"],
  isActive: true
};

// Access properties
user.name        // "Braeden"
user["age"]      // 20 (bracket notation -- useful when key is a variable)

// Add or update
user.email = "braeden@example.com";
user.age = 21;

// Check all keys
Object.keys(user)    // ["name", "age", "skills", "isActive", "email"]
```

---

## Common Mistakes

1. **Using `=` instead of `===` in a condition.** `if (x = 5)` assigns 5 to x. `if (x === 5)` checks if x is 5.

2. **Off-by-one in loops.** `for (let i = 0; i <= 5; i++)` runs 6 times (0 through 5 inclusive). `i < 5` runs 5 times.

3. **Calling a function before defining it** (only matters for arrow functions / `const` functions -- regular `function` declarations are hoisted).

4. **Forgetting `return`.** A function without a `return` statement returns `undefined`.

5. **Mutating a `const` array and thinking it won't work.** `const` prevents reassignment, not mutation. `const arr = []; arr.push(1);` is valid.

---

## What to Read Next

- `07_javascript-dom.md` - Select and change HTML elements with JavaScript
- MDN JavaScript basics: https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web/JavaScript_basics

---

## Activity: Build a Simple HTTP Server

1. Install Node.js if not already installed
1. Create server.js using Node's built-in http module (no external libraries)
1. The server should listen on port 3000 and respond to GET requests:
1. GET / returns a plain HTML page with 'Hello from Node.js'
1. GET /time returns a JSON response: {time: '2026-09-29T10:00:00.000Z'}
1. GET /echo?msg=hello returns JSON: {message: 'hello'}
1. Any other path returns a 404 with a plain text 'Not found' message
1. Run it with node server.js and test with curl http://localhost:3000/ in a second terminal
1. You know you are done when all four routes work correctly

---

## Check-in Questions

Write your answers down before your next session.

1. What is the difference between Node.js and browser JavaScript?
2. What is npm and what is it used for?
3. How is Node.js different from a Python web server like Flask?
4. What does require() do in Node.js? How does it differ from browser ES module import?
