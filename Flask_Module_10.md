# MODULE 10 — The Response Object

> **Course:** Flask — Zero to Production
> **Module:** 10 of 39
> **Style:** Practical-first, complete code examples

---

## 1. What Is the Response Object?

In Module 9 we learned about `request` — everything the **browser sends to you**.

The Response Object is the opposite — it's everything **you send back to the browser**.

Every HTTP response has three parts:
```
1. Status code  →  200 OK, 404 Not Found, 500 Server Error...
2. Headers      →  Content-Type, Set-Cookie, Location...
3. Body         →  The actual HTML / JSON / file content
```

When you `return "Hello"` from a view function, Flask automatically builds a response for you. But sometimes you need **full control** — setting custom headers, cookies, status codes, or content types. That's when you use the Response object directly.

---

## 2. Two Ways to Build Responses

### Way 1 — Let Flask do it automatically (simple cases)

```python
return "Hello"                   # Flask builds the response for you
return {"name": "Arjun"}         # Flask converts to JSON automatically
return "Not found", 404          # Flask sets status 404
```

### Way 2 — Build it yourself (when you need control)

```python
from flask import make_response

response = make_response("Hello")
response.status_code = 200
response.headers["X-Custom"] = "MyValue"
return response
```

---

## 3. `make_response()` — Full Control Over Your Response

`make_response()` takes the same things you'd normally `return` and gives you back a `Response` object you can customize.

### Complete Example

**`app.py`:**

```python
from flask import Flask, make_response

app = Flask(__name__)


@app.route("/")
def home():
    return """
    <!DOCTYPE html>
    <html>
    <head><title>Home</title></head>
    <body>
        <h1>Welcome!</h1>
        <a href="/custom">See custom response</a>
    </body>
    </html>
    """


@app.route("/custom")
def custom_response():
    # Build the response manually
    response = make_response("""
    <!DOCTYPE html>
    <html>
    <head><title>Custom Response</title></head>
    <body>
        <h1>This response has custom headers!</h1>
        <p>Open DevTools → Network tab to see them.</p>
    </body>
    </html>
    """)

    # Set status code
    response.status_code = 200

    # Add custom headers
    response.headers["X-App-Name"]    = "My Flask App"
    response.headers["X-Developer"]   = "Arjun"
    response.headers["Cache-Control"] = "no-cache"

    return response


if __name__ == "__main__":
    app.run(debug=True)
```

**How to see the custom headers:**
1. Run the app
2. Visit `http://localhost:5000/custom`
3. Press `F12` → Network tab → click the request → Headers section
4. You'll see `X-App-Name: My Flask App` in the response headers

---

## 4. Setting the Content-Type

`Content-Type` tells the browser **what kind of data** is in the response body.

```python
from flask import Flask, make_response

app = Flask(__name__)


# Returning plain text (not HTML)
@app.route("/text")
def plain_text():
    response = make_response("Hello, this is plain text. No HTML tags.")
    response.headers["Content-Type"] = "text/plain"
    return response


# Returning XML
@app.route("/xml")
def xml_response():
    xml_data = """<?xml version="1.0"?>
    <user>
        <name>Arjun</name>
        <email>arjun@test.com</email>
    </user>"""
    response = make_response(xml_data)
    response.headers["Content-Type"] = "application/xml"
    return response


# Returning CSV
@app.route("/csv")
def csv_response():
    csv_data = "Name,Age,City\nArjun,25,Mumbai\nPriya,28,Delhi"
    response = make_response(csv_data)
    response.headers["Content-Type"] = "text/csv"
    response.headers["Content-Disposition"] = "attachment; filename=users.csv"
    return response


if __name__ == "__main__":
    app.run(debug=True)
```

Visit `/csv` — the browser will **download** a CSV file called `users.csv` instead of showing it on screen. That's what `Content-Disposition: attachment; filename=...` does.

---

## 5. Status Codes — Setting Them the Right Way

You already know you can return a tuple:

```python
return "Not found", 404
```

With `make_response()`:

