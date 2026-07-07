# MODULE 11 — Templates

> **Course:** Flask — Zero to Production
> **Module:** 11 of 39
> **Style:** Practical-first, complete code + complete HTML files

---

## 1. What Are Templates?

Until now, we've been writing HTML inside Python strings like this:

```python
return """
<!DOCTYPE html>
<html>
<body><h1>Hello Arjun</h1></body>
</html>
"""
```

This works for tiny examples, but it's terrible for real apps because:
- HTML inside Python strings has **no syntax highlighting**
- **No auto-complete** in your editor
- Mixing HTML and Python logic makes both messy and hard to read
- If you have 10 pages that all share a navbar, you'd copy-paste the navbar 10 times

**Templates** solve this. A template is a **separate `.html` file** that:
1. Lives in the `templates/` folder
2. Contains normal HTML
3. Has special **placeholder tags** where dynamic data gets inserted

You keep Python in `.py` files and HTML in `.html` files — clean separation.

---

## 2. How `render_template()` Works

```python
from flask import render_template

return render_template("index.html", name="Arjun", age=25)
```

Flask:
1. Looks inside the `templates/` folder for `index.html`
2. Reads the file
3. Finds placeholders like `{{ name }}` and `{{ age }}`
4. Replaces them with the values you passed (`"Arjun"`, `25`)
5. Returns the final HTML string as the response

The engine that does this replacement is called **Jinja2** (covered deeply in Module 12). For now, you just need to know these basics:

| Jinja2 syntax | What it does |
|---|---|
| `{{ variable }}` | Prints the value of a variable |
| `{% if ... %}` | Conditional block |
| `{% for ... %}` | Loop |
| `{# comment #}` | A comment (not sent to browser) |

---

## 3. Project Structure for This Module

```
my_app/
├── app.py
└── templates/
    ├── index.html
    ├── about.html
    ├── profile.html
    ├── products.html
    ├── dashboard.html
    └── bmi.html
```

Everything goes into the `templates/` folder. Flask finds it automatically.

---

## 4. Example 1 — Basic Template (Passing Variables)

### `app.py`

```python
from flask import Flask, render_template

app = Flask(__name__)


@app.route("/")
def home():
    return render_template("index.html", title="My Website", message="Welcome to Flask!")


@app.route("/about")
def about():
    return render_template("about.html", title="About Us")


if __name__ == "__main__":
    app.run(debug=True)
```

---

