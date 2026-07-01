# MODULE 7 — View Functions

> **Course:** Flask — Zero to Production
> **Module:** 7 of 39
> **Builds on:** Modules 1–6

---

## 1. Introduction

In Module 6, we learned that Flask looks at a URL and finds the right Python function to call.

That Python function — the one Flask calls — is called a **view function**.

That's it. A view function is just a regular Python function that:
1. Flask calls when someone visits a URL
2. Does some work (maybe looks up data, maybe just returns text)
3. **Returns something** to send back to the browser

This whole module is about step 3 — **what can you return**, and **how does Flask turn it into a real web response?**

---

## 2. Why Do We Need This?

Let's think about what a browser actually receives.

When you visit a website, your browser doesn't receive a Python string. It receives an **HTTP response** — which looks like this:

```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 27

<h1>Hello from Flask!</h1>
```

This has three parts:
1. **Status line** — `200 OK` means "everything went fine." (`404` means "not found." `500` means "server crashed.")
2. **Headers** — extra information like "what type of content is this?" or "how long is it?"
3. **Body** — the actual content the browser shows (HTML, JSON, file, etc.)

Your view function only returns the **body** (or maybe the body + a status code). Flask's job is to take whatever you return and wrap it into this full HTTP response format automatically.

**If Flask didn't do this**, you'd have to manually build every HTTP response from scratch — writing the status line, the headers, and the body by hand for every single view function. That would be exhausting.

---

## 3. The Problem Without This

Imagine if every time you wrote a view function, you had to do this:

```python
def home():
    body = b"<h1>Hello!</h1>"
    status = "200 OK"
    headers = [("Content-Type", "text/html"), ("Content-Length", "15")]
    return body, status, headers
```

Every. Single. Function. Every route. Every page.

Flask fixes this by being smart about **what you return**. You return a simple Python value (like a string), and Flask figures out the rest automatically.

---

## 4. Real-Life Analogy

Think of a **restaurant order system**.

- You are the **chef** (view function). Your job is to make the food.
- The **waiter** is Flask. The waiter takes your food and packages it properly for delivery.
- The **customer** is the browser. They receive a full, properly packaged meal.

You (the chef) just shout "here's the pasta!" and hand it to the waiter (Flask). You don't need to find a box, add a fork, seal it, label it, and calculate the delivery price. The waiter does all that.

Similarly: you just `return "Hello!"` and Flask packages it into a complete HTTP response.

---

## 5. Internal Working — What Happens to Your Return Value

After your view function returns something, Flask does the following (simplified):

```
Your view function returns a value
         ↓
Flask checks: what type is this return value?
         ↓
  ┌──────────────────────────────────────────────┐
  │  String → wrap in Response(status=200,       │
  │            Content-Type="text/html")          │
  │                                              │
  │  Dict   → convert to JSON automatically      │
  │            (since Flask 1.0)                 │
  │                                              │
  │  Tuple  → (body, status_code) or            │
  │            (body, headers) or               │
  │            (body, status_code, headers)      │
  │                                              │
  │  Response object → use as-is                 │
  └──────────────────────────────────────────────┘
         ↓
Flask sends it back to the browser as a complete HTTP response
```

This automatic conversion is done inside a Flask method called `make_response()`. Internally, Flask calls this on whatever you return, turning it into a proper `Response` object before sending it.

---

## 6. Syntax — 5 Ways to Return From a View Function

### Way 1 — Just Return a String (Simplest)

```python
@app.route("/")
def home():
    return "Hello, World!"
```

Flask automatically sets:
- Status code: `200 OK`
- Content-Type: `text/html`

---

### Way 2 — Return a String + Status Code

```python
@app.route("/not-found")
def not_found():
    return "This page doesn't exist", 404
```

You return a **tuple**: `(body, status_code)`.

Flask sees the tuple and uses the second item as the HTTP status code.

---

### Way 3 — Return a String + Status Code + Custom Headers

```python
@app.route("/special")
def special():
    return "Special Response", 200, {"X-Custom-Header": "MyValue"}
```

You return a **tuple**: `(body, status_code, headers_dict)`.

---

### Way 4 — Return a Dictionary (Automatic JSON)

