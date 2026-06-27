# MODULE 2 — Installing Flask

> **Course:** Flask — Zero to Production
> **Module:** 2 of 39
> **Builds on:** Module 1 (Introduction to Flask)

---

## 1. Introduction

Installing Flask sounds like it should be "just run one command," and technically it is — `pip install flask`. But if that's *all* we cover, you'll get a working install today and a broken, conflicting mess of a machine six months from now when you start a second Python project. So this module is really about two things:

1. **The mechanics**: how to actually install Flask and verify it worked.
2. **The discipline**: **virtual environments** — the single most important habit for any Python web developer, and the thing almost every beginner skips and regrets later.

By the end of this module, you'll not only have Flask installed — you'll understand *why* it's installed the way it is, what files actually land on your disk, and how to keep every project's dependencies completely separate from every other project's.

---

## 2. Why Do We Need This?

Let's think about what "installing a package" really means in Python.

When you run `pip install flask`, pip does roughly this:
1. Looks up `flask` on PyPI (the Python Package Index).
2. Downloads the package files.
3. Also downloads Flask's own **dependencies** (other packages Flask itself needs to run) — Werkzeug, Jinja2, Click, itsdangerous, blinker, MarkupSafe.
4. Copies all of these into a folder Python searches when you write `import flask`.

The question is: **which folder?** By default, Python has *one* global folder per Python installation (`site-packages`). If you install Flask there, every single Python project on your computer shares that same folder.

This becomes a real problem the moment you have two projects with conflicting needs — for example, Project A needs Flask version 2.0, but Project B needs Flask version 3.1 with a newer feature. With only one global folder, you literally cannot have both versions installed at once. Installing one overwrites the other.

**Virtual environments** solve this by giving *each project its own private, isolated folder* of installed packages — so Project A and Project B can each have their own Flask version, with zero conflict.

---

## 3. Problem Before Virtual Environments Existed

Before tools like `venv` (built into Python 3.3+) and its predecessor `virtualenv` became standard practice, developers genuinely ran into "dependency hell":

- Installing a new project's requirements would silently **upgrade or downgrade** a package that an older project depended on, breaking that older project without any obvious warning.
- Different operating system package managers (e.g., the system Python on Linux) would mix with manually `pip install`-ed packages, and upgrading the OS could break your Python tools.
- Two developers on the same team, with slightly different globally-installed package versions, would get "works on my machine" bugs that took hours to diagnose, because the *code* was identical — only the *environment* differed.

Virtual environments were the direct answer: instead of trying to carefully manage one shared set of packages for the whole machine, just **give every project a clean, disposable, fully isolated copy.**

---

## 4. Real-Life Analogy

Think of your computer's global Python installation like a **shared kitchen pantry** in an apartment building. If every tenant stores their ingredients in the same shared pantry, one tenant's "upgrade the flour to a different brand" could ruin another tenant's recipe that depended on the old flour.

A **virtual environment** is like giving each tenant their **own private mini-fridge** inside their own apartment. Tenant A can stock whatever brand and version of ingredients their recipe needs. Tenant B does the same, completely independently. Nobody's mini-fridge affects anyone else's.

---

## 5. Internal Working — What Actually Happens on Disk

### 5.1 Creating a Virtual Environment

When you run:

```bash
python -m venv venv
```

Here is what happens internally, step by step:

1. `python -m venv venv` tells Python to run the `venv` module as a script, passing `venv` as the **name of the folder to create**.
2. Python creates a new directory (here, named `venv`) containing:
   - A **copy or symlink** of the Python interpreter binary itself.
   - A fresh, **empty** `site-packages` folder (or a near-empty one) — this is the "private mini-fridge."
   - Activation scripts (`activate` for macOS/Linux, `activate.bat`/`Activate.ps1` for Windows) that temporarily modify your shell's `PATH` variable.
3. Crucially, this new environment does **not** copy your globally-installed packages into it by default — it starts clean.

### 5.2 Activating It

Running the activation script (e.g., `source venv/bin/activate` on macOS/Linux, or `venv\Scripts\activate` on Windows) does one simple but powerful thing: it **temporarily edits your terminal session's `PATH`** so that when you type `python` or `pip`, your shell finds the interpreter *inside* `venv/` *before* it finds the system-wide one.

