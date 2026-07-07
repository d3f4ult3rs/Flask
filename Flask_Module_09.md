# MODULE 9 — The Request Object

> **Course:** Flask — Zero to Production
> **Module:** 9 of 39
> **Style:** Practical-first

---

## 1. What Is the Request Object?

When a user visits your site or submits a form, their browser sends an **HTTP request**. That request contains a LOT of information:

- What URL did they visit?
- What HTTP method did they use (GET, POST)?
- What data did they send (form fields, JSON)?
- What files did they upload?
- What browser are they using?
- What cookies do they have?

Flask wraps all of this into one convenient Python object called **`request`**.

You just import it and use it inside any view function:

```python
from flask import request
```

That's it. Flask automatically fills this object with the current request's data before calling your view function.

---

## 2. Quick Overview — What's Inside `request`

```
request
 ├── request.method          → "GET", "POST", "PUT", "DELETE", etc.
 ├── request.args            → Query string parameters (?name=Arjun)
 ├── request.form            → HTML form data (POST body)
 ├── request.json            → JSON body (API requests)
 ├── request.data            → Raw request body as bytes
 ├── request.files           → Uploaded files
 ├── request.headers         → HTTP headers
 ├── request.cookies         → Cookies sent by browser
 ├── request.url             → Full URL of the request
 ├── request.path            → Just the path part (/about)
 ├── request.host            → Domain name (localhost:5000)
 └── request.remote_addr     → IP address of the client
```

Let's go through each one practically.

---

## 3. `request.method`

**What it is:** The HTTP method used — `"GET"`, `"POST"`, `"PUT"`, `"DELETE"`, etc.

**Most common use:** Handling a form that both displays (GET) and processes (POST) on the same route.

```python
from flask import Flask, request

app = Flask(__name__)

@app.route("/login", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        return "Processing your login..."
    return "Please show the login form"
```

**Test it:**
- Visit `GET /login` in browser → sees the form message
- Submit a form to `POST /login` → sees the processing message

---

## 4. `request.args` — Query String Parameters

**What it is:** Data passed in the URL after the `?` sign.

Example URL: `http://localhost:5000/search?q=flask&page=2`

- `q` = `"flask"`
- `page` = `"2"`

```python
@app.route("/search")
def search():
    query = request.args.get("q", "")          # "flask"
    page  = request.args.get("page", "1")      # "2"
    return f"Searching for: {query}, Page: {page}"
```

**How to use `request.args`:**

```python
# Method 1 — .get() with default (SAFE — won't crash if key missing)
name = request.args.get("name", "Guest")

# Method 2 — direct access (UNSAFE — raises KeyError if key missing)
name = request.args["name"]

# Get all values for a repeated key: ?tag=python&tag=flask
tags = request.args.getlist("tag")   # ["python", "flask"]

# Check if a key exists
if "q" in request.args:
    print("Search query present")

# See all arguments as a plain dict
all_args = request.args.to_dict()
```

**Common Methods**
```python
| Method                              | Description                        | Example Result                  |
| ----------------------------------- | ---------------------------------- | ------------------------------- |
|  request.args.get("name")           | Get first value or `None`          | `"abhi"`                        |
|  request.args.get("name", "Guest")  | Get value with default             | `"Guest"`                       |
|  request.args.get("age", type=int)  | Convert to integer                 | `20`                            |
|  request.args["name"]               | Get value, raises error if missing | `"abhi"`                        |
|  request.args.getlist("name")       | Get all values                     | `["python", "flask"]`           |
|  "name" in request.args             | Check if key exists                | `True`                          |
|  request.args.items()               | Iterate over key-value pairs       | `("name", "abhi")`              |
|  request.args.keys()                | Get all keys                       | `["name", "age"]`               |
|  request.args.values()              | Get all values                     | `["abhi", "20"]`                |
|  request.args.to_dict()             | Convert to standard dictionary     | `{"name": "abhi", "age": "20"}` |
```

> **Important:** All values from `request.args` are **strings**. If you need a number, convert manually:
> ```python
> page = int(request.args.get("page", 1))
> ```

---

## 5. `request.form` — HTML Form Data

**What it is:** Data submitted via an HTML `<form>` with `method="POST"`.

**A complete working example:**