```python
from flask import Flask, make_response, request, redirect, url_for, abort

app = Flask(__name__)


@app.route("/")
def home():
    return """
    <!DOCTYPE html>
    <html>
    <head><title>Status Code Demo</title></head>
    <body>
        <h1>Status Code Demo</h1>
        <ul>
            <li><a href="/success">200 - Success</a></li>
            <li><a href="/created">201 - Created</a></li>
            <li><a href="/no-content">204 - No Content</a></li>
            <li><a href="/redirect-me">302 - Redirect</a></li>
            <li><a href="/forbidden">403 - Forbidden</a></li>
            <li><a href="/not-found">404 - Not Found</a></li>
            <li><a href="/server-error">500 - Server Error</a></li>
        </ul>
    </body>
    </html>
    """


@app.route("/success")
def success():
    response = make_response("<h1>Everything worked!</h1>")
    response.status_code = 200   # Default — usually no need to set explicitly
    return response


@app.route("/created")
def created():
    # 201 = "Something was created" — common in APIs after POST
    response = make_response("<h1>New item created!</h1>")
    response.status_code = 201
    return response


@app.route("/no-content")
def no_content():
    # 204 = "Success, but nothing to show" — e.g., after delete
    response = make_response("")
    response.status_code = 204
    return response


@app.route("/redirect-me")
def redirect_me():
    # Sends browser to /success
    return redirect(url_for("success"))


@app.route("/forbidden")
def forbidden():
    abort(403)   # Stops immediately — returns 403 Forbidden


@app.route("/not-found")
def not_found():
    abort(404)   # Stops immediately — returns 404 Not Found


@app.route("/server-error")
def server_error():
    abort(500)   # Stops immediately — returns 500 Internal Server Error


if __name__ == "__main__":
    app.run(debug=True)
```

---

## 6. Redirects — Sending the Browser Somewhere Else

A redirect tells the browser: **"Don't stay here — go to this other URL."**

The browser receives a response with:
- Status: `302 Found` (or `301 Moved Permanently`)
- Header: `Location: /new-url`
- The browser automatically follows the `Location` header

```python
from flask import Flask, redirect, url_for, request

app = Flask(__name__)


@app.route("/")
def home():
    return """
    <!DOCTYPE html>
    <html>
    <head><title>Redirect Demo</title></head>
    <body>
        <h1>Redirect Demo</h1>
        <ul>
            <li><a href="/old-page">Old page (302 redirect)</a></li>
            <li><a href="/google">Go to Google (external)</a></li>
            <li><a href="/login-required">Login required page</a></li>
        </ul>

        <h2>Login Form</h2>
        <form action="/login" method="POST">
            <input type="text"     name="username" placeholder="Username"><br><br>
            <input type="password" name="password" placeholder="Password"><br><br>
            <button type="submit">Login</button>
        </form>
    </body>
    </html>
    """


# 302 Temporary redirect — old URL moved to new URL temporarily
@app.route("/old-page")
def old_page():
    return redirect(url_for("new_page"))    # Redirect to new_page endpoint


@app.route("/new-page")
def new_page():
    return """
    <!DOCTYPE html>
    <html>
    <head><title>New Page</title></head>
    <body>
        <h1>You were redirected here from /old-page!</h1>
        <a href="/">Back to home</a>
    </body>
    </html>
    """


# 301 Permanent redirect — for SEO (browser caches this forever)
@app.route("/old-about")
def old_about():
    return redirect(url_for("about"), code=301)


@app.route("/about")
def about():
    return """
    <!DOCTYPE html>
    <html>
    <head><title>About</title></head>
    <body><h1>About Page</h1><a href="/">Home</a></body>
    </html>
    """


# External redirect
@app.route("/google")
def go_google():
    return redirect("https://www.google.com")


# Redirect after login check
@app.route("/login-required")
def login_required():
    logged_in = False   # Pretend no one is logged in
    if not logged_in:
        return redirect(url_for("home"))    # Send them back to home
    return "<h1>Secret content!</h1>"


# POST → Redirect → GET pattern
@app.route("/login", methods=["POST"])
def login():
    username = request.form.get("username", "")
    password = request.form.get("password", "")

    if username == "admin" and password == "1234":
        return redirect(url_for("dashboard"))
    return redirect(url_for("home"))


@app.route("/dashboard")
def dashboard():
    return """
    <!DOCTYPE html>
    <html>
    <head><title>Dashboard</title></head>
    <body>
        <h1>Welcome to your Dashboard!</h1>
        <a href="/">Logout</a>
    </body>
    </html>
    """


if __name__ == "__main__":
    app.run(debug=True)
```

---

## 7. Setting Cookies on the Response