```python
@app.route("/api/user")
def api_user():
    return {"name": "Arjun", "age": 25}
```

Since Flask 1.0, if you return a Python **dictionary**, Flask automatically converts it to **JSON** (using `jsonify` internally) and sets the `Content-Type` to `application/json`.

This is extremely useful for building APIs.

---

### Way 5 — Return a Response Object (Full Control)

```python
from flask import Flask, make_response

app = Flask(__name__)

@app.route("/custom")
def custom():
    response = make_response("Hello with custom headers!")
    response.status_code = 200
    response.headers["X-Author"] = "Arjun"
    response.headers["Content-Type"] = "text/plain"
    return response
```

`make_response()` gives you a `Response` object you can fully customize before returning.

---

## 7. Small Example — All 5 Ways Together

```python
from flask import Flask, make_response, jsonify

app = Flask(__name__)


# Way 1 — Simple string
@app.route("/")
def home():
    return "Welcome Home!"


# Way 2 — String + status code
@app.route("/error")
def error():
    return "Something went wrong", 500


# Way 3 — String + status code + headers
@app.route("/headers")
def with_headers():
    return "Look at my headers!", 200, {"X-My-Header": "Hello"}


# Way 4 — Dictionary (automatic JSON)
@app.route("/api/data")
def api_data():
    return {"message": "success", "count": 42}


# Way 5 — Full Response object
@app.route("/full")
def full_response():
    response = make_response("<h1>Full Control!</h1>")
    response.status_code = 201
    response.headers["X-Creator"] = "Flask"
    return response


if __name__ == "__main__":
    app.run(debug=True)
```

---

## 8. Step-by-Step Explanation

Let's go through every line of the example above.

---

**`from flask import Flask, make_response, jsonify`**

We are importing three things from the `flask` package:
- `Flask` — the class to create our app (we know this already)
- `make_response` — a function that creates a `Response` object we can manually customize
- `jsonify` — a function that converts a Python dictionary (or list) into a proper JSON HTTP response with the correct `Content-Type: application/json` header

Why import `make_response`? Because when you just `return "a string"`, Flask calls `make_response("a string")` internally anyway — but importing it lets you call it yourself when you need extra control.

---

**`app = Flask(__name__)`**
Creates the Flask application object — the central filing cabinet (Module 4).

---

**`@app.route("/")`**
Registers the `/` URL pattern in Flask's URL Map (Module 6).

---

**`def home():`**
The view function for the root URL.

---

**`return "Welcome Home!"`**
Returns a plain Python string. Internally, Flask does:
```python
response = make_response("Welcome Home!")
# Response object created:
#   body    = b"Welcome Home!"
#   status  = 200
#   Content-Type = "text/html; charset=utf-8"
```

The browser receives a complete HTTP response with this body.

---

**`return "Something went wrong", 500`**
Returns a **tuple** — two items: the body string and a status code integer.

Flask sees this is a tuple with 2 items, and:
- Uses item 0 (`"Something went wrong"`) as the body.
- Uses item 1 (`500`) as the HTTP status code.

The number `500` means "Internal Server Error" — it tells the browser "something broke on the server side."

---

**`return "Look at my headers!", 200, {"X-My-Header": "Hello"}`**
Returns a **tuple** with 3 items: body, status code, and a dictionary of extra HTTP headers.

Flask merges these extra headers into the response on top of the defaults.

`X-My-Header` is a custom header. The `X-` prefix is a convention for non-standard, custom headers (though this convention is now deprecated in modern HTTP — it still works).

---

**`return {"message": "success", "count": 42}`**
Returns a Python **dictionary**. Flask detects this and automatically does the equivalent of:
```python
response = jsonify({"message": "success", "count": 42})
# body    = '{"message": "success", "count": 42}'
# status  = 200
# Content-Type = "application/json"
```

The browser (or an API client like Postman) receives properly formatted JSON, not a Python dictionary string.

---

**`response = make_response("<h1>Full Control!</h1>")`**
`make_response()` creates a `Response` object — think of it as a "blank packaged meal" that you then customize before handing it to the waiter (Flask). You pass the body as the first argument.

