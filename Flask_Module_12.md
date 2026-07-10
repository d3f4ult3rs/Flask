# MODULE 12 — Jinja2

> **Course:** Flask — Zero to Production
> **Module:** 12 of 39
> **Style:** Practical-first, complete code + complete HTML files

---

## 1. What Is Jinja2?

Jinja2 is the **templating engine** Flask uses to process your `.html` files.

In Module 11 we used the basics — `{{ variable }}`, `{% if %}`, `{% for %}`. That's just the beginning. Jinja2 has many powerful features that let you:

- **Format and transform data** inside the template (filters)
- **Check what type a value is** (tests)
- **Create reusable HTML components** (macros)
- **Set variables** inside a template
- **Add custom Python functions** usable in templates

This module covers all of these practically, with complete working examples.

---

## 2. Project Structure for This Module

```
my_app/
├── app.py
└── templates/
    ├── filters.html
    ├── tests.html
    ├── macros.html
    ├── custom_filter.html
    └── full_demo.html
```

---

## 3. Filters — Transforming Data in Templates

A **filter** transforms a value before displaying it.

**Syntax:** `{{ value | filter_name }}`
**With arguments:** `{{ value | filter_name(argument) }}`
**Chaining:** `{{ value | filter1 | filter2 }}`

Think of it like a pipe — the value flows through each filter and comes out transformed.

---

### `app.py` (for all filter examples)

```python
from flask import Flask, render_template

app = Flask(__name__)


@app.route("/filters")
def filters_demo():
    return render_template("filters.html",
        name         = "arjun sharma",
        description  = "  Flask is a lightweight web framework.  ",
        price        = 1999.5678,
        rating       = 4.3,
        items        = ["Apple", "Banana", "Cherry", "Date", "Elderberry"],
        tags         = ["python", "flask", "web"],
        bio          = None,
        long_text    = "This is a very long description that goes on and on and might need to be cut short for display purposes.",
        html_content = "<script>alert('XSS!')</script>Hello",
        count        = 7
    )


if __name__ == "__main__":
    app.run(debug=True)
```

---

