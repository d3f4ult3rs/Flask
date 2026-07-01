# MODULE 3 — Project Structure

> **Course:** Flask — Zero to Production
> **Module:** 3 of 39
> **Builds on:** Module 1 (Introduction), Module 2 (Installing Flask)

---

## 1. Introduction

Before we write a single route, we need to talk about **how to organize the files of a Flask project.** This might feel premature — "I don't even have any code yet!" — but project structure is exactly the kind of thing that's painless to set up correctly from day one, and painful to retrofit after your app has grown to 40 files.

Flask, true to its "micro-framework" philosophy from Module 1, **does not force a folder structure on you.** Django says "your project *must* look like this." Flask says "here are some conventions people commonly use — pick what fits." That freedom is powerful, but it also means beginners often either (a) dump everything into one giant file, or (b) over-engineer a tiny project with folders it doesn't need yet.

This module gives you a clear, practical decision framework: what structure to use for a tiny app, and how that naturally evolves into a structure that scales to a large app — without a painful rewrite later.

---

## 2. Why Do We Need This?

Imagine writing every single view function, every database model, every configuration value, and every helper function in **one file** called `app.py`. For a "Hello World" app, this is totally fine — even ideal, since splitting it up would just be unnecessary ceremony.

But now imagine that file growing to contain:
- 30 routes
- Database model definitions
- Authentication logic
- Email-sending helper functions
- Configuration for 3 different environments (development, testing, production)

At that point, `app.py` becomes a 2,000-line file where finding anything requires `Ctrl+F`, two developers editing different "sections" of the same file constantly create Git merge conflicts, and there's no clear boundary between "this is a route" vs. "this is a database model" vs. "this is configuration."

**Project structure exists to create boundaries** — physical, file-based boundaries that mirror logical boundaries in your application's responsibilities. Good structure is what lets a codebase grow from 1 file to 100 files without becoming unmanageable.

---

## 3. The Problem Before Conventions Existed

In the earliest days of dynamic web scripting (think raw PHP or early CGI scripts), it was extremely common to see **one script per URL**, with HTML, business logic, and even raw SQL queries mixed together in the same file, repeated with copy-pasted boilerplate across dozens of files. There was no shared structure to separate "what handles a request" from "what renders a page" from "what talks to the database" — this mixing is sometimes called **"spaghetti code"**, precisely because tracing a piece of logic meant following tangled threads across many files with no consistent pattern.

The web development community gradually converged on patterns like **MVC (Model-View-Controller)** and its many variants to solve this — establishing that a well-organized app should clearly separate:
- **Data/Models** (what your data looks like, e.g., a `User`)
- **Views/Templates** (what the user sees)
- **Controllers/Routes** (the logic deciding what to show, in response to a request)

Flask doesn't force MVC by name, but its conventions (a `templates/` folder, a `static/` folder, separate files for routes vs. models) are clearly inspired by this same separation-of-concerns idea.

---

## 4. Real-Life Analogy

Think of project structure like organizing a **household**. A studio apartment (tiny Flask app) can reasonably have one room that's bedroom + kitchen + living room combined — adding walls would be silly for one person living alone.

But a family of six living in a house needs **separate rooms with clear purposes**: a kitchen for cooking, bedrooms for sleeping, a living room for socializing. If you tried to keep "one big combined room" as the family grew, mornings would be chaos — someone cooking breakfast while someone else is trying to sleep, with no walls to separate the activities.

Flask project structure is exactly this: small apps get one room (`app.py`); growing apps get walls (separate files/folders) added exactly where the chaos would otherwise begin.

---

## 5. Internal Working — How Flask "Discovers" Your Structure

This is important and often glossed over: **Flask doesn't scan your whole project to "find" your structure.** Flask only knows about three structural things automatically, and only because you tell it (directly or via convention):