### `templates/index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ title }}</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 60px auto;
            text-align: center;
            background-color: #f8f9fa;
        }
        h1 { color: #343a40; }
        p  { color: #6c757d; font-size: 18px; }
        a  { color: #007bff; text-decoration: none; font-size: 16px; }
        a:hover { text-decoration: underline; }
    </style>
</head>
<body>
    <h1>{{ message }}</h1>
    <p>This HTML is coming from a template file, not a Python string!</p>
    <a href="/about">Go to About Page</a>
</body>
</html>
```

---

### `templates/about.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{{ title }}</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 60px auto;
            text-align: center;
        }
        a { color: #007bff; text-decoration: none; }
    </style>
</head>
<body>
    <h1>{{ title }}</h1>
    <p>We are a team of developers building Flask apps.</p>
    <a href="/">Back to Home</a>
</body>
</html>
```

**What happens:**
- `{{ title }}` is replaced with `"My Website"` or `"About Us"` depending on the route
- `{{ message }}` is replaced with `"Welcome to Flask!"`
- Everything else is plain HTML — untouched

---

## 5. Step-by-Step Explanation

**`render_template("index.html", title="My Website", message="Welcome to Flask!")`**

- `"index.html"` — the filename Flask looks for inside `templates/`
- `title="My Website"` — keyword argument; creates a variable called `title` available inside the template
- `message="Welcome to Flask!"` — same; creates a `message` variable in the template

**Inside `index.html`:**

`{{ title }}` — the double curly braces tell Jinja2: "replace this with the value of the `title` variable." Result: `My Website`.

`{{ message }}` — replaced with `"Welcome to Flask!"`.

---

## 6. Example 2 — If / Else in Templates

### `app.py`

```python
from flask import Flask, render_template

app = Flask(__name__)


@app.route("/profile/<username>")
def profile(username):
    users = {
        "arjun":  {"name": "Arjun Sharma",  "role": "admin",  "age": 25},
        "priya":  {"name": "Priya Patel",   "role": "user",   "age": 28},
        "rahul":  {"name": "Rahul Verma",   "role": "user",   "age": 22},
    }

    user = users.get(username.lower())

    return render_template("profile.html", user=user, username=username)


if __name__ == "__main__":
    app.run(debug=True)
```

---

### `templates/profile.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Profile Page</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 600px;
            margin: 60px auto;
            padding: 20px;
        }
        .card {
            background: #ffffff;
            border: 1px solid #dee2e6;
            border-radius: 10px;
            padding: 30px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        .badge-admin {
            background: #dc3545;
            color: white;
            padding: 3px 10px;
            border-radius: 4px;
            font-size: 14px;
        }
        .badge-user {
            background: #28a745;
            color: white;
            padding: 3px 10px;
            border-radius: 4px;
            font-size: 14px;
        }
        .error {
            background: #fff3cd;
            border: 1px solid #ffc107;
            border-radius: 8px;
            padding: 20px;
            text-align: center;
        }
        a { color: #007bff; text-decoration: none; }
    </style>
</head>
<body>

    {% if user %}

        <div class="card">
            <h2>{{ user.name }}</h2>
            <p>Age: {{ user.age }}</p>

            {% if user.role == "admin" %}
                <span class="badge-admin">Admin</span>
            {% else %}
                <span class="badge-user">User</span>
            {% endif %}
        </div>

    {% else %}

        <div class="error">
            <h2>User Not Found</h2>
            <p>No user with username "<strong>{{ username }}</strong>" exists.</p>
        </div>

    {% endif %}

    <br>
    <a href="/">Back to Home</a>

</body>
</html>
```

**Test URLs:**
- `/profile/arjun` — Shows Arjun's card with red Admin badge
- `/profile/priya` — Shows Priya's card with green User badge
- `/profile/nobody` — Shows User Not Found warning

**Key Jinja2 syntax:**

```html
{% if user %}              starts an if block
{% else %}                 the else branch
{% endif %}                MUST close every if block
{% if user.role == "admin" %}   comparison inside if
```

---

## 7. Example 3 — Loops in Templates (Displaying Lists)

### `app.py`

```python
from flask import Flask, render_template

app = Flask(__name__)


@app.route("/products")
def products():
    product_list = [
        {"id": 1, "name": "Laptop",     "price": 55000, "in_stock": True},
        {"id": 2, "name": "Mouse",      "price": 799,   "in_stock": True},
        {"id": 3, "name": "Keyboard",   "price": 1299,  "in_stock": False},
        {"id": 4, "name": "Monitor",    "price": 18000, "in_stock": True},
        {"id": 5, "name": "Headphones", "price": 3500,  "in_stock": False},
    ]
    return render_template("products.html", products=product_list, total=len(product_list))


if __name__ == "__main__":
    app.run(debug=True)
```

---

### `templates/products.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Products</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 40px auto;
            padding: 20px;
        }
        h1 { color: #343a40; }
        .subtitle { color: #6c757d; margin-bottom: 20px; }
        table { width: 100%; border-collapse: collapse; }
        th {
            background: #343a40;
            color: white;
            padding: 12px 15px;
            text-align: left;
        }
        td { padding: 12px 15px; border-bottom: 1px solid #dee2e6; }
        tr:hover { background: #f8f9fa; }
        .in-stock  { color: #28a745; font-weight: bold; }
        .out-stock { color: #dc3545; font-weight: bold; }
        .empty-msg { text-align: center; padding: 40px; color: #6c757d; }
    </style>
</head>
<body>

    <h1>Our Products</h1>
    <p class="subtitle">Showing {{ total }} product(s)</p>

    {% if products %}

        <table>
            <thead>
                <tr>
                    <th>#</th>
                    <th>Product Name</th>
                    <th>Price</th>
                    <th>Availability</th>
                </tr>
            </thead>
            <tbody>
                {% for product in products %}
                <tr>
                    <td>{{ loop.index }}</td>
                    <td>{{ product.name }}</td>
                    <td>{{ product.price }}</td>
                    <td>
                        {% if product.in_stock %}
                            <span class="in-stock">In Stock</span>
                        {% else %}
                            <span class="out-stock">Out of Stock</span>
                        {% endif %}
                    </td>
                </tr>
                {% endfor %}
            </tbody>
        </table>

    {% else %}

        <p class="empty-msg">No products available right now.</p>

    {% endif %}

</body>
</html>
```

**Key Jinja2 syntax:**

```html
{% for product in products %}   loops through the list
{{ loop.index }}                current iteration number (starts at 1)
{{ product.name }}              access dictionary key with dot notation
{% endfor %}                    MUST close every for loop
```

**Built-in `loop` variables:**

| Variable | What it gives you |
|---|---|
| `loop.index` | Current count (starts at 1) |
| `loop.index0` | Current count (starts at 0) |
| `loop.first` | `True` if this is the first item |
| `loop.last` | `True` if this is the last item |
| `loop.length` | Total number of items |

---

## 8. Example 4 — Passing Different Data Types

### `app.py`

```python
from flask import Flask, render_template

app = Flask(__name__)


@app.route("/dashboard")
def dashboard():
    return render_template(
        "dashboard.html",
        username   = "Arjun",
        score      = 95.5,
        is_premium = True,
        tasks      = ["Buy groceries", "Write code", "Read book"],
        stats      = {"posts": 42, "followers": 1200, "following": 300},
        last_login = None
    )


if __name__ == "__main__":
    app.run(debug=True)
```

---

### `templates/dashboard.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Dashboard - {{ username }}</title>
    <style>
        body  { font-family: Arial, sans-serif; max-width: 700px; margin: 40px auto; padding: 20px; }
        .card { background: #f8f9fa; border-radius: 10px; padding: 20px; margin: 15px 0; }
        .premium { background: #fff3cd; border: 1px solid #ffc107; }
        h3   { margin: 0 0 10px 0; color: #343a40; }
        ul   { margin: 0; padding-left: 20px; }
        li   { margin: 5px 0; }
        .stat-box {
            display: inline-block;
            background: #007bff;
            color: white;
            border-radius: 8px;
            padding: 10px 20px;
            margin: 5px;
            text-align: center;
        }
        .stat-box span { display: block; font-size: 24px; font-weight: bold; }
    </style>
</head>
<body>

    <h1>Welcome, {{ username }}!</h1>

    {# This is a Jinja2 comment - it will NOT appear in browser page source #}

    {% if is_premium %}
        <div class="card premium">
            You are a <strong>Premium</strong> member!
        </div>
    {% else %}
        <div class="card">
            You are on the free plan. <a href="#">Upgrade to Premium</a>
        </div>
    {% endif %}

    <div class="card">
        <h3>Your Score</h3>
        <p>{{ score }} / 100</p>
    </div>

    <div class="card">
        <h3>Stats</h3>
        <div class="stat-box">
            <span>{{ stats.posts }}</span>Posts
        </div>
        <div class="stat-box">
            <span>{{ stats.followers }}</span>Followers
        </div>
        <div class="stat-box">
            <span>{{ stats.following }}</span>Following
        </div>
    </div>

    <div class="card">
        <h3>Your Tasks ({{ tasks | length }})</h3>
        {% if tasks %}
            <ul>
                {% for task in tasks %}
                    <li>{{ loop.index }}. {{ task }}</li>
                {% endfor %}
            </ul>
        {% else %}
            <p>No tasks. Enjoy your day!</p>
        {% endif %}
    </div>

    <div class="card">
        <h3>Last Login</h3>
        {% if last_login %}
            <p>{{ last_login }}</p>
        {% else %}
            <p style="color: #6c757d;">No login recorded yet.</p>
        {% endif %}
    </div>

</body>
</html>
```

**New things shown:**

| Syntax | Meaning |
|---|---|
| `{{ tasks \| length }}` | A filter — `length` counts items in a list |
| `{# comment #}` | Jinja2 comment — NOT sent to browser |
| `{{ stats.posts }}` | Access dictionary value using dot notation |

---

## 9. Example 5 — Form + Template Together (BMI Calculator)

### `app.py`

```python
from flask import Flask, render_template, request

app = Flask(__name__)


@app.route("/bmi", methods=["GET", "POST"])
def bmi_calculator():
    result   = None
    category = None
    error    = None

    if request.method == "POST":
        try:
            weight = float(request.form.get("weight", 0))
            height = float(request.form.get("height", 0))

            if weight <= 0 or height <= 0:
                error = "Please enter valid weight and height."
            else:
                height_m = height / 100
                bmi      = round(weight / (height_m ** 2), 1)
                result   = bmi

                if bmi < 18.5:
                    category = "Underweight"
                elif bmi < 25:
                    category = "Normal"
                elif bmi < 30:
                    category = "Overweight"
                else:
                    category = "Obese"

        except ValueError:
            error = "Please enter numbers only."

    return render_template("bmi.html", result=result, category=category, error=error)


if __name__ == "__main__":
    app.run(debug=True)
```

---

### `templates/bmi.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>BMI Calculator</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 500px;
            margin: 60px auto;
            padding: 20px;
            background: #f0f2f5;
        }
        .card {
            background: white;
            border-radius: 12px;
            padding: 30px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
        }
        h1   { text-align: center; color: #343a40; margin-bottom: 25px; }
        label { display: block; margin-bottom: 5px; font-weight: bold; color: #495057; }
        input {
            width: 100%;
            padding: 10px;
            border: 1px solid #ced4da;
            border-radius: 6px;
            margin-bottom: 20px;
            font-size: 16px;
            box-sizing: border-box;
        }
        button {
            width: 100%;
            padding: 12px;
            background: #007bff;
            color: white;
            border: none;
            border-radius: 6px;
            font-size: 16px;
            cursor: pointer;
        }
        button:hover { background: #0056b3; }
        .error {
            background: #f8d7da;
            color: #842029;
            border: 1px solid #f5c2c7;
            border-radius: 6px;
            padding: 12px;
            margin-top: 20px;
        }
        .result {
            text-align: center;
            margin-top: 20px;
            padding: 20px;
            border-radius: 8px;
        }
        .Underweight { background: #cfe2ff; color: #084298; }
        .Normal      { background: #d1e7dd; color: #0a3622; }
        .Overweight  { background: #fff3cd; color: #664d03; }
        .Obese       { background: #f8d7da; color: #842029; }
        .bmi-number  { font-size: 48px; font-weight: bold; margin: 10px 0; }
        .bmi-cat     { font-size: 20px; }
    </style>
</head>
<body>
    <div class="card">
        <h1>BMI Calculator</h1>

        <form method="POST">
            <label for="weight">Weight (kg):</label>
            <input
                type="number"
                id="weight"
                name="weight"
                placeholder="e.g. 70"
                step="0.1"
                value="{{ request.form.get('weight', '') }}"
            >

            <label for="height">Height (cm):</label>
            <input
                type="number"
                id="height"
                name="height"
                placeholder="e.g. 175"
                step="0.1"
                value="{{ request.form.get('height', '') }}"
            >

            <button type="submit">Calculate BMI</button>
        </form>

        {% if error %}
            <div class="error">
                {{ error }}
            </div>
        {% endif %}

        {% if result %}
            <div class="result {{ category }}">
                <div class="bmi-number">{{ result }}</div>
                <div class="bmi-cat">{{ category }}</div>
            </div>
        {% endif %}

    </div>
</body>
</html>
```

**New things shown:**

- `{{ request.form.get('weight', '') }}` — Flask makes `request` available in templates automatically. This re-fills form fields after submission.
- `class="{{ category }}"` — you can use variables anywhere in HTML, including inside attributes.

---

## 10. Visual Diagram — How render_template Works

```
Python (app.py)
   |
   |  render_template("products.html", products=[...])
   |
   v
Flask looks in templates/ folder
   |
   |  reads templates/products.html
   |
   v
Jinja2 Engine processes the file:
   |
   |  {{ product.name }}    replaced with "Laptop"
   |  {% for ... %}         repeats the block for each item
   |  {% if ... %}          includes or skips sections
   |
   v
Final HTML string (all {{ }} and {% %} are gone, replaced with real values)
   |
   v
Browser receives and renders the page
```

---

## 11. `url_for()` in Templates

```html
<!-- BAD: hardcoded URL, breaks if you rename the route -->
<a href="/about">About</a>

<!-- GOOD: uses endpoint name, survives renaming -->
<a href="{{ url_for('about') }}">About</a>

<!-- With a URL variable -->
<a href="{{ url_for('profile', username='arjun') }}">Arjun's Profile</a>

<!-- For static files (CSS, images, JS) -->
<link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
<img src="{{ url_for('static', filename='logo.png') }}" alt="Logo">
```

---

## 12. Accessing `request` Inside Templates

Flask automatically makes `request`, `session`, `g`, and `config` available inside every template:

```html
<!-- Show current URL path -->
<p>You are at: {{ request.path }}</p>

<!-- Check the request method -->
{% if request.method == "POST" %}
    <p>Form was just submitted!</p>
{% endif %}

<!-- Show a value from app config -->
<p>App: {{ config['APP_NAME'] }}</p>
```

---

## 13. Common Mistakes

### Mistake 1: Wrong folder name

```
my_app/
├── app.py
└── template/       <- WRONG: missing the 's'
    └── index.html
```

Flask looks for `templates/` (with an `s`). Wrong name = `TemplateNotFound` error.

---

### Mistake 2: Forgetting to close Jinja2 blocks

```html
<!-- WRONG: no endif -->
{% if user %}
    <p>Hello {{ user.name }}</p>

<!-- CORRECT -->
{% if user %}
    <p>Hello {{ user.name }}</p>
{% endif %}
```

Every `{% if %}` needs `{% endif %}`. Every `{% for %}` needs `{% endfor %}`.

---

### Mistake 3: Using `{{ }}` for logic instead of `{% %}`

```html
<!-- WRONG -->
{{ if user }}

<!-- CORRECT -->
{% if user %}
```

Simple rule:
- `{{ }}` = **print a value**
- `{% %}` = **do something** (if, for, extends, block...)

---

### Mistake 4: Trying to run arbitrary Python in templates

```html
<!-- WRONG: Jinja2 is NOT a Python interpreter -->
{% import os %}
{% x = 2 + 3 %}
```

Do all calculations in `app.py`, pass the final result to the template.

---

### Mistake 5: Not passing a variable the template needs

```python
# app.py: forgot to pass "user"
return render_template("profile.html")
```

```html
<!-- profile.html: UndefinedError! -->
{{ user.name }}
```

Fix: pass the variable, or guard with `{% if user is defined and user %}`.

---

## 14. Interview Questions

**Q1: What is `render_template()` and what does it return?**
A: A Flask function that loads an HTML file from `templates/`, passes it through Jinja2 to replace `{{ variable }}` and process `{% %}` tags, and returns the final HTML string as a response.

**Q2: What is the difference between `{{ }}` and `{% %}`?**
A: `{{ }}` outputs/prints a value. `{% %}` is for control statements like `if`, `for`, `block` that don't output directly.

**Q3: How do you pass data from Python to a template?**
A: As keyword arguments to `render_template()`: `render_template("page.html", name="Arjun")`. Each keyword becomes a variable inside the template.

**Q4: Does Flask make the `request` object available inside templates?**
A: Yes — Flask automatically injects `request`, `session`, `g`, and `config` into every template without you needing to pass them.

**Q5: Why use `url_for()` in templates instead of hardcoded URL strings?**
A: If you rename a route, `url_for()` automatically generates the correct new URL. Hardcoded strings would all break.

---

## 15. Best Practices

- Keep all templates in the `templates/` folder.
- Use `url_for()` for all links and static file references.
- Do all calculations in Python — pass only final data to templates.
- Always check `{% if variable %}` before using a variable that might be `None`.
- Close all `{% if %}` with `{% endif %}` and all `{% for %}` with `{% endfor %}`.
- Break large templates into reusable pieces using template inheritance (Module 13).

---

## 16. Mini Project — Student Report Card

Build a complete app:

**`app.py`:** Route `GET /report/<student_name>` that passes student data to a template.

**`templates/report.html`:** Shows:
- Student name, class, roll number
- A table of subjects and marks
- Total marks and percentage
- Pass/Fail (pass if percentage >= 40)
- Green background for pass, red for fail

Student data to use:

```python
students = {
    "arjun": {
        "name": "Arjun Sharma", "class": "10th", "roll": 42,
        "marks": {"Maths": 88, "Science": 76, "English": 65, "Hindi": 70, "History": 55}
    },
    "priya": {
        "name": "Priya Patel", "class": "10th", "roll": 15,
        "marks": {"Maths": 35, "Science": 28, "English": 40, "Hindi": 32, "History": 38}
    }
}
```

---

## 17. Practice Exercises

**5 Easy Questions**
1. Which folder must template files be placed in?
2. What Jinja2 syntax prints the value of a variable called `username`?
3. How do you start and end a for loop in a Jinja2 template?
4. True/False: You can import Python modules inside a Jinja2 template.
5. What function do you call in Python to render a template?

**5 Medium Questions**
1. How do you pass a list of items to a template and display each one in a `<li>` tag?
2. What is `loop.index` and where can you use it?
3. How do you display "No items found" when a list is empty?
4. What is the difference between `{# comment #}` and `<!-- comment -->`?
5. How do you access a dictionary key `user_name` inside a template?

**5 Hard Questions**
1. A template uses `{{ products | length }}`. What is `length` here?
2. Why can't you do complex Python calculations inside Jinja2 templates?
3. How does Flask make `request` available in templates without you passing it?
4. If `render_template("page.html")` cannot find `page.html`, what error is raised?
5. You want row numbers (1, 2, 3...) in a table loop. What two ways can you do it?

**2 Debugging Questions**
1. A developer's template shows the literal text `{{ username }}` instead of the actual name. What went wrong?
2. A developer gets `jinja2.exceptions.TemplateSyntaxError` on a `{% for %}` loop. What are two likely causes?

**2 Interview Questions**
1. "Explain how `render_template()` works internally."
2. "Why is it bad practice to put business logic inside Jinja2 templates?"

---

## 18. Summary

| Concept | Key Takeaway |
|---|---|
| Templates | Separate `.html` files in the `templates/` folder |
| `render_template()` | Loads, processes, and returns the HTML file |
| `{{ variable }}` | Prints a variable's value |
| `{% if %} / {% endif %}` | Conditional block |
| `{% for x in list %} / {% endfor %}` | Loop block |
| `loop.index` | Current loop number (starts at 1) |
| `{# comment #}` | Jinja2 comment — NOT sent to browser |
| `url_for()` in templates | Generates safe, refactorable URLs |
| Auto-injected objects | `request`, `session`, `g`, `config` always available |

### Memory Trick
**"Templates are letters with blanks. Python fills in the blanks before mailing. `{{ }}` = fill in this blank. `{% %}` = follow these instructions."**

---

**End of Module 11.**