```python
from flask import Flask, request

app = Flask(__name__)

@app.route("/register", methods=["GET", "POST"])
def register():
    if request.method == "POST":
        username = request.form.get("username", "")
        email    = request.form.get("email", "")
        password = request.form.get("password", "")

        # Basic validation
        if not username or not email or not password:
            return "All fields are required!", 400

        return f"Registered: {username} ({email})"

    # Show the form on GET
    return """
        <form method="POST">
            <input type="text"     name="username"  placeholder="Username"><br>
            <input type="email"    name="email"     placeholder="Email"><br>
            <input type="password" name="password"  placeholder="Password"><br>
            <button type="submit">Register</button>
        </form>
    """

if __name__ == "__main__":
    app.run(debug=True)
```

**Step-by-step — what happens on form submit:**

```
User fills form and clicks submit
   ↓
Browser sends POST /register
   ↓
Request body contains:
   username=Arjun&email=arjun@test.com&password=secret123
   ↓
Flask parses this into request.form:
   {"username": "Arjun", "email": "arjun@test.com", "password": "secret123"}
   ↓
Your view function reads request.form.get("username")  → "Arjun"
   ↓
Returns response
```

**How to use `request.form`:**

```python
# .get() with a default — safe, no crash if field missing
username = request.form.get("username", "")

# Direct access — crashes with 400 if key missing (actually useful for required fields)
username = request.form["username"]

# Get a checkbox (returns "on" if checked, None if not)
subscribe = request.form.get("subscribe")     # "on" or None

# Get repeated fields: <select multiple> or checkboxes with same name
hobbies = request.form.getlist("hobby")       # ["cricket", "chess"]

# See all form data as plain dict
all_data = request.form.to_dict()
```

---

## 6. `request.args` vs `request.form` — Side by Side

| Feature | `request.args` | `request.form` |
|---|---|---|
| Where is the data? | URL: `?key=value` | Request body |
| Which method? | Usually `GET` | Usually `POST` |
| Visible in browser? | Yes (in URL bar) | No |
| Good for | Search, filters, pagination | Login, registration, any form |
| Max size | Limited by URL length | Much larger (browser limit) |

---

## 7. `request.json` — JSON Data (For APIs)

**What it is:** JSON data sent in the request body. Used when building APIs that receive JSON from a frontend (React, mobile app, Postman, etc.).

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/api/create-user", methods=["POST"])
def create_user():
    data = request.get_json()    # Parse JSON body → Python dict

    if data is None:
        return jsonify({"error": "No JSON received"}), 400

    name  = data.get("name", "")
    email = data.get("email", "")

    if not name or not email:
        return jsonify({"error": "name and email required"}), 400

    return jsonify({
        "message": "User created",
        "user": {"name": name, "email": email}
    }), 201
```

**Testing this with `curl`:**

```bash
curl -X POST http://localhost:5000/api/create-user \
     -H "Content-Type: application/json" \
     -d '{"name": "Arjun", "email": "arjun@test.com"}'
```

**`request.get_json()` vs `request.json`:**

```python
# Recommended — returns None if JSON is invalid (safe)
data = request.get_json()

# Short form — raises exception if JSON is invalid (can cause 500 errors)
data = request.json

# Force parsing even without Content-Type header (useful for testing)
data = request.get_json(force=True)

# Silent mode — returns None on error instead of raising
data = request.get_json(silent=True)
```

> **Always use `request.get_json(silent=True)` in APIs** — it returns `None` gracefully instead of crashing when the client sends malformed JSON.

---

## 8. `request.files` — Uploaded Files

**What it is:** Files uploaded via an HTML form with `enctype="multipart/form-data"`.

```python
import os
from flask import Flask, request

app = Flask(__name__)
app.config["UPLOAD_FOLDER"] = "uploads"

@app.route("/upload", methods=["GET", "POST"])
def upload():
    if request.method == "POST":
        # Check if file was included
        if "photo" not in request.files:
            return "No file part in the form", 400

        file = request.files["photo"]

        # Check if user actually selected a file
        if file.filename == "":
            return "No file selected", 400

        # Save it
        save_path = os.path.join(app.config["UPLOAD_FOLDER"], file.filename)
        file.save(save_path)

        return f"File saved: {file.filename}"

    return """
        <form method="POST" enctype="multipart/form-data">
            <input type="file" name="photo">
            <button>Upload</button>
        </form>
    """
```

**Key attributes of an uploaded file:**

```python
file = request.files["photo"]

