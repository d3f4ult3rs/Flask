# MODULE 6 — Routing

> **Course:** Flask — Zero to Production
> **Module:** 6 of 39
> **Builds on:** Modules 1–5

---

## 1. Introduction

**Routing** is the mechanism that decides: *"Which piece of Python code should run in response to this specific URL?"*

When a browser visits `http://yoursite.com/about`, the server must figure out: does an "about" page exist? What function returns it? Flask's routing system is the answer to both questions. It is arguably the **most central feature** of Flask — without it, you'd have nothing but a WSGI function with a giant `if/elif` chain for every URL (recall the raw WSGI example from Module 1).

In Flask, routing is built on top of **Werkzeug's URL routing system**, exposed to you through the elegant `@app.route()` decorator. This module digs deeply into how that decorator works, what data structures it builds internally, how URL matching actually happens at request time, and all the nuances of HTTP methods, trailing slashes, and URL rules.

---

## 2. Why Do We Need This?

An HTTP request always contains a **path** (e.g., `/`, `/about`, `/user/42/profile`) and a **method** (e.g., `GET`, `POST`). The server needs to look at both of these and decide what code to run.

Without a routing system, *you*, the developer, would have to manually inspect every incoming request's path in a giant conditional block:

```python
def application(environ, start_response):
    path = environ['PATH_INFO']
    method = environ['REQUEST_METHOD']

    if path == '/' and method == 'GET':
        # home page logic...
    elif path == '/about' and method == 'GET':
        # about page logic...
    elif path == '/user/1/profile' and method == 'GET':
        # profile for user 1...
    elif path == '/user/2/profile' and method == 'GET':
        # profile for user 2...
    # ... and so on, forever
```

This has immediate and obvious problems:
- It doesn't scale — every new URL needs a new `elif`.
- URL variables (like `/user/<id>/profile`) require manual string parsing.
- Multiple developers editing the same block creates merge conflicts constantly.
- HTTP method handling (GET vs POST for the same path) gets tangled.
- There's no clean way to generate URLs from code (what if `/user/profile` changes to `/profile/user`? You'd have to update every hardcoded string everywhere).

A routing system solves **all of this** by turning URL patterns into clean, independently registered Python functions — each one completely self-contained.

---

## 3. The Problem Before Flask's Routing

Early WSGI-based Python code had exactly the "giant `if/elif`" problem described above. Even early routing attempts (in pre-Flask libraries) were often implemented as lists of `(regex_pattern, handler_function)` tuples that got scanned linearly for every request — meaning the server literally checked every registered route one by one until it found a match. With 100 routes, every request evaluated up to 100 regex comparisons. Werkzeug's routing system (which Flask uses) replaced this with a far more efficient **compiled regex map** — a single compiled regex that can test all rules in one pass.

---

## 4. Real-Life Analogy

Think of routing like a **hotel's front desk directory**. The hotel (your web app) has many rooms (pages/endpoints). When a guest arrives and says "I want Room 304" (the URL), the front desk (routing system) immediately looks them up in a directory (the URL Map) and says "Go to the third floor, third door on the right" (calls your view function).

Without the directory, every arriving guest would need to walk the entire hotel, knocking on every door until they found the right room. The directory exists precisely to make lookup instant, regardless of how many rooms the hotel has.

The `@app.route("/about")` decorator is exactly like an entry you write *into that directory*: "Room `about` → managed by the `about_page()` function."

---

## 5. Internal Working — Deeply

This is where most tutorials wave their hands and say "Flask routes URLs to functions, moving on." We're not going to do that.

### 5.1 What `@app.route()` Actually Is

In Python, a **decorator** is syntactic sugar for a function that wraps another function. `@app.route("/")` is equivalent to:

```python
def home():
    return "Hello!"

home = app.route("/")(home)
```

