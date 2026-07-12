# MODULE 14 — Forms

> **Course:** Flask — Zero to Production
> **Module:** 14 of 39
> **Style:** Practical-first, complete code + complete HTML files

---

## 1. Two Ways to Handle Forms in Flask

There are two approaches to forms in Flask:

**Approach 1 — Raw HTML Forms (no library)**
- You write the HTML form yourself
- You read `request.form` manually
- You write your own validation logic
- Good for: simple forms, quick prototypes

**Approach 2 — Flask-WTF (recommended for real apps)**
- A library that handles form creation, validation, and CSRF protection
- Forms are defined as Python classes
- Validation rules are written in Python — not in the template
- Good for: production apps, complex forms, consistent validation

This module covers **both** — starting with raw forms so you understand what's happening, then Flask-WTF which is what you'd use in real projects.

---

## 2. Project Structure

```
my_app/
├── app.py
├── forms.py                ← Flask-WTF form classes (Part 2)
└── templates/
    ├── base.html
    ├── raw_login.html
    ├── raw_register.html
    ├── wtf_login.html
    ├── wtf_register.html
    └── success.html
```

---

## 3. PART 1 — Raw HTML Forms (No Library)

### Example 1 — Login Form (Raw)

### `app.py`

```python
from flask import Flask, render_template, request, redirect, url_for, session

app = Flask(__name__)
app.secret_key = "dev-secret-key-change-in-production"

# Fake user database
USERS = {
    "arjun@test.com": {"name": "Arjun Sharma",  "password": "password123"},
    "priya@test.com": {"name": "Priya Patel",   "password": "mypassword"},
}


@app.route("/")
def home():
    user = session.get("user")
    return render_template("base.html", user=user)


@app.route("/raw/login", methods=["GET", "POST"])
def raw_login():
    errors = {}
    form_data = {}

    if request.method == "POST":
        email    = request.form.get("email", "").strip()
        password = request.form.get("password", "").strip()

        # Keep form values to refill the form on error
        form_data["email"] = email

        # Validate fields
        if not email:
            errors["email"] = "Email is required."
        elif "@" not in email or "." not in email:
            errors["email"] = "Enter a valid email address."

        if not password:
            errors["password"] = "Password is required."

        # Only check credentials if no field errors
        if not errors:
            user = USERS.get(email)
            if not user or user["password"] != password:
                errors["general"] = "Invalid email or password."
            else:
                session["user"] = user["name"]
                return redirect(url_for("raw_success"))

    return render_template("raw_login.html", errors=errors, form_data=form_data)


@app.route("/raw/register", methods=["GET", "POST"])
def raw_register():
    errors = {}
    form_data = {}

    if request.method == "POST":
        name     = request.form.get("name", "").strip()
        email    = request.form.get("email", "").strip()
        password = request.form.get("password", "").strip()
        confirm  = request.form.get("confirm_password", "").strip()
        agree    = request.form.get("agree")   # checkbox: "on" or None

        # Keep form values
        form_data = {"name": name, "email": email}

        # Validate
        if not name:
            errors["name"] = "Full name is required."
        elif len(name) < 2:
            errors["name"] = "Name must be at least 2 characters."

        if not email:
            errors["email"] = "Email is required."
        elif "@" not in email:
            errors["email"] = "Enter a valid email address."
        elif email in USERS:
            errors["email"] = "This email is already registered."

        if not password:
            errors["password"] = "Password is required."
        elif len(password) < 8:
            errors["password"] = "Password must be at least 8 characters."

        if not confirm:
            errors["confirm_password"] = "Please confirm your password."
        elif password and confirm and password != confirm:
            errors["confirm_password"] = "Passwords do not match."

        if not agree:
            errors["agree"] = "You must accept the terms and conditions."

        if not errors:
            # In real app: hash password and save to database
            USERS[email] = {"name": name, "password": password}
            session["user"] = name
            return redirect(url_for("raw_success"))

    return render_template("raw_register.html", errors=errors, form_data=form_data)


@app.route("/raw/success")
def raw_success():
    user = session.get("user", "Guest")
    return render_template("success.html", user=user)


@app.route("/logout")
def logout():
    session.clear()
    return redirect(url_for("raw_login"))


if __name__ == "__main__":
    app.run(debug=True)
```

---