1. **The `Flask(__name__)` call** — Flask uses the `__name__` of your module to figure out the **root path** of your application. This root path is the anchor point Flask uses to look for the next two things.
2. **The `templates/` folder** — by default, Flask looks for a folder literally named `templates` *sitting next to* your main application file (or package), and registers it as the place `render_template()` should search.
3. **The `static/` folder** — similarly, Flask looks for a folder literally named `static` next to your app, and automatically creates a route (`/static/<filename>`) that serves files from it.

Everything else — how you organize your routes, models, helper functions — is **entirely up to you**. Flask has no opinion. The "recommended structures" you'll see below are community convention, not a Flask requirement.

### 5.1 Diagram — What Flask Actually Looks For

```
my_project/
├── app.py              <- Flask(__name__) is created here
├── templates/          <- Flask AUTOMATICALLY looks here for render_template()
│   └── index.html
└── static/             <- Flask AUTOMATICALLY serves this at /static/...
    └── style.css
```

Anything outside `templates/` and `static/` (like a `models.py` or a `routes/` folder) is something **you** decide to create and **you** decide to `import` into `app.py` — Flask has zero built-in awareness of it.

---

## 6. Syntax — Two Common Structures

### 6.1 Structure A — Single-File App (small projects)

```
my_project/
├── venv/
├── app.py
├── templates/
│   └── index.html
├── static/
│   └── style.css
└── requirements.txt
```

This is perfectly appropriate for a learning project, a small internal tool, or anything with fewer than ~10 routes and no real data layer yet.

### 6.2 Structure B — Package-Based App (growing/production projects)

```
my_project/
├── venv/
├── requirements.txt
├── run.py                     <- the file you actually execute to start the app
└── app/                        <- this is now a PYTHON PACKAGE (note: no .py extension)
    ├── __init__.py             <- creates the Flask app (the "Application Factory" — Module 20)
    ├── routes.py               <- view functions
    ├── models.py               <- database models
    ├── config.py               <- configuration classes/values
    ├── templates/
    │   ├── base.html
    │   └── index.html
    └── static/
        ├── css/
        └── js/
```

We will build *into* this exact structure starting around Module 19–20, so don't worry about memorizing it perfectly yet — just recognize the shape.

---

## 7. Small Example

**`app.py` (Structure A):**

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route("/")
def home():
    return render_template("index.html")

if __name__ == "__main__":
    app.run(debug=True)
```

**`templates/index.html`:**

```html
<!DOCTYPE html>
<html>
<head><title>My App</title></head>
<body>
    <h1>Hello from a template file!</h1>
</body>
</html>
```

---

## 8. Step-by-Step Explanation

**`from flask import Flask, render_template`**
Imports two names from the `flask` package: the `Flask` class (to create the app object) and the `render_template` function (to find and render an HTML file from the `templates/` folder). Why both here: this example demonstrates the *connection* between your Python code and your templates folder — the whole point of this module.

**`app = Flask(__name__)`**
Creates the central application object. The `__name__` argument is what lets Flask compute the root path of your project on disk, which it then uses to locate the `templates/` and `static/` folders *relative to this file's location* — this is the direct link back to Section 5's explanation.

**`@app.route("/")`**
Registers the function below as the handler for the root URL `/`. (Full deep-dive in Module 6.)

**`def home():`**
The view function — the code that decides what happens for this URL.

**`return render_template("index.html")`**
Calls `render_template`, passing the **filename** of the template (as a string) it should look for *inside* the `templates/` folder. Internally, Flask uses Jinja2's template loader to search `templates/index.html`, read its contents, process any Jinja2 syntax inside it (none yet, in this simple example), and return the final HTML as a string, which becomes the HTTP response body.

**`if __name__ == "__main__":`**
A standard Python idiom — this block only runs when you execute this file *directly* (e.g., `python app.py`), not when this file is *imported* by another file. Why this matters here: it prevents `app.run()` from accidentally firing if, say, another module imports `app` from this file for testing purposes.

**`app.run(debug=True)`**
Starts Flask's built-in development server. (Full deep-dive in Module 5.)

### Program Flow

```
You run: python app.py
   ↓
__name__ == "__main__" is True
   ↓