file.filename        # Original filename: "profile.jpg"
file.content_type    # MIME type: "image/jpeg"
file.read()          # Read the file content as bytes
file.save("path/to/save")   # Save to disk
```

> **Security Warning:** Never trust `file.filename` directly — a malicious user could send `filename="../../../etc/passwd"` to overwrite system files. Always sanitize it:
> ```python
> from werkzeug.utils import secure_filename
> safe_name = secure_filename(file.filename)   # Strips dangerous characters
> ```

---

## 9. `request.headers` — HTTP Headers

**What it is:** All HTTP headers sent by the client.

```python
@app.route("/headers")
def show_headers():
    user_agent   = request.headers.get("User-Agent")
    content_type = request.headers.get("Content-Type")
    auth_token   = request.headers.get("Authorization")

    return f"""
        User-Agent: {user_agent}
        Content-Type: {content_type}
        Auth Token: {auth_token}
    """
```

**Common headers you'll read:**

| Header | What it contains | Example |
|---|---|---|
| `User-Agent` | Browser/client info | `Mozilla/5.0 ...` |
| `Authorization` | Auth token/credentials | `Bearer eyJhbG...` |
| `Content-Type` | Format of the request body | `application/json` |
| `Accept` | Formats the client can handle | `text/html, application/json` |
| `Referer` | Where the request came from | `https://google.com` |
| `X-Forwarded-For` | Real client IP behind a proxy | `203.0.113.42` |

---

## 10. `request.cookies` — Reading Cookies

**What it is:** Cookies that the browser has stored for your domain and is sending with this request.

```python
@app.route("/dashboard")
def dashboard():
    username = request.cookies.get("username")

    if not username:
        return "No cookie found — please log in first"

    return f"Welcome back, {username}!"
```

We cover setting cookies (on the response side) in Module 17.

---

## 11. Other Useful `request` Attributes

```python
@app.route("/info")
def info():
    return f"""
    Full URL:      {request.url}
    Path only:     {request.path}
    Host:          {request.host}
    Method:        {request.method}
    Client IP:     {request.remote_addr}
    Is AJAX?:      {request.is_json}
    Is Secure?:    {request.is_secure}
    """
```

**What each one gives you:**

| Attribute | Example value |
|---|---|
| `request.url` | `http://localhost:5000/search?q=flask` |
| `request.path` | `/search` |
| `request.host` | `localhost:5000` |
| `request.method` | `"GET"` |
| `request.remote_addr` | `"127.0.0.1"` |
| `request.is_json` | `True` if Content-Type is `application/json` |
| `request.is_secure` | `True` if request came over HTTPS |
| `request.referrer` | URL of the page that linked here |

---

## 12. Complete Practical Example — A Contact Form

Here's a real, complete form that uses `request.method`, `request.form`, and validation all together:

```python
from flask import Flask, request, redirect, url_for

app = Flask(__name__)

# Fake "database" for storing messages
messages = []

@app.route("/contact", methods=["GET", "POST"])
def contact():
    errors = {}

    if request.method == "POST":
        name    = request.form.get("name", "").strip()
        email   = request.form.get("email", "").strip()
        message = request.form.get("message", "").strip()

        # Validate inputs
        if not name:
            errors["name"] = "Name is required"
        if not email or "@" not in email:
            errors["email"] = "Valid email is required"
        if not message:
            errors["message"] = "Message cannot be empty"
        if len(message) > 500:
            errors["message"] = "Message too long (max 500 chars)"

        # If no errors, save and redirect
        if not errors:
            messages.append({
                "name": name,
                "email": email,
                "message": message
            })
            return redirect(url_for("thank_you"))

        # If errors, fall through to show form again with error messages

    # Build error HTML snippets for display
    name_err    = f"<span style='color:red'>{errors.get('name','')}</span>"
    email_err   = f"<span style='color:red'>{errors.get('email','')}</span>"
    message_err = f"<span style='color:red'>{errors.get('message','')}</span>"

    return f"""
        <h2>Contact Us</h2>
        <form method="POST">
            <label>Name: {name_err}</label><br>
            <input type="text" name="name" value="{request.form.get('name','')}"><br><br>

            <label>Email: {email_err}</label><br>
            <input type="email" name="email" value="{request.form.get('email','')}"><br><br>

            <label>Message: {message_err}</label><br>
            <textarea name="message" rows="4">{request.form.get('message','')}</textarea><br><br>

            <button type="submit">Send Message</button>
        </form>
    """


@app.route("/thank-you")
def thank_you():
    return f"<h2>Thank you! We have {len(messages)} message(s) so far.</h2>"


if __name__ == "__main__":
    app.run(debug=True)
```

