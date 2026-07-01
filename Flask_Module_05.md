# MODULE 5 — Running Flask

> **Course:** Flask — Zero to Production
> **Module:** 5 of 39
> **Builds on:** Modules 1–4

---

## 1. Introduction

You've created the application object (Module 4). Now we need to actually **start a server** so a browser can talk to it. There are two common ways to do this:

1. `app.run()` — calling a method directly inside your Python script.
2. `flask run` — a command-line tool that starts the server *without* you writing `app.run()` at all.

Both ultimately do the same underlying job: start Werkzeug's built-in **development server**, which listens on a port and feeds incoming requests into your Flask app. This module explains both approaches, what **debug mode** really does (it's more than "shows nicer errors"), and why this server is explicitly *not* meant for production.

---

## 2. Why Do We Need This?

Recall from Module 1 that a web server's core job is: open a socket, listen on a port, accept connections, and feed parsed requests into a WSGI application. Someone has to actually *do* that listening. Flask itself doesn't contain socket-handling code — that lives in **Werkzeug**. `app.run()` is essentially a convenience method that says: *"Take Werkzeug's simple development server, point it at me (since I'm a valid WSGI app — Module 4), and start listening."*

Without `app.run()` or `flask run`, you'd have to manually write something like:

```python
from werkzeug.serving import run_simple
run_simple('127.0.0.1', 5000, app)
```

Flask gives you `app.run()` so you don't need to know Werkzeug's serving API directly — but underneath, that's exactly what's happening.

---

## 3. The Problem Before This Existed

Before Werkzeug's development server (and tools like it) existed for Python web apps, getting *any* feedback loop going while developing locally meant either:
- Manually configuring a full production-grade server (like Apache with `mod_python`) just to see a single test page — heavy and slow to iterate with.
- Writing your own minimal socket server from scratch for local testing — error-prone and a distraction from actually building your app.