**`response.status_code = 201`**
Sets the HTTP status to `201 Created` — often used by APIs to say "your request succeeded and something new was created." We're setting it manually here.

**`response.headers["X-Creator"] = "Flask"`**
Adds a custom header to this specific response. The `headers` attribute is a dictionary-like object you can set key-value pairs on.

**`return response`**
Returns the fully built `Response` object. Since it's already a `Response`, Flask uses it as-is without any automatic conversion.

---

### Program Flow

```
Browser: GET /api/data
   ↓
Flask matches URL → api_data() view function called
   ↓
api_data() returns {"message": "success", "count": 42}
   ↓
Flask sees: return value is a dict
   ↓
Flask internally calls jsonify() on the dict
   ↓
Creates Response object:
  - body: '{"message": "success", "count": 42}'
  - status: 200
  - Content-Type: application/json
   ↓
Browser receives JSON response
```

---

## 9. Visual Diagram

```
Your View Function
   │
   │  returns one of these ↓
   │
   ├─── "a string"                → Response(body="a string",
   │                                  status=200, type=text/html)
   │
   ├─── ("string", 404)           → Response(body="string",
   │                                  status=404, type=text/html)
   │
   ├─── ("string", 200, headers)  → Response(body="string",
   │                                  status=200, extra headers added)
   │
   ├─── {"key": "value"}          → Response(body='{"key":"value"}',
   │                                  status=200, type=application/json)
   │
   └─── Response object           → Used as-is, no conversion needed
              │
              ▼
         Flask sends HTTP response to browser
```

---

## 10. Important Flask Functions for Responses

### `jsonify()`

```python
from flask import jsonify

@app.route("/api/users")
def get_users():
    users = [
        {"id": 1, "name": "Arjun"},
        {"id": 2, "name": "Priya"}
    ]
    return jsonify(users)
```

`jsonify()` does two things:
1. Converts your Python dictionary or list into a **JSON string**.
2. Sets the `Content-Type` header to `application/json` so the browser/client knows it's receiving JSON.

