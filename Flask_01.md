# MODULE 1 — Introduction to Flask

> **Course:** Flask — Zero to Production
> **Module:** 1 of 39
> **Prerequisite:** Comfortable with core Python (functions, classes, imports, decorators-as-a-concept)

---

## 1. Introduction

**Flask** is a *web framework* written in Python. Before we even touch Flask's syntax, we need to be crystal clear on two words: "web" and "framework."

- A **web application** is just a program that talks to a browser (or any HTTP client) over the internet using the **HTTP protocol** — it receives a *request* and sends back a *response*.
- A **framework** is a pre-built skeleton of code that handles the repetitive, error-prone plumbing of a task, so you only write the *business logic* — the part that's unique to your app.

Flask specifically is called a **micro-framework**. "Micro" does **not** mean "limited" or "toy." It means Flask makes almost *zero* decisions for you. It does not ship with a built-in database layer, a built-in form validator, or a built-in authentication system. It gives you:

1. **Routing** — mapping a URL to a Python function.
2. **Request/Response handling** — reading what the browser sent, and sending something back.
3. **Templating** — generating dynamic HTML (via Jinja2).
4. **A development server** — so you can test locally.

Everything else (databases, login systems, REST tooling) is added later through **extensions** — separate packages that plug into Flask. This is a deliberate design philosophy: *"give the developer Lego bricks, not a finished castle."*

Flask is built on top of two other libraries created by the same author (Armin Ronacher):
- **Werkzeug** — a WSGI utility library that does the actual heavy lifting of parsing HTTP requests, building responses, and routing.
- **Jinja2** — a templating engine that lets you embed Python-like logic inside HTML files.

Flask itself is best thought of as a **thin, elegant wrapper** around Werkzeug + Jinja2, glued together with a clean, decorator-based API.

---

## 2. Why Do We Need This?

To answer "why Flask," we first have to answer "why *any* web framework." Let's reason from first principles.

A web server, at its core, is a program that:
1. Opens a network **socket** and listens on a port (e.g., port 5000 or 80).
2. Accepts incoming **TCP connections**.
3. Reads raw bytes off that connection.
4. Parses those bytes according to the **HTTP specification** (method, path, headers, body).
5. Decides what to do based on the path (e.g., `/about` should show the About page).
6. Builds a response: a status line (`200 OK`), headers (`Content-Type: text/html`), and a body.
7. Writes those bytes back onto the socket.
8. Closes or keeps the connection alive, and loops back to step 2.

If you had to do this manually for every Python web project, you would be re-writing the same ~500 lines of socket-and-parsing code every single time, and one small bug (e.g., mis-parsing a header) could break your entire application or open a security hole. Flask exists so that **steps 1, 2, 3, 4, 6, and 7 are handled for you**, and you only write the *decision logic* in step 5 — "what should happen when someone visits `/about`?"

**In one sentence:** Flask exists so you can focus on *what your application should do*, not *how raw bytes travel over a network*.

---

## 3. The Problem Before Frameworks Like Flask Existed

This isn't just theoretical — it's actual web history, and understanding it tells you *why* Flask is shaped the way it is.

| Era | Approach | Problem |
|---|---|---|
| Early 1990s | **CGI (Common Gateway Interface)** | Every single HTTP request spawned a brand-new OS process to run your script. Extremely slow under load — process creation is expensive. |
| Mid-1990s–2000s | **Server-specific APIs** (e.g., `mod_python` for Apache) | Faster than CGI (no new process per request), but your code became tightly coupled to *one specific web server*. Code written for Apache wouldn't run on another server without rewriting. |
| 2003 | **WSGI (Web Server Gateway Interface) — PEP 333** | A *standard contract* was defined: any Python web application that follows this contract can run on *any* WSGI-compliant server. This decoupled "the application" from "the server" for good. |
| 2004 onward | **Frameworks built on WSGI** (Django 2005, Flask 2010) | Frameworks could now be written once and deployed on many different servers (Gunicorn, uWSGI, Waitress, etc.) without modification. |

Flask was created in 2010, specifically as a *lightweight* alternative to the already-popular but "batteries-included" Django. The unspoken problem it solved was: *"What if I don't want an ORM, an admin panel, and a templating engine forced on me — I just want routing and templates, and I'll choose the rest myself?"*

---

## 4. Real-Life Analogy

Imagine you want to open a restaurant.