app.run(debug=True) starts the dev server
   ↓
Browser requests GET /
   ↓
Flask matches "/" to home()
   ↓
home() calls render_template("index.html")
   ↓
Flask/Jinja2 looks inside templates/ folder
   ↓
index.html is found, read, and processed
   ↓
Resulting HTML string becomes the Response
   ↓
Browser displays the page
```

---

## 9. Visual Diagram — Folder-to-Code Relationship

```
              Flask(__name__)
                    │
        determines root path of app
                    │
        ┌───────────┴───────────┐
        ▼                       ▼
  looks for "templates/"   looks for "static/"
        │                       │
        ▼                       ▼
 render_template("x.html")   /static/<filename>
   searches templates/x.html   serves static/filename
```

---

## 10. Common Mistakes

1. **Naming the templates folder something other than `templates`** (e.g., `views/` or `html/`) without telling Flask — `render_template()` will fail with a `TemplateNotFound` error, because Flask only auto-detects the exact name `templates`.
2. **Putting `templates/` or `static/` in the wrong location** — they must sit in the same directory as the file where `Flask(__name__)` is created (or be explicitly configured otherwise).
3. **Over-structuring a tiny project** — creating `routes/`, `models/`, `services/`, `repositories/` folders for a 1-route hello-world app adds complexity with zero benefit at that scale.
4. **Under-structuring a growing project** — leaving 50 routes and 10 models all crammed into one `app.py` file long after it's become painful to navigate.
5. **Forgetting `__init__.py` when converting a folder into a package** (Structure B) — without it, older Python versions (and some tooling) won't recognize `app/` as an importable package. (Python 3.3+ supports "namespace packages" without it, but explicit `__init__.py` remains the clear, conventional choice for Flask apps, especially since it's where the Application Factory function lives — Module 20.)

> **⚠️ Warning:** `render_template()` does **not** search your whole project for a matching filename — it only searches the registered template folder(s). A common beginner panic is "but the file exists!" — yes, but not *inside* `templates/`.

---

## 11. Interview Questions (with Answers)

**Q1: Does Flask require a specific project structure?**
A: No. Flask is unopinionated about structure — it only has built-in conventions for two folder names: `templates/` and `static/`. Everything else is developer choice.

**Q2: How does Flask know where to find your templates folder?**
A: Through the `Flask(__name__)` call — Flask computes the application's root path from the module name and looks for a `templates/` folder relative to it.

**Q3: When would you switch from a single-file app to a package-based structure?**
A: When the project grows enough that separating routes, models, and configuration into different files improves clarity and reduces merge conflicts — there's no fixed line, but once a single file becomes hard to navigate (commonly cited as several hundred lines mixing concerns), it's time.

**Q4: What's the purpose of an `__init__.py` file in a Flask package structure?**
A: It marks a folder as a Python package and, by Flask convention, typically contains the **Application Factory function** that creates and configures the Flask app instance.

**Q5: If you renamed `templates/` to `templates_html/`, would `render_template()` still work without changes?**
A: No — it would raise a `TemplateNotFound` error unless you explicitly reconfigure Flask's template folder location.

---

## 12. Best Practices

- Start with **Structure A** (single file) for learning and tiny projects — don't pre-optimize.
- Migrate to **Structure B** (package-based) the moment you introduce a database, multiple unrelated feature areas, or more than ~2–3 contributors.
- Keep `templates/` and `static/` named exactly that, unless you have a strong, specific reason not to (and document it clearly if you change it).
- Group templates into subfolders mirroring your routes once you have more than a handful (e.g., `templates/auth/login.html`, `templates/blog/post.html`).
- Never mix "things Flask auto-discovers" (`templates/`, `static/`) with arbitrary custom folder names for the same purpose — pick one convention and stay consistent.

---

## 13. Real-World Usage

Production Flask codebases almost universally use a package-based structure once they pass the prototype stage, very often combined with **Blueprints** (Module 18) — which let you split routes by *feature* (e.g., an `auth` blueprint, a `blog` blueprint), each with its own templates and static subfolder, all registered onto one central app. We'll build toward exactly this by Module 20.

---

## 14. Mini Project

> Take the small example from Section 7 and expand it into **Structure B**: create a `run.py` at the top level, and an `app/` package containing `__init__.py` (which creates the Flask app and a single route), plus `app/templates/index.html`. Confirm it runs by executing `python run.py`. (Don't worry if `__init__.py`'s exact contents feel unfamiliar — we'll formalize the Application Factory pattern fully in Module 20. For now, just practice physically arranging the files correctly.)

---

## 15. Practice Exercises

**5 Easy Questions**
1. What two folder names does Flask automatically recognize by convention?
2. True/False: Flask requires a `routes.py` file.
3. What problem does separating "models," "routes," and "templates" into different files solve?
4. In Structure B, what is the purpose of `run.py`?
5. What error would you get if `render_template()` can't find the requested file?

**5 Medium Questions**
1. Why does `Flask(__name__)` matter for locating the `templates/` folder?
2. Describe a scenario where Structure A (single file) is clearly the right choice.
3. Describe a scenario where Structure A would clearly start causing pain.
4. What does `__init__.py` signal to Python about a folder?
5. Why might production Flask projects organize templates into subfolders matching feature areas?

**5 Hard Questions**
1. If you wanted Flask to look for templates in a folder named `views/` instead of `templates/`, conceptually, what would you need to configure (you don't need exact syntax yet — just explain the idea)?
2. Why is "Flask doesn't enforce structure" both a strength and a potential risk for inexperienced teams?
3. Explain why merge conflicts are more likely in a single giant `app.py` than in a well-separated package structure.
4. In what way does Flask's "convention over no convention, but not enforced convention" differ from Django's structural philosophy from Module 1?
5. Why does converting to a package-based structure usually happen *around the same time* a project introduces a database layer?

**2 Debugging Questions**
1. A teammate's project has `templates/index.html`, but their `app.py` lives in a *different* parent folder than the project root. `render_template("index.html")` fails. What's the most likely structural mistake?
2. Someone renamed their `app.py` to `server.py` and now static files return 404. Is the filename change itself the cause? What should you actually check?

**2 Interview Questions**
1. "Explain the trade-offs between a single-file Flask app and a package-based Flask app."
2. "How does Flask locate the `templates/` and `static/` folders internally?"

**1 Mini Project**
See Section 14 above.

---

## 16. Summary

| Concept | Key Takeaway |
|---|---|
| Flask's structural opinion | None, except `templates/` and `static/` by convention |
| Structure A | Single `app.py` — best for small/learning projects |
| Structure B | Package (`app/` with `__init__.py`, `routes.py`, `models.py`, etc.) — best for growing/production projects |
| `Flask(__name__)` | Anchors the root path Flask uses to find `templates/`/`static/` |
| `__init__.py` | Marks a folder as a package; conventionally hosts the Application Factory (Module 20) |

### Comparison Table — Single-File vs Package Structure

| Aspect | Single-File (`app.py`) | Package-Based (`app/`) |
|---|---|---|
| Best for | Learning, prototypes, tiny tools | Growing/production apps |
| Separation of concerns | Minimal | Strong (routes, models, config separated) |
| Merge-conflict risk | Higher as it grows | Lower |
| Onboarding new devs | Simple at first, confusing later | Slightly more setup, clearer long-term |
| Supports Blueprints cleanly | Awkward | Natural fit |

### 💡 Memory Trick
**"templates and static are the only two rooms Flask already knows about in the house — every other room, you build and label yourself."**

### ❓ FAQ
- **"Do I need to decide my final structure before I start coding?"** No — start simple (Structure A) and refactor into Structure B when the pain of a single file actually shows up. Don't pre-optimize.
- **"Will moving from Structure A to B break my templates?"** Not if you move the `templates/` folder along with your app code and keep it adjacent to wherever the `Flask(__name__)` call now lives.

---

**End of Module 3.**