Cookies are small pieces of data the server tells the browser to **store and send back** on every future request.

```
Server → "Store this cookie: username=Arjun"
Browser → stores it
Browser → sends it back with every future request to this site
Server → reads it via request.cookies.get("username")
```

```python
from flask import Flask, make_response, request, redirect, url_for

app = Flask(__name__)


@app.route("/")
def home():
    # Read cookie if it exists
    username = request.cookies.get("username", None)

    if username:
        greeting = f"<p>Welcome back, <strong>{username}</strong>!</p>"
        logout_link = '<a href="/logout">Logout</a>'
    else:
        greeting = "<p>You are not logged in.</p>"
        logout_link = ""

    return f"""
    <!DOCTYPE html>
    <html>
    <head><title>Cookie Demo</title></head>
    <body>
        <h1>Cookie Demo</h1>
        {greeting}
        {logout_link}

        <h2>Set Your Name</h2>
        <form action="/set-cookie" method="POST">
            <input type="text" name="username" placeholder="Enter your name">
            <button type="submit">Remember Me</button>
        </form>
    </body>
    </html>
    """


@app.route("/set-cookie", methods=["POST"])
def set_cookie():
    username = request.form.get("username", "Guest")

    # Redirect to home, but set a cookie on the response
    response = make_response(redirect(url_for("home")))

    response.set_cookie(
        "username",           # Cookie name
        username,             # Cookie value
        max_age=60*60*24*7,   # Expires in 7 days (seconds)
        httponly=True,        # Can't be read by JavaScript (security)
        samesite="Lax"        # Protects against CSRF (security)
    )
    return response


@app.route("/logout")
def logout():
    response = make_response(redirect(url_for("home")))
    response.delete_cookie("username")   # Remove the cookie
    return response


if __name__ == "__main__":
    app.run(debug=True)
```

**What the `set_cookie()` parameters do:**

| Parameter | What it does |
|---|---|
| `key` | Name of the cookie |
| `value` | Value to store |
| `max_age` | Seconds until cookie expires. `None` = expires when browser closes |
| `expires` | Specific datetime when cookie expires |
| `path` | Which URL path the cookie applies to. Default `"/"` means all paths |
| `domain` | Which domain the cookie applies to |
| `secure` | If `True`, cookie only sent over HTTPS |
| `httponly` | If `True`, JavaScript cannot read this cookie (prevents XSS theft) |
| `samesite` | `"Lax"` or `"Strict"` — helps prevent CSRF attacks |

> **Security Rule:** Always set `httponly=True` on cookies that store sensitive data (like session IDs). This single setting prevents a whole category of XSS attacks where an attacker's JavaScript tries to steal your cookies.

---

## 8. Custom Error Pages

By default, Flask shows a plain, ugly error page for 404 and 500 errors. You can replace these with beautiful custom pages using `@app.errorhandler`.

```python
from flask import Flask, render_template_string, request

app = Flask(__name__)


# Custom 404 page
@app.errorhandler(404)
def page_not_found(error):
    return """
    <!DOCTYPE html>
    <html>
    <head>
        <title>404 - Page Not Found</title>
        <style>
            body {
                font-family: Arial, sans-serif;
                text-align: center;
                padding: 80px;
                background: #f8f9fa;
            }
            h1 { font-size: 80px; color: #dc3545; margin: 0; }
            h2 { color: #6c757d; }
            a  { color: #007bff; text-decoration: none; font-size: 18px; }
            a:hover { text-decoration: underline; }
        </style>
    </head>
    <body>
        <h1>404</h1>
        <h2>Oops! Page not found.</h2>
        <p>The page you are looking for doesn't exist.</p>
        <a href="/">← Go back to Home</a>
    </body>
    </html>
    """, 404   # ← Must return the status code too!


# Custom 500 page
@app.errorhandler(500)
def server_error(error):
    return """
    <!DOCTYPE html>
    <html>
    <head>
        <title>500 - Server Error</title>
        <style>
            body {
                font-family: Arial, sans-serif;
                text-align: center;
                padding: 80px;
                background: #fff3cd;
            }
            h1 { font-size: 80px; color: #856404; margin: 0; }
            h2 { color: #6c757d; }
            a  { color: #007bff; text-decoration: none; font-size: 18px; }
        </style>
    </head>
    <body>
        <h1>500</h1>
        <h2>Something went wrong on our end.</h2>
        <p>We are working on fixing this. Please try again later.</p>
        <a href="/">← Go back to Home</a>
    </body>
    </html>
    """, 500


# Custom 403 page
@app.errorhandler(403)
def forbidden(error):
    return """
    <!DOCTYPE html>
    <html>
    <head>
        <title>403 - Forbidden</title>
        <style>
            body {
                font-family: Arial, sans-serif;
                text-align: center;
                padding: 80px;
            }
            h1 { font-size: 80px; color: #fd7e14; margin: 0; }
            a  { color: #007bff; text-decoration: none; }
        </style>
    </head>
    <body>
        <h1>403</h1>
        <h2>Access Denied</h2>
        <p>You do not have permission to view this page.</p>
        <a href="/">← Go back to Home</a>
    </body>
    </html>
    """, 403


# Normal routes
@app.route("/")
def home():
    return """
    <!DOCTYPE html>
    <html>
    <head><title>Home</title></head>
    <body>
        <h1>Home Page</h1>
        <ul>
            <li><a href="/missing-page">Trigger 404</a></li>
            <li><a href="/crash">Trigger 500</a></li>
        </ul>
    </body>
    </html>
    """


@app.route("/crash")
def crash():
    raise Exception("This is a deliberate crash to test 500 error page!")


if __name__ == "__main__":
    app.run(debug=False)   # debug=False so our error handlers show (not Werkzeug's debugger)
```