**What this demonstrates:**
- `request.method` — detecting GET vs POST
- `request.form.get()` — reading form fields safely
- `.strip()` — trimming whitespace from user input
- Validation logic — checking for empty fields, email format, length
- **Post/Redirect/Get pattern** — after successful POST, redirect to prevent duplicate submissions on refresh
- Re-populating form fields on error — using `request.form.get("name", "")` to show what the user typed

---

## 13. Step-by-Step — What Happens on Form Submit

```
User fills form → clicks "Send Message"
   ↓
Browser sends POST /contact
   Body: name=Arjun&email=arjun@test.com&message=Hello!
   ↓
Werkzeug parses request body
   ↓
Flask builds request object:
   request.method = "POST"
   request.form = {"name": "Arjun", "email": "arjun@test.com", "message": "Hello!"}
   ↓
Flask calls contact()
   ↓
request.method == "POST" → True
   ↓
name    = request.form.get("name") → "Arjun"
email   = request.form.get("email") → "arjun@test.com"
message = request.form.get("message") → "Hello!"
   ↓
Validation passes → no errors
   ↓
Message saved → redirect(url_for("thank_you"))
   ↓
Browser receives 302 Redirect → follows to GET /thank-you
   ↓
thank_you() runs → shows confirmation page
```

---

## 14. Visual Diagram — `request` Data Sources

```
Browser sends HTTP Request
         │
         ├── URL: /search?q=flask&page=2
         │              └──────────────── → request.args
         │
         ├── Method: POST
         │              └──────────────── → request.method
         │
         ├── Headers:
         │   Content-Type: application/json
         │   Authorization: Bearer xyz
         │              └──────────────── → request.headers
         │
         ├── Body (form):
         │   username=Arjun&password=secret
         │              └──────────────── → request.form
         │
         ├── Body (JSON):
         │   {"name": "Arjun", "age": 25}
         │              └──────────────── → request.get_json()
         │
         ├── Files:
         │   photo=<binary data>
         │              └──────────────── → request.files
         │
         └── Cookie: username=Arjun
                        └──────────────── → request.cookies
```

---

## 15. Common Mistakes

### Mistake 1: Using `request.form` for JSON data

```python
# ❌ WRONG — this returns None when client sends JSON
username = request.form.get("username")

# ✅ CORRECT — use get_json() for JSON requests
data = request.get_json()
username = data.get("username")
```

---

### Mistake 2: Direct access without `.get()` — crashing on missing fields

```python
# ❌ WRONG — crashes with 400 if "name" not in form
name = request.form["name"]

# ✅ SAFE — returns "" if "name" not in form
name = request.form.get("name", "")
```

---

### Mistake 3: Forgetting `methods=["GET", "POST"]` on the route

```python
# ❌ WRONG — will 405 on POST
@app.route("/login")
def login():
    if request.method == "POST":   # This block never runs!
        ...

# ✅ CORRECT
@app.route("/login", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        ...
```

---

### Mistake 4: Trusting `file.filename` without sanitizing

```python
# ❌ DANGEROUS
file.save(os.path.join("uploads", file.filename))  # Path traversal attack!

# ✅ SAFE
from werkzeug.utils import secure_filename
file.save(os.path.join("uploads", secure_filename(file.filename)))
```

---

### Mistake 5: Forgetting `enctype` on file upload forms

```html
<!-- ❌ WRONG — file won't be in request.files -->
<form method="POST">
    <input type="file" name="photo">
</form>

<!-- ✅ CORRECT -->
<form method="POST" enctype="multipart/form-data">
    <input type="file" name="photo">
</form>
```

---

## 16. Interview Questions

**Q1: What is `request` in Flask and where does it come from?**
A: `request` is a global-like proxy object Flask provides — it's imported from `flask` and automatically contains all data from the current HTTP request (method, form data, JSON, headers, files, cookies).

**Q2: What's the difference between `request.args` and `request.form`?**
A: `request.args` holds data from the URL query string (`?key=value`), available on any method. `request.form` holds data from an HTML form submitted as POST body, only populated when `Content-Type: application/x-www-form-urlencoded`.