Breaking that down:
- `app.route("/")` is called first — it does **NOT** call `home()`. It returns an **intermediate function** (a closure) that "remembers" the URL rule `"/"`.
- That intermediate function is then immediately called with `home` as its argument.
- Inside this call, Flask takes the `home` function and **registers it** in Werkzeug's URL Map alongside the rule `"/"`.
- The `home` function itself is returned unchanged — the decorator doesn't modify `home`'s behavior, it just *registers* it as a side effect.

So the entire job of `@app.route()` is a **registration side effect**, not a wrapping. Your function works exactly the same as before; Flask has just secretly filed it in its routing directory.

### 5.2 The URL Map — Werkzeug's `Map`

Inside `app.url_map` (recall Module 4), Flask stores a Werkzeug `Map` object. The `Map` holds a collection of `Rule` objects. Each `Rule` represents one registered route and contains:

- The **URL pattern string** (e.g., `"/user/<int:id>"`)
- The **allowed HTTP methods** (e.g., `["GET", "POST"]`)
- The **endpoint name** (a string — by default the name of your view function, e.g., `"user_profile"`)

The `Map` is compiled into an efficient regex structure the first time it's needed, so matching at request time is fast even with many routes.

### 5.3 Endpoint vs View Function — A Critical Distinction

This distinction confuses nearly every beginner, and it's crucial for understanding `url_for()` later (Module 7).

Flask's routing system has **two layers**:

```
URL Pattern  →  Endpoint Name  →  View Function
"/about"     →  "about"        →  about()
```

- An **endpoint** is just a **string name** (by default, the name of the view function).
- The URL Map maps *URL patterns* to *endpoint names*.
- A separate dictionary (`app.view_functions`) maps *endpoint names* to actual *Python function objects*.

Why two steps? Because Blueprints (Module 18) namespace endpoint names (e.g., `"auth.login"` instead of just `"login"`), and `url_for()` uses endpoint names (not URL strings) to generate URLs — making URL generation refactorable without touching every template.

### 5.4 How URL Matching Happens at Request Time

When a request arrives (e.g., `GET /user/42`):

1. Flask calls `app.url_map.bind(environ)` to create a **MapAdapter** — a Werkzeug object that knows the current request's host/script name and can perform matching.
2. The MapAdapter's `match()` method tests the path and method against all registered `Rule` objects in the Map.
3. If a match is found, `match()` returns a **tuple**: `(endpoint_name, url_variable_dict)` — e.g., `("user_profile", {"id": 42})`.
4. Flask looks up `endpoint_name` in `app.view_functions` to get the actual Python function.
5. Flask calls that function, passing any URL variables as keyword arguments.
6. If *no* match is found: a 404 Not Found is returned. If a match is found but the HTTP method is wrong: a 405 Method Not Allowed is returned.

### 5.5 Trailing Slash Behavior — `strict_slashes`

Werkzeug makes an important distinction between routes with and without trailing slashes, and this behavior trips up nearly every beginner at least once:

```python
@app.route("/about")       # Rule: no trailing slash
def about():
    return "About"

@app.route("/blog/")       # Rule: WITH trailing slash (the "folder" convention)
def blog():
    return "Blog"
```

| URL visited | Route registered as | What happens |
|---|---|---|
| `/about` | `/about` | ✅ Match — returns "About" |
| `/about/` | `/about` | 🔄 404 (strict_slashes=True, which is the default) |
| `/blog/` | `/blog/` | ✅ Match — returns "Blog" |
| `/blog` | `/blog/` | 🔄 301 Redirect to `/blog/` |

The logic: routes *with* a trailing slash behave like a folder (you can access it either way; Flask redirects the "wrong" form to the "right" one). Routes *without* a trailing slash behave like a specific file (the slash version gets a 404, not a redirect).

### 5.6 Diagram — Request Routing Flow