> **Note:** Custom error handlers only show when `debug=False`. In debug mode, Flask shows the interactive Werkzeug debugger instead (from Module 5).

---

## 9. Complete Example — Everything Together

Here is one complete app demonstrating `make_response`, custom headers, cookies, redirect, and custom error pages all working together.

**`app.py`:**

```python
from flask import Flask, make_response, redirect, url_for, request, abort

app = Flask(__name__)


# ─── HOME ──────────────────────────────────────────────────────────────────
@app.route("/")
def home():
    username = request.cookies.get("username", None)
    if username:
        welcome = f"<p>Logged in as: <strong>{username}</strong> | <a href='/logout'>Logout</a></p>"
    else:
        welcome = "<p>Not logged in. <a href='/login'>Login here</a></p>"

    return f"""
    <!DOCTYPE html>
    <html>
    <head>
        <title>Response Demo App</title>
        <style>
            body {{ font-family: Arial, sans-serif; max-width: 600px; margin: 40px auto; }}
            a    {{ display: block; margin: 8px 0; color: #007bff; }}
            .box {{ background: #f0f0f0; padding: 15px; border-radius: 8px; margin: 10px 0; }}
        </style>
    </head>
    <body>
        <h1>Response Object Demo</h1>
        {welcome}

        <div class="box">
            <h3>Try these:</h3>
            <a href="/headers-demo">View custom response headers</a>
            <a href="/download-csv">Download a CSV file</a>
            <a href="/old-url">Old URL (redirects to /new-url)</a>
            <a href="/admin">Admin page (403 if not logged in)</a>
            <a href="/broken-page">Trigger a 404 error</a>
        </div>
    </body>
    </html>
    """


# ─── LOGIN / LOGOUT ────────────────────────────────────────────────────────
@app.route("/login", methods=["GET", "POST"])
def login():
    error = ""

    if request.method == "POST":
        username = request.form.get("username", "").strip()
        password = request.form.get("password", "").strip()

        if username == "admin" and password == "1234":
            response = make_response(redirect(url_for("home")))
            response.set_cookie("username", username, max_age=3600, httponly=True, samesite="Lax")
            return response
        else:
            error = "<p style='color:red'>Invalid username or password.</p>"

    return f"""
    <!DOCTYPE html>
    <html>
    <head><title>Login</title>
        <style>
            body  {{ font-family: Arial, sans-serif; max-width: 400px; margin: 80px auto; }}
            input {{ display: block; width: 100%; padding: 8px; margin: 8px 0; }}
            button {{ padding: 10px 20px; background: #007bff; color: white; border: none; cursor: pointer; }}
        </style>
    </head>
    <body>
        <h2>Login</h2>
        {error}
        <form method="POST">
            <label>Username:</label>
            <input type="text" name="username" placeholder="admin">
            <label>Password:</label>
            <input type="password" name="password" placeholder="1234">
            <button type="submit">Login</button>
        </form>
        <a href="/">← Back to Home</a>
    </body>
    </html>
    """


@app.route("/logout")
def logout():
    response = make_response(redirect(url_for("home")))
    response.delete_cookie("username")
    return response


# ─── CUSTOM HEADERS DEMO ───────────────────────────────────────────────────
@app.route("/headers-demo")
def headers_demo():
    response = make_response("""
    <!DOCTYPE html>
    <html>
    <head><title>Custom Headers</title></head>
    <body>
        <h1>Custom Headers Set!</h1>
        <p>Open DevTools → Network tab → click this request → Response Headers</p>
        <p>You will see X-App-Name and X-Developer headers.</p>
        <a href="/">← Home</a>
    </body>
    </html>
    """)
    response.headers["X-App-Name"]    = "Response Demo App"
    response.headers["X-Developer"]   = "Arjun Sharma"
    response.headers["Cache-Control"] = "no-store"
    return response


# ─── CSV DOWNLOAD ──────────────────────────────────────────────────────────
@app.route("/download-csv")
def download_csv():
    csv_content = "Name,Age,City\nArjun,25,Mumbai\nPriya,28,Delhi\nRahul,30,Bangalore"
    response = make_response(csv_content)
    response.headers["Content-Type"]        = "text/csv"
    response.headers["Content-Disposition"] = "attachment; filename=users.csv"
    return response


# ─── REDIRECT ──────────────────────────────────────────────────────────────
@app.route("/old-url")
def old_url():
    return redirect(url_for("new_url"), code=301)


@app.route("/new-url")
def new_url():
    return """
    <!DOCTYPE html>
    <html>
    <head><title>New URL</title></head>
    <body>
        <h1>You were redirected from /old-url</h1>
        <p>This was a 301 Permanent Redirect.</p>
        <a href="/">← Home</a>
    </body>
    </html>
    """


# ─── ADMIN (COOKIE-PROTECTED) ──────────────────────────────────────────────
@app.route("/admin")
def admin():
    username = request.cookies.get("username")
    if not username:
        abort(403)
    return f"""
    <!DOCTYPE html>
    <html>
    <head><title>Admin</title></head>
    <body>
        <h1>Admin Panel</h1>
        <p>Welcome, {username}. You can see this because you are logged in.</p>
        <a href="/">← Home</a>
    </body>
    </html>
    """


# ─── CUSTOM ERROR HANDLERS ─────────────────────────────────────────────────
@app.errorhandler(403)
def forbidden(e):
    return """
    <!DOCTYPE html>
    <html>
    <head><title>403 Forbidden</title>
        <style>
            body {{ font-family: Arial, sans-serif; text-align: center; padding: 80px; }}
            h1   {{ font-size: 72px; color: #fd7e14; margin: 0; }}
        </style>
    </head>
    <body>
        <h1>403</h1>
        <h2>Access Denied</h2>
        <p>You need to be logged in to view this page.</p>
        <a href="/login">Login here</a> | <a href="/">Home</a>
    </body>
    </html>
    """, 403


@app.errorhandler(404)
def not_found(e):
    return """
    <!DOCTYPE html>
    <html>
    <head><title>404 Not Found</title>
        <style>
            body {{ font-family: Arial, sans-serif; text-align: center; padding: 80px; }}
            h1   {{ font-size: 72px; color: #dc3545; margin: 0; }}
        </style>
    </head>
    <body>
        <h1>404</h1>
        <h2>Page Not Found</h2>
        <p>The page you are looking for does not exist.</p>
        <a href="/">← Go Home</a>
    </body>
    </html>
    """, 404


if __name__ == "__main__":
    app.run(debug=False)
```