**Q3: Why use `request.get_json(silent=True)` instead of `request.json`?**
A: `request.json` raises an exception if the JSON is malformed or the Content-Type header is missing, causing a 500 error. `get_json(silent=True)` returns `None` gracefully, letting you handle the error yourself.

**Q4: How do you safely handle uploaded file names?**
A: Use `werkzeug.utils.secure_filename()` which strips path separators and dangerous characters from the filename before saving.

**Q5: What is the Post/Redirect/Get pattern and why use it?**
A: After a successful POST (form submission), redirect the browser to a GET request. This prevents duplicate form submissions when the user refreshes the page — because refreshing a GET is harmless, but refreshing a POST re-submits the form.

---

## 17. Best Practices

- Always use `.get("key", default)` — never `["key"]` directly — to avoid crashes on missing fields.
- Always validate and sanitize form/JSON data before using it.
- Use `secure_filename()` for any uploaded file names.
- Use the **Post/Redirect/Get** pattern after successful form submissions.
- Use `request.get_json(silent=True)` in API routes — never `request.json` directly.
- Never trust client input — check types, length, and format on every field.

---

## 18. Mini Project — Simple Login System

Build a login system that:
1. `GET /login` → Shows a form with username and password fields
2. `POST /login` → Checks if `username == "admin"` and `password == "1234"`:
   - If correct → return "Welcome Admin!"
   - If wrong → return the form again with error "Invalid credentials" (status 401)
3. `GET /profile?user=Arjun` → Returns "Profile of Arjun" using `request.args`
4. `POST /api/echo` → Receives JSON `{"message": "hello"}` and returns `{"you_said": "hello"}` using `request.get_json()`

---

## 19. Practice Exercises

**5 Easy Questions**
1. Which `request` attribute gives you data from `?name=Arjun` in the URL?
2. Which `request` attribute gives you data submitted via an HTML form POST?
3. True/False: `request.form.get("name")` will crash if "name" is not in the form.
4. What attribute holds uploaded files?
5. What does `request.method` return when a user just visits a page in a browser normally?

**5 Medium Questions**
1. Why must file upload forms have `enctype="multipart/form-data"`?
2. What does `request.form.getlist("hobby")` return if three checkboxes named "hobby" are checked?
3. When would you use `request.get_json(force=True)`?
4. What is `secure_filename()` and why is it critical for file uploads?
5. Why should you redirect after a successful POST instead of returning a response directly?

**5 Hard Questions**
1. A client sends a POST request with `Content-Type: application/json` and a body of `{"name": "Arjun"}`. Will `request.form.get("name")` return "Arjun"? Why or why not?
2. How does Flask's `request` object know which request it belongs to when multiple users are using the app at the same time?
3. What's the difference between `request.data` and `request.get_json()`?
4. A form has two checkboxes both named `agree`. The user checks both. What does `request.form.get("agree")` return vs `request.form.getlist("agree")`?
5. A user sends `?page=abc` to a route that does `page = int(request.args.get("page", 1))`. What happens? How do you fix it?

**2 Debugging Questions**
1. A developer has `@app.route("/submit")` (no `methods=`), and inside it checks `if request.method == "POST"`. When they submit their form they get a 405 error. Fix it.
2. A developer uploads a file and tries `file.save("uploads/" + file.filename)`. It works locally but on a Linux server it creates a file with a strange name like `uploads/../../etc/bad`. What's the vulnerability and fix?

**2 Interview Questions**
1. "Walk me through everything Flask puts in the `request` object for a POST form submission."
2. "What is the Post/Redirect/Get pattern? Why does it matter?"

---

## 20. Summary

| Attribute | What it holds | Use `.get()` safely? |
|---|---|---|
| `request.method` | HTTP method string | N/A — always exists |
| `request.args` | URL query parameters | Yes — `request.args.get("key", default)` |
| `request.form` | HTML form POST data | Yes — `request.form.get("key", default)` |
| `request.get_json()` | Parsed JSON body | N/A — returns `None` if missing/invalid |
| `request.files` | Uploaded files | Check `"key" in request.files` first |
| `request.headers` | HTTP request headers | `request.headers.get("Name")` |
| `request.cookies` | Browser cookies | `request.cookies.get("name")` |
| `request.url` | Full URL string | N/A — always exists |
| `request.remote_addr` | Client IP address | N/A — always exists |

### 💡 Memory Trick
**"Everything the browser sends to you lives inside `request`. Everything you send back to the browser is the `response`."**

---

**End of Module 9.**