```
Browser: GET /user/42
   ↓
Werkzeug dev server
   ↓
app.__call__(environ, start_response)       [Module 4]
   ↓
Flask builds Request object from environ    [Module 9]
   ↓
app.url_map.bind(environ) → MapAdapter
   ↓
MapAdapter.match("/user/42", method="GET")
   ↓
Found: endpoint="user_profile", values={"id": 42}
   ↓
app.view_functions["user_profile"]  → user_profile function
   ↓
user_profile(id=42) called
   ↓
Return value → Flask builds Response        [Module 10]
   ↓
Response sent to browser
```

---

## 6. Syntax

### 6.1 Basic Route

```python
@app.route("/about")
def about():
    return "About Page"
```

### 6.2 Multiple URL Rules for One Function

```python
@app.route("/")
@app.route("/home")
def home():
    return "Home Page"
```

Both `/` and `/home` point to the same function. Each decorator registers a separate `Rule` in the URL Map with the same endpoint name (`"home"`).

### 6.3 Specifying HTTP Methods

```python
@app.route("/submit", methods=["GET", "POST"])
def submit():
    return "Submit Page"
```

By default, a route only accepts `GET` (and implicitly `HEAD` and `OPTIONS` — Flask handles these automatically). You must explicitly list `"POST"` (or others) to allow them.

### 6.4 URL Variables (Preview — Deep Dive in Module 8)

```python
@app.route("/user/<username>")
def user_profile(username):
    return f"Profile: {username}"

@app.route("/post/<int:post_id>")
def show_post(post_id):
    return f"Post: {post_id}"
```