- **Raw sockets/no framework** = You buy an empty plot of land. You must lay your own electrical wiring, plumbing, and gas lines before you can even think about cooking. Most of your time goes into infrastructure, not food.
- **Django (a "full-stack" framework)** = You rent a fully furnished restaurant — kitchen equipment, menu templates, a built-in ordering system, a built-in cashier system, all installed and configured. Fast to start, but you must work within the layout that's already there.
- **Flask (a micro-framework)** = You rent a bare commercial kitchen space with electricity, water, and gas already connected (that's Werkzeug doing the HTTP wiring for you), but **no** furniture, **no** menu system, **no** cashier. You bring in exactly the equipment (extensions) you want — your own oven brand, your own POS system — and arrange the kitchen exactly how you like.

This is why Flask is loved for small-to-medium projects, APIs, and microservices: you only add what you actually need.

---

## 5. Internal Working — What Is Flask *Actually* Made Of?

This is the part most beginner tutorials skip — let's not skip it.

### 5.1 The WSGI Contract

WSGI defines that a Python web application must be a **callable** (a function or an object with `__call__`) with this exact signature:

```python
def application(environ, start_response):
    # environ: a dict containing all request data (method, path, headers, etc.)
    # start_response: a function you call to set the status code and headers
    start_response('200 OK', [('Content-Type', 'text/plain')])
    return [b'Hello, World!']
```

That's it. That is the *entire* foundation that every Python web framework (Flask, Django, FastAPI's underlying ASGI cousin, etc.) is ultimately built on top of. A web server (like Werkzeug's dev server, or Gunicorn in production) calls this function for every incoming request.

### 5.2 Where Flask Fits In

When you write:

```python
app = Flask(__name__)
```

You are creating an instance of the `Flask` class. This object is itself a **WSGI application** — meaning it implements `__call__`, so it can be invoked exactly like the `application` function above. Internally:

1. `Flask.__call__(environ, start_response)` is triggered by the server for every request.
2. It delegates to `Flask.wsgi_app(environ, start_response)`.
3. `wsgi_app` builds a **Request object** (using Werkzeug) from the raw `environ` dict — this gives you the friendly `request.form`, `request.args`, etc. you'll use later.
4. It pushes a **Request Context** and an **Application Context** onto an internal stack (we cover this deeply in a later module — for now, just know: Flask needs to "remember" *which request is currently being handled* even though many requests can be in flight at once).
5. It asks Werkzeug's **URL routing system** ("the Map") which Python function should handle this URL.
6. It calls that function (your *view function*) and gets back a return value.
7. It converts that return value into a proper **Response object**.
8. It calls `start_response(...)` with the status and headers, and returns the body — fulfilling the WSGI contract.
9. It pops the contexts off the stack, cleaning up.

### 5.3 Diagram — The Layers

```
   ┌───────────────────────────────────┐
   │         Your Application          │   <- routes, view functions, templates
   ├───────────────────────────────────┤
   │              Flask                │   <- decorators, context, request/response glue
   ├────────────────────┬──────────────┤
   │      Werkzeug       │    Jinja2   │   <- HTTP parsing, routing | HTML templating
   ├────────────────────┴──────────────┤
   │               WSGI                │   <- the universal contract (PEP 3333)
   ├───────────────────────────────────┤
   │   Werkzeug Dev Server / Gunicorn  │   <- the actual server listening on a port
   ├───────────────────────────────────┤
   │      Operating System (TCP/IP)    │
   └───────────────────────────────────┘
```

> **Important Note:** Flask does not reinvent HTTP parsing or routing logic — it borrows Werkzeug's. Flask's real contribution is *developer experience*: decorators, contexts, and a clean API on top of Werkzeug's lower-level pieces.

---

## 6. Syntax (Preview Only)

We will dissect this line-by-line in **Module 5 (Running Flask)** and **Module 6 (Routing)**. For now, just look at the shape of a Flask app:

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Hello, Flask!"
```

Quick preview (not the full deep-dive yet):

| Line | What it is (brief) |
|---|---|
| `from flask import Flask` | Imports the `Flask` *class* from the `flask` package. |
| `app = Flask(__name__)` | Creates one instance of your application — the central object. |
| `@app.route("/")` | A decorator that tells Flask: "when someone visits `/`, run the function below." |
| `def home():` | A *view function* — the code that runs for that URL. |
| `return "Hello, Flask!"` | Whatever you return becomes the HTTP response body. |

We'll explain *exactly* what `__name__` does, how `@app.route` works internally (it modifies Werkzeug's URL Map), and what happens to that return string between Module 5 and 7.

---

## 7. Small Example

Since Module 1 is conceptual, here's the smallest *mental model* example rather than runnable code — picture WSGI without Flask:

```python
def application(environ, start_response):
    path = environ.get('PATH_INFO', '/')
    if path == '/':
        body = b'Welcome Home'
    elif path == '/about':
        body = b'About Us'
    else:
        start_response('404 Not Found', [('Content-Type', 'text/plain')])
        return [b'Not Found']
    start_response('200 OK', [('Content-Type', 'text/plain')])
    return [body]
```

This is literally what Flask is sparing you from writing by hand — manual `if/elif` path matching, manual status strings, manual byte encoding.

---

## 8. Step-by-Step Explanation (The Request/Response Journey)

Here is what happens, end to end, the moment a user visits your Flask site in a browser. We will go deeper into each box in later modules, but you should be able to picture this whole journey *right now*.

1. **Browser** sends an HTTP request: `GET / HTTP/1.1` plus headers.
2. The request travels over TCP/IP to your server's IP address and port.
3. The **web server** (Werkzeug's dev server locally, or Gunicorn in production) accepts the raw bytes.
4. The server constructs the WSGI `environ` dictionary and calls your Flask `app` (because `app` is a WSGI callable).
5. Flask builds a **Request object**, pushes the **Request Context**.
6. Flask's router (Werkzeug's `Map`/`MapAdapter`) matches the path `/` to your `home()` function.
7. Flask calls `home()`.
8. `home()` returns a value (a string, in our example).
9. Flask converts that into a **Response object** with a default status `200 OK` and `Content-Type: text/html`.
10. Flask pops the contexts, hands the response back to the server.
11. The server sends the bytes back over the socket.
12. The **browser** receives them and renders the page.

---

## 9. Visual Diagram

```
 Browser
    │
    │  GET /
    ▼
 ┌─────────────────────────────┐
 │   Web Server (Werkzeug /    │
 │   Gunicorn) — listens on    │
 │   a TCP port                │
 └──────────────┬──────────────┘
                │ builds WSGI environ
                ▼
 ┌─────────────────────────────┐
 │           Flask              │
 │  1. Build Request object     │
 │  2. Push Request Context     │
 │  3. Match URL → view func    │
 │  4. Call view function       │
 │  5. Build Response object    │
 └──────────────┬──────────────┘
                │ HTTP response bytes
                ▼
            Browser renders page
```

---

## 10. Common Mistakes (Beginner Pitfalls)

1. **Thinking Flask = a database.** Flask has no idea what a database is. You add SQLAlchemy or another tool separately (Module 21–22).
2. **Thinking Flask = Django, just smaller.** They have different philosophies — Flask gives you choice, Django gives you convention. Neither is "smaller Django."
3. **Running the development server in production.** The built-in server (`app.run()`) is single-threaded by default, not hardened for security, and explicitly says in its own startup banner that it's a dev server. Production needs Gunicorn/Waitress + a real web server like Nginx (Module 31).
4. **Confusing WSGI with HTTP.** WSGI is *not* a network protocol — it's a Python-level calling convention between a server and an app. HTTP is the actual protocol that travels over the network.
5. **Assuming Flask automatically secures your app.** CSRF, XSS, SQL injection protection — none of these are automatic. You must deliberately add them (we'll cover each in the Security sections of later modules).

> **⚠️ Warning:** Never deploy `app.run(debug=True)` to a public server. Debug mode can expose an interactive Python debugger to anyone who can trigger an error — this can lead to **remote code execution**. We'll dig into this in the Security section of Module 31 (Deployment).

---

## 11. Interview Questions (with Answers)

**Q1: What is Flask?**
A: A lightweight Python web framework (a "micro-framework") built on Werkzeug (WSGI toolkit) and Jinja2 (templating engine), used to build web applications and APIs.

**Q2: Why is Flask called a "micro" framework?**
A: Not because it's small in capability, but because it doesn't make architectural decisions for you (no built-in ORM, no built-in form validation, no built-in admin panel). You add these via extensions as needed.

**Q3: What is WSGI, and why does Flask need it?**
A: WSGI (Web Server Gateway Interface) is a standard Python interface (PEP 3333) between web servers and web applications. Flask needs it so that the same Flask app can run unmodified on different servers (the built-in dev server, Gunicorn, uWSGI, etc.).

**Q4: Is Flask faster than Django?**
A: "Faster" depends on the metric. Flask has less overhead per request because it does less by default, but a fair comparison depends on what features Django's equivalent app would also need (which Flask would need to add via extensions, adding its own overhead back).

**Q5: Name the two core libraries Flask is built on.**
A: Werkzeug (WSGI utilities, routing, request/response objects) and Jinja2 (templating engine).

---

## 12. Best Practices (Foundational, Pre-Installation)

- Always develop inside a **virtual environment** (covered fully in Module 2) — never install Flask globally.
- Understand *what* Flask gives you natively versus what needs an extension, before reaching for a third-party package — sometimes you don't need one.
- Read error messages fully; Flask's debug mode gives you a full traceback for a reason.
- Treat the development server as **strictly local** — never expose it to the public internet.

---

## 13. Real-World Usage

Flask is widely used for:
- **Internal tools and dashboards** at many tech companies, because it's quick to stand up.
- **Microservices and REST APIs**, where a small, focused service doesn't need Django's full toolkit.
- **Prototypes and MVPs**, where speed of iteration matters more than built-in structure.
- **Data science / ML model serving**, where a lightweight API in front of a Python model (e.g., using `scikit-learn` or `PyTorch`) is a very common pattern.

Companies in the Python ecosystem (e.g., teams at Netflix, Reddit, Airbnb, and Lyft) have publicly discussed using Flask for some of their internal or service-level Python applications — typically not their entire monolith, but specific services where its lightness was a good fit.

---

## 14. Mini "Project" for This Module (Conceptual — No Code Yet)

Since we haven't installed Flask yet (that's Module 2), this module's mini-project is a **research + reasoning task**:

> Pick any website or app you use daily. Write 5–8 sentences hypothesizing: would Flask be a good fit to build its *backend*, or would something like Django or FastAPI be a better fit? Justify using the concepts from this module (micro vs full-stack, WSGI, extensibility).

---

## 15. Practice Exercises

**5 Easy Questions**
1. What does "WSGI" stand for?
2. What is the name of the templating engine Flask uses?
3. True/False: Flask includes a built-in database system.
4. What is the term for the smallest unit of code that handles a specific URL in Flask?
5. Name one library Flask is built directly on top of.

**5 Medium Questions**
1. Explain in your own words why CGI was slow for handling many requests.
2. What problem did the WSGI standard specifically solve?
3. Why is Flask described as giving the developer "choice" rather than "convention"?
4. What is the difference between a web *server* and a web *framework*?
5. Why shouldn't you run Flask's development server in production?

**5 Hard Questions**
1. Walk through, in your own words, every step that happens between a browser sending `GET /` and Flask calling your view function.
2. Why is it accurate to say "Flask *is* a WSGI application" rather than "Flask *uses* WSGI"?
3. If WSGI didn't exist, what specific coupling problem would re-emerge between frameworks and servers?
4. Werkzeug handles routing — what, specifically, does Flask add on top of that raw routing capability?
5. Explain why "micro-framework" is a description of philosophy, not capability.

**2 Debugging Questions**
1. A teammate says: "I called my WSGI app and it worked, but the browser shows a blank page with no error." Given the WSGI contract `(environ, start_response)`, what's a likely root cause to investigate first?
2. Another teammate's WSGI function returns a Python `str` directly instead of a list of `bytes` objects. Based on the WSGI spec, why might this fail or behave unexpectedly on some servers?

**2 Interview Questions**
1. "Compare Flask and Django at a philosophical level — not feature by feature, but in terms of design philosophy."
2. "If you were choosing between Flask and Django for a brand-new project, what three questions would you ask before deciding?"

**1 Mini Project**
See Section 14 above.

---

## 16. Summary

| Concept | Key Takeaway |
|---|---|
| Flask | A Python micro-framework for building web apps/APIs |
| Built on | Werkzeug (WSGI/routing) + Jinja2 (templates) |
| WSGI | The standard contract that lets any Python app run on any compatible server |
| Philosophy | Minimal core, extend only what you need |
| Not included by default | Database layer, form validation, admin panel, auth system |
| Dev server | For local development only — never production |

### Quick Comparison Table — Flask vs Django vs FastAPI

| Feature | Flask | Django | FastAPI |
|---|---|---|---|
| Philosophy | Minimal, flexible | Batteries-included, opinionated | Modern, async-first, type-hint driven |
| Built-in ORM | No | Yes (Django ORM) | No |
| Built-in Admin Panel | No | Yes | No |
| Async support | Limited (improving) | Limited (improving) | Native (built on ASGI) |
| Templating | Jinja2 | Django Template Language | Usually Jinja2 too |
| Best for | Small–medium apps, APIs, microservices | Large apps needing structure fast | High-performance async APIs |
| Underlying spec | WSGI | WSGI (ASGI optionally) | ASGI |

### 💡 Memory Trick
Think **"Flask = Werkzeug's plumbing + Jinja2's paint + your own furniture."** Werkzeug handles the pipes (HTTP), Jinja2 handles the look (HTML), and *you* decide everything else (database, auth, structure).

### ❓ FAQ
- **"Can Flask build large applications?"** Yes — with good structure (Blueprints, Application Factory — Modules 18 & 20), Flask scales to large codebases. It just doesn't enforce that structure for you.
- **"Do I need to know WSGI to use Flask?"** No, not to get started — but understanding it (as we did here) means you'll never be confused about *why* Flask code is shaped the way it is, and it makes debugging deployment issues far easier later.

---

**End of Module 1.**