This is why your terminal prompt changes to show `(venv)` — that's your visual confirmation that `python` and `pip` are now pointing at the isolated copy.

### 5.3 Installing Flask Inside It

Now when you run:

```bash
pip install flask
```

`pip` (which is now the *venv's* pip, not the global one) downloads Flask **and its dependencies** into `venv/lib/.../site-packages/`, completely separate from any other project on your machine.

### 5.4 What Actually Gets Installed

`flask` itself is a relatively small package, but installing it pulls in several **dependencies** automatically:

```
flask
 ├── Werkzeug      (WSGI utilities, routing, request/response objects)
 ├── Jinja2        (templating engine)
 │    └── MarkupSafe  (escapes strings safely for HTML — prevents XSS)
 ├── itsdangerous  (cryptographically signs data — used for secure sessions/cookies)
 ├── click         (the library powering the `flask` command-line tool)
 └── blinker       (signals/event system used internally for some Flask hooks)
```

> **Important Note:** You never installed Werkzeug, Jinja2, or the others yourself — `pip` resolved them automatically because Flask's own packaging metadata declares them as required dependencies. This is called **dependency resolution**.

### 5.5 Diagram — Isolation

```
 Your Machine
 ┌────────────────────────────────────────────────────────┐
 │  Global Python Install (site-packages)                 │
 │   (Should stay mostly EMPTY of project-specific stuff) │
 └────────────────────────────────────────────────────────┘

 ┌───────────────────────┐      ┌───────────────────────┐
 │   Project A           │      │   Project B           │
 │   venv/               │      │   venv/               │
 │   ├─ flask 2.0        │      │   ├─ flask 3.1         │
 │   ├─ werkzeug 2.0     │      │   ├─ werkzeug 3.0      │
 │   └─ jinja2 3.0       │      │   └─ jinja2 3.1        │
 └───────────────────────┘      └───────────────────────┘
        ▲ isolated                    ▲ isolated
        no conflict between A and B's package versions
```

---

## 6. Syntax

### 6.1 Creating the Virtual Environment

```bash
python -m venv venv
```

| Part | Explanation |
|---|---|
| `python` | Invokes your system's Python 3 interpreter (on some systems this is `python3`). |
| `-m venv` | The `-m` flag tells Python "run this **module** as a script." `venv` is a built-in standard-library module whose job is to create virtual environments. |
| `venv` (second one) | This is an **argument**, not part of the command name — it's the *name of the folder* to create. You could name it `.venv`, `env`, anything; `venv` is just the common convention. |

### 6.2 Activating It

macOS / Linux:
```bash
source venv/bin/activate
```
Windows (PowerShell):
```powershell
venv\Scripts\Activate.ps1
```
Windows (cmd.exe):
```cmd
venv\Scripts\activate.bat
```

### 6.3 Installing Flask

```bash
pip install flask
```

| Part | Explanation |
|---|---|
| `pip` | Python's package installer — itself a Python program that talks to PyPI's web API. |
| `install` | A **subcommand** telling pip what action to perform (others include `uninstall`, `list`, `freeze`). |
| `flask` | The **package name** as registered on PyPI — case-insensitive, must match the published name exactly. |

### 6.4 Verifying the Install

```bash
python -m flask --version
```
or, inside a Python shell:
```python
import flask
print(flask.__version__)
```

### 6.5 Deactivating

```bash
deactivate
```
This restores your shell's `PATH` back to normal, "leaving" the virtual environment.

### 6.6 Freezing Dependencies (for sharing your project)

```bash
pip freeze > requirements.txt
```

This writes every installed package and its **exact version** into a text file, so anyone else (or you, on another machine) can recreate the identical environment with:

```bash
pip install -r requirements.txt
```

---

## 7. Small Example — Full Sequence

```bash
mkdir my_flask_project
cd my_flask_project
python -m venv venv
source venv/bin/activate
pip install flask
pip freeze > requirements.txt
```

---

## 8. Step-by-Step Explanation

**Line 1: `mkdir my_flask_project`**
Creates a new, empty folder. Why this line exists: keeping every project in its own folder is what lets each one have its own `venv/` subfolder later, with no overlap.

**Line 2: `cd my_flask_project`**
Changes your terminal's current working directory into that folder. Why: every following command (creating the venv, installing packages) should happen *inside* this project, not somewhere else on disk.

**Line 3: `python -m venv venv`**
Creates the isolated environment, as explained in Section 5.1. This is the object/structure being created: an entire mini Python installation living inside `my_flask_project/venv/`.

**Line 4: `source venv/bin/activate`**
Switches your *current terminal session* to use that mini installation's `python`/`pip` instead of the system-wide ones. Why this is called: without it, `pip install flask` later would install Flask globally, defeating the whole purpose.

**Line 5: `pip install flask`**
Downloads and installs Flask plus its dependency tree (Section 5.4) into `venv/lib/.../site-packages/`.

**Line 6: `pip freeze > requirements.txt`**
`pip freeze` prints every installed package with its exact version to the terminal; the `>` symbol **redirects** that output into a new file called `requirements.txt` instead of printing it to the screen. Why this line exists: it's how you make your project's dependencies *reproducible* for teammates, deployment servers, or future-you.

### Program Flow

```
mkdir + cd
   ↓
python -m venv venv   (create isolated environment)
   ↓
activate              (point shell's python/pip at venv)
   ↓
pip install flask     (download Flask + dependencies into venv)
   ↓
pip freeze > requirements.txt   (record exact versions)
   ↓
Ready to write Flask code
```

---

## 9. Visual Diagram

```
 Terminal (before activation)
    │
    │  which python  →  /usr/bin/python (global)
    ▼
 source venv/bin/activate
    │
    │  PATH variable temporarily modified
    ▼
 Terminal (after activation)
    │
    │  which python  →  /project/venv/bin/python (isolated)
    ▼
 pip install flask
    │
    ▼
 venv/lib/python3.x/site-packages/
    ├── flask/
    ├── werkzeug/
    ├── jinja2/
    ├── markupsafe/
    ├── itsdangerous/
    ├── click/
    └── blinker/
```

---

## 10. Common Mistakes

1. **Forgetting to activate the venv** before running `pip install flask` — you end up installing globally without realizing it, and later wonder why your project "can't find Flask" when run differently.
2. **Committing the `venv/` folder to Git.** It's machine-specific, often huge, and unnecessary — commit `requirements.txt` instead, and add `venv/` to `.gitignore`.
3. **Mixing `pip` and `conda` installs in the same environment**, which can cause subtle, hard-to-debug conflicts.
4. **Installing Flask with `sudo pip install flask`** on Linux/macOS — this installs into system-protected directories and risks breaking your OS's own Python tooling. Always prefer a venv.
5. **Assuming `python` always means Python 3.** On some systems, `python` still points to Python 2. Use `python3 -m venv venv` if unsure.

> **⚠️ Warning:** If `pip install flask` appears to succeed but `import flask` still fails in your code, the #1 cause is that your venv isn't actually activated in the terminal/IDE you're running the code from. Always check `which python` (macOS/Linux) or `where python` (Windows) to confirm.

---

## 11. Interview Questions (with Answers)

**Q1: What is a virtual environment, and why is it important?**
A: An isolated, self-contained copy of Python and its installed packages, scoped to one project. It prevents version conflicts between projects and makes deployments reproducible.

**Q2: What command creates a virtual environment using Python's standard library?**
A: `python -m venv <name>` (using the built-in `venv` module — no third-party tool required since Python 3.3).

**Q3: What's the purpose of `requirements.txt`?**
A: It lists exact package names and versions used by a project, so the same environment can be recreated elsewhere with `pip install -r requirements.txt`.

**Q4: Does installing Flask also install Werkzeug and Jinja2 automatically? Why?**
A: Yes — because Flask declares them as required dependencies in its packaging metadata, and `pip` automatically resolves and installs a package's full dependency tree.

**Q5: What's the risk of using `sudo pip install` on a global Python installation?**
A: It can overwrite or conflict with packages your operating system itself relies on, since many Linux distributions use system Python internally for OS-level scripts/tools.

---

## 12. Best Practices

- **One virtual environment per project** — always, no exceptions, even for tiny scripts.
- Name your venv folder consistently (`venv` or `.venv`) and add it to `.gitignore` immediately.
- Pin versions in `requirements.txt` (e.g., `flask==3.1.0`) for production projects, so a future `pip install` doesn't silently pull in a breaking new version.
- Re-create your venv from `requirements.txt` rather than manually reinstalling packages one by one when setting up on a new machine.
- Consider tools like `pip-tools` or `poetry` for larger projects (mentioned here for awareness — out of scope for this beginner module).

---

## 13. Real-World Usage

In real engineering teams, this exact workflow — `venv` + `requirements.txt` (or a `pyproject.toml`/`Pipfile` equivalent) — is how a project's dependencies travel with the codebase. Continuous Integration (CI) servers, Docker containers, and new teammates' machines all run essentially the same two commands you just learned (`python -m venv venv` + `pip install -r requirements.txt`) to get an identical, working environment, every single time.

---

## 14. Mini Project

> Set up a brand-new folder called `flask_playground`. Create a virtual environment inside it, activate it, install Flask, and generate a `requirements.txt`. Then run `pip list` and write down (in your own notes) every package that got installed and, in one sentence each, what role you *think* it plays (you can guess — we'll confirm Werkzeug's and Jinja2's roles in upcoming modules).

---

## 15. Practice Exercises

**5 Easy Questions**
1. What command creates a virtual environment named `env`?
2. What terminal command installs Flask once a venv is activated?
3. True/False: `venv/` should normally be committed to Git.
4. What file records the exact versions of installed packages?
5. What command deactivates an active virtual environment?

**5 Medium Questions**
1. Explain why two projects on the same machine might need two different Flask versions.
2. What does the `-m` flag do in `python -m venv venv`?
3. What does `pip freeze` actually do, technically?
4. Name three packages that get installed automatically as Flask's dependencies.
5. Why is it risky to install Python packages with `sudo` on Linux/macOS?

**5 Hard Questions**
1. Explain, internally, what changes in your shell session when you "activate" a virtual environment.
2. If `requirements.txt` only pins `flask==3.1.0` but not `werkzeug`, what could go wrong months later when someone re-installs from that file?
3. Why does a virtual environment not, by default, include your globally-installed packages?
4. What's the practical difference between `pip install flask` and `pip install -r requirements.txt`?
5. If `python` on a machine points to Python 2.7, what specifically would go wrong when trying `python -m venv venv`, and why?

**2 Debugging Questions**
1. A user says: "I activated my venv, ran `pip install flask`, but `import flask` still fails in VS Code." What are two likely causes related to *which interpreter* is actually being used?
2. A teammate's `requirements.txt` has no version numbers at all, just package names. Six months later, a fresh install breaks the project. What's the likely root cause?

**2 Interview Questions**
1. "Walk me through, step by step, what happens internally when someone runs `python -m venv venv` followed by activation."
2. "Why do professional teams avoid installing project dependencies globally?"

**1 Mini Project**
See Section 14 above.

---

## 16. Summary

| Concept | Key Takeaway |
|---|---|
| Virtual environment | An isolated, per-project copy of Python + packages |
| `python -m venv venv` | Standard-library command to create one |
| Activation | Temporarily redirects `python`/`pip` in your shell to the venv's copies |
| `pip install flask` | Installs Flask + its dependency tree (Werkzeug, Jinja2, etc.) |
| `requirements.txt` | A reproducible snapshot of exact installed versions |
| Golden rule | One project = one venv, always |

### Comparison Table — Global Install vs Virtual Environment

| Aspect | Global Install | Virtual Environment |
|---|---|---|
| Isolation | None — shared across all projects | Fully isolated per project |
| Version conflicts | Common | Avoided |
| Reproducibility | Poor | Excellent (via `requirements.txt`) |
| Risk to OS tooling | Possible (especially with `sudo`) | None |
| Recommended for real projects | ❌ | ✅ |

### 💡 Memory Trick
**"venv = a clean rental apartment, every project gets its own — never let projects share a fridge."**

### ❓ FAQ
- **"Do I need to activate the venv every time I open a new terminal?"** Yes — activation is per-terminal-session; it doesn't persist automatically across new terminal windows or reboots.
- **"What's the difference between `venv` and `virtualenv`?"** `venv` is built into Python 3.3+ standard library; `virtualenv` is an older third-party tool that still exists and has a few extra features, but for this course, the built-in `venv` is all you need.

---

**End of Module 2.**
Say **"Next"** when you're ready for **Module 3 — Project Structure** (how to organize files and folders in a real Flask project, and why structure matters as a project grows).