---

## 10. Visual Diagram — Response Object Structure

```
make_response("Hello")
        │
        ▼
  Response Object
  ┌─────────────────────────────────┐
  │  status_code  = 200             │
  │                                 │
  │  headers = {                    │
  │    "Content-Type": "text/html", │
  │    "Content-Length": "5",       │
  │    "X-Custom": "...",           │  ← you can add custom headers
  │    "Set-Cookie": "...",         │  ← response.set_cookie() adds this
  │    "Location": "...",           │  ← redirect() adds this
  │  }                              │
  │                                 │
  │  data = b"Hello"               │  ← the body (as bytes)
  └─────────────────────────────────┘
        │
        ▼
  Sent to browser as raw HTTP bytes
```

---

## 11. Common Mistakes

### Mistake 1: Forgetting to return the 404 status code from error handlers

```python
# ❌ WRONG — browser receives status 200, even though it's a "not found" page
@app.errorhandler(404)
def not_found(e):
    return "<h1>Page not found</h1>"   # Missing status code!

# ✅ CORRECT
@app.errorhandler(404)
def not_found(e):
    return "<h1>Page not found</h1>", 404
```

---

### Mistake 2: Setting a cookie but forgetting `httponly=True`

```python
# ❌ RISKY — JavaScript can steal this cookie
response.set_cookie("session_id", "abc123")

# ✅ SECURE
response.set_cookie("session_id", "abc123", httponly=True, samesite="Lax")
```