### `templates/base.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Flask Forms{% endblock %}</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: 'Segoe UI', Arial, sans-serif;
            background: #f0f2f5;
            min-height: 100vh;
        }
        nav {
            background: #2d3436;
            padding: 0 30px;
            height: 56px;
            display: flex;
            align-items: center;
            justify-content: space-between;
        }
        .nav-brand { color: #74b9ff; font-weight: bold; font-size: 18px; text-decoration: none; }
        .nav-links a { color: #dfe6e9; text-decoration: none; margin-left: 20px; font-size: 14px; }
        .nav-links a:hover { color: #74b9ff; }
        .container {
            max-width: 960px;
            margin: 40px auto;
            padding: 0 20px;
        }
        .form-card {
            background: white;
            border-radius: 14px;
            padding: 36px 40px;
            box-shadow: 0 4px 20px rgba(0,0,0,0.08);
            max-width: 480px;
            margin: 0 auto;
        }
        .form-title { font-size: 26px; font-weight: bold; margin-bottom: 6px; color: #2d3436; }
        .form-sub   { color: #636e72; margin-bottom: 28px; font-size: 14px; }
        .form-group { margin-bottom: 20px; }
        .form-group label {
            display: block;
            font-weight: 600;
            margin-bottom: 6px;
            color: #2d3436;
            font-size: 14px;
        }
        .form-group input[type="text"],
        .form-group input[type="email"],
        .form-group input[type="password"],
        .form-group input[type="number"],
        .form-group select,
        .form-group textarea {
            width: 100%;
            padding: 11px 14px;
            border: 1.5px solid #dfe6e9;
            border-radius: 8px;
            font-size: 15px;
            font-family: inherit;
            transition: border-color 0.2s, box-shadow 0.2s;
            background: #fafafa;
        }
        .form-group input:focus,
        .form-group select:focus,
        .form-group textarea:focus {
            outline: none;
            border-color: #0984e3;
            background: white;
            box-shadow: 0 0 0 3px rgba(9,132,227,0.12);
        }
        .form-group input.error-field {
            border-color: #d63031;
            background: #fff5f5;
        }
        .form-group textarea { height: 110px; resize: vertical; }
        .error-msg {
            color: #d63031;
            font-size: 12px;
            margin-top: 5px;
            display: flex;
            align-items: center;
            gap: 4px;
        }
        .alert {
            padding: 13px 16px;
            border-radius: 8px;
            margin-bottom: 22px;
            font-size: 14px;
        }
        .alert-danger  { background: #ffeaea; color: #c0392b; border-left: 4px solid #d63031; }
        .alert-success { background: #eafaf1; color: #1e8449; border-left: 4px solid #00b894; }
        .btn {
            display: inline-block;
            padding: 11px 22px;
            border-radius: 8px;
            border: none;
            font-size: 15px;
            font-family: inherit;
            cursor: pointer;
            text-decoration: none;
            transition: opacity 0.2s, transform 0.1s;
        }
        .btn:active { transform: scale(0.98); }
        .btn:hover  { opacity: 0.88; }
        .btn-primary  { background: #0984e3; color: white; }
        .btn-success  { background: #00b894; color: white; }
        .btn-block    { width: 100%; text-align: center; }
        .divider { text-align: center; color: #b2bec3; margin: 18px 0; font-size: 13px; }
        .link-row { text-align: center; font-size: 14px; color: #636e72; margin-top: 18px; }
        .link-row a { color: #0984e3; text-decoration: none; font-weight: 600; }
        .checkbox-row {
            display: flex;
            align-items: flex-start;
            gap: 10px;
        }
        .checkbox-row input[type="checkbox"] { margin-top: 3px; cursor: pointer; }
        .checkbox-row label { font-weight: normal; cursor: pointer; font-size: 14px; color: #636e72; }
        .checkbox-row label a { color: #0984e3; }
    </style>
    {% block extra_css %}{% endblock %}
</head>
<body>
    <nav>
        <a href="/" class="nav-brand">⚡ FlaskForms</a>
        <div class="nav-links">
            <a href="/raw/login">Login</a>
            <a href="/raw/register">Register</a>
            {% if user %}<a href="/logout">Logout ({{ user }})</a>{% endif %}
        </div>
    </nav>
    <div class="container">
        {% block content %}{% endblock %}
    </div>
</body>
</html>
```

---

### `templates/raw_login.html`

```html
{% extends "base.html" %}
{% block title %}Login — FlaskForms{% endblock %}

{% block content %}
<div class="form-card">

    <div class="form-title">Welcome back 👋</div>
    <div class="form-sub">Sign in to your account</div>

    {# Show general error (wrong credentials) #}
    {% if errors.general %}
        <div class="alert alert-danger">
            🔒 {{ errors.general }}
        </div>
    {% endif %}

    <form method="POST" action="/raw/login" novalidate>

        <!-- Email Field -->
        <div class="form-group">
            <label for="email">Email Address</label>
            <input
                type="email"
                id="email"
                name="email"
                placeholder="you@example.com"
                value="{{ form_data.get('email', '') }}"
                class="{{ 'error-field' if errors.email }}"
            >
            {% if errors.email %}
                <div class="error-msg">⚠ {{ errors.email }}</div>
            {% endif %}
        </div>

        <!-- Password Field -->
        <div class="form-group">
            <label for="password">Password</label>
            <input
                type="password"
                id="password"
                name="password"
                placeholder="Enter your password"
                class="{{ 'error-field' if errors.password }}"
            >
            {% if errors.password %}
                <div class="error-msg">⚠ {{ errors.password }}</div>
            {% endif %}
        </div>

        <!-- Submit Button -->
        <button type="submit" class="btn btn-primary btn-block">
            Sign In →
        </button>

    </form>

    <div class="divider">— or —</div>

    <div class="link-row">
        Don't have an account? <a href="/raw/register">Create one</a>
    </div>

    <div class="link-row" style="margin-top: 10px;">
        <small style="color: #b2bec3;">
            Test: arjun@test.com / password123
        </small>
    </div>

</div>
{% endblock %}
```

---

### `templates/raw_register.html`

```html
{% extends "base.html" %}
{% block title %}Register — FlaskForms{% endblock %}

{% block content %}
<div class="form-card">

    <div class="form-title">Create Account 🎉</div>
    <div class="form-sub">Join us today — it's free!</div>

    <form method="POST" action="/raw/register" novalidate>

        <!-- Full Name -->
        <div class="form-group">
            <label for="name">Full Name</label>
            <input
                type="text"
                id="name"
                name="name"
                placeholder="Arjun Sharma"
                value="{{ form_data.get('name', '') }}"
                class="{{ 'error-field' if errors.name }}"
            >
            {% if errors.name %}
                <div class="error-msg">⚠ {{ errors.name }}</div>
            {% endif %}
        </div>

        <!-- Email -->
        <div class="form-group">
            <label for="email">Email Address</label>
            <input
                type="email"
                id="email"
                name="email"
                placeholder="you@example.com"
                value="{{ form_data.get('email', '') }}"
                class="{{ 'error-field' if errors.email }}"
            >
            {% if errors.email %}
                <div class="error-msg">⚠ {{ errors.email }}</div>
            {% endif %}
        </div>

        <!-- Password -->
        <div class="form-group">
            <label for="password">Password</label>
            <input
                type="password"
                id="password"
                name="password"
                placeholder="At least 8 characters"
                class="{{ 'error-field' if errors.password }}"
            >
            {% if errors.password %}
                <div class="error-msg">⚠ {{ errors.password }}</div>
            {% endif %}
        </div>

        <!-- Confirm Password -->
        <div class="form-group">
            <label for="confirm_password">Confirm Password</label>
            <input
                type="password"
                id="confirm_password"
                name="confirm_password"
                placeholder="Repeat your password"
                class="{{ 'error-field' if errors.confirm_password }}"
            >
            {% if errors.confirm_password %}
                <div class="error-msg">⚠ {{ errors.confirm_password }}</div>
            {% endif %}
        </div>

        <!-- Terms Checkbox -->
        <div class="form-group">
            <div class="checkbox-row">
                <input type="checkbox" id="agree" name="agree">
                <label for="agree">
                    I agree to the <a href="#">Terms of Service</a>
                    and <a href="#">Privacy Policy</a>
                </label>
            </div>
            {% if errors.agree %}
                <div class="error-msg" style="margin-top: 6px;">⚠ {{ errors.agree }}</div>
            {% endif %}
        </div>

        <button type="submit" class="btn btn-success btn-block">
            Create My Account →
        </button>

    </form>

    <div class="link-row" style="margin-top: 18px;">
        Already have an account? <a href="/raw/login">Sign in</a>
    </div>

</div>
{% endblock %}
```

---

### `templates/success.html`

```html
{% extends "base.html" %}
{% block title %}Welcome!{% endblock %}

{% block content %}
<div style="text-align: center; padding: 60px 20px;">
    <div style="font-size: 72px; margin-bottom: 20px;">🎉</div>
    <h1 style="font-size: 36px; color: #2d3436; margin-bottom: 12px;">
        Welcome, {{ user }}!
    </h1>
    <p style="color: #636e72; font-size: 18px; margin-bottom: 30px;">
        You have successfully logged in (or registered).
    </p>
    <a href="/logout" class="btn btn-primary">Logout</a>
</div>
{% endblock %}
```

---

## 4. How Raw Form Validation Works — Diagram

```
Browser: POST /raw/login
         email=arjun@test.com
         password=wrongpass
              ↓
Flask receives → request.form populated
              ↓
View function runs validation:
  ├── email empty?     → NO
  ├── email valid?     → YES
  ├── password empty?  → NO
  └── credentials OK?  → NO → errors["general"] = "Invalid..."
              ↓
errors dict is NOT empty
              ↓
render_template("raw_login.html", errors=errors, form_data=form_data)
              ↓
Template shows form AGAIN with:
  - error messages in red
  - email field re-filled (from form_data)
  - password field empty (never re-fill passwords)
```

---

## 5. PART 2 — Flask-WTF (The Professional Way)

### Why Flask-WTF?

| Raw Forms | Flask-WTF |
|---|---|
| Validation logic mixed in view function | Validation rules in the Form class |
| No automatic CSRF protection | Automatic CSRF token on every form |
| Easy to forget validation rules | Reusable form classes |
| Manual error handling | Built-in error messages |
| Gets messy with many forms | Clean and organised |

### Install It

```bash
pip install flask-wtf
```

---

### 5.1 What is CSRF?

**CSRF (Cross-Site Request Forgery)** is an attack where a malicious website tricks your browser into submitting a form to YOUR site using YOUR cookies.

**Example of the attack:**
1. You are logged in to `yourbank.com`
2. You visit `evil.com` which has a hidden form that submits to `yourbank.com/transfer`
3. Your browser sends your `yourbank.com` cookie along with the request
4. The bank thinks it's you — and transfers your money

**Flask-WTF's fix:** Every form gets a secret, unique **CSRF token** (a random string). The form submission must include this token. An attacker can't know this token because it's different every time and hidden in the form. Without the token, the request is rejected.

---

### `forms.py`

```python
from flask_wtf import FlaskForm
from wtforms import (
    StringField,
    EmailField,
    PasswordField,
    TextAreaField,
    SelectField,
    BooleanField,
    IntegerField,
    SubmitField,
)
from wtforms.validators import (
    DataRequired,
    Email,
    Length,
    EqualTo,
    NumberRange,
    Optional,
    Regexp,
)
from wtforms import ValidationError


# ─── LOGIN FORM ───────────────────────────────────────────────────────────────
class LoginForm(FlaskForm):
    email    = EmailField("Email Address",
                          validators=[DataRequired(message="Email is required."),
                                      Email(message="Enter a valid email address.")])
    password = PasswordField("Password",
                             validators=[DataRequired(message="Password is required.")])
    submit   = SubmitField("Sign In")


# ─── REGISTRATION FORM ────────────────────────────────────────────────────────
class RegisterForm(FlaskForm):
    name     = StringField("Full Name",
                           validators=[DataRequired(message="Name is required."),
                                       Length(min=2, max=80, message="Name must be 2–80 characters.")])

    email    = EmailField("Email Address",
                          validators=[DataRequired(message="Email is required."),
                                      Email(message="Enter a valid email address.")])

    password = PasswordField("Password",
                             validators=[DataRequired(message="Password is required."),
                                         Length(min=8, message="Password must be at least 8 characters.")])

    confirm  = PasswordField("Confirm Password",
                             validators=[DataRequired(message="Please confirm your password."),
                                         EqualTo("password", message="Passwords must match.")])

    agree    = BooleanField("I agree to the Terms of Service",
                            validators=[DataRequired(message="You must accept the terms.")])

    submit   = SubmitField("Create Account")


# ─── PROFILE FORM ─────────────────────────────────────────────────────────────
class ProfileForm(FlaskForm):
    name    = StringField("Full Name",
                          validators=[DataRequired(), Length(min=2, max=80)])

    bio     = TextAreaField("Bio",
                            validators=[Optional(), Length(max=300, message="Bio max 300 characters.")])

    age     = IntegerField("Age",
                           validators=[Optional(),
                                       NumberRange(min=13, max=120, message="Age must be between 13–120.")])

    gender  = SelectField("Gender",
                          choices=[("", "— Select —"), ("male", "Male"), ("female", "Female"), ("other", "Other")],
                          validators=[Optional()])

    website = StringField("Website URL",
                          validators=[Optional(),
                                      Regexp(r'^https?://', message="URL must start with http:// or https://")])

    submit  = SubmitField("Save Profile")


# ─── CONTACT FORM ─────────────────────────────────────────────────────────────
class ContactForm(FlaskForm):
    name    = StringField("Name",
                          validators=[DataRequired(), Length(min=2, max=80)])

    email   = EmailField("Email",
                         validators=[DataRequired(), Email()])

    subject = StringField("Subject",
                          validators=[DataRequired(), Length(min=5, max=120)])

    message = TextAreaField("Message",
                            validators=[DataRequired(), Length(min=20, max=2000,
                                                              message="Message must be 20–2000 characters.")])

    submit  = SubmitField("Send Message")
```

---

### Updated `app.py` (adding Flask-WTF routes)

```python
from flask import Flask, render_template, request, redirect, url_for, session, flash
from forms import LoginForm, RegisterForm, ProfileForm, ContactForm

app = Flask(__name__)
app.secret_key = "dev-secret-key-change-in-production"

# Flask-WTF requires this config key
app.config["WTF_CSRF_ENABLED"] = True

# Fake user database
USERS = {
    "arjun@test.com": {"name": "Arjun Sharma", "password": "password123"},
    "priya@test.com": {"name": "Priya Patel",  "password": "mypassword"},
}


# ─── RAW FORM ROUTES (from Part 1) ────────────────────────────────────────────
@app.route("/")
def home():
    return redirect(url_for("wtf_login"))


@app.route("/logout")
def logout():
    session.clear()
    return redirect(url_for("wtf_login"))


# ─── FLASK-WTF: LOGIN ─────────────────────────────────────────────────────────
@app.route("/login", methods=["GET", "POST"])
def wtf_login():
    form = LoginForm()   # create an instance of the form

    if form.validate_on_submit():   # True only on valid POST
        email    = form.email.data
        password = form.password.data

        user = USERS.get(email)
        if not user or user["password"] != password:
            form.email.errors.append("Invalid email or password.")
        else:
            session["user"] = user["name"]
            flash(f"Welcome back, {user['name']}! 👋", "success")
            return redirect(url_for("wtf_success"))

    return render_template("wtf_login.html", form=form)


# ─── FLASK-WTF: REGISTER ──────────────────────────────────────────────────────
@app.route("/register", methods=["GET", "POST"])
def wtf_register():
    form = RegisterForm()

    if form.validate_on_submit():
        email = form.email.data
        if email in USERS:
            form.email.errors.append("This email is already registered.")
        else:
            USERS[email] = {
                "name":     form.name.data,
                "password": form.password.data
            }
            session["user"] = form.name.data
            flash(f"Account created! Welcome, {form.name.data}! 🎉", "success")
            return redirect(url_for("wtf_success"))

    return render_template("wtf_register.html", form=form)


# ─── FLASK-WTF: PROFILE ───────────────────────────────────────────────────────
@app.route("/profile", methods=["GET", "POST"])
def profile():
    form = ProfileForm()

    if form.validate_on_submit():
        flash("Profile saved successfully!", "success")
        return redirect(url_for("profile"))

    return render_template("wtf_profile.html", form=form)


# ─── FLASK-WTF: CONTACT ───────────────────────────────────────────────────────
@app.route("/contact", methods=["GET", "POST"])
def contact():
    form = ContactForm()

    if form.validate_on_submit():
        flash("Your message has been sent! We'll reply soon. 📬", "success")
        return redirect(url_for("contact"))

    return render_template("wtf_contact.html", form=form)


# ─── SUCCESS PAGE ─────────────────────────────────────────────────────────────
@app.route("/success")
def wtf_success():
    user = session.get("user", "Guest")
    return render_template("success.html", user=user)


if __name__ == "__main__":
    app.run(debug=True)
```

---

### `templates/wtf_login.html`

```html
{% extends "base.html" %}
{% block title %}Login — Flask-WTF{% endblock %}

{% block content %}
<div class="form-card">

    <div class="form-title">Welcome back 👋</div>
    <div class="form-sub">Sign in with Flask-WTF</div>

    {# Flash messages #}
    {% with messages = get_flashed_messages(with_categories=True) %}
        {% for category, message in messages %}
            <div class="alert alert-{{ category }}">{{ message }}</div>
        {% endfor %}
    {% endwith %}

    <form method="POST" novalidate>

        {# CSRF Token — Flask-WTF adds this automatically #}
        {{ form.hidden_tag() }}

        <!-- Email -->
        <div class="form-group">
            {{ form.email.label }}
            {{ form.email(
                placeholder="you@example.com",
                class="error-field" if form.email.errors
            ) }}
            {% for error in form.email.errors %}
                <div class="error-msg">⚠ {{ error }}</div>
            {% endfor %}
        </div>

        <!-- Password -->
        <div class="form-group">
            {{ form.password.label }}
            {{ form.password(
                placeholder="Enter your password",
                class="error-field" if form.password.errors
            ) }}
            {% for error in form.password.errors %}
                <div class="error-msg">⚠ {{ error }}</div>
            {% endfor %}
        </div>

        <button type="submit" class="btn btn-primary btn-block">
            Sign In →
        </button>

    </form>

    <div class="link-row" style="margin-top: 18px;">
        No account? <a href="/register">Create one</a>
    </div>
    <div class="link-row" style="margin-top:8px;">
        <small style="color:#b2bec3;">Test: arjun@test.com / password123</small>
    </div>

</div>
{% endblock %}
```

---

### `templates/wtf_register.html`

```html
{% extends "base.html" %}
{% block title %}Register — Flask-WTF{% endblock %}

{% block content %}
<div class="form-card">

    <div class="form-title">Create Account 🎉</div>
    <div class="form-sub">Powered by Flask-WTF</div>

    <form method="POST" novalidate>
        {{ form.hidden_tag() }}

        <!-- Name -->
        <div class="form-group">
            {{ form.name.label }}
            {{ form.name(placeholder="Arjun Sharma",
                         class="error-field" if form.name.errors) }}
            {% for error in form.name.errors %}
                <div class="error-msg">⚠ {{ error }}</div>
            {% endfor %}
        </div>

        <!-- Email -->
        <div class="form-group">
            {{ form.email.label }}
            {{ form.email(placeholder="you@example.com",
                          class="error-field" if form.email.errors) }}
            {% for error in form.email.errors %}
                <div class="error-msg">⚠ {{ error }}</div>
            {% endfor %}
        </div>

        <!-- Password -->
        <div class="form-group">
            {{ form.password.label }}
            {{ form.password(placeholder="At least 8 characters",
                             class="error-field" if form.password.errors) }}
            {% for error in form.password.errors %}
                <div class="error-msg">⚠ {{ error }}</div>
            {% endfor %}
        </div>

        <!-- Confirm Password -->
        <div class="form-group">
            {{ form.confirm.label }}
            {{ form.confirm(placeholder="Repeat your password",
                            class="error-field" if form.confirm.errors) }}
            {% for error in form.confirm.errors %}
                <div class="error-msg">⚠ {{ error }}</div>
            {% endfor %}
        </div>

        <!-- Terms Checkbox -->
        <div class="form-group">
            <div class="checkbox-row">
                {{ form.agree() }}
                {{ form.agree.label }}
            </div>
            {% for error in form.agree.errors %}
                <div class="error-msg" style="margin-top:6px;">⚠ {{ error }}</div>
            {% endfor %}
        </div>

        <button type="submit" class="btn btn-success btn-block">
            Create My Account →
        </button>

    </form>

    <div class="link-row" style="margin-top: 18px;">
        Already registered? <a href="/login">Sign in</a>
    </div>

</div>
{% endblock %}
```

---

### `templates/wtf_profile.html`

```html
{% extends "base.html" %}
{% block title %}Edit Profile{% endblock %}

{% block content %}
<div class="form-card" style="max-width: 560px;">

    <div class="form-title">Edit Profile ✏️</div>
    <div class="form-sub">Update your personal information</div>

    {% with messages = get_flashed_messages(with_categories=True) %}
        {% for category, message in messages %}
            <div class="alert alert-{{ category }}">{{ message }}</div>
        {% endfor %}
    {% endwith %}

    <form method="POST" novalidate>
        {{ form.hidden_tag() }}

        <!-- Full Name -->
        <div class="form-group">
            {{ form.name.label }}
            {{ form.name(placeholder="Arjun Sharma",
                         class="error-field" if form.name.errors) }}
            {% for error in form.name.errors %}
                <div class="error-msg">⚠ {{ error }}</div>
            {% endfor %}
        </div>

        <!-- Bio (Textarea) -->
        <div class="form-group">
            {{ form.bio.label }}
            {{ form.bio(placeholder="Tell us about yourself...",
                        class="error-field" if form.bio.errors) }}
            {% for error in form.bio.errors %}
                <div class="error-msg">⚠ {{ error }}</div>
            {% endfor %}
        </div>

        <!-- Age (Number) -->
        <div class="form-group">
            {{ form.age.label }}
            {{ form.age(placeholder="25",
                        class="error-field" if form.age.errors) }}
            {% for error in form.age.errors %}
                <div class="error-msg">⚠ {{ error }}</div>
            {% endfor %}
        </div>

        <!-- Gender (Dropdown) -->
        <div class="form-group">
            {{ form.gender.label }}
            {{ form.gender(class="error-field" if form.gender.errors) }}
            {% for error in form.gender.errors %}
                <div class="error-msg">⚠ {{ error }}</div>
            {% endfor %}
        </div>

        <!-- Website URL -->
        <div class="form-group">
            {{ form.website.label }}
            {{ form.website(placeholder="https://yourwebsite.com",
                            class="error-field" if form.website.errors) }}
            {% for error in form.website.errors %}
                <div class="error-msg">⚠ {{ error }}</div>
            {% endfor %}
        </div>

        <button type="submit" class="btn btn-primary btn-block">
            Save Profile
        </button>

    </form>

</div>
{% endblock %}
```

---

### `templates/wtf_contact.html`

```html
{% extends "base.html" %}
{% block title %}Contact Us{% endblock %}

{% block content %}
<div class="form-card" style="max-width: 560px;">

    <div class="form-title">Contact Us 📬</div>
    <div class="form-sub">We'd love to hear from you</div>

    {% with messages = get_flashed_messages(with_categories=True) %}
        {% for category, message in messages %}
            <div class="alert alert-{{ category }}">{{ message }}</div>
        {% endfor %}
    {% endwith %}

    <form method="POST" novalidate>
        {{ form.hidden_tag() }}

        <div class="form-group">
            {{ form.name.label }}
            {{ form.name(placeholder="Arjun Sharma",
                         class="error-field" if form.name.errors) }}
            {% for error in form.name.errors %}
                <div class="error-msg">⚠ {{ error }}</div>
            {% endfor %}
        </div>

        <div class="form-group">
            {{ form.email.label }}
            {{ form.email(placeholder="you@example.com",
                          class="error-field" if form.email.errors) }}
            {% for error in form.email.errors %}
                <div class="error-msg">⚠ {{ error }}</div>
            {% endfor %}
        </div>

        <div class="form-group">
            {{ form.subject.label }}
            {{ form.subject(placeholder="What is this about?",
                            class="error-field" if form.subject.errors) }}
            {% for error in form.subject.errors %}
                <div class="error-msg">⚠ {{ error }}</div>
            {% endfor %}
        </div>

        <div class="form-group">
            {{ form.message.label }}
            {{ form.message(placeholder="Write your message here (min 20 characters)...",
                            class="error-field" if form.message.errors) }}
            {% for error in form.message.errors %}
                <div class="error-msg">⚠ {{ error }}</div>
            {% endfor %}
        </div>

        <button type="submit" class="btn btn-primary btn-block">
            Send Message →
        </button>

    </form>

</div>
{% endblock %}
```

---

## 6. WTForms Validators — Complete Reference

| Validator | Import | What it does |
|---|---|---|
| `DataRequired()` | wtforms.validators | Field must not be empty |
| `Optional()` | wtforms.validators | Field is optional — skip other validators if empty |
| `Email()` | wtforms.validators | Must be valid email format |
| `Length(min, max)` | wtforms.validators | String length between min and max |
| `EqualTo("field")` | wtforms.validators | Must match another field (for confirm password) |
| `NumberRange(min, max)` | wtforms.validators | Number must be in range |
| `URL()` | wtforms.validators | Must be a valid URL |
| `Regexp(pattern)` | wtforms.validators | Must match a regex pattern |
| `AnyOf(values)` | wtforms.validators | Value must be one of a list |
| `NoneOf(values)` | wtforms.validators | Value must NOT be one of a list |
| `InputRequired()` | wtforms.validators | Like DataRequired but also catches `0` and `False` |

---

## 7. WTForms Field Types — Complete Reference

| Field | Class | Use for |
|---|---|---|
| Text | `StringField` | Names, usernames, titles |
| Email | `EmailField` | Email addresses (validates format) |
| Password | `PasswordField` | Passwords (masked input) |
| Textarea | `TextAreaField` | Long text, messages, bios |
| Number | `IntegerField` | Whole numbers (age, quantity) |
| Decimal | `DecimalField` | Decimal numbers (price) |
| Dropdown | `SelectField` | Single choice from options |
| Multi-select | `SelectMultipleField` | Multiple choices |
| Checkbox | `BooleanField` | True/False, agree/disagree |
| Radio | `RadioField` | Choose one from visible options |
| File | `FileField` | File uploads |
| Hidden | `HiddenField` | Hidden values |
| Submit | `SubmitField` | Submit button |

---

## 8. `form.validate_on_submit()` — How It Works

```python
if form.validate_on_submit():
```

This single line does TWO checks:

1. **Is this a POST request?** (`request.method == "POST"`)
2. **Did all validators pass?** (checks all field rules)

Only if BOTH are true does it return `True`. If the page was just loaded (GET), or if any validator failed (POST with errors), it returns `False` and the template is rendered with errors shown.

```
GET /login
    ↓
form.validate_on_submit() → False (it's a GET, not POST)
    ↓
render template with empty form


POST /login (bad data)
    ↓
form.validate_on_submit()
    ↓
  Is POST? YES
  Is valid? NO (email is wrong format)
    ↓
Returns False
    ↓
render template with errors shown in red


POST /login (good data)
    ↓
form.validate_on_submit()
    ↓
  Is POST? YES
  Is valid? YES
    ↓
Returns True
    ↓
Your business logic runs
    ↓
Redirect to success page
```

---

## 9. Flash Messages

`flash()` stores a temporary message that shows on the **next page** (after a redirect). It's perfect for "Account created!" or "Login failed" messages.

```python
# In view function
from flask import flash

flash("Login successful!", "success")   # message + category
flash("Invalid password.", "danger")    # categories match CSS class names
return redirect(url_for("dashboard"))
```

```html
<!-- In template (usually in base.html so it shows on every page) -->
{% with messages = get_flashed_messages(with_categories=True) %}
    {% for category, message in messages %}
        <div class="alert alert-{{ category }}">
            {{ message }}
        </div>
    {% endfor %}
{% endwith %}
```

Flash messages are stored in the **session** and are automatically deleted after they are displayed once.

---

## 10. Common Mistakes

### Mistake 1: Forgetting `form.hidden_tag()` in the template

```html
<!-- WRONG: no CSRF token — will get 400 Bad Request -->
<form method="POST">
    {{ form.email() }}
    ...
</form>

<!-- CORRECT: always include hidden_tag() -->
<form method="POST">
    {{ form.hidden_tag() }}
    {{ form.email() }}
    ...
</form>
```

---

### Mistake 2: Forgetting `secret_key`

Flask-WTF needs `app.secret_key` to sign the CSRF token. Without it you get a `RuntimeError`.

```python
# WRONG: no secret key
app = Flask(__name__)

# CORRECT: always set a secret key
app = Flask(__name__)
app.secret_key = "your-secret-key-here"
```

---

### Mistake 3: Using `form.validate()` instead of `form.validate_on_submit()`

```python
# WRONG: form.validate() runs on GET requests too, always triggering errors
if form.validate():
    ...

# CORRECT: only validates on POST
if form.validate_on_submit():
    ...
```

---

### Mistake 4: Reading form data incorrectly

```python
# WRONG: raw request.form — bypasses WTForms
name = request.form.get("name")

# CORRECT: read from the form object
name = form.name.data
```

---

### Mistake 5: Not redirecting after successful POST (Post/Redirect/Get)

```python
# WRONG: if user refreshes, form is submitted again!
if form.validate_on_submit():
    save_data(form)
    return render_template("success.html")  # no redirect!

# CORRECT: redirect after successful POST
if form.validate_on_submit():
    save_data(form)
    flash("Saved!", "success")
    return redirect(url_for("success"))
```

---

## 11. Interview Questions

**Q1: What is CSRF and how does Flask-WTF protect against it?**
A: CSRF (Cross-Site Request Forgery) is an attack where a malicious site tricks a browser into making an authenticated request to your site. Flask-WTF adds a hidden CSRF token to every form. On submission, it verifies the token matches what was sent. An attacker can't know this token, so forged requests are rejected.

**Q2: What does `form.validate_on_submit()` do?**
A: It returns `True` only when the current request is a POST AND all form validators pass. On GET requests or failed validation, it returns `False`, letting you re-render the form with error messages.

**Q3: Why redirect after a successful form submission instead of rendering a template?**
A: This is the Post/Redirect/Get (PRG) pattern. If you render after POST, refreshing the page re-submits the form, potentially duplicating data. A redirect converts the POST into a GET, so refreshing just reloads the result page harmlessly.

**Q4: How do you access submitted form data in Flask-WTF?**
A: Through `form.field_name.data` (e.g., `form.email.data`). Not through `request.form.get()` — that bypasses WTForms processing.

**Q5: What is `form.hidden_tag()` and why is it required?**
A: It renders a hidden `<input>` field containing the CSRF token. Without it, every POST request to a Flask-WTF protected form returns `400 Bad Request` because the CSRF check fails.

---

## 12. Best Practices

- Always use **Flask-WTF for production forms** — raw forms are fine for prototypes only.
- Always set a strong `secret_key` — never hardcode it in production (use environment variables — Module 19).
- Always include `{{ form.hidden_tag() }}` in every form template.
- Always use the **Post/Redirect/Get** pattern after successful submission.
- Always add `novalidate` to your `<form>` tag to disable browser's built-in validation — let WTForms handle it for consistent error styling.
- Never re-fill password fields after a failed login — it's a security best practice.
- Use `flash()` for success messages after redirects, not before renders.

---

## 13. Mini Project — Complete Registration + Login System

Build a working auth system with:

**Forms (`forms.py`):**
- `LoginForm` — email, password
- `RegisterForm` — name, email, password, confirm, checkbox

**Routes (`app.py`):**
- `GET/POST /register` — register with Flask-WTF
- `GET/POST /login` — login with Flask-WTF
- `GET /dashboard` — protected page (redirect to login if not logged in)
- `GET /logout` — clear session and redirect

**Templates (all extending base.html):**
- `register.html` — registration form with all errors shown
- `login.html` — login form
- `dashboard.html` — welcome page showing session username
- `base.html` — with flash message display block in navbar area

---

## 14. Practice Exercises

**5 Easy Questions**
1. What does `form.hidden_tag()` render in the HTML?
2. What Python class do you inherit from to create a Flask-WTF form?
3. Which validator ensures a field is not empty?
4. True/False: `form.validate_on_submit()` returns `True` on GET requests.
5. What does the `EqualTo("password")` validator check?

**5 Medium Questions**
1. What is CSRF and why is it dangerous?
2. What is the difference between `DataRequired` and `Optional` validators?
3. How do you add a custom error message to a validator?
4. How do you add an error to a specific field from inside the view function (not from a validator)?
5. Where must flash messages be displayed, and why does it need `{% with %}`?

**5 Hard Questions**
1. Explain the complete lifecycle of a Flask-WTF form — from GET request to successful POST redirect.
2. Why does Flask-WTF need `app.secret_key` to work?
3. How would you create a custom validator that checks if an email is from a specific domain (e.g., only `@company.com`)?
4. What happens if `WTF_CSRF_ENABLED = False` is set? When might that be appropriate?
5. How would you pre-populate a Flask-WTF form with existing data (e.g., editing a profile that already has saved values)?

**2 Debugging Questions**
1. A developer's form always shows "The CSRF token is missing" on POST. They checked and `{{ form.hidden_tag() }}` is in the template. What else could cause this?
2. A developer uses `form.validate_on_submit()` but the form never validates even with correct data. They check and all validators seem correct. What is one very common overlooked issue?

**2 Interview Questions**
1. "What is the Post/Redirect/Get pattern? Why is it important for forms?"
2. "Compare raw HTML form handling in Flask vs Flask-WTF. When would you use each?"

---

## 15. Summary

| Concept | Raw Forms | Flask-WTF |
|---|---|---|
| Form definition | HTML only | Python class + HTML rendering |
| Validation | Manual `if` checks in view | Validators in form class |
| CSRF protection | Manual (or none) | Automatic via `hidden_tag()` |
| Error messages | Manual dict passing | Built into `form.field.errors` |
| Best for | Quick prototypes | All production apps |

### Key Code Patterns

```python
# Create form
form = MyForm()

# Check valid POST
if form.validate_on_submit():
    data = form.field_name.data
    flash("Done!", "success")
    return redirect(url_for("next_page"))

return render_template("page.html", form=form)
```

```html
<!-- Template essentials -->
<form method="POST">
    {{ form.hidden_tag() }}          <!-- CSRF token — required! -->
    {{ form.field_name.label }}      <!-- Label tag -->
    {{ form.field_name() }}          <!-- Input element -->
    {% for error in form.field_name.errors %}
        <div class="error">{{ error }}</div>
    {% endfor %}
</form>
```

### 💡 Memory Trick
**"Flask-WTF = form security guard. `hidden_tag()` is the ID check at the door. `validate_on_submit()` is the metal detector. Without both, anyone can walk in."**

---

**End of Module 14.**