### `templates/filters.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Jinja2 Filters Demo</title>
    <style>
        body     { font-family: Arial, sans-serif; max-width: 750px; margin: 40px auto; padding: 20px; }
        h1       { color: #343a40; border-bottom: 2px solid #007bff; padding-bottom: 10px; }
        h3       { color: #495057; margin-top: 30px; }
        .box     { background: #f8f9fa; border-left: 4px solid #007bff; padding: 12px 16px; margin: 8px 0; border-radius: 4px; }
        .label   { color: #6c757d; font-size: 13px; font-weight: bold; text-transform: uppercase; }
        .value   { font-size: 16px; margin-top: 4px; }
        code     { background: #e9ecef; padding: 2px 6px; border-radius: 3px; font-size: 14px; }
    </style>
</head>
<body>

<h1>Jinja2 Filters Demo</h1>

<!-- ═══════════════════════════════════════════════════
     STRING FILTERS
═══════════════════════════════════════════════════ -->
<h3>String Filters</h3>

<div class="box">
    <div class="label">upper — converts to UPPERCASE</div>
    <div class="value">{{ name | upper }}</div>
    <code>{{ "{{ name | upper }}" }}</code>
</div>

<div class="box">
    <div class="label">lower — converts to lowercase</div>
    <div class="value">{{ name | lower }}</div>
    <code>{{ "{{ name | lower }}" }}</code>
</div>

<div class="box">
    <div class="label">title — Title Case Every Word</div>
    <div class="value">{{ name | title }}</div>
    <code>{{ "{{ name | title }}" }}</code>
</div>

<div class="box">
    <div class="label">capitalize — Only first letter capitalized</div>
    <div class="value">{{ name | capitalize }}</div>
    <code>{{ "{{ name | capitalize }}" }}</code>
</div>

<div class="box">
    <div class="label">strip — removes whitespace from both sides</div>
    <div class="value">"{{ description | strip }}"</div>
    <code>{{ "{{ description | strip }}" }}</code>
</div>

<div class="box">
    <div class="label">replace — replaces text</div>
    <div class="value">{{ name | replace("arjun", "Arjun") }}</div>
    <code>{{ '{{ name | replace("arjun", "Arjun") }}' }}</code>
</div>

<div class="box">
    <div class="label">length — count characters or items</div>
    <div class="value">Name length: {{ name | length }}</div>
    <code>{{ "{{ name | length }}" }}</code>
</div>

<div class="box">
    <div class="label">truncate — cut long text with ...</div>
    <div class="value">{{ long_text | truncate(40) }}</div>
    <code>{{ "{{ long_text | truncate(40) }}" }}</code>
</div>

<div class="box">
    <div class="label">wordcount — count number of words</div>
    <div class="value">{{ long_text | wordcount }} words</div>
    <code>{{ "{{ long_text | wordcount }}" }}</code>
</div>

<div class="box">
    <div class="label">reverse — reverses a string or list</div>
    <div class="value">{{ name | reverse }}</div>
    <code>{{ "{{ name | reverse }}" }}</code>
</div>

<!-- ═══════════════════════════════════════════════════
     NUMBER FILTERS
═══════════════════════════════════════════════════ -->
<h3>Number Filters</h3>

<div class="box">
    <div class="label">round — round to N decimal places</div>
    <div class="value">{{ price | round(2) }}</div>
    <code>{{ "{{ price | round(2) }}" }}</code>
</div>

<div class="box">
    <div class="label">round(0) — round to whole number</div>
    <div class="value">{{ price | round(0) | int }}</div>
    <code>{{ "{{ price | round(0) | int }}" }}</code>
</div>

<div class="box">
    <div class="label">abs — absolute value (removes negative sign)</div>
    <div class="value">{{ -42 | abs }}</div>
    <code>{{ "{{ -42 | abs }}" }}</code>
</div>

<!-- ═══════════════════════════════════════════════════
     LIST FILTERS
═══════════════════════════════════════════════════ -->
<h3>List Filters</h3>

<div class="box">
    <div class="label">length — count items in list</div>
    <div class="value">{{ items | length }} items</div>
    <code>{{ "{{ items | length }}" }}</code>
</div>

<div class="box">
    <div class="label">first — get the first item</div>
    <div class="value">{{ items | first }}</div>
    <code>{{ "{{ items | first }}" }}</code>
</div>

<div class="box">
    <div class="label">last — get the last item</div>
    <div class="value">{{ items | last }}</div>
    <code>{{ "{{ items | last }}" }}</code>
</div>

<div class="box">
    <div class="label">join — join list items into a string</div>
    <div class="value">{{ items | join(", ") }}</div>
    <code>{{ '{{ items | join(", ") }}' }}</code>
</div>

<div class="box">
    <div class="label">sort — sort a list alphabetically</div>
    <div class="value">{{ items | sort | join(", ") }}</div>
    <code>{{ "{{ items | sort | join(', ') }}" }}</code>
</div>

<div class="box">
    <div class="label">reverse — reverse a list</div>
    <div class="value">{{ items | reverse | list | join(", ") }}</div>
    <code>{{ "{{ items | reverse | list | join(', ') }}" }}</code>
</div>

<div class="box">
    <div class="label">unique — remove duplicates</div>
    {% set dupes = ["apple", "banana", "apple", "cherry", "banana"] %}
    <div class="value">{{ dupes | unique | list | join(", ") }}</div>
    <code>{{ "{{ dupes | unique | list | join(', ') }}" }}</code>
</div>

<!-- ═══════════════════════════════════════════════════
     DEFAULT & SAFETY FILTERS
═══════════════════════════════════════════════════ -->
<h3>Default and Safety Filters</h3>

<div class="box">
    <div class="label">default — show fallback if value is None or empty</div>
    <div class="value">{{ bio | default("No bio provided.") }}</div>
    <code>{{ '{{ bio | default("No bio provided.") }}' }}</code>
</div>

<div class="box">
    <div class="label">default with boolean — also catches False, 0, ""</div>
    <div class="value">{{ bio | default("No bio.", boolean=True) }}</div>
    <code>{{ '{{ bio | default("No bio.", boolean=True) }}' }}</code>
</div>

<div class="box">
    <div class="label">escape — safely escape HTML characters (prevents XSS)</div>
    <div class="value">{{ html_content | escape }}</div>
    <code>{{ "{{ html_content | escape }}" }}</code>
</div>

<div class="box">
    <div class="label">safe — mark a string as SAFE HTML (do not escape)</div>
    <div class="value">Use carefully — only on trusted content!</div>
    <div class="value">{{ "&lt;strong&gt;Bold&lt;/strong&gt;" | safe }}</div>
    <code>{{ '{{ trusted_html | safe }}' }}</code>
</div>

<!-- ═══════════════════════════════════════════════════
     CHAINING FILTERS
═══════════════════════════════════════════════════ -->
<h3>Chaining Multiple Filters</h3>

<div class="box">
    <div class="label">Chain: title + truncate(20) + upper</div>
    <div class="value">{{ name | title | truncate(20) | upper }}</div>
    <code>{{ "{{ name | title | truncate(20) | upper }}" }}</code>
</div>

<div class="box">
    <div class="label">Chain: tags list → join with # → upper</div>
    <div class="value">{{ tags | join(" #") | upper }}</div>
    <code>{{ '{{ tags | join(" #") | upper }}' }}</code>
</div>

</body>
</html>
```

---

## 4. Jinja2 Tests — Checking What a Value Is

A **test** checks a condition about a value and returns `True` or `False`.

**Syntax:** `{{ value is test_name }}` or `{% if value is test_name %}`

### `app.py` (add these routes)

```python
@app.route("/tests")
def tests_demo():
    return render_template("tests.html",
        name        = "Arjun",
        empty_name  = "",
        none_value  = None,
        number      = 42,
        odd_num     = 7,
        even_num    = 8,
        items       = ["Apple", "Banana"],
        empty_list  = [],
        mapping     = {"key": "value"},
        score       = 85,
    )
```

---

### `templates/tests.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Jinja2 Tests Demo</title>
    <style>
        body  { font-family: Arial, sans-serif; max-width: 700px; margin: 40px auto; padding: 20px; }
        h1    { color: #343a40; border-bottom: 2px solid #28a745; padding-bottom: 10px; }
        h3    { color: #495057; margin-top: 30px; }
        .box  { background: #f8f9fa; border-left: 4px solid #28a745; padding: 12px 16px; margin: 8px 0; border-radius: 4px; }
        .true  { color: #28a745; font-weight: bold; }
        .false { color: #dc3545; font-weight: bold; }
        code  { background: #e9ecef; padding: 2px 6px; border-radius: 3px; font-size: 14px; }
    </style>
</head>
<body>

<h1>Jinja2 Tests Demo</h1>

<h3>None and Defined Tests</h3>

<div class="box">
    <code>none_value is none</code> →
    {% if none_value is none %}
        <span class="true">True — value IS None</span>
    {% else %}
        <span class="false">False</span>
    {% endif %}
</div>

<div class="box">
    <code>name is none</code> →
    {% if name is none %}
        <span class="true">True</span>
    {% else %}
        <span class="false">False — "Arjun" is not None</span>
    {% endif %}
</div>

<div class="box">
    <code>name is defined</code> →
    {% if name is defined %}
        <span class="true">True — variable "name" exists</span>
    {% else %}
        <span class="false">False</span>
    {% endif %}
</div>

<div class="box">
    <code>ghost_var is defined</code> →
    {% if ghost_var is defined %}
        <span class="true">True</span>
    {% else %}
        <span class="false">False — "ghost_var" was never passed</span>
    {% endif %}
</div>

<div class="box">
    <code>name is undefined</code> →
    {% if name is undefined %}
        <span class="true">True</span>
    {% else %}
        <span class="false">False — "name" is defined</span>
    {% endif %}
</div>

<h3>Number Tests</h3>

<div class="box">
    <code>odd_num is odd</code> (odd_num = {{ odd_num }}) →
    {% if odd_num is odd %}
        <span class="true">True — {{ odd_num }} is odd</span>
    {% else %}
        <span class="false">False</span>
    {% endif %}
</div>

<div class="box">
    <code>even_num is even</code> (even_num = {{ even_num }}) →
    {% if even_num is even %}
        <span class="true">True — {{ even_num }} is even</span>
    {% else %}
        <span class="false">False</span>
    {% endif %}
</div>

<div class="box">
    <code>number is number</code> →
    {% if number is number %}
        <span class="true">True — {{ number }} is a number type</span>
    {% else %}
        <span class="false">False</span>
    {% endif %}
</div>

<div class="box">
    <code>number is divisibleby(6)</code> (42 / 6 = 7) →
    {% if number is divisibleby(6) %}
        <span class="true">True — 42 is divisible by 6</span>
    {% else %}
        <span class="false">False</span>
    {% endif %}
</div>

<h3>Type Tests</h3>

<div class="box">
    <code>name is string</code> →
    {% if name is string %}
        <span class="true">True</span>
    {% else %}
        <span class="false">False</span>
    {% endif %}
</div>

<div class="box">
    <code>items is iterable</code> →
    {% if items is iterable %}
        <span class="true">True — lists are iterable</span>
    {% else %}
        <span class="false">False</span>
    {% endif %}
</div>

<div class="box">
    <code>mapping is mapping</code> →
    {% if mapping is mapping %}
        <span class="true">True — dicts are mappings</span>
    {% else %}
        <span class="false">False</span>
    {% endif %}
</div>

<h3>Truthy / Falsy Tests</h3>

<div class="box">
    <code>items is truthy</code> (has items) →
    {% if items %}
        <span class="true">True — list has items</span>
    {% else %}
        <span class="false">False</span>
    {% endif %}
</div>

<div class="box">
    <code>empty_list is falsy</code> (empty list) →
    {% if not empty_list %}
        <span class="true">True — empty list is falsy</span>
    {% else %}
        <span class="false">False</span>
    {% endif %}
</div>

<h3>Practical: Score Grade Example</h3>

<div class="box">
    Score is {{ score }}.
    {% if score >= 90 %}
        <span class="true">Grade: A</span>
    {% elif score >= 75 %}
        <span class="true">Grade: B</span>
    {% elif score >= 60 %}
        Grade: C
    {% elif score >= 40 %}
        <span class="false">Grade: D</span>
    {% else %}
        <span class="false">Grade: F — Fail</span>
    {% endif %}
</div>

</body>
</html>
```

---

## 5. `{% set %}` — Setting Variables Inside a Template

Sometimes you need to create a temporary variable inside the template itself.

```html
{% set full_name = first_name ~ " " ~ last_name %}
<p>Hello, {{ full_name }}!</p>

{% set total = price * quantity %}
<p>Total: ₹{{ total | round(2) }}</p>

{% set items_count = products | length %}
<p>{{ items_count }} products found</p>
```

The `~` operator is Jinja2's **string concatenation** (like `+` for strings in Python).

---

## 6. Macros — Reusable HTML Components

A **macro** is like a Python function but written inside a template. You define it once and call it anywhere. It's perfect for reusable UI components like buttons, form fields, or cards.

### `app.py`

```python
@app.route("/macros")
def macros_demo():
    users = [
        {"name": "Arjun Sharma",  "email": "arjun@test.com",  "role": "admin",  "active": True},
        {"name": "Priya Patel",   "email": "priya@test.com",  "role": "user",   "active": True},
        {"name": "Rahul Verma",   "email": "rahul@test.com",  "role": "user",   "active": False},
        {"name": "Sneha Mehta",   "email": "sneha@test.com",  "role": "editor", "active": True},
    ]
    return render_template("macros.html", users=users)
```

---

### `templates/macros.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Jinja2 Macros Demo</title>
    <style>
        body  { font-family: Arial, sans-serif; max-width: 800px; margin: 40px auto; padding: 20px; }
        h1    { color: #343a40; }

        /* Button styles */
        .btn        { padding: 8px 16px; border: none; border-radius: 5px; cursor: pointer; font-size: 14px; text-decoration: none; display: inline-block; margin: 3px; }
        .btn-primary   { background: #007bff; color: white; }
        .btn-danger    { background: #dc3545; color: white; }
        .btn-success   { background: #28a745; color: white; }
        .btn-secondary { background: #6c757d; color: white; }
        .btn-sm { padding: 4px 10px; font-size: 12px; }
        .btn-lg { padding: 12px 24px; font-size: 18px; }

        /* Badge styles */
        .badge        { padding: 3px 10px; border-radius: 12px; font-size: 12px; font-weight: bold; }
        .badge-admin  { background: #dc3545; color: white; }
        .badge-editor { background: #fd7e14; color: white; }
        .badge-user   { background: #28a745; color: white; }

        /* Card styles */
        .card         { border: 1px solid #dee2e6; border-radius: 10px; padding: 20px; margin: 10px 0; display: flex; justify-content: space-between; align-items: center; }
        .card-inactive { opacity: 0.5; }
        .card-name    { font-weight: bold; font-size: 16px; }
        .card-email   { color: #6c757d; font-size: 14px; }

        /* Alert styles */
        .alert         { padding: 12px 16px; border-radius: 6px; margin: 10px 0; }
        .alert-success { background: #d1e7dd; color: #0a3622; border: 1px solid #a3cfbb; }
        .alert-danger  { background: #f8d7da; color: #842029; border: 1px solid #f1aeb5; }
        .alert-warning { background: #fff3cd; color: #664d03; border: 1px solid #ffe69c; }
        .alert-info    { background: #cff4fc; color: #055160; border: 1px solid #9eeaf9; }

        /* Input field styles */
        .form-group   { margin: 15px 0; }
        .form-group label  { display: block; font-weight: bold; margin-bottom: 5px; }
        .form-group input  { width: 100%; padding: 9px 12px; border: 1px solid #ced4da; border-radius: 5px; font-size: 15px; box-sizing: border-box; }
        .form-group input:focus { outline: none; border-color: #007bff; box-shadow: 0 0 0 3px rgba(0,123,255,0.15); }
        .form-group .error-msg { color: #dc3545; font-size: 13px; margin-top: 4px; }
    </style>
</head>
<body>

<h1>Jinja2 Macros Demo</h1>

{# ═══════════════════════════════════════════════
   DEFINE MACROS
   (Usually these go at the top, or in a separate file)
═══════════════════════════════════════════════ #}

{# MACRO 1: Button component #}
{% macro render_button(label, type="primary", size="", href="#") %}
    <a href="{{ href }}" class="btn btn-{{ type }} {{ 'btn-' + size if size }}">
        {{ label }}
    </a>
{% endmacro %}


{# MACRO 2: Badge / role tag #}
{% macro render_badge(role) %}
    <span class="badge badge-{{ role }}">{{ role | upper }}</span>
{% endmacro %}


{# MACRO 3: User card #}
{% macro render_user_card(user) %}
    <div class="card {{ 'card-inactive' if not user.active }}">
        <div>
            <div class="card-name">{{ user.name }}</div>
            <div class="card-email">{{ user.email }}</div>
        </div>
        <div>
            {{ render_badge(user.role) }}
            {% if user.active %}
                {{ render_button("Edit",   type="secondary", size="sm") }}
                {{ render_button("Delete", type="danger",    size="sm") }}
            {% else %}
                <span style="color: #dc3545; font-size: 13px;">Inactive</span>
            {% endif %}
        </div>
    </div>
{% endmacro %}


{# MACRO 4: Alert box #}
{% macro render_alert(message, type="info", dismissible=False) %}
    <div class="alert alert-{{ type }}">
        {% if type == "success" %}✓{% elif type == "danger" %}✗{% elif type == "warning" %}⚠{% else %}ℹ{% endif %}
        {{ message }}
        {% if dismissible %}
            <a href="#" style="float:right; color: inherit;">&times;</a>
        {% endif %}
    </div>
{% endmacro %}


{# MACRO 5: Form input field with label and optional error #}
{% macro render_input(name, label, type="text", value="", error="", placeholder="") %}
    <div class="form-group">
        <label for="{{ name }}">{{ label }}</label>
        <input
            type="{{ type }}"
            id="{{ name }}"
            name="{{ name }}"
            value="{{ value }}"
            placeholder="{{ placeholder }}"
            {% if error %}style="border-color: #dc3545;"{% endif %}
        >
        {% if error %}
            <div class="error-msg">{{ error }}</div>
        {% endif %}
    </div>
{% endmacro %}


{# ═══════════════════════════════════════════════
   USE THE MACROS
═══════════════════════════════════════════════ #}

<h2>Buttons (render_button macro)</h2>

{{ render_button("Primary Button",   type="primary") }}
{{ render_button("Danger Button",    type="danger") }}
{{ render_button("Success Button",   type="success") }}
{{ render_button("Small Button",     type="primary",   size="sm") }}
{{ render_button("Large Button",     type="success",   size="lg") }}
{{ render_button("Go to Google",     type="secondary", href="https://google.com") }}

<h2>Alerts (render_alert macro)</h2>

{{ render_alert("Your profile was saved successfully!", type="success") }}
{{ render_alert("You are not logged in.", type="warning") }}
{{ render_alert("An error occurred. Please try again.", type="danger") }}
{{ render_alert("Here is some helpful information.", type="info") }}

<h2>User Cards (render_user_card macro)</h2>

{% for user in users %}
    {{ render_user_card(user) }}
{% endfor %}

<h2>Form Inputs (render_input macro)</h2>

<form method="POST" action="#">
    {{ render_input("username", "Username",       placeholder="Enter your username") }}
    {{ render_input("email",    "Email Address",  type="email", placeholder="you@example.com") }}
    {{ render_input("password", "Password",       type="password", error="Password must be at least 8 characters") }}
    {{ render_input("phone",    "Phone Number",   placeholder="+91 9876543210", value="9876543210") }}
    {{ render_button("Submit Form", type="primary", size="lg") }}
</form>

</body>
</html>
```

**Why macros are powerful:**
- You define `render_input` once — it handles label, input field, and error message together
- Every form field looks consistent
- If you want to change how all input fields look, you change ONE macro
- No copy-pasting the same HTML block over and over

---

## 7. Custom Filters — Adding Your Own Python Functions as Filters

You can add any Python function as a custom Jinja2 filter using `@app.template_filter()`.

### `app.py` (complete, with custom filters)

```python
from flask import Flask, render_template
from datetime import datetime

app = Flask(__name__)


# ─── CUSTOM FILTER 1: Format a number as Indian currency ──────────────────
@app.template_filter("inr")
def inr_format(value):
    """Formats a number like: 1250000 → ₹12,50,000"""
    try:
        value = int(value)
        s = str(value)
        if len(s) <= 3:
            return f"₹{s}"
        # Indian number system: last 3 digits, then groups of 2
        result = s[-3:]
        s = s[:-3]
        while s:
            result = s[-2:] + "," + result
            s = s[:-2]
        return f"₹{result.lstrip(',')}"
    except (ValueError, TypeError):
        return f"₹{value}"


# ─── CUSTOM FILTER 2: Time ago (like "3 hours ago") ──────────────────────
@app.template_filter("timeago")
def time_ago(dt):
    """Converts a datetime to '5 minutes ago' style string"""
    now  = datetime.utcnow()
    diff = now - dt
    seconds = int(diff.total_seconds())

    if seconds < 60:
        return f"{seconds} seconds ago"
    elif seconds < 3600:
        return f"{seconds // 60} minutes ago"
    elif seconds < 86400:
        return f"{seconds // 3600} hours ago"
    elif seconds < 604800:
        return f"{seconds // 86400} days ago"
    else:
        return dt.strftime("%d %b %Y")


# ─── CUSTOM FILTER 3: Initials from a full name ───────────────────────────
@app.template_filter("initials")
def get_initials(name):
    """'Arjun Sharma' → 'AS'"""
    parts = name.strip().split()
    return "".join(p[0].upper() for p in parts if p)


# ─── CUSTOM FILTER 4: Mask email for privacy ─────────────────────────────
@app.template_filter("mask_email")
def mask_email(email):
    """'arjun@gmail.com' → 'ar***@gmail.com'"""
    try:
        local, domain = email.split("@")
        masked = local[:2] + "*" * (len(local) - 2)
        return f"{masked}@{domain}"
    except Exception:
        return email


# ─── CUSTOM FILTER 5: Pluralise words ────────────────────────────────────
@app.template_filter("pluralise")
def pluralise(count, singular, plural=None):
    """{{ 1 | pluralise('item') }} → '1 item'
       {{ 3 | pluralise('item') }} → '3 items'"""
    if plural is None:
        plural = singular + "s"
    return f"{count} {singular if count == 1 else plural}"


# ─── ROUTE ────────────────────────────────────────────────────────────────
@app.route("/custom-filters")
def custom_filters_demo():
    from datetime import timedelta
    return render_template("custom_filter.html",
        salary      = 1250000,
        product_price = 49999,
        post_time   = datetime.utcnow() - timedelta(minutes=37),
        login_time  = datetime.utcnow() - timedelta(hours=5),
        old_date    = datetime.utcnow() - timedelta(days=10),
        users       = [
            {"name": "Arjun Sharma",   "email": "arjun.sharma@gmail.com"},
            {"name": "Priya Patel",    "email": "priya.patel@yahoo.com"},
            {"name": "Rahul Verma",    "email": "rahulv@company.org"},
        ],
        cart_count  = 1,
        file_count  = 5,
    )


if __name__ == "__main__":
    app.run(debug=True)
```

---

### `templates/custom_filter.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Custom Filters Demo</title>
    <style>
        body   { font-family: Arial, sans-serif; max-width: 750px; margin: 40px auto; padding: 20px; }
        h1     { color: #343a40; border-bottom: 2px solid #6f42c1; padding-bottom: 10px; }
        h3     { color: #495057; margin-top: 30px; }
        .box   { background: #f8f9fa; border-left: 4px solid #6f42c1; padding: 12px 16px; margin: 8px 0; border-radius: 4px; }
        .label { color: #6c757d; font-size: 13px; font-weight: bold; text-transform: uppercase; }
        .value { font-size: 18px; margin-top: 5px; color: #212529; }
        code   { background: #e9ecef; padding: 2px 6px; border-radius: 3px; font-size: 13px; }
        table  { width: 100%; border-collapse: collapse; margin: 10px 0; }
        th     { background: #6f42c1; color: white; padding: 10px 14px; text-align: left; }
        td     { padding: 10px 14px; border-bottom: 1px solid #dee2e6; }
        .avatar {
            display: inline-flex; align-items: center; justify-content: center;
            width: 40px; height: 40px; border-radius: 50%; background: #6f42c1;
            color: white; font-weight: bold; font-size: 14px;
        }
    </style>
</head>
<body>

<h1>Custom Filters Demo</h1>

<!-- ─── INR Filter ────────────────────────────────────── -->
<h3>Custom Filter: inr (Indian Rupee Format)</h3>

<div class="box">
    <div class="label">Salary</div>
    <div class="value">{{ salary | inr }}</div>
    <code>{{ "{{ salary | inr }}" }}</code>
</div>

<div class="box">
    <div class="label">Product Price</div>
    <div class="value">{{ product_price | inr }}</div>
    <code>{{ "{{ product_price | inr }}" }}</code>
</div>

<!-- ─── Time Ago Filter ───────────────────────────────── -->
<h3>Custom Filter: timeago</h3>

<div class="box">
    <div class="label">Post was created</div>
    <div class="value">{{ post_time | timeago }}</div>
    <code>{{ "{{ post_time | timeago }}" }}</code>
</div>

<div class="box">
    <div class="label">Last login</div>
    <div class="value">{{ login_time | timeago }}</div>
</div>

<div class="box">
    <div class="label">Old date</div>
    <div class="value">{{ old_date | timeago }}</div>
</div>

<!-- ─── Initials + Masked Email ──────────────────────── -->
<h3>Custom Filters: initials + mask_email</h3>

<table>
    <thead>
        <tr>
            <th>Avatar</th>
            <th>Full Name</th>
            <th>Masked Email</th>
        </tr>
    </thead>
    <tbody>
        {% for user in users %}
        <tr>
            <td><span class="avatar">{{ user.name | initials }}</span></td>
            <td>{{ user.name }}</td>
            <td>{{ user.email | mask_email }}</td>
        </tr>
        {% endfor %}
    </tbody>
</table>

<!-- ─── Pluralise Filter ──────────────────────────────── -->
<h3>Custom Filter: pluralise</h3>

<div class="box">
    <div class="label">Cart</div>
    <div class="value">{{ cart_count | pluralise("item") }}</div>
    <code>{{ "{{ cart_count | pluralise('item') }}" }}</code>
</div>

<div class="box">
    <div class="label">Files</div>
    <div class="value">{{ file_count | pluralise("file") }}</div>
</div>

<div class="box">
    <div class="label">Using custom plural form</div>
    <div class="value">{{ 3 | pluralise("child", "children") }}</div>
    <code>{{ "{{ 3 | pluralise('child', 'children') }}" }}</code>
</div>

</body>
</html>
```

---

## 8. Full Reference — All Built-in Filters

### String Filters

| Filter | Example | Result |
|---|---|---|
| `upper` | `{{ "hello" \| upper }}` | `HELLO` |
| `lower` | `{{ "HELLO" \| lower }}` | `hello` |
| `title` | `{{ "hello world" \| title }}` | `Hello World` |
| `capitalize` | `{{ "hello world" \| capitalize }}` | `Hello world` |
| `strip` | `{{ "  hi  " \| strip }}` | `hi` |
| `replace(old, new)` | `{{ "cat" \| replace("c","b") }}` | `bat` |
| `truncate(n)` | `{{ "long text..." \| truncate(8) }}` | `long ...` |
| `length` | `{{ "hello" \| length }}` | `5` |
| `reverse` | `{{ "abc" \| reverse }}` | `cba` |
| `wordcount` | `{{ "one two three" \| wordcount }}` | `3` |

### Number Filters

| Filter | Example | Result |
|---|---|---|
| `round(n)` | `{{ 3.14159 \| round(2) }}` | `3.14` |
| `int` | `{{ 3.9 \| int }}` | `3` |
| `float` | `{{ "3" \| float }}` | `3.0` |
| `abs` | `{{ -5 \| abs }}` | `5` |

### List Filters

| Filter | Example | Result |
|---|---|---|
| `length` | `{{ [1,2,3] \| length }}` | `3` |
| `first` | `{{ [1,2,3] \| first }}` | `1` |
| `last` | `{{ [1,2,3] \| last }}` | `3` |
| `join(sep)` | `{{ [1,2,3] \| join("-") }}` | `1-2-3` |
| `sort` | `{{ ["b","a","c"] \| sort }}` | `['a','b','c']` |
| `reverse` | `{{ [1,2,3] \| reverse \| list }}` | `[3,2,1]` |
| `unique` | `{{ [1,1,2,2,3] \| unique \| list }}` | `[1,2,3]` |
| `min` | `{{ [3,1,2] \| min }}` | `1` |
| `max` | `{{ [3,1,2] \| max }}` | `3` |
| `sum` | `{{ [1,2,3] \| sum }}` | `6` |
| `random` | `{{ [1,2,3] \| random }}` | random item |

### Safety Filters

| Filter | What it does |
|---|---|
| `escape` or `e` | Converts `<`, `>`, `&` to HTML entities — **prevents XSS** |
| `safe` | Marks a string as safe — renders HTML tags as actual HTML |
| `default(val)` | Returns `val` if the variable is `None` or undefined |
| `default(val, boolean=True)` | Returns `val` if falsy (`None`, `""`, `0`, `False`) |

---

## 9. Whitespace Control

By default, Jinja2 preserves newlines around `{% %}` blocks, which can produce messy HTML. Add `-` to trim whitespace:

```html
{%- if user %}       ← removes whitespace BEFORE this tag
    Hello!
{% endif -%}         ← removes whitespace AFTER this tag
```

For most beginner use cases, you don't need to worry about this — browsers ignore extra whitespace in HTML.

---

## 10. Common Mistakes

### Mistake 1: Using `safe` on user-generated content

```html
<!-- NEVER do this if the content came from a user -->
{{ user_comment | safe }}

<!-- This is fine only for your own trusted HTML -->
{{ my_app_html_variable | safe }}
```

`safe` disables XSS protection. Only use it on content you fully control and trust.

---

### Mistake 2: Forgetting `{% endmacro %}`

```html
<!-- WRONG -->
{% macro render_badge(role) %}
    <span>{{ role }}</span>
{# forgot endmacro! #}

<!-- CORRECT -->
{% macro render_badge(role) %}
    <span>{{ role }}</span>
{% endmacro %}
```

---

### Mistake 3: Calling a macro before defining it

```html
<!-- WRONG: calling before defining -->
{{ render_badge("admin") }}

{% macro render_badge(role) %}
    <span>{{ role }}</span>
{% endmacro %}

<!-- CORRECT: define first, call after -->
{% macro render_badge(role) %}
    <span>{{ role }}</span>
{% endmacro %}

{{ render_badge("admin") }}
```

---

### Mistake 4: Using `|safe` instead of `|escape` for user input

```html
<!-- XSS vulnerability! -->
Search results for: {{ query | safe }}

<!-- Safe: -->
Search results for: {{ query | escape }}
```

---

### Mistake 5: Forgetting `@app.template_filter()` decorator name

```python
# WRONG: the filter name in the decorator and the usage must match
@app.template_filter("inr_format")
def inr_format(value):
    ...

# In template: {{ price | inr_format }}   ← uses decorator name "inr_format"

# CORRECT: keep them consistent
@app.template_filter("inr")    ← short name
def inr_format(value):         ← function name can differ
    ...

# In template: {{ price | inr }}          ← uses decorator name "inr"
```

---

## 11. Interview Questions

**Q1: What is a Jinja2 filter?**
A: A transformation applied to a value before displaying it, using the pipe syntax `{{ value | filter }}`. Filters are built-in (like `upper`, `length`, `default`) or custom functions registered with `@app.template_filter()`.

**Q2: What is the difference between `| escape` and `| safe`?**
A: `escape` converts HTML special characters (`<`, `>`, `&`) into safe HTML entities, preventing XSS. `safe` does the opposite — it marks content as trusted HTML, disabling escaping. Use `safe` only on content you fully control; never on user input.

**Q3: What is a Jinja2 macro?**
A: A reusable template component defined with `{% macro name(args) %}...{% endmacro %}` and called like a function. It's used to avoid repeating the same HTML block (like a form input or card) across a template.

**Q4: How do you add a custom Python function as a Jinja2 filter?**
A: Decorate the function with `@app.template_filter("filter_name")`. The function receives the value being filtered as its first argument and returns the transformed value.

**Q5: What does `{{ bio | default("No bio.") }}` do?**
A: If `bio` is `None` or undefined, it displays `"No bio."` instead. With `boolean=True`, it also falls back on `""`, `0`, and `False`.

---

## 12. Best Practices

- Use built-in filters whenever possible — they're optimized and well-tested.
- **Never use `| safe` on user-generated content** — it's the number one XSS mistake.
- Define macros at the **top** of the template (or in a separate `macros.html` file you import).
- Keep custom filters **pure functions** — they receive a value, transform it, and return it — no side effects.
- Use `| default("fallback")` whenever a variable might be `None` — prevents ugly "None" showing in your UI.
- Use `| e` (shorthand for `| escape`) liberally on any data that came from user input.

---

## 13. Mini Project — Product Catalog

Build a complete product catalog with:

**`app.py`:**
- A `/catalog` route that passes a list of 6+ products with `name`, `price`, `discount`, `category`, `in_stock`, `created_at` (datetime)
- A custom filter `discount_price` that calculates the final price after discount

**`templates/catalog.html`:**
- Display all products in cards
- Use `| inr` for prices (or your own currency format)
- Use `| timeago` to show when each product was added
- Use `| upper` on category names
- Use a macro `render_product_card(product)` for the card layout
- Show "OUT OF STOCK" badge if `in_stock` is `False`
- Show a "X% OFF" badge if discount > 0

---

## 14. Practice Exercises

**5 Easy Questions**
1. What syntax applies a filter to a variable in Jinja2?
2. Which filter converts a string to all capitals?
3. Which filter shows a fallback value when a variable is `None`?
4. True/False: `| safe` should be used on all user-submitted content.
5. What keyword starts and ends a macro definition?

**5 Medium Questions**
1. How do you chain two filters together? Give an example.
2. What is the difference between `| length` on a string vs on a list?
3. Write a Jinja2 line that joins a list `["a", "b", "c"]` into the string `"a | b | c"`.
4. How do you register a custom Python function as a template filter?
5. Inside a `{% for %}` loop, how do you check if the current item is the last one?

**5 Hard Questions**
1. Why is Jinja2's `| escape` filter applied automatically in Flask, and when would it NOT be applied?
2. Explain how `{% set %}` inside a `{% for %}` loop behaves differently to what Python developers expect, and what `namespace()` solves.
3. A macro takes `(label, type="text", error="")` as arguments. How would you call it with only `label` and `error`, skipping `type`?
4. What's the difference between `{{ bio | default("N/A") }}` and `{{ bio | default("N/A", boolean=True) }}`?
5. How would you make a custom filter available globally in all templates vs only applying it on a specific variable?

**2 Debugging Questions**
1. A developer's custom filter `@app.template_filter("format_price")` is defined but the template throws `no filter named 'format_price'`. What are two likely causes?
2. A template displays `&lt;strong&gt;Hello&lt;/strong&gt;` instead of **Hello**. The developer is confused because they put real HTML in the variable. What is happening and how do you fix it?

**2 Interview Questions**
1. "What are Jinja2 filters and macros? When would you use each?"
2. "What is the security risk with `| safe` and how would you handle user-submitted HTML safely?"

---

## 15. Summary

| Feature | Syntax | Purpose |
|---|---|---|
| Filter | `{{ value \| filter }}` | Transform a value before displaying |
| Filter with args | `{{ value \| filter(arg) }}` | Transform with options |
| Chain filters | `{{ value \| f1 \| f2 }}` | Apply multiple transforms |
| Test | `{% if value is odd %}` | Check type or condition |
| Set variable | `{% set x = value %}` | Create a template variable |
| Macro define | `{% macro name(args) %}...{% endmacro %}` | Create reusable component |
| Macro call | `{{ name(args) }}` | Use the component |
| Custom filter | `@app.template_filter("name")` | Add Python function as filter |

### 💡 Memory Trick
**"Filters are like Instagram filters — they transform how your data looks without changing the original. Macros are like reusable stamp templates — define once, use everywhere."**

---

**End of Module 12.**
