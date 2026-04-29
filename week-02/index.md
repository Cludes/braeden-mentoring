---
layout: page
title: "HTML Forms and Links"
week_number: "2"
release_date: "2026-04-28"
phase_label: "Phase 1"
phase_name: "Web Foundations"
phase_color: "#4361ee"
permalink: /week-02/
---

> Week 2. This week we are covering forms -- how web pages collect information from users.
> 
> Forms are everywhere: login pages, sign-up pages, search bars, contact pages. Understanding how they work is important for both web development and cybersecurity.
> 
> Goal for this week: Read the guide, then add a contact form to the About Me page you built last week.

---

# HTML Forms and Links

## What You Will Be Able to Do

By the end of this guide you will be able to:

- Build a working HTML form with inputs, labels, and a submit button
- Use the correct input types for different kinds of data
- Explain what happens when a form is submitted
- Link pages together correctly

---

## Forms: The Basics

A form collects data from the user and sends it somewhere.

```html
<form action="/submit" method="POST">
  <label for="username">Username:</label>
  <input type="text" id="username" name="username" required>

  <button type="submit">Sign Up</button>
</form>
```

Key attributes on `<form>`:

| Attribute | What it does |
|-----------|-------------|
| `action` | The URL to send the data to |
| `method` | How to send it: `GET` (visible in URL) or `POST` (hidden in request body) |

---

## The `<label>` Element

Every input must have a label. This is required for accessibility (screen readers, keyboard navigation).

The `for` attribute on the label must match the `id` on the input:

```html
<label for="email">Email address:</label>
<input type="email" id="email" name="email">
```

When the label and input are linked:
- Clicking the label focuses the input
- Screen readers read the label aloud before the input

---

## Input Types

HTML has many input types. The browser automatically validates the correct format for each type.

```html
<!-- Plain text -->
<input type="text" name="username">

<!-- Email address (validates @ symbol) -->
<input type="email" name="email">

<!-- Password (characters are hidden) -->
<input type="password" name="password">

<!-- Number (shows up/down arrows, validates number) -->
<input type="number" name="age" min="1" max="120">

<!-- Checkbox (yes/no) -->
<input type="checkbox" name="agree" id="agree">
<label for="agree">I agree to the terms</label>

<!-- Radio buttons (pick one from a group) -->
<input type="radio" name="skill" id="beginner" value="beginner">
<label for="beginner">Beginner</label>

<input type="radio" name="skill" id="advanced" value="advanced">
<label for="advanced">Advanced</label>

<!-- File upload -->
<input type="file" name="resume">

<!-- Hidden field (sent with form but not shown to user) -->
<input type="hidden" name="source" value="signup-page">

<!-- Date picker -->
<input type="date" name="birthday">
```

**Important:** All radio buttons in a group must share the same `name` attribute. The browser uses `name` to know they are in the same group.

---

## Useful Input Attributes

```html
<input
  type="text"
  name="username"
  id="username"
  placeholder="Enter your name"   <!-- hint text inside the field -->
  required                         <!-- form cannot submit if empty -->
  minlength="3"                    <!-- minimum character count -->
  maxlength="50"                   <!-- maximum character count -->
  autocomplete="off"               <!-- disable browser autofill -->
  value="default text"             <!-- pre-filled value -->
>
```

---

## The `<select>` Dropdown

```html
<label for="country">Country:</label>
<select id="country" name="country">
  <option value="">-- Select a country --</option>
  <option value="au">Australia</option>
  <option value="us">United States</option>
  <option value="uk">United Kingdom</option>
</select>
```

The `value` attribute on each `<option>` is what gets sent to the server. The text between the tags is what the user sees.

---

## Multi-line Text: `<textarea>`

```html
<label for="message">Your message:</label>
<textarea id="message" name="message" rows="5" cols="40">
Default text here if needed.
</textarea>
```

Unlike `<input>`, `<textarea>` has a closing tag and the content goes between the tags.

---

## Buttons

```html
<!-- Submits the form -->
<button type="submit">Send</button>

<!-- Resets all fields to their default values -->
<button type="reset">Clear</button>

<!-- Does nothing by default (used with JavaScript) -->
<button type="button" onclick="doSomething()">Click Me</button>
```

Always specify `type` on buttons inside forms. Without it, the default type is `submit`, which can cause accidental form submissions.

---

## A Complete Form Example

```html
<form action="/register" method="POST">
  <h2>Create Account</h2>

  <label for="fullname">Full Name:</label>
  <input type="text" id="fullname" name="fullname" required minlength="2">

  <label for="email">Email:</label>
  <input type="email" id="email" name="email" required>

  <label for="password">Password:</label>
  <input type="password" id="password" name="password" required minlength="8">

  <label for="skill">Skill level:</label>
  <select id="skill" name="skill">
    <option value="beginner">Beginner</option>
    <option value="intermediate">Intermediate</option>
    <option value="advanced">Advanced</option>
  </select>

  <label for="bio">About you:</label>
  <textarea id="bio" name="bio" rows="4"></textarea>

  <label>
    <input type="checkbox" name="terms" required>
    I agree to the terms and conditions
  </label>

  <button type="submit">Create Account</button>
</form>
```

---

## What Happens When a Form Submits?

1. Browser checks all `required` fields and validates `type` constraints
2. If validation passes, data is packaged as key=value pairs (the `name` attributes become the keys)
3. Data is sent to the `action` URL using the `method` (GET or POST)
4. Server receives the data and responds

Without JavaScript, the page reloads. With JavaScript you can intercept the submission and send data in the background (AJAX).

---

## GET vs POST

| | GET | POST |
|-|-----|------|
| Data location | URL query string | Request body |
| Visible to user | Yes (`?name=alice&age=30`) | No |
| Bookmarkable | Yes | No |
| Best for | Search forms, filters | Login, registration, file upload |
| Length limit | Yes (URL max ~2000 chars) | No practical limit |

Example GET URL after form submit:
```
https://example.com/search?query=cybersecurity&level=beginner
```

---

## Common Mistakes

1. **Forgetting `name` attribute on inputs.** The `name` is the key in the submitted data. Without it the field is ignored.

2. **Using `id` instead of `name` for form submission.** `id` is for the browser (linking labels, CSS, JS). `name` is for the server.

3. **Not linking labels to inputs.** Every visible input needs a `<label for="...">`.

4. **Using `GET` for passwords.** Passwords in the URL are visible in browser history, server logs, and anyone looking over your shoulder.

---

## What to Read Next

- `06_javascript-intro.md` - Add behaviour to your forms (validation, dynamic content)
- `07_javascript-dom.md` - Manipulate HTML elements with JavaScript

---

## Activity: Add a Contact Form

1. Open the about.html file you built in Week 1
1. Add a new section called 'Contact' below your existing content
1. Build a form with: a name field (text input), an email field (email input), a message field (textarea), and a submit button
1. Every input must have a label linked with for= and id=
1. Add required to the name and email fields
1. Open the page in your browser and try submitting the form with an invalid email address
1. You know you are done when: the browser prevents submission for an invalid email, each label is clickable and focuses the correct field

---

## Check-in Questions

Write your answers down before your next session.

1. What is the difference between GET and POST? When would you use each?
2. What happens if an input does not have a name attribute when the form is submitted?
3. Why do all radio buttons in a group need the same name attribute?
4. What does event.preventDefault() do and when do you need it?
