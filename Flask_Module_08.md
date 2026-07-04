# MODULE 8 — URL Variables

> **Course:** Flask — Zero to Production
> **Module:** 8 of 39
> **Builds on:** Modules 1–7

---

## 1. Introduction

In Module 6, we learned how to route a URL like `/about` to a view function.

But what about URLs like these?

- `/user/42` → Show profile of user number 42
- `/user/99` → Show profile of user number 99
- `/post/hello-world` → Show a blog post called "hello-world"
- `/product/laptop-dell-xps` → Show a product page

You can't create a separate route for every user ID or every blog post title — there could be millions of them. What you want is one route that works for **all** of them.

**URL Variables** (also called **URL parameters** or **dynamic segments**) are the solution. They let you put a placeholder inside your URL pattern that **captures whatever the user actually typed** and passes it to your view function.

---

## 2. Why Do We Need This?

Imagine you're building Instagram. Instagram shows each user's profile at a URL like:

```
/profile/arjun_sharma
/profile/priya_patel
/profile/rahul_verma
```

Each of those is a different URL, but they all do the same thing: look up a username and show their profile.

Would you write this?

```python
@app.route("/profile/arjun_sharma")
def arjun_profile():
    return "Arjun's profile"

@app.route("/profile/priya_patel")
def priya_profile():
    return "Priya's profile"

# ... and 1 billion more routes for every user on Instagram?
```

Obviously not. That's impossible.

Instead, you write **one route** with a variable placeholder:

```python
@app.route("/profile/<username>")
def show_profile(username):
    return f"Profile of {username}"
```

Now `<username>` captures whatever comes after `/profile/` in the URL, and passes it to your function. One route handles all users. This is the power of URL variables.

---

## 3. The Problem Without URL Variables

Without URL variables, you'd be forced to use query strings for everything:

```
/profile?username=arjun_sharma
/profile?username=priya_patel
```

Query strings (the `?key=value` part) work, but they're not ideal for things that are core to the resource being identified. Clean, readable URLs like `/profile/arjun_sharma` are:
- More **human-readable**
- Better for **SEO** (search engines understand them better)
- More in line with **REST API design principles** (Module 26)
- What users expect from modern web apps

URL variables give you this clean URL design.

---

## 4. Real-Life Analogy

Think of a URL variable like a **blank form field** on a sign.

Imagine a hotel corridor with a sign that says:

```
Room ______   →   Turn Left
```

The blank is filled in dynamically. Room 101 → Turn Left. Room 204 → Turn Left. Room 999 → Turn Left. One sign handles all rooms.

In Flask:

```python
@app.route("/room/<int:number>")
def go_to_room(number):
    return f"Room {number} → Turn Left"
```

The `<int:number>` is that blank. Flask fills it in with whatever number appears in the URL.

---

## 5. Internal Working

When Flask sees a route like:

```python
@app.route("/user/<int:user_id>")
```

Werkzeug converts this pattern into a **regular expression** internally. Roughly:

```
/user/<int:user_id>   →   /user/([0-9]+)
```

When a request for `/user/42` arrives:

1. Werkzeug matches `/user/42` against this regex.
2. It **captures** `42` from the URL (as a string — all URL bytes are strings).
3. The `int:` **converter** takes that captured string `"42"` and converts it to Python integer `42`.
4. Flask calls your view function with `user_id=42` as a keyword argument.

If the URL is `/user/abc` and you used `<int:user_id>`:
- The regex `([0-9]+)` doesn't match `abc`.
- Werkzeug returns a **404 Not Found** — not a 400 Bad Request. This means "no route exists that matches this URL." (Because from Werkzeug's perspective, this pattern simply doesn't match.)

This is an important security and design benefit: **type converters act as automatic validators** for your URL segments.

### 5.1 Diagram — How URL Matching Works With Variables

```
Registered route: "/user/<int:user_id>"
                       ↓
Werkzeug compiles to regex: "^/user/([0-9]+)$"
                       ↓
Request arrives: GET /user/42
                       ↓
Regex matches! Captured group: "42"
                       ↓
int converter: "42" → 42 (Python int)
                       ↓
Flask calls: show_user(user_id=42)
```