> **Why not just use `json.dumps()` (Python's built-in)?**
> `json.dumps()` only converts the data — it returns a plain string. `jsonify()` wraps it in a proper Flask `Response` object with the correct `Content-Type` header already set.

---

### `redirect()`

```python
from flask import redirect, url_for

@app.route("/old-page")
def old_page():
    return redirect("/new-page")

# Better way (using endpoint name):
@app.route("/old-page")
def old_page():
    return redirect(url_for("new_page"))

@app.route("/new-page")
def new_page():
    return "This is the new page!"
```

`redirect()` sends a response that tells the browser: **"Go to this other URL instead."**

Internally, this is a response with:
- Status code: `302 Found` (temporary redirect) by default
- A `Location` header set to the URL you want to redirect to

The browser reads the `Location` header and automatically sends a new request to that URL.

**redirect() + status codes:**

```python
redirect("/new-page", code=301)  # 301 = Permanent redirect (browser caches it)
redirect("/new-page", code=302)  # 302 = Temporary redirect (default)
```

---

### `abort()`

```python
from flask import abort

@app.route("/secret")
def secret():
    user_logged_in = False
    if not user_logged_in:
        abort(403)   # Immediately stop and return 403 Forbidden
    return "Secret content!"
```

`abort()` **immediately stops** your view function and sends back an HTTP error response.

It's like a fire alarm — when called, everything stops and Flask sends the error response immediately. Any code after `abort()` is never reached.

Common status codes you'll use with `abort()`:

| Code | Meaning | When to use |
|---|---|---|
| `400` | Bad Request | User sent invalid data |
| `401` | Unauthorized | User needs to log in |
| `403` | Forbidden | User is logged in but not allowed |
| `404` | Not Found | Resource doesn't exist |
| `500` | Internal Server Error | Something crashed |

---

## 11. HTTP Status Codes — The Complete Beginner's Guide

Status codes are **3-digit numbers** that tell the browser whether the request worked, and if not, why.

They are grouped into 5 families:

| Range | Family | Simple meaning |
|---|---|---|
| `1xx` | Informational | "I'm processing, hang on..." |
| `2xx` | Success ✅ | "It worked!" |
| `3xx` | Redirection 🔄 | "Go to this other URL instead" |
| `4xx` | Client Error ❌ | "You made a mistake in your request" |
| `5xx` | Server Error 💥 | "We (the server) made a mistake" |

**Most important ones for Flask developers:**

| Code | Name | When Flask uses it |
|---|---|---|
| `200` | OK | Default for successful responses |
| `201` | Created | Used in APIs when something new is created |
| `204` | No Content | Success, but no body to return |
| `301` | Moved Permanently | Permanent redirect |
| `302` | Found (Temp. Redirect) | `redirect()` default |
| `400` | Bad Request | User sent bad data |
| `401` | Unauthorized | Not logged in |
| `403` | Forbidden | Logged in but not allowed |
| `404` | Not Found | URL doesn't exist |
| `405` | Method Not Allowed | Wrong HTTP method (Module 6) |
| `500` | Internal Server Error | Unhandled exception in your code |

---

## 12. Common Mistakes

1. **Forgetting to return anything from a view function**

```python
# WRONG - returns None
@app.route("/")
def home():
    print("Hello!")   # This only prints to the server terminal, not the browser!
    # No return statement!
```

Flask will raise a `ValueError: The view function did not return a valid response`. Always return something.

---

2. **Using `print()` thinking it shows in the browser**

`print()` outputs to your **server's terminal**, not to the browser. To send something to the browser, you must `return` it.

---

3. **Returning a list directly without `jsonify()`**

```python
# WRONG in older Flask versions
@app.route("/items")
def items():
    return [1, 2, 3]   # In Flask 2.2+, this works (auto-jsonifies lists too)

# SAFE for all versions
@app.route("/items")
def items():
    return jsonify([1, 2, 3])
```

---

4. **Confusing status codes**

Don't use `404` when you mean `403`. These have very different meanings:
- `404` = "I don't know what you're talking about" (the resource doesn't exist)
- `403` = "I know what you want, but I'm not letting you have it"

---

5. **Trying to return two values without a tuple**

```python
# WRONG
@app.route("/")
def home():
    return "Hello", "200"   # This IS a tuple, but status code must be an int, not a string

# RIGHT
@app.route("/")
def home():
    return "Hello", 200     # Status code as an integer
```

---

## 13. Interview Questions (with Answers)

**Q1: What is a view function in Flask?**
A: A regular Python function that Flask calls when a matching URL is requested. It must return something — a string, dictionary, tuple, or Response object — that Flask can convert into an HTTP response.

**Q2: What are the three valid forms of a tuple return value in Flask?**
A:
- `(body, status_code)`
- `(body, headers_dict)`
- `(body, status_code, headers_dict)`

**Q3: What is the difference between `return {"key": "val"}` and `return jsonify({"key": "val"})`?**
A: Functionally the same since Flask 1.0 — Flask automatically calls `jsonify()` on returned dictionaries. But `jsonify()` is still needed for returning lists, for more control, or for code clarity.

**Q4: What does `abort(404)` do internally?**
A: It raises a special `HTTPException` — a Python exception Flask intercepts and converts into an appropriate HTTP error response (404 Not Found), immediately stopping the view function.

**Q5: What happens if a view function returns `None` (i.e., no `return` statement)?**
A: Flask raises a `ValueError` because `None` is not a valid response value.

---

## 14. Best Practices

- **Always return something** from every view function.
- For **APIs**: return dictionaries or use `jsonify()` — never return HTML.
- For **web pages**: return rendered templates (Module 11) — not raw HTML strings.
- Use **`abort()`** for error conditions — it's cleaner than manually building error responses.
- Use **`redirect(url_for("endpoint_name"))`** instead of hardcoded URL strings — this is safer and makes URL changes painless (we cover `url_for` deeply next module).
- Respond with **semantically correct status codes** — `201` for created resources, `400` for bad input, `204` for successful deletes with no body.

---

## 15. Real-World Usage

In a real-world Flask API, your view functions look like this:

```python
from flask import jsonify, abort, make_response

@app.route("/api/users/<int:user_id>", methods=["GET"])
def get_user(user_id):
    user = db.get_user(user_id)         # fetch from database (Module 21)
    if user is None:
        abort(404)                       # user not found
    return jsonify(user.to_dict())       # return JSON representation

@app.route("/api/users", methods=["POST"])
def create_user():
    data = request.get_json()            # read incoming JSON (Module 9)
    if not data or "name" not in data:
        abort(400)                       # bad request - missing field
    new_user = db.create_user(data)
    return jsonify(new_user.to_dict()), 201   # 201 Created
```

This pattern — fetch/validate, abort on error, return JSON with correct status code — is the backbone of virtually every Flask REST API.

---

## 16. Mini Project

> Build a small "calculator API" with these routes:
> - `GET /add?a=5&b=3` → returns `{"result": 8}`
> - `GET /multiply?a=4&b=6` → returns `{"result": 24}`
> - `GET /divide?a=10&b=0` → returns `{"error": "Cannot divide by zero"}` with status code `400`
>
> (Don't worry about reading query parameters (`?a=5`) yet — use hardcoded numbers for now. We cover `request.args` in Module 9. Focus on getting the *return values and status codes right*.)

---

## 17. Practice Exercises

**5 Easy Questions**
1. What is a view function?
2. What does Flask do if you return a plain string from a view function?
3. What function do you call to immediately stop a view function and return an error?
4. What status code means "Not Found"?
5. True/False: `print("Hello")` inside a view function sends "Hello" to the browser.

**5 Medium Questions**
1. What is the difference between `jsonify()` and Python's `json.dumps()`?
2. When would you use a `201` status code instead of `200`?
3. What's the difference between `abort(401)` and `abort(403)`?
4. Write a view function that returns a response with a custom header `X-App-Name: MyApp`.
5. What does `redirect()` actually send to the browser? (What's in the HTTP response?)

**5 Hard Questions**
1. Explain, step by step, what Flask does internally after your view function returns `"Hello", 404`.
2. Why is `abort()` implemented as a Python exception internally? What advantage does this give?
3. If you wanted a `DELETE` request to return success but with no body (the HTTP standard way), what status code and return value would you use?
4. What's the difference between `make_response()` and just returning a tuple? When would you *need* `make_response()` specifically?
5. A view function returns a `Response` object but also sets `response.status_code` after returning. Will the code after the `return` statement run? Explain.

**2 Debugging Questions**
1. A developer's route returns `return "Hello", "200"` (status as a string). The response body appears fine but the app behaves oddly with status checks. What's the bug?
2. A developer returns `return jsonify(data), 200, {"Content-Type": "text/plain"}`. They expect JSON but their API client says the content type is `text/plain`. What happened?

**2 Interview Questions**
1. "Walk me through all the different things a Flask view function can return, and how Flask handles each one."
2. "What's the difference between `return abort(404)` and `abort(404)` (without return)?"

**1 Mini Project**
See Section 16 above.

---

## 18. Summary

| What you return | What Flask does |
|---|---|
| `"string"` | Wraps in Response: status=200, type=text/html |
| `("string", 404)` | Wraps in Response: status=404, type=text/html |
| `("string", 200, {headers})` | Wraps in Response: adds custom headers |
| `{"key": "val"}` | Converts to JSON: status=200, type=application/json |
| `make_response(...)` object | Uses as-is, no conversion needed |

### Key Functions Quick Reference

| Function | What it does |
|---|---|
| `make_response(body)` | Creates a Response object you can customize |
| `jsonify(data)` | Converts dict/list to JSON Response with correct Content-Type |
| `redirect(url)` | Sends browser to a different URL (302 by default) |
| `abort(status_code)` | Immediately stops the view and returns an error response |

### 💡 Memory Trick
**"Your view function is the chef. Flask is the waiter. You just make the food — the waiter figures out the plate, the packaging, and the delivery."**

### ❓ FAQ
- **"Do I always need `jsonify()`?"** No — returning a dict directly works in Flask 1.0+. But for lists, or when you need to set a custom status code alongside JSON (`return jsonify(data), 201`), you still need it.
- **"Is there a limit to what I can put in a response body?"** Technically no — but be thoughtful. Sending huge files as string bodies is inefficient; Flask has dedicated file-sending tools for that (covered later).

---

**End of Module 7.**