The `<variable>` syntax captures a segment of the URL and passes it as a keyword argument to the view function. The optional `int:` prefix is a **converter** — it converts the captured string to an integer and also only matches if the segment looks like an integer (e.g., `/post/abc` would 404, because `"abc"` can't be converted to `int`).

We cover converters in exhaustive detail in Module 8.

### 6.5 The Full Signature of `@app.route()`

```python
@app.route(
    rule,
    methods=None,
    endpoint=None,
    strict_slashes=None,
    redirect_to=None,
    host=None,
)
```

| Parameter | Meaning |
|---|---|
| `rule` | The URL pattern string (required) |
| `methods` | List of allowed HTTP methods. Defaults to `["GET"]`. |
| `endpoint` | Custom endpoint name string. Defaults to the function's `__name__`. |
| `strict_slashes` | If `False`, the route accepts both `/path` and `/path/`. |
| `redirect_to` | If set, immediately redirects to another URL or endpoint instead of running the view. Rarely used. |
| `host` | For host-based routing — matches only requests to a specific hostname. |

---

## 7. Small Example — Three Routes, Different Methods

```python
from flask import Flask, request

app = Flask(__name__)


@app.route("/")
def home():
    return "<h1>Home Page</h1>"


@app.route("/about")
def about():
    return "<h1>About Us</h1>"


@app.route("/contact", methods=["GET", "POST"])
def contact():
    if request.method == "POST":
        return "Form submitted!"
    return "<form method='post'><button>Submit</button></form>"


if __name__ == "__main__":
    app.run(debug=True)
```

---

## 8. Step-by-Step Explanation

**`from flask import Flask, request`**
Imports the `Flask` class (as before) and the `request` object — a special object that Flask makes available inside every view function, representing the *current* HTTP request. We'll deeply dissect `request` in Module 9; we introduce it here just to demonstrate method checking.

**`app = Flask(__name__)`**
Creates the application object with an empty URL Map (Module 4).

---

**`@app.route("/")`**
This is the decorator call. At Python's import time (when this file is first loaded), `app.route("/")` is called, which:
- Creates a Werkzeug `Rule` object for the pattern `"/"`, accepting only `GET` (default).
- Does NOT call `home()`.
- Returns an intermediate "registrar" function.
That registrar is then immediately called with `home` as its argument, which adds the Rule + endpoint mapping to `app.url_map` and adds `{"home": home_function}` to `app.view_functions`. Then it returns `home` unchanged.

**`def home():`**
The view function for `/`. The name `"home"` automatically becomes its endpoint name.

**`return "<h1>Home Page</h1>"`**
Flask's response system (Module 10) automatically converts a plain returned string to an HTTP response with status `200 OK` and `Content-Type: text/html`.

---

**`@app.route("/about")`**
Same registration process as above, but for the pattern `"/about"` with endpoint `"about"`.

---

**`@app.route("/contact", methods=["GET", "POST"])`**
Registers `"/contact"` accepting *both* `GET` and `POST` requests. Without `methods=["GET", "POST"]`, visiting this URL via a form `POST` would return a `405 Method Not Allowed`.

**`if request.method == "POST":`**
`request.method` is a string — `"GET"`, `"POST"`, `"PUT"`, etc. — reflecting the HTTP method of the current request. This single conditional makes the `contact` function handle both "show the form" (GET) and "process the form submission" (POST) in one place. This is the classic **PRG (Post/Redirect/Get)** pattern's starting point — Module 14 covers it in full.

---

### Program Flow — Three Scenarios

**Scenario 1: Browser visits `GET /`**
```
GET /
 ↓
MapAdapter.match("/", "GET") → ("home", {})
 ↓
app.view_functions["home"]() → "<h1>Home Page</h1>"
 ↓
200 OK response
```

**Scenario 2: Browser visits `GET /about`**
```
GET /about
 ↓
MapAdapter.match("/about", "GET") → ("about", {})
 ↓
app.view_functions["about"]() → "<h1>About Us</h1>"
 ↓
200 OK response
```

**Scenario 3: Browser submits form to `POST /contact`**
```
POST /contact
 ↓
MapAdapter.match("/contact", "POST") → ("contact", {})
 ↓
app.view_functions["contact"]()
 ↓
request.method == "POST" → True
 ↓
returns "Form submitted!" → 200 OK response
```

**Scenario 4: Browser visits `DELETE /about` (not registered)**
```
DELETE /about
 ↓
MapAdapter.match("/about", "DELETE")
 ↓
Match found for path "/about" but not for method "DELETE"
 ↓
405 Method Not Allowed (automatic, Flask handles it)
```

---

## 9. Visual Diagram — The URL Map After Registration

```
 app.url_map (Werkzeug Map)
 ┌───────────────────────────────────────────────────────────────────────────────────┐
 │  Rule("/" , methods=["GET","HEAD","OPTIONS"], endpoint="home")                    │
 │                                                                                   │
 │  Rule("/about", methods=["GET","HEAD","OPTIONS"], endpoint="about")               │
 │                                                                                   │
 │  Rule("/contact"   , methods=["GET","POST","HEAD","OPTIONS"], endpoint="contact") │
 │                                                                                   │
 │  Rule("/static/<path:filename>", endpoint="static")   ← auto-added by Flask       |              │
 │                                                                                   │
 └───────────────────────────────────────────────────────────────────────────────────┘

 app.view_functions
 ┌─────────────────────────────────────────┐
 │  "home"    → <function home at 0x...>   │
 │  "about"   → <function about at 0x...>  │
 │  "contact" → <function contact at 0x...>│
 │  "static"  → Flask's internal handler   │
 └─────────────────────────────────────────┘
```

---

## 10. Common Mistakes

1. **Forgetting `methods=["GET", "POST"]`** for a form submission route — getting a confusing `405 Method Not Allowed` and thinking the route doesn't exist, when it exists but simply refuses `POST`.
2. **Registering the same endpoint name twice** (two functions with the same name) — Flask will raise an `AssertionError` at startup: `"View function mapping is overwriting an existing endpoint function."` Always give view functions unique names.
3. **Confusing the URL pattern with the function name** — the URL pattern (`"/about"`) and the endpoint/function name (`about`) are related by convention but are separate concepts; you can name them differently via the `endpoint=` parameter.
4. **Expecting `/user/` and `/user` to both work without configuration** — without `strict_slashes=False`, they are distinct, and the `strict_slashes` default behavior (Section 5.5) may surprise you.
5. **Not accounting for `HEAD` and `OPTIONS`** — Flask automatically handles these for you: `HEAD` returns the same response as `GET` but without a body; `OPTIONS` returns the allowed methods. You almost never need to explicitly add these to `methods=`.

> **⚠️ Warning:** The `@app.route()` decorator runs and registers routes at **import time** — when Python loads your file. This means route registration order matters: if you try to register the same URL pattern twice, whichever runs second "wins" (or Flask raises an error). It also means routes are registered before the server starts accepting requests — not per-request.

---

## 11. Interview Questions (with Answers)

**Q1: What does the `@app.route()` decorator actually do internally?**
A: It calls `app.route()`, which creates a Werkzeug `Rule` object for the given URL pattern and registers it in `app.url_map`. It also stores the mapping between the endpoint name and the view function in `app.view_functions`. The view function itself is returned unchanged — the decorator's job is purely registration.

**Q2: What is the difference between an endpoint and a view function in Flask?**
A: An endpoint is a **string name** (by default the function's `__name__`). The URL Map maps URL patterns to endpoint names. A separate registry (`app.view_functions`) maps endpoint names to actual Python functions. This two-step indirection allows Blueprints to namespace endpoints and allows `url_for()` to work by name, not hardcoded URL strings.

**Q3: What HTTP response code does Flask return when a URL matches but the HTTP method doesn't?**
A: `405 Method Not Allowed`.

**Q4: Why does Flask automatically add `HEAD` and `OPTIONS` to every route's allowed methods?**
A: `HEAD` is required by the HTTP spec and is identical to `GET` but without a response body — Flask handles it automatically by running the `GET` handler and stripping the body. `OPTIONS` is used by browsers as part of CORS preflight checks — Flask generates the correct `Allow` header response automatically.

**Q5: What happens to `@app.route()` decorators at runtime?**
A: They run at **import time** (when Python loads the module), not at request time. By the time the server starts accepting traffic, all routes are already fully registered in the URL Map.

---

## 12. Best Practices

- Use **clear, descriptive function names** for view functions — they become endpoint names and appear in logs and error messages.
- Keep your view functions **thin** — they should delegate business logic to separate service/helper functions, not contain it inline. (A view function should mostly be: validate input → call a service → return a response.)
- Prefer **explicit `methods=`** even when only `GET` — it makes the contract clear to future readers.
- Use `url_for("endpoint_name")` (Module 7) to generate URLs in your code and templates instead of hardcoding strings — this makes URL refactoring painless.
- When a route handles both `GET` and `POST`, consider whether a `redirect` after successful `POST` (the **Post/Redirect/Get** pattern) is appropriate — it prevents duplicate form submissions on browser refresh.

---

## 13. Comparison Table — GET vs POST

| Aspect | GET | POST |
|---|---|---|
| Purpose | Retrieve data | Submit/send data |
| Data in URL? | Yes (query string) | No (request body) |
| Bookmarkable? | Yes | No |
| Browser "back" button re-triggers? | Yes | Yes (with a warning) |
| Should be idempotent? | Yes | Not necessarily |
| Use for forms that change data? | No | Yes |
| Visible in browser history/logs? | Yes | No (body is not logged by default) |

---

## 14. Real-World Usage

In production Flask apps, routing is typically split across multiple **Blueprints** (Module 18), each responsible for a feature area. A real project might look like:

```python
# auth/routes.py
@auth_bp.route("/login", methods=["GET", "POST"])
def login(): ...

@auth_bp.route("/logout")
def logout(): ...

# blog/routes.py
@blog_bp.route("/posts")
def list_posts(): ...

@blog_bp.route("/posts/<int:id>")
def show_post(id): ...
```

But the underlying mechanism — `Rule` objects in a `Map`, matched by a `MapAdapter` at request time — is identical to what we've covered here.

---

## 15. Mini Project

> Build a small Flask app with five routes:
> - `GET /` → Home page
> - `GET /about` → About page
> - `GET /contact` → Shows a plain text form (no real HTML needed yet)
> - `POST /contact` → Returns "Message received"
> - `GET /help` (also accessible at `GET /faq`) → Returns "Help / FAQ page" (hint: use two `@app.route()` decorators on the same function)
>
> After building it, open `python` shell, import your app, and print `app.url_map` — observe all the registered rules directly in Werkzeug's internal representation.

---

## 16. Practice Exercises

**5 Easy Questions**
1. What is the default HTTP method a Flask route accepts if `methods=` is not specified?
2. What HTTP status code is returned if a URL is found but the method is wrong?
3. True/False: `@app.route()` calls the view function when it is applied.
4. What is an "endpoint" in Flask's routing system?
5. What attribute of the `app` object stores all registered routes?

**5 Medium Questions**
1. Explain the difference between `@app.route("/blog")` and `@app.route("/blog/")` in terms of trailing slash behavior.
2. How would you make a route accessible at both `/` and `/home`?
3. What's the difference between the URL Map and `app.view_functions`?
4. Why does Flask add `HEAD` to routes automatically, and what does a `HEAD` response look like?
5. What does Flask do internally when `@app.route()` is applied at import time?

**5 Hard Questions**
1. Explain, at the level of Python internals, exactly how `@app.route("/")` as a decorator achieves "registration as a side effect" without modifying the wrapped function.
2. Why is the two-step indirection (URL → endpoint → function) more powerful than a direct (URL → function) mapping?
3. A Flask app has 200 routes. How does Werkzeug's URL Map avoid checking all 200 rules linearly for every request?
4. What would happen if two view functions in the same app happened to have the same Python function name (e.g., both named `index`)? Explain Flask's behavior.
5. Why would a REST API use `methods=["GET", "POST", "PUT", "DELETE"]` on one route vs. separate routes per method, and what are the trade-offs?

**2 Debugging Questions**
1. A developer registers `@app.route("/profile")` but gets a `404` when visiting `/profile/`. They're confused because "the route exists." Explain the cause and fix.
2. A developer runs their app and gets: `AssertionError: View function mapping is overwriting an existing endpoint function: home`. What caused this, and what are two ways to fix it?

**2 Interview Questions**
1. "Explain Flask's routing internals — from `@app.route()` being applied to a view function being called at request time."
2. "What is the difference between an endpoint and a view function? Why does Flask separate these concepts?"

**1 Mini Project**
See Section 15 above.

---

## 16. Summary

| Concept | Key Takeaway |
|---|---|
| `@app.route()` | A registration decorator — adds a `Rule` to `app.url_map` and the function to `app.view_functions` |
| URL Map | Werkzeug's `Map` object inside `app` — holds all registered `Rule` objects |
| Endpoint | A string name (defaults to function name) — the bridge between URL Map and view function |
| `app.view_functions` | Dictionary mapping endpoint names to actual Python callables |
| Matching | `MapAdapter.match()` at request time → returns `(endpoint, url_variables)` |
| `methods=` | Explicit list of allowed HTTP methods; defaults to `["GET"]` |
| Trailing slash | Routes with trailing slash redirect missing slash; routes without do not |
| 405 | Automatic response when path matches but HTTP method doesn't |

### 💡 Memory Trick
**"Routing is a dictionary with two levels: URL → name → function. The name is the middle layer that makes everything refactorable."**

### ❓ FAQ
- **"Can a single view function be registered under multiple URLs?"** Yes — stack multiple `@app.route()` decorators on the same function. Each one creates a separate `Rule` entry in the URL Map.
- **"Can I register routes dynamically at runtime, after the app starts?"** Technically yes, but it's discouraged — the URL Map is typically compiled/finalized at startup for performance, and runtime additions can cause subtle caching issues. The standard approach is to register all routes at import time via decorators or Blueprints.

---

**End of Module 6.**