---

### Mistake 3: Trying to set cookies on `return` directly

```python
# ❌ WRONG — you can't set cookies on a plain string return
@app.route("/login", methods=["POST"])
def login():
    return redirect(url_for("home"))   # No way to set cookie here!

# ✅ CORRECT — use make_response() first
@app.route("/login", methods=["POST"])
def login():
    response = make_response(redirect(url_for("home")))
    response.set_cookie("user", "Arjun", httponly=True)
    return response
```

---

### Mistake 4: Using error handlers with `debug=True`

When `debug=True`, Flask shows the interactive Werkzeug debugger instead of your custom error handlers. Always test error pages with `debug=False`.

---

### Mistake 5: 301 vs 302 — Using the wrong redirect code

```python
# ❌ WRONG for temporary redirects — 301 gets cached by browsers FOREVER
return redirect(url_for("new_page"), code=301)   # Use only for PERMANENT moves

# ✅ CORRECT for most cases — 302 is temporary and not cached
return redirect(url_for("new_page"))   # Default is 302
```

---

## 12. Interview Questions

**Q1: What is `make_response()` and when do you need it?**
A: `make_response()` creates a `Response` object from a return value, giving you full control to set custom headers, cookies, status codes, and content types. You need it whenever a simple `return` statement isn't enough — e.g., when setting cookies or adding custom headers.

**Q2: How do you set a cookie in Flask?**
A: Build a `Response` object with `make_response()`, then call `response.set_cookie(name, value, ...)` on it before returning it.

**Q3: What does `httponly=True` do when setting a cookie?**
A: It tells the browser that JavaScript cannot read this cookie — only the browser can send it with HTTP requests. This protects session cookies from being stolen by XSS attacks.

**Q4: What is the difference between a 301 and 302 redirect?**
A: `301 Moved Permanently` — the browser caches this redirect forever; used when a URL has permanently moved. `302 Found` — temporary redirect; browser doesn't cache it; used for most redirects in web apps.

**Q5: Why must custom `errorhandler` functions return the status code as the second value?**
A: Because Flask uses the return value to build the HTTP response. Without specifying the status code, Flask defaults to 200 — so your "page not found" page would be sent with a 200 status, misleading browsers, search engines, and API clients.

---

## 13. Best Practices

- Use `make_response()` whenever you need to set cookies or custom headers — it's the correct, clean way.
- Always set `httponly=True` and `samesite="Lax"` on cookies storing sensitive data.
- Always use `code=302` (or the default) for redirects — use `code=301` only when you're absolutely sure a URL is permanently gone.
- Always return the correct status code from `@app.errorhandler` functions.
- Use `Content-Disposition: attachment; filename=...` when you want the browser to download a file instead of displaying it.
- Test your custom error pages with `debug=False`.

---

## 14. Summary

| What you want to do | How to do it |
|---|---|
| Set a custom header | `response.headers["X-Key"] = "Value"` |
| Set status code | `response.status_code = 201` or `return body, 201` |
| Set a cookie | `response.set_cookie("name", "value", httponly=True)` |
| Delete a cookie | `response.delete_cookie("name")` |
| Redirect temporarily | `return redirect(url_for("endpoint"))` (302) |
| Redirect permanently | `return redirect(url_for("endpoint"), code=301)` |
| Force file download | `response.headers["Content-Disposition"] = "attachment; filename=x.csv"` |
| Custom error page | `@app.errorhandler(404)` → `return html, 404` |

### 💡 Memory Trick
**"`request` = everything the browser sends you. `response` = everything you send back. `make_response()` = the tool you use when you need to control every detail of what you send back."**

---

**End of Module 10.**