The development server solves this by giving you a **zero-configuration, instantly-startable server** purpose-built for local iteration — with features like auto-reloading on code changes and an interactive debugger, which a production server would never include (for good security reasons we'll get to).

---

## 4. Real-Life Analogy

Think of the development server like a **flight simulator** versus a **real airplane**. The simulator (dev server) is fast to start, forgiving of mistakes, shows you detailed diagnostic information when something goes wrong, and lets you "rewind" easily. A real airplane (production server, like Gunicorn — Module 31) is built for the real world: efficient, hardened, and absolutely not designed to show a stranger your cockpit's internal wiring if something breaks mid-flight.

You'd never fly paying passengers in a flight simulator — and you should never serve real internet traffic with the development server.

---

## 5. Internal Working

### 5.1 What `app.run()` Does, Step by Step

```python
app.run(debug=True)
```

Internally, this:
1. Reads the parameters you passed (`host`, `port`, `debug`, etc.), falling back to sensible defaults (`host="127.0.0.1"`, `port=5000`, `debug=False`).
2. If `debug=True`, enables two distinct features (often confused as one):
   - **The reloader** — a separate watcher process that monitors your `.py` files and automatically restarts the server when it detects a change, so you don't have to manually stop/start it after every edit.
   - **The interactive debugger** — when an unhandled exception occurs, instead of just showing a generic error, Werkzeug renders a detailed, **interactive** traceback page directly in the browser, including a live Python console at each stack frame.
3. Calls Werkzeug's `run_simple(host, port, self, ...)`, passing **itself** (`self`, the Flask app object) as the WSGI application to serve.
4. Werkzeug opens a TCP socket, binds it to the given host/port, and enters a loop: accept a connection → parse the HTTP request into a WSGI `environ` → call `app(environ, start_response)` → take the returned response → write it back to the socket → repeat.

### 5.2 What `flask run` Does Differently

The `flask` command-line tool is installed automatically when you `pip install flask` (recall the `click` dependency from Module 2 — it powers this CLI). Running:

```bash
flask run
```

does the same underlying job as `app.run()`, but it needs to first **find your application object**. It does this via an environment variable:

```bash
export FLASK_APP=app.py      # macOS/Linux
set FLASK_APP=app.py         # Windows cmd
$env:FLASK_APP="app.py"      # Windows PowerShell
```

`flask run` reads `FLASK_APP`, imports that module, looks for a variable named `app` (or a factory function — Module 20) inside it, and starts the server using that object — **without requiring an `if __name__ == "__main__": app.run()` block in your file at all.**

### 5.3 Diagram — Two Paths, Same Destination

```
            (Path A)                      (Path B)
    python app.py                     flask run
        │                                 │
   if __name__=="__main__"          reads FLASK_APP env var
        │                                 │
   app.run(...) called               imports that module,
        │                            finds the app object
        ▼                                 ▼
        └──────────► Werkzeug's run_simple() ◄──────────┘
                              │
                    Opens TCP socket, listens,
                    feeds requests into app(environ, start_response)
```

---

## 6. Syntax

### 6.1 `app.run()` Parameters

```python
app.run(host="127.0.0.1", port=5000, debug=False, threaded=False, ssl_context=None)
```

| Parameter | Meaning |
|---|---|
| `host` | The IP address to bind to. `"127.0.0.1"` (localhost-only) by default; `"0.0.0.0"` makes it reachable from other devices on your network. |
| `port` | The TCP port to listen on. `5000` by default. |
| `debug` | Enables the reloader + interactive debugger when `True`. |
| `threaded` | If `True`, allows the dev server to handle multiple requests concurrently using threads, rather than one at a time. |
| `ssl_context` | Lets you serve over HTTPS locally with a certificate, for testing secure-cookie behavior, etc. |

### 6.2 `flask run` Command Options

```bash
flask run --host=0.0.0.0 --port=8000 --debug
```

| Flag | Meaning |
|---|---|
| `--host` | Same meaning as `app.run()`'s `host`. |
| `--port` | Same meaning as `app.run()`'s `port`. |
| `--debug` | Equivalent to `debug=True` — enables reloader + debugger. |

---

## 7. Small Example

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Server is running!"

if __name__ == "__main__":
    app.run(debug=True)
```

Run it with:

```bash
python app.py
```

You'll see terminal output similar to:

```
 * Serving Flask app 'app'
 * Debug mode: on
 * Running on http://127.0.0.1:5000
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 123-456-789
```

---

## 8. Step-by-Step Explanation

**`if __name__ == "__main__":`**
As covered in Module 3, this ensures the code inside only runs when this file is executed directly, not when imported elsewhere (e.g., by a test file that imports `app` for testing without wanting to start a real server).

**`app.run(debug=True)`**
Triggers the full sequence described in Section 5.1: reads defaults for `host`/`port`, enables the reloader and interactive debugger, and calls Werkzeug's `run_simple()` with the `app` object itself.

**Terminal output line: `* Debug mode: on`**
Confirms the reloader and debugger are both active — this is your visual signal that this is a development run, not production.

**Terminal output line: `* Restarting with stat`**
Indicates the reloader is using the `stat` system call to periodically check file modification times, to detect when you've saved a code change.

**Terminal output line: `* Debugger PIN: 123-456-789`**
This PIN is a real security feature — to use the interactive in-browser Python console (which lets you literally execute arbitrary Python code at the point of failure), you must enter this PIN first. This single line is *why* `debug=True` is dangerous on a public-facing server: an attacker who can trigger an exception and figure out (or brute-force, in misconfigured cases) this PIN could potentially execute arbitrary code on your machine.

### Program Flow

```
python app.py
   ↓
__name__ == "__main__" → True
   ↓
app.run(debug=True)
   ↓
Reloader process starts watching files
   ↓
Werkzeug's run_simple() opens socket on 127.0.0.1:5000
   ↓
Browser sends GET /
   ↓
Werkzeug parses request → builds WSGI environ
   ↓
app(environ, start_response) called  (Flask's __call__, Module 4)
   ↓
Flask matches "/" → calls home()
   ↓
Response sent back through Werkzeug → browser
```

---

## 9. Visual Diagram

```
   Browser
      │  GET http://127.0.0.1:5000/
      ▼
 ┌─────────────────────────────┐
 │ Werkzeug Dev Server          │
 │  - listening on 127.0.0.1:5000│
 │  - reloader watching files    │
 │  - debugger ready on error    │
 └──────────────┬───────────────┘
                │ environ dict
                ▼
 ┌─────────────────────────────┐
 │        Flask app object      │
 │  matches route, runs view fn │
 └──────────────┬───────────────┘
                │ response
                ▼
            Back to Browser
```

---

## 10. Common Mistakes

1. **Using `host="0.0.0.0"` with `debug=True` on a network anyone else can reach** — this exposes the interactive debugger (and its remote-code-execution risk) to anyone on that network.
2. **Believing `debug=True` is just "nicer error messages."** It also enables the reloader, which can behave oddly with certain background threads or global state that gets initialized twice (because the reloader actually runs your script in two processes — the watcher and the real one).
3. **Forgetting to set `FLASK_APP`** before running `flask run`, leading to a "could not locate a Flask application" error.
4. **Trying to handle real, concurrent production traffic with `app.run()`**, even with `threaded=True` — it's still not designed or hardened for that role (Module 31 covers proper production servers).
5. **Hardcoding `debug=True` and forgetting to remove it before deployment** — one of the most common real-world Flask security incidents.

> **⚠️ Warning:** Never set `debug=True` (or equivalently `FLASK_DEBUG=1`) in any environment reachable by the public internet. This is not a style preference — it is a documented, serious security risk (potential remote code execution via the interactive debugger).

---

## 11. Interview Questions (with Answers)

**Q1: What two distinct features does `debug=True` actually enable?**
A: The auto-reloader (restarts the server on code changes) and the interactive in-browser debugger (detailed tracebacks with a live Python console).

**Q2: How does `flask run` find your application without you calling `app.run()`?**
A: It reads the `FLASK_APP` environment variable, imports that module, and looks for an `app` object (or an application factory function) inside it.

**Q3: Why is the development server unsuitable for production?**
A: It isn't designed or hardened for handling real concurrent traffic securely and efficiently, and debug mode (if accidentally left on) introduces a serious remote-code-execution risk via the interactive debugger.

**Q4: What is the Debugger PIN, and why does it exist?**
A: A security code required to use the interactive debugger's live console — meant to prevent anyone who simply triggers an error from automatically getting code-execution access.

**Q5: What's the underlying library actually providing the development server Flask uses?**
A: Werkzeug — specifically its `run_simple()` function.

---

## 12. Best Practices

- Use `debug=True` (or `flask run --debug`) **only** in local development, on `127.0.0.1`.
- Use environment variables or config files (Module 19) to control debug mode, rather than hardcoding it — so it's easy to guarantee it's off in production.
- Prefer `flask run` with `FLASK_APP` set via a `.flaskenv` file (loaded automatically if you install `python-dotenv`) for a clean, script-free way to run your app during development.
- When you need to test how your app behaves under multiple simultaneous requests locally, use `threaded=True` — but remember this still isn't a production-grade concurrency model.

---

## 13. Real-World Usage

In real teams, `app.run()` essentially never appears in production code paths at all — production deployments use a WSGI server like Gunicorn (`gunicorn myapp:app`) or uWSGI, sitting behind a reverse proxy like Nginx (all covered in depth in Module 31). `app.run()`/`flask run` exist purely for **local development and quick testing** — that's their entire intended scope.

---

## 14. Mini Project

> Take your Module 4 mini-project app and run it two different ways: once via `python app.py` (using `app.run(debug=True)`), and once via `flask run` (after setting `FLASK_APP`). Confirm both reach the same page in your browser. Then deliberately introduce a bug (e.g., reference an undefined variable inside your view function) and observe the interactive debugger page that appears — note what information it shows you (file, line number, local variables at each frame).

---

## 15. Practice Exercises

**5 Easy Questions**
1. What is the default port `app.run()` uses?
2. What environment variable does `flask run` rely on to find your app?
3. True/False: `debug=True` is safe to use on a publicly reachable server.
4. What two features does debug mode enable?
5. Which library actually implements the development server's socket-handling code?

**5 Medium Questions**
1. Why does the reloader sometimes cause code to run twice on startup?
2. What does `host="0.0.0.0"` change compared to the default `"127.0.0.1"`?
3. What's the security purpose of the Debugger PIN?
4. Why isn't `threaded=True` enough to make the dev server production-ready?
5. What's the practical difference between running via `python app.py` vs `flask run`?

**5 Hard Questions**
1. Explain, step by step, what happens internally between calling `app.run()` and a browser actually receiving a response.
2. Why is "shows nicer error messages" an incomplete description of what `debug=True` does?
3. If the reloader detects a file change mid-request, what risk might that introduce, conceptually?
4. Why does `flask run` need `FLASK_APP` to be set, when `app.run()` doesn't need any equivalent environment variable?
5. From a security standpoint, why is an "interactive Python console reachable via a web error page" inherently risky, regardless of a PIN?

**2 Debugging Questions**
1. A developer runs `flask run` and gets "Error: Could not locate a Flask application." What's the most likely missing step?
2. A teammate's app behaves correctly the first time they save a change, but on the *second* save, the server seems to hang or double-print startup logs. What dev-server feature is the likely cause, and why?

**2 Interview Questions**
1. "What exactly happens internally when you call `app.run(debug=True)`?"
2. "Why should the Flask development server never be used in production, beyond just 'it's slow'?"

**1 Mini Project**
See Section 14 above.

---

## 16. Summary

| Concept | Key Takeaway |
|---|---|
| `app.run()` | Convenience method that calls Werkzeug's `run_simple()`, passing the Flask app as the WSGI application |
| `flask run` | CLI alternative that locates your app via the `FLASK_APP` environment variable |
| `debug=True` | Enables BOTH the auto-reloader AND the interactive debugger |
| Debugger PIN | A security gate in front of the live, in-browser Python console |
| Production readiness | The dev server is never appropriate for real, public traffic |

### Comparison Table — Development Server vs Production Server

| Aspect | Development Server (`app.run()`/`flask run`) | Production Server (Gunicorn, etc. — Module 31) |
|---|---|---|
| Purpose | Local iteration and testing | Serving real traffic |
| Concurrency model | Minimal/single-threaded by default | Robust worker/process models |
| Debug info on error | Full interactive traceback (if enabled) | Generic error responses only |
| Security hardening | Minimal | Production-grade |
| Auto-reload on code change | Yes (with debug mode) | No (intentionally) |

### 💡 Memory Trick
**"Debug mode doesn't just turn on a flashlight — it opens a door. Never leave that door open on a server strangers can reach."**

### ❓ FAQ
- **"Can I use `flask run` without setting `FLASK_APP` manually every time?"** Yes — many developers create a `.flaskenv` file with `FLASK_APP=app.py` inside it, combined with the `python-dotenv` package, so it's loaded automatically.
- **"Does `app.run()` work fine for showing my project to a friend on the same Wi-Fi?"** Technically yes, with `host="0.0.0.0"` and debug mode **off** — but this is still not the same as real production deployment, and should be temporary/trusted-network-only.

---

**End of Module 5.**