---

## 6. Syntax — The Basics

### 6.1 Basic Variable (No Type — Captures Any String)

```python
@app.route("/hello/<name>")
def hello(name):
    return f"Hello, {name}!"
```

- `<name>` — the angle brackets mark a **variable segment**.
- `name` — this exact name is passed as a **keyword argument** to the view function.
- Without a type prefix, it defaults to the `string` converter — captures any text that doesn't contain a `/`.

### 6.2 With a Type Converter

```python
@app.route("/user/<int:user_id>")
def show_user(user_id):
    return f"User ID is: {user_id}"
```

- `int:` — the **converter**. It converts the URL segment to an integer.
- `user_id` — the variable name passed to the function.

### 6.3 Multiple URL Variables

```python
@app.route("/user/<int:user_id>/post/<int:post_id>")
def show_post(user_id, post_id):
    return f"User {user_id}, Post {post_id}"
```

You can have as many variables in a URL as you need. Each one becomes a keyword argument.

---

## 7. Converters — All Built-In Types

Flask comes with **6 built-in URL converters**. Think of them as filters — they define what kind of text a variable can match.

| Converter | Syntax | What it accepts | Python type returned |
|---|---|---|---|
| `string` | `<name>` or `<string:name>` | Any text **without** a `/` | `str` |
| `int` | `<int:id>` | Positive whole numbers only (`0`, `1`, `42`, etc.) | `int` |
| `float` | `<float:price>` | Decimal numbers (`3.14`, `0.5`) | `float` |
| `path` | `<path:subpath>` | Any text **including** `/` characters | `str` |
| `uuid` | `<uuid:uid>` | A UUID format string (e.g., `550e8400-e29b-41d4-a716-446655440000`) | `uuid.UUID` |
| `any` | `<any(cat,dog):animal>` | One of a defined set of values | `str` |

Let's look at each with a real example.

---

### Converter 1: `string` (Default)

```python
@app.route("/greet/<name>")
def greet(name):
    return f"Hello, {name}!"
```

- `/greet/Arjun` → `"Hello, Arjun!"`
- `/greet/Priya` → `"Hello, Priya!"`
- `/greet/Hello World` → **Won't match cleanly** (spaces in URLs are problematic — browsers encode them as `%20`)
- `/greet/some/path` → **Won't match** — `string` converter **stops at `/`**

> **Important:** The `string` converter does NOT match forward slashes. This is what makes `/user/<name>` different from `/user/<path:name>`.

---

### Converter 2: `int`

```python
@app.route("/post/<int:post_id>")
def show_post(post_id):
    return f"Post number {post_id}"
```

- `/post/1` → `show_post(post_id=1)` ✅
- `/post/100` → `show_post(post_id=100)` ✅
- `/post/abc` → **404** (doesn't match — `abc` is not an integer) ✅ (good! prevents bad requests)
- `/post/-5` → **404** (negative numbers not accepted by `int` converter)
- `/post/3.14` → **404** (decimals not accepted by `int` converter)

This is very useful. If your database stores post IDs as integers, this converter ensures your view function **only gets called with valid integer IDs** — you don't need to manually validate inside the function.

---

### Converter 3: `float`

```python
@app.route("/temperature/<float:temp>")
def show_temperature(temp):
    return f"Temperature is {temp}°C"
```

- `/temperature/36.6` → `show_temperature(temp=36.6)` ✅
- `/temperature/100` → **404** (no decimal point — `int` format doesn't match `float` converter)
- `/temperature/36.6.1` → **404** (not a valid float)

> **Note:** `float` requires a decimal point. `/temperature/100` would 404 because `100` matches `int` format, not `float`. Use `float` only when you truly expect decimal numbers.

---

### Converter 4: `path`

```python
@app.route("/files/<path:filename>")
def show_file(filename):
    return f"File: {filename}"
```

- `/files/image.png` → `show_file(filename="image.png")` ✅
- `/files/photos/2024/summer.jpg` → `show_file(filename="photos/2024/summer.jpg")` ✅
- `/files/a/b/c/d/file.txt` → `show_file(filename="a/b/c/d/file.txt")` ✅

`path` is just like `string` but it **also matches forward slashes**. This is useful for file paths, nested category URLs, etc.

> **Warning:** Be careful with `path` — because it matches slashes, it will consume everything after the prefix. This means `<path:filename>` should usually be the **last segment** of your URL pattern.

---

### Converter 5: `uuid`

```python
import uuid

@app.route("/item/<uuid:item_id>")
def show_item(item_id):
    return f"Item UUID: {item_id}"
```

- `/item/550e8400-e29b-41d4-a716-446655440000` → matches ✅
- `/item/not-a-uuid` → **404** ✅

UUIDs are used when you don't want to expose sequential integer IDs (e.g., `/user/1`, `/user/2` reveals how many users you have). Random UUIDs like `550e8400-...` are unpredictable and harder to guess.

---

### Converter 6: `any`

```python
@app.route("/category/<any(news, sports, tech):section>")
def show_section(section):
    return f"You're in the {section} section"
```

- `/category/news` → `show_section(section="news")` ✅
- `/category/sports` → `show_section(section="sports")` ✅
- `/category/food` → **404** (not in the allowed list)

`any` is like a whitelist — only the values you specify are valid.

---

## 8. Small Example — All Converters in One App

```python
from flask import Flask

app = Flask(__name__)


# Default string converter
@app.route("/hello/<name>")
def hello(name):
    return f"Hello, {name}!"


# Integer converter
@app.route("/user/<int:user_id>")
def user_profile(user_id):
    return f"User ID: {user_id} (type: {type(user_id).__name__})"


# Float converter
@app.route("/price/<float:amount>")
def show_price(amount):
    return f"Price: ₹{amount:.2f}"


# Path converter
@app.route("/files/<path:filepath>")
def show_file(filepath):
    return f"File path: {filepath}"


# Any converter
@app.route("/status/<any(active, inactive, banned):state>")
def user_status(state):
    return f"User status: {state}"


if __name__ == "__main__":
    app.run(debug=True)
```

---

## 9. Step-by-Step Explanation

**`@app.route("/hello/<name>")`**

The `<name>` is a URL variable placeholder. When Python loads this file, Flask registers a route in its URL Map. Instead of a fixed pattern like `/hello`, it registers a **dynamic pattern** that Werkzeug internally converts to a regex like `^/hello/([^/]+)$`.

The capture group `([^/]+)` means: "match any characters that are not a forward slash."

**`def hello(name):`**

The parameter `name` in the function signature **must match the variable name** used inside `< >` in the route. Flask passes the captured URL segment as a keyword argument with that exact name.

If you write `<name>` in the route but `def hello(username):` in the function — Flask will raise a `TypeError` because the keyword argument names don't match.

**`return f"Hello, {name}!"`**

The captured value `name` is just a regular Python string you can use anywhere.

---

**`@app.route("/user/<int:user_id>")`**

`int:` is the converter prefix. Before passing the value to your function, Werkzeug:
1. Checks the URL segment matches `[0-9]+`
2. Calls Python's `int()` on it to convert `"42"` → `42`

**`def user_profile(user_id):`**

`user_id` arrives as a Python `int`, not a string. You can immediately do math on it, pass it to a database query, etc. — no manual `int()` conversion needed.

**`return f"User ID: {user_id} (type: {type(user_id).__name__})"`**

The `type(user_id).__name__` will be `"int"` — proving Flask converted it for us.

---

**`@app.route("/files/<path:filepath>")`**

`path:` means the captured segment can include `/` characters.

If someone visits `/files/documents/reports/q4.pdf`, then `filepath` will be the string `"documents/reports/q4.pdf"` — the entire path after `/files/`, slashes included.

---

### Program Flow — Visiting `/user/42`

```
Browser: GET /user/42
   ↓
Werkzeug: match "/user/42" against all registered rules
   ↓
Rule "/user/<int:user_id>" → regex: "^/user/([0-9]+)$"
   ↓
"42" matches ([0-9]+) ✓
   ↓
int converter: "42" → 42
   ↓
endpoint = "user_profile", values = {"user_id": 42}
   ↓
Flask calls: user_profile(user_id=42)
   ↓
Returns: "User ID: 42 (type: int)"
   ↓
Flask sends 200 OK response to browser
```

---

### Program Flow — Visiting `/user/abc` (Invalid)

```
Browser: GET /user/abc
   ↓
Werkzeug: match "/user/abc" against all registered rules
   ↓
Rule "/user/<int:user_id>" → regex: "^/user/([0-9]+)$"
   ↓
"abc" does NOT match ([0-9]+) ✗
   ↓
No matching rule found
   ↓
Flask returns: 404 Not Found
```

---

## 10. Visual Diagram — Full Picture

```
URL: /user/<int:user_id>/post/<int:post_id>
           │                    │
           │                    │
    int converter          int converter
    "42" → 42              "7" → 7
           │                    │
           └────────┬───────────┘
                    ↓
    view_function(user_id=42, post_id=7)
                    ↓
              Your Python code runs
                    ↓
              Returns a response
```

---

## 11. Variable Names Must Match

This is the #1 mistake beginners make with URL variables. The name inside `< >` in the route **must be identical** to the parameter name in the function.

```python
# ✅ CORRECT — names match
@app.route("/user/<int:user_id>")
def show_user(user_id):     # "user_id" matches "<int:user_id>"
    return f"User {user_id}"


# ❌ WRONG — names don't match
@app.route("/user/<int:user_id>")
def show_user(id):          # "id" doesn't match "user_id" → TypeError!
    return f"User {id}"
```

---

## 12. `url_for()` — Generating URLs from Variable Routes

`url_for()` is a Flask function that **generates a URL** for you by looking up a route by its **endpoint name**. We introduced it briefly in Module 6. Now that we have URL variables, it becomes even more powerful.

```python
from flask import Flask, url_for

app = Flask(__name__)

@app.route("/user/<int:user_id>")
def show_user(user_id):
    return f"User {user_id}"

with app.test_request_context():
    # Generate the URL for show_user with user_id=42
    url = url_for("show_user", user_id=42)
    print(url)   # /user/42
```

**Why use `url_for()` instead of just writing `"/user/42"`?**

| Hardcoded string | `url_for()` |
|---|---|
| `"/user/" + str(user_id)` | `url_for("show_user", user_id=user_id)` |
| If you rename the route later, you must update every string | `url_for()` automatically uses the new URL |
| Easy to make typos | Flask raises an error if the endpoint doesn't exist |
| Can't handle query parameters automatically | Extra keyword args become query parameters |

**Extra keyword arguments become query parameters:**

```python
url_for("show_user", user_id=42, tab="posts")
# produces: /user/42?tab=posts
```

This is why professional Flask code uses `url_for()` everywhere — in templates, in redirects, everywhere.

---

## 13. Common Mistakes

### Mistake 1: Mismatched variable names

```python
# ❌ WRONG
@app.route("/post/<int:post_id>")
def show_post(id):           # should be "post_id", not "id"
    return f"Post {id}"
```

**Error:** `TypeError: show_post() got an unexpected keyword argument 'post_id'`

---

### Mistake 2: Forgetting the type prefix when you need it

```python
# ❌ Without converter — user_id is a STRING "42", not int 42
@app.route("/user/<user_id>")
def show_user(user_id):
    user = db.get(user_id)   # might fail if db expects an int

# ✅ With converter — user_id is int 42
@app.route("/user/<int:user_id>")
def show_user(user_id):
    user = db.get(user_id)   # works correctly
```

---

### Mistake 3: Using `path` converter when `string` is enough

`path` is a powerful converter but using it carelessly can "eat" URL segments you didn't intend to capture.

```python
# ❌ Risky — <path:name> eats everything including slashes
@app.route("/hello/<path:name>")
def hello(name):
    return f"Hello {name}"

# /hello/john/doe → name = "john/doe"  (probably not what you wanted)

# ✅ Correct — <string:name> stops at the first slash
@app.route("/hello/<string:name>")
def hello(name):
    return f"Hello {name}"

# /hello/john → name = "john" ✅
# /hello/john/doe → 404 ✅ (correctly rejects)
```

---

### Mistake 4: Negative numbers with `int` converter

```python
@app.route("/offset/<int:n>")
def paginate(n):
    return f"Offset: {n}"
```

`/offset/-5` → **404**, not `-5`. The `int` converter only accepts non-negative integers. If you need negative numbers, use `string` converter and manually convert with a try/except.

---

### Mistake 5: Trying to make URL variables optional

Flask does **not** support optional URL variables natively. You cannot write:

```python
# ❌ This is NOT valid Flask syntax
@app.route("/user/<int:user_id>?")
```

**Workaround** — register two routes pointing to the same function:

```python
@app.route("/user/")
@app.route("/user/<int:user_id>")
def show_user(user_id=None):
    if user_id is None:
        return "Show all users"
    return f"Show user {user_id}"
```

---

## 14. Interview Questions (with Answers)

**Q1: What is a URL variable in Flask?**
A: A dynamic segment in a URL pattern, written as `<variable_name>` or `<converter:variable_name>`, that captures the corresponding part of the actual URL and passes it as a keyword argument to the view function.

**Q2: What are URL converters, and why are they useful?**
A: Converters are type specifications applied to URL variables (e.g., `int:`, `float:`, `path:`). They serve two purposes: they constrain what URLs match the route (a route with `<int:id>` won't match `/user/abc`), and they automatically convert the captured string to the appropriate Python type.

**Q3: What happens when you visit `/user/abc` on a route registered as `/user/<int:user_id>`?**
A: Flask returns a `404 Not Found`, because `abc` doesn't satisfy the `int` converter's requirement (only digits allowed). The route simply doesn't match.

**Q4: What is `url_for()` and why use it over hardcoded URLs?**
A: `url_for("endpoint_name", variable=value)` generates a URL for a named endpoint. It's preferred because it automatically updates if you rename/move routes, raises errors for typos instead of silently creating wrong links, and correctly handles query parameters and URL encoding.

**Q5: What's the difference between `string` and `path` converters?**
A: Both capture text, but `string` stops at the first `/` while `path` captures slashes too. Use `path` for file-system-like paths; use `string` for simple single-segment captures.

---

## 15. Best Practices

- Always use **type converters** (`int:`, `float:`) when you know the expected type — they act as free input validation.
- **Match the variable name** in the route and the function parameter exactly — it's a common mistake that causes a `TypeError`.
- Use **`url_for()`** everywhere instead of hardcoded URL strings — your future self will thank you.
- Put `<path:...>` variables at the **end** of URL patterns — they consume everything after, including slashes, which can cause unexpected behavior if placed in the middle.
- For **optional URL segments**, use two stacked `@app.route()` decorators on the same function with a default parameter value.

---

## 16. Real-World Usage

In a real blog application:

```python
@app.route("/blog/<int:year>/<int:month>/<slug>")
def blog_post(year, month, slug):
    # year = 2024 (int, auto-converted)
    # month = 7 (int, auto-converted)
    # slug = "my-first-post" (string)
    post = Post.query.filter_by(year=year, month=month, slug=slug).first()
    if post is None:
        abort(404)
    return render_template("post.html", post=post)
```

This route handles URLs like `/blog/2024/7/my-first-post` cleanly. Flask auto-converts `year` and `month` to integers (useful for database queries), and `slug` stays as a string.

---

## 17. Mini Project

> Build a simple **User Profile viewer** with these routes:
>
> - `GET /` → Returns "Welcome! Visit /user/<any number> to see a profile"
> - `GET /user/<int:user_id>` → Returns `"Profile page for User ID: X"` where X is the number from the URL
> - `GET /category/<any(tech, science, sports):name>` → Returns `"Category: X"` — but visiting `/category/food` should give a 404
> - `GET /files/<path:filepath>` → Returns `"You requested file: X/Y/Z"` (where X/Y/Z is whatever path they gave)
>
> After building it, test each route in your browser. Try visiting an invalid URL (like `/user/abc` or `/category/cooking`) and confirm you get a 404.

---

## 18. Practice Exercises

**5 Easy Questions**
1. How do you mark a URL segment as a variable in Flask?
2. What is the default converter type if you write `<name>` without a prefix?
3. Which converter should you use to capture a file path that includes slashes?
4. True/False: `<int:id>` will match `/user/abc` and pass `"abc"` to your function.
5. What does `url_for("show_user", user_id=5)` return if the route is `/user/<int:user_id>`?

**5 Medium Questions**
1. What Python type does the `int` converter return — `str`, `int`, or `float`?
2. What's the difference between the `string` and `path` converters?
3. How would you create a route that accepts only `"cat"`, `"dog"`, or `"bird"` as a URL segment?
4. Why does `/user/-5` return 404 on an `<int:user_id>` route?
5. How do you add a query parameter to a URL generated by `url_for()`?

**5 Hard Questions**
1. Explain, internally, how Werkzeug converts `<int:user_id>` into a regex pattern and applies the converter.
2. Why does Flask return `404` (not `400`) when an `int` converter fails to match?
3. How would you implement an optional URL variable in Flask, and what's the cleanest way to do it?
4. If two routes are registered — `/user/<string:name>` and `/user/<int:user_id>` — and someone visits `/user/42`, which route wins? Why?
5. What could go wrong if you use `<path:filepath>` in the middle of a URL pattern instead of at the end?

**2 Debugging Questions**
1. A developer writes `@app.route("/post/<int:post_id>")` but their function signature is `def show(id):`. Running the app works, but visiting `/post/5` gives a `TypeError`. What's wrong?
2. A developer expects `/files/a/b/c.txt` to work with `@app.route("/files/<string:name>")`. It returns 404. They're confused because "the file path is in the URL." Explain the problem and solution.

**2 Interview Questions**
1. "What are URL converters in Flask? Name all built-in ones and explain when you'd use each."
2. "Why is `url_for()` better than hardcoded URL strings? Give a practical example."

**1 Mini Project**
See Section 17 above.

---

## 19. Summary

| Syntax | What it does |
|---|---|
| `<name>` | Captures any text without slashes (string converter) |
| `<string:name>` | Same as above — explicit string |
| `<int:name>` | Only matches digits, converts to Python int |
| `<float:name>` | Only matches decimal numbers, converts to Python float |
| `<path:name>` | Captures text including slashes |
| `<uuid:name>` | Only matches UUID format, converts to `uuid.UUID` |
| `<any(a,b):name>` | Only matches one of the listed values |

### Converter Quick Reference

| Converter | Matches | Returns | Rejects |
|---|---|---|---|
| `string` | Any text (no `/`) | `str` | Slashes |
| `int` | `0, 1, 2, ...` | `int` | Letters, negatives, decimals |
| `float` | `1.5, 3.14, ...` | `float` | Text, integers without `.` |
| `path` | Any text (with `/`) | `str` | Nothing (greedy) |
| `uuid` | UUID format | `uuid.UUID` | Non-UUID text |
| `any` | Listed values only | `str` | Anything not in the list |

### 💡 Memory Trick
**"Think of `< >` as a blank form field in your URL. The converter type is the validation rule written on the form: 'only fill this in with a number', 'only fill this in with one of these options', etc."**

### ❓ FAQ
- **"Can I have two variable segments next to each other?"** Flask/Werkzeug requires a separator (like `/` or `-`) between variable segments. `/user/<first><last>` is not valid. `/user/<first>-<last>` works.
- **"What if I need to accept negative integers?"** Use `<string:n>` and manually do `int(n)` inside your function, wrapped in a `try/except` to handle invalid input yourself.
- **"Does `url_for()` work inside templates too?"** Yes! It's even more commonly used in Jinja2 templates than in Python code. We'll see this in Module 11.

---

**End of Module 8.**
