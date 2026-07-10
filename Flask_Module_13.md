# MODULE 13 — Template Inheritance

> **Course:** Flask — Zero to Production
> **Module:** 13 of 39
> **Style:** Practical-first, complete code + complete HTML files

---

## 1. The Problem Template Inheritance Solves

Look at the templates we built in Modules 11 and 12. Every single `.html` file started with:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Page Title</title>
    <style>
        body { font-family: Arial ... }
        /* same styles every time */
    </style>
</head>
<body>
    <!-- same navbar every time -->
    <nav>...</nav>
```

And ended with:

```html
    <!-- same footer every time -->
    <footer>...</footer>
</body>
</html>
```

If your app has 20 pages, that's the same navbar, footer, CSS, and boilerplate copied into **20 files**. Now imagine your client says "Change the navbar colour." You'd have to edit 20 files. Miss even one and your site looks inconsistent.

**Template Inheritance** solves this. You create **one** `base.html` with the shared layout, and every other page just **extends** it and fills in only the parts that change.

---

## 2. How It Works — The Core Idea

**`base.html`** = the skeleton of every page. It has:
- The `<!DOCTYPE>`, `<html>`, `<head>`, `<body>` tags
- Your navbar, footer, CSS links
- **Named blocks** — holes you can fill in from child templates

**Child templates** (e.g., `home.html`, `about.html`) = just say:
1. `{% extends "base.html" %}` — "I inherit from base"
2. Fill in the named blocks with their own content

That's the entire concept.

---

## 3. Key Syntax — Just 3 Things to Learn

```html
{# In base.html — define a block (a "hole") #}
{% block block_name %}
    optional default content here
{% endblock %}


{# In child template — inherit from base #}
{% extends "base.html" %}


{# In child template — fill in a block #}
{% block block_name %}
    Your content for this page goes here
{% endblock %}
```

And one optional extra:

```html
{# In child template — include parent's default content PLUS add more #}
{% block block_name %}
    {{ super() }}
    Your extra content here
{% endblock %}
```

That's everything you need to know conceptually. Now let's build it.

---

## 4. Project Structure

```
my_app/
├── app.py
└── templates/
    ├── base.html           ← THE skeleton (shared layout)
    ├── home.html           ← extends base.html
    ├── about.html          ← extends base.html
    ├── contact.html        ← extends base.html
    ├── blog.html           ← extends base.html
    └── blog_post.html      ← extends base.html
```

---

## 5. Example 1 — Basic Inheritance (The Foundation)

### `app.py`

```python
from flask import Flask, render_template, request, redirect, url_for

app = Flask(__name__)


@app.route("/")
def home():
    features = [
        {"icon": "🚀", "title": "Fast",     "desc": "Built with performance in mind."},
        {"icon": "🔒", "title": "Secure",   "desc": "Security is our top priority."},
        {"icon": "🎨", "title": "Beautiful","desc": "Clean, modern design out of the box."},
    ]
    return render_template("home.html", features=features)


@app.route("/about")
def about():
    team = [
        {"name": "Arjun Sharma",  "role": "Founder & CEO",     "img": "👨‍💼"},
        {"name": "Priya Patel",   "role": "Lead Developer",    "img": "👩‍💻"},
        {"name": "Rahul Verma",   "role": "UI/UX Designer",    "img": "👨‍🎨"},
    ]
    return render_template("about.html", team=team)


@app.route("/contact", methods=["GET", "POST"])
def contact():
    success = False
    if request.method == "POST":
        # In real app, you'd send email or save to DB here
        success = True
    return render_template("contact.html", success=success)


@app.route("/blog")
def blog():
    posts = [
        {"id": 1, "title": "Getting Started with Flask",
         "excerpt": "Learn the basics of Flask web framework in this beginner-friendly guide.",
         "author": "Arjun", "date": "10 Jan 2025", "tag": "Flask"},
        {"id": 2, "title": "Jinja2 Template Tips and Tricks",
         "excerpt": "Master Jinja2 filters, macros, and template inheritance with real examples.",
         "author": "Priya", "date": "15 Jan 2025", "tag": "Jinja2"},
        {"id": 3, "title": "Deploying Flask to Production",
         "excerpt": "A step-by-step guide to deploy your Flask app using Gunicorn and Nginx.",
         "author": "Rahul", "date": "20 Jan 2025", "tag": "Deployment"},
    ]
    return render_template("blog.html", posts=posts)


@app.route("/blog/<int:post_id>")
def blog_post(post_id):
    posts = {
        1: {"title": "Getting Started with Flask",
            "content": "Flask is a micro web framework written in Python. It is classified as a microframework because it does not require particular tools or libraries. It has no database abstraction layer, form validation, or any other components where pre-existing third-party libraries provide common functions.",
            "author": "Arjun", "date": "10 Jan 2025", "tag": "Flask",
            "read_time": "5 min read"},
        2: {"title": "Jinja2 Template Tips and Tricks",
            "content": "Jinja2 is a modern and designer-friendly templating language for Python, modelled after Django's templates. It is fast, widely used, and secure with the optional sandboxed template execution environment.",
            "author": "Priya", "date": "15 Jan 2025", "tag": "Jinja2",
            "read_time": "8 min read"},
    }
    post = posts.get(post_id)
    if not post:
        return render_template("404.html"), 404
    return render_template("blog_post.html", post=post)


@app.errorhandler(404)
def not_found(e):
    return render_template("404.html"), 404


if __name__ == "__main__":
    app.run(debug=True)
```

---

### `templates/base.html`

This is the most important file. Read every comment.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    {# ── BLOCK: title ─────────────────────────────────────────────────────
       Child templates fill this in.
       If they don't, "My Flask App" is used as the default.
    ── #}
    <title>{% block title %}My Flask App{% endblock %}</title>

    <!-- Shared CSS for ALL pages -->
    <style>
        /* ── Reset & Base ── */
        * { margin: 0; padding: 0; box-sizing: border-box; }

        body {
            font-family: 'Segoe UI', Arial, sans-serif;
            background: #f5f6fa;
            color: #2d3436;
            min-height: 100vh;
            display: flex;
            flex-direction: column;
        }

        /* ── Navbar ── */
        nav {
            background: #2d3436;
            padding: 0 40px;
            display: flex;
            align-items: center;
            justify-content: space-between;
            height: 60px;
            position: sticky;
            top: 0;
            z-index: 100;
        }
        .nav-brand {
            color: #74b9ff;
            font-size: 20px;
            font-weight: bold;
            text-decoration: none;
        }
        .nav-links a {
            color: #dfe6e9;
            text-decoration: none;
            margin-left: 25px;
            font-size: 15px;
            transition: color 0.2s;
        }
        .nav-links a:hover       { color: #74b9ff; }
        .nav-links a.active      { color: #74b9ff; font-weight: bold; }

        /* ── Main content wrapper ── */
        .main-content {
            flex: 1;
            max-width: 1000px;
            width: 100%;
            margin: 0 auto;
            padding: 40px 20px;
        }

        /* ── Footer ── */
        footer {
            background: #2d3436;
            color: #b2bec3;
            text-align: center;
            padding: 20px;
            font-size: 14px;
        }
        footer a { color: #74b9ff; text-decoration: none; }

        /* ── Shared utility classes ── */
        .btn {
            display: inline-block;
            padding: 10px 22px;
            border-radius: 6px;
            text-decoration: none;
            font-size: 15px;
            border: none;
            cursor: pointer;
            transition: opacity 0.2s;
        }
        .btn:hover        { opacity: 0.85; }
        .btn-primary      { background: #0984e3; color: white; }
        .btn-success      { background: #00b894; color: white; }
        .btn-danger       { background: #d63031; color: white; }
        .btn-outline      { background: transparent; color: #0984e3; border: 2px solid #0984e3; }

        .card {
            background: white;
            border-radius: 12px;
            padding: 24px;
            box-shadow: 0 2px 12px rgba(0,0,0,0.07);
            margin-bottom: 20px;
        }

        .alert {
            padding: 14px 18px;
            border-radius: 8px;
            margin-bottom: 20px;
        }
        .alert-success { background: #d1e7dd; color: #0a3622; border-left: 4px solid #00b894; }
        .alert-danger  { background: #f8d7da; color: #842029; border-left: 4px solid #d63031; }
        .alert-info    { background: #cff4fc; color: #055160; border-left: 4px solid #0984e3; }

        .tag {
            display: inline-block;
            background: #dfe6e9;
            color: #636e72;
            padding: 3px 10px;
            border-radius: 20px;
            font-size: 12px;
        }
        .tag-blue   { background: #cce5ff; color: #004085; }
        .tag-green  { background: #d4edda; color: #155724; }
        .tag-orange { background: #fff3cd; color: #856404; }
    </style>

    {# ── BLOCK: extra_css ──────────────────────────────────────────────────
       Child templates can add their own page-specific CSS here.
       This block is empty by default (no default content).
    ── #}
    {% block extra_css %}{% endblock %}
</head>
<body>

    <!-- ── NAVBAR (same on every page) ── -->
    <nav>
        <a href="{{ url_for('home') }}" class="nav-brand">⚡ FlaskApp</a>
        <div class="nav-links">
            <a href="{{ url_for('home') }}"
               class="{{ 'active' if request.endpoint == 'home' }}">Home</a>
            <a href="{{ url_for('about') }}"
               class="{{ 'active' if request.endpoint == 'about' }}">About</a>
            <a href="{{ url_for('blog') }}"
               class="{{ 'active' if request.endpoint in ['blog', 'blog_post'] }}">Blog</a>
            <a href="{{ url_for('contact') }}"
               class="{{ 'active' if request.endpoint == 'contact' }}">Contact</a>
        </div>
    </nav>

    <!-- ── MAIN CONTENT ── -->
    <div class="main-content">

        {# ── BLOCK: content ───────────────────────────────────────────────
           This is the MAIN block.
           Every child template fills this with its unique page content.
        ── #}
        {% block content %}
            <p>No content provided.</p>
        {% endblock %}

    </div>

    <!-- ── FOOTER (same on every page) ── -->
    <footer>
        <p>
            &copy; 2025 FlaskApp. Built with ❤️ using
            <a href="https://flask.palletsprojects.com">Flask</a>.
        </p>
    </footer>

    {# ── BLOCK: extra_js ───────────────────────────────────────────────────
       Child templates can add their own JavaScript here.
       Placed at bottom so it doesn't block page rendering.
    ── #}
    {% block extra_js %}{% endblock %}

</body>
</html>
```

---

### `templates/home.html`

```html
{% extends "base.html" %}

{# Set the page title #}
{% block title %}Home — FlaskApp{% endblock %}

{# Page-specific CSS only for this page #}
{% block extra_css %}
<style>
    .hero {
        background: linear-gradient(135deg, #0984e3, #6c5ce7);
        color: white;
        text-align: center;
        padding: 70px 30px;
        border-radius: 16px;
        margin-bottom: 40px;
    }
    .hero h1       { font-size: 48px; margin-bottom: 15px; }
    .hero p        { font-size: 20px; opacity: 0.9; margin-bottom: 30px; }
    .features-grid {
        display: grid;
        grid-template-columns: repeat(3, 1fr);
        gap: 20px;
        margin-top: 20px;
    }
    .feature-card  { text-align: center; padding: 30px 20px; }
    .feature-icon  { font-size: 48px; margin-bottom: 15px; }
    .feature-title { font-size: 20px; font-weight: bold; margin-bottom: 8px; }
    .feature-desc  { color: #636e72; line-height: 1.6; }
</style>
{% endblock %}

{# Main page content — this replaces {% block content %} in base.html #}
{% block content %}

    <!-- Hero Section -->
    <div class="hero">
        <h1>Build Web Apps Fast ⚡</h1>
        <p>A clean Flask starter with template inheritance, routing, and more.</p>
        <a href="{{ url_for('blog') }}" class="btn btn-outline"
           style="color:white; border-color:white;">Read the Blog</a>
        &nbsp;&nbsp;
        <a href="{{ url_for('contact') }}" class="btn btn-success">Get in Touch</a>
    </div>

    <!-- Features Section -->
    <h2 style="margin-bottom: 20px;">Why FlaskApp?</h2>
    <div class="features-grid">
        {% for feature in features %}
        <div class="card feature-card">
            <div class="feature-icon">{{ feature.icon }}</div>
            <div class="feature-title">{{ feature.title }}</div>
            <div class="feature-desc">{{ feature.desc }}</div>
        </div>
        {% endfor %}
    </div>

{% endblock %}
```

---

### `templates/about.html`

```html
{% extends "base.html" %}

{% block title %}About Us — FlaskApp{% endblock %}

{% block extra_css %}
<style>
    .page-header {
        text-align: center;
        margin-bottom: 40px;
    }
    .page-header h1 { font-size: 40px; color: #2d3436; }
    .page-header p  { font-size: 18px; color: #636e72; margin-top: 10px; }

    .team-grid {
        display: grid;
        grid-template-columns: repeat(3, 1fr);
        gap: 20px;
        margin-top: 30px;
    }
    .team-card       { text-align: center; padding: 30px 20px; }
    .team-avatar     { font-size: 60px; margin-bottom: 15px; }
    .team-name       { font-size: 18px; font-weight: bold; }
    .team-role       { color: #636e72; margin-top: 5px; font-size: 14px; }

    .mission-box {
        background: linear-gradient(135deg, #00b894, #00cec9);
        color: white;
        border-radius: 12px;
        padding: 40px;
        text-align: center;
        margin-top: 40px;
    }
    .mission-box h2 { font-size: 28px; margin-bottom: 15px; }
    .mission-box p  { font-size: 17px; opacity: 0.9; line-height: 1.7; }
</style>
{% endblock %}

{% block content %}

    <div class="page-header">
        <h1>About Us</h1>
        <p>We are a passionate team of developers building great things with Flask.</p>
    </div>

    <h2>Meet the Team</h2>
    <div class="team-grid">
        {% for member in team %}
        <div class="card team-card">
            <div class="team-avatar">{{ member.img }}</div>
            <div class="team-name">{{ member.name }}</div>
            <div class="team-role">{{ member.role }}</div>
        </div>
        {% endfor %}
    </div>

    <div class="mission-box">
        <h2>Our Mission</h2>
        <p>
            To make web development accessible, enjoyable, and powerful for
            every Python developer — from beginners to experts.
        </p>
    </div>

{% endblock %}
```

---

### `templates/contact.html`

```html
{% extends "base.html" %}

{% block title %}Contact — FlaskApp{% endblock %}

{% block extra_css %}
<style>
    .contact-wrapper {
        max-width: 580px;
        margin: 0 auto;
    }
    .page-header       { text-align: center; margin-bottom: 35px; }
    .page-header h1    { font-size: 38px; }
    .page-header p     { color: #636e72; margin-top: 8px; }
    label              { display: block; font-weight: 600; margin-bottom: 6px; color: #2d3436; }
    input, textarea    {
        width: 100%;
        padding: 11px 14px;
        border: 1px solid #dfe6e9;
        border-radius: 8px;
        font-size: 15px;
        margin-bottom: 20px;
        font-family: inherit;
        box-sizing: border-box;
        transition: border-color 0.2s;
    }
    input:focus, textarea:focus {
        outline: none;
        border-color: #0984e3;
        box-shadow: 0 0 0 3px rgba(9,132,227,0.15);
    }
    textarea           { height: 140px; resize: vertical; }
    .submit-btn        { width: 100%; padding: 13px; font-size: 16px; }
</style>
{% endblock %}

{% block content %}

    <div class="contact-wrapper">
        <div class="page-header">
            <h1>Get in Touch</h1>
            <p>We'd love to hear from you. Send us a message!</p>
        </div>

        {# Show success message after form submit #}
        {% if success %}
            <div class="alert alert-success">
                ✓ Thank you! Your message has been sent. We'll get back to you soon.
            </div>
        {% endif %}

        <div class="card">
            <form method="POST">
                <label for="name">Full Name</label>
                <input type="text" id="name" name="name"
                       placeholder="Arjun Sharma" required>

                <label for="email">Email Address</label>
                <input type="email" id="email" name="email"
                       placeholder="arjun@example.com" required>

                <label for="subject">Subject</label>
                <input type="text" id="subject" name="subject"
                       placeholder="What is this about?">

                <label for="message">Message</label>
                <textarea id="message" name="message"
                          placeholder="Write your message here..." required></textarea>

                <button type="submit" class="btn btn-primary submit-btn">
                    Send Message →
                </button>
            </form>
        </div>
    </div>

{% endblock %}
```

---

### `templates/blog.html`

```html
{% extends "base.html" %}

{% block title %}Blog — FlaskApp{% endblock %}

{% block extra_css %}
<style>
    .page-header    { margin-bottom: 30px; }
    .page-header h1 { font-size: 38px; }
    .page-header p  { color: #636e72; margin-top: 8px; }
    .post-card      { display: flex; flex-direction: column; }
    .post-header    { display: flex; justify-content: space-between; align-items: flex-start; margin-bottom: 10px; }
    .post-title     { font-size: 20px; font-weight: bold; color: #2d3436; }
    .post-title a   { text-decoration: none; color: inherit; }
    .post-title a:hover { color: #0984e3; }
    .post-excerpt   { color: #636e72; line-height: 1.6; margin: 10px 0; }
    .post-meta      { font-size: 13px; color: #b2bec3; margin-top: 12px; }
    .read-more      { font-size: 14px; margin-top: 12px; }
    .no-posts       { text-align: center; padding: 60px 20px; color: #636e72; }
</style>
{% endblock %}

{% block content %}

    <div class="page-header">
        <h1>📝 Blog</h1>
        <p>Tutorials, tips, and insights about Flask and Python web development.</p>
    </div>

    {% if posts %}

        {% for post in posts %}
        <div class="card post-card">
            <div class="post-header">
                <div class="post-title">
                    <a href="{{ url_for('blog_post', post_id=post.id) }}">
                        {{ post.title }}
                    </a>
                </div>
                <span class="tag tag-blue">{{ post.tag }}</span>
            </div>
            <p class="post-excerpt">{{ post.excerpt }}</p>
            <div class="post-meta">
                ✍ {{ post.author }} &nbsp;·&nbsp; 📅 {{ post.date }}
            </div>
            <div class="read-more">
                <a href="{{ url_for('blog_post', post_id=post.id) }}" class="btn btn-outline" style="font-size:13px; padding:6px 14px;">
                    Read More →
                </a>
            </div>
        </div>
        {% endfor %}

    {% else %}

        <div class="card no-posts">
            <p style="font-size:48px;">📭</p>
            <h3>No posts yet</h3>
            <p>Check back soon for new articles!</p>
        </div>

    {% endif %}

{% endblock %}
```

---

### `templates/blog_post.html`

```html
{% extends "base.html" %}

{% block title %}{{ post.title }} — FlaskApp{% endblock %}

{% block extra_css %}
<style>
    .post-wrapper   { max-width: 720px; margin: 0 auto; }
    .post-tag       { margin-bottom: 15px; }
    .post-title     { font-size: 36px; line-height: 1.3; margin-bottom: 15px; }
    .post-meta      { color: #b2bec3; font-size: 14px; margin-bottom: 30px; padding-bottom: 20px; border-bottom: 1px solid #dfe6e9; }
    .post-body      { font-size: 17px; line-height: 1.8; color: #2d3436; }
    .back-link      { margin-top: 40px; }
    .share-section  { margin-top: 40px; padding-top: 20px; border-top: 1px solid #dfe6e9; }
    .share-section h3 { margin-bottom: 12px; }
</style>
{% endblock %}

{% block content %}

    <div class="post-wrapper">

        <div class="post-tag">
            <span class="tag tag-blue">{{ post.tag }}</span>
            &nbsp;
            <span class="tag">{{ post.read_time }}</span>
        </div>

        <h1 class="post-title">{{ post.title }}</h1>

        <div class="post-meta">
            ✍ Written by <strong>{{ post.author }}</strong>
            &nbsp;·&nbsp;
            📅 {{ post.date }}
        </div>

        <div class="card post-body">
            <p>{{ post.content }}</p>
        </div>

        <div class="back-link">
            <a href="{{ url_for('blog') }}" class="btn btn-outline">← Back to Blog</a>
        </div>

        <div class="share-section">
            <h3>Enjoyed this post?</h3>
            <a href="{{ url_for('contact') }}" class="btn btn-primary">Get in Touch</a>
        </div>

    </div>

{% endblock %}
```

---

### `templates/404.html`

```html
{% extends "base.html" %}

{% block title %}404 — Page Not Found{% endblock %}

{% block content %}
<div style="text-align: center; padding: 60px 20px;">
    <div style="font-size: 80px;">😕</div>
    <h1 style="font-size: 60px; color: #d63031; margin: 10px 0;">404</h1>
    <h2 style="color: #636e72;">Page Not Found</h2>
    <p style="color: #b2bec3; margin: 15px 0 30px;">
        The page you are looking for doesn't exist or has been moved.
    </p>
    <a href="{{ url_for('home') }}" class="btn btn-primary">← Back to Home</a>
</div>
{% endblock %}
```

---

## 6. How It All Connects — Visual Diagram

```
base.html
├── <!DOCTYPE html>
├── <head>
│   ├── <title>{% block title %}My Flask App{% endblock %}</title>
│   ├── shared CSS
│   └── {% block extra_css %}{% endblock %}
├── <body>
│   ├── <nav>  ← SAME on every page
│   ├── <div class="main-content">
│   │   └── {% block content %}{% endblock %}  ← CHANGES per page
│   └── <footer>  ← SAME on every page
└── {% block extra_js %}{% endblock %}


home.html                  about.html               contact.html
├── extends "base.html"    ├── extends "base.html"  ├── extends "base.html"
├── block title            ├── block title           ├── block title
│   "Home — FlaskApp"      │   "About Us — FlaskApp" │   "Contact — FlaskApp"
├── block extra_css        ├── block extra_css       ├── block extra_css
│   (hero + feature CSS)   │   (team card CSS)       │   (form CSS)
└── block content          └── block content         └── block content
    (hero + features)          (team grid)               (contact form)
```

---

## 7. `{{ super() }}` — Keeping Parent Content AND Adding More

Sometimes you want to keep what the parent block already has and add more. Use `{{ super() }}`.

**Example — adding extra CSS without losing the parent's CSS:**

Imagine `base.html` has this:

```html
{% block extra_css %}
    <link rel="stylesheet" href="{{ url_for('static', filename='common.css') }}">
{% endblock %}
```

A child page can use `{{ super() }}` to keep that AND add its own:

```html
{% block extra_css %}
    {{ super() }}   ← includes parent's <link> tag first
    <style>
        /* only this page's extra styles */
        .dashboard-grid { display: grid; ... }
    </style>
{% endblock %}
```

Without `{{ super() }}`, the child's `block extra_css` **replaces** the parent's entirely. With `{{ super() }}`, it **adds to** it.

**Practical use in `base.html`:**

```html
{# base.html - default meta tags #}
{% block head_extra %}
    <meta name="author" content="FlaskApp Team">
{% endblock %}
```

```html
{# blog_post.html - keep the default meta, add post-specific one #}
{% block head_extra %}
    {{ super() }}
    <meta name="description" content="{{ post.excerpt }}">
    <meta property="og:title" content="{{ post.title }}">
{% endblock %}
```

---

## 8. Three-Level Inheritance — Base → Layout → Page

You can chain inheritance beyond two levels. This is useful for complex apps with multiple layouts (e.g., a regular layout and an admin layout both extending the same base).

```
base.html                  ← universal skeleton (DOCTYPE, nav, footer)
    └── admin_base.html    ← extends base.html, adds sidebar
            ├── admin_dashboard.html   ← extends admin_base.html
            └── admin_users.html      ← extends admin_base.html
```

### `templates/admin_base.html`

```html
{% extends "base.html" %}

{% block extra_css %}
<style>
    .admin-layout  { display: grid; grid-template-columns: 220px 1fr; gap: 25px; }
    .sidebar       { background: white; border-radius: 12px; padding: 20px; height: fit-content; box-shadow: 0 2px 12px rgba(0,0,0,0.07); }
    .sidebar h3    { font-size: 13px; text-transform: uppercase; color: #b2bec3; margin-bottom: 12px; }
    .sidebar a     { display: block; padding: 9px 12px; color: #2d3436; text-decoration: none; border-radius: 6px; margin-bottom: 4px; font-size: 14px; }
    .sidebar a:hover  { background: #f5f6fa; }
    .sidebar a.active { background: #0984e3; color: white; }
    .admin-content { min-width: 0; }
</style>
{% endblock %}

{# Override content block to add a sidebar layout #}
{% block content %}
<div class="admin-layout">

    <!-- Sidebar — same in ALL admin pages -->
    <div class="sidebar">
        <h3>Admin Panel</h3>
        <a href="/admin"        class="{{ 'active' if request.path == '/admin' }}">📊 Dashboard</a>
        <a href="/admin/users"  class="{{ 'active' if request.path == '/admin/users' }}">👥 Users</a>
        <a href="/admin/posts"  class="{{ 'active' if request.path == '/admin/posts' }}">📝 Posts</a>
        <a href="/admin/settings" class="{{ 'active' if request.path == '/admin/settings' }}">⚙️ Settings</a>
        <hr style="margin: 15px 0; border: none; border-top: 1px solid #dfe6e9;">
        <a href="{{ url_for('home') }}">← Back to Site</a>
    </div>

    <!-- Main area — each admin page fills this -->
    <div class="admin-content">
        {% block admin_content %}
            <p>No admin content.</p>
        {% endblock %}
    </div>

</div>
{% endblock %}
```

---

### `templates/admin_dashboard.html`

```html
{% extends "admin_base.html" %}

{% block title %}Dashboard — Admin{% endblock %}

{% block admin_content %}

    <h1 style="margin-bottom: 25px;">📊 Dashboard</h1>

    <div style="display: grid; grid-template-columns: repeat(3,1fr); gap: 15px; margin-bottom: 25px;">
        <div class="card" style="text-align:center;">
            <div style="font-size:36px; font-weight:bold; color:#0984e3;">1,248</div>
            <div style="color:#636e72; margin-top:5px;">Total Users</div>
        </div>
        <div class="card" style="text-align:center;">
            <div style="font-size:36px; font-weight:bold; color:#00b894;">342</div>
            <div style="color:#636e72; margin-top:5px;">Blog Posts</div>
        </div>
        <div class="card" style="text-align:center;">
            <div style="font-size:36px; font-weight:bold; color:#6c5ce7;">89%</div>
            <div style="color:#636e72; margin-top:5px;">Uptime</div>
        </div>
    </div>

    <div class="card">
        <h3 style="margin-bottom:15px;">Recent Activity</h3>
        <p style="color:#636e72;">No recent activity to show.</p>
    </div>

{% endblock %}
```

---

### Add admin routes to `app.py`

```python
@app.route("/admin")
def admin_dashboard():
    return render_template("admin_dashboard.html")
```

**The result:** `admin_dashboard.html` gets:
- `base.html`'s navbar and footer (level 1)
- `admin_base.html`'s sidebar layout (level 2)
- Its own dashboard stats (level 3)

No code repeated anywhere.

---

## 9. Step-by-Step — What Happens When You Visit `/`

```
Browser: GET /
   ↓
Flask calls home() view function
   ↓
render_template("home.html", features=[...])
   ↓
Jinja2 reads home.html
   ↓
Sees {% extends "base.html" %}
   ↓
Jinja2 reads base.html first (the parent)
   ↓
Jinja2 finds all {% block %} tags in base.html:
   - block title      → filled by home.html's block title
   - block extra_css  → filled by home.html's block extra_css
   - block content    → filled by home.html's block content
   - block extra_js   → empty (home.html doesn't define it)
   ↓
Jinja2 merges everything:
   - base.html's structure + nav + footer
   - home.html's blocks inserted at the right spots
   ↓
Final HTML sent to browser
```

---

## 10. Active Nav Link — Highlighting Current Page

Notice in `base.html` we used:

```html
<a href="{{ url_for('home') }}"
   class="{{ 'active' if request.endpoint == 'home' }}">Home</a>
```

This adds the CSS class `active` only when the current page's endpoint is `'home'`. Flask makes `request` available in templates automatically (Module 12), so this works without passing anything extra.

Result: The current page's link is highlighted blue in the navbar on every page — no extra Python code needed.

---

## 11. Common Mistakes

### Mistake 1: Putting content outside blocks in a child template

```html
{% extends "base.html" %}

{# WRONG: this text is outside any block — it will be IGNORED #}
<p>This will never show up!</p>

{% block content %}
    <p>This WILL show up.</p>
{% endblock %}
```

In a child template that `extends` another, **any content outside a `{% block %}` is silently ignored**. Everything must be inside a block.

---

### Mistake 2: Forgetting `{% extends %}` must be the FIRST line

```html
<!-- WRONG: text before extends -->
<!DOCTYPE html>
{% extends "base.html" %}

<!-- CORRECT: extends must be the very first thing -->
{% extends "base.html" %}
```

`{% extends %}` must be the **first tag** in the file. Any whitespace or content before it causes a Jinja2 error.

---

### Mistake 3: Using the same block name twice in a child template

```html
{% extends "base.html" %}

{% block content %}First definition{% endblock %}

{# WRONG: can't define same block twice #}
{% block content %}Second definition{% endblock %}
```

Each block can only be defined **once** per template. Only the first definition is used.

---

### Mistake 4: Forgetting `{% endblock %}`

```html
{% block content %}
    <h1>Hello</h1>
    {# Missing {% endblock %} — causes TemplateSyntaxError #}
```

Every `{% block %}` must have a matching `{% endblock %}`.

---

### Mistake 5: Inheriting from a child template that extends another

This is not a mistake — it's called multi-level inheritance (Section 8). But beginners get confused thinking inheritance can only be 2 levels. It can be as deep as you need.

---

## 12. Interview Questions

**Q1: What is template inheritance in Flask/Jinja2?**
A: A mechanism where a `base.html` file defines the shared layout (navbar, footer, CSS) with named `{% block %}` placeholders, and child templates use `{% extends "base.html" %}` to inherit that layout and fill in only their unique content blocks.

**Q2: What does `{% block content %}{% endblock %}` do in a base template?**
A: It defines a named "hole" called `content` where child templates can insert their unique page content. If no child fills it, the block shows its default content (or nothing if the block is empty).

**Q3: What is `{{ super() }}` used for?**
A: It includes the parent block's content inside a child block's override. Without it, the child's block completely replaces the parent's content. With `{{ super() }}`, the child adds to the parent's content.

**Q4: What happens if a child template has content outside any `{% block %}` tags?**
A: That content is silently ignored by Jinja2. In a child template that extends another, everything must be inside a `{% block %}`.

**Q5: Can you have more than 2 levels of template inheritance?**
A: Yes. You can chain as many levels as needed — for example, `base.html` → `admin_base.html` → `admin_dashboard.html`. Each level adds its own layer of shared structure.

---

## 13. Best Practices

- Name your base template `base.html` — it's the universal convention.
- Define blocks for **title**, **extra_css**, **content**, and **extra_js** at minimum — these cover almost every case.
- Use **meaningful block names** — `{% block sidebar %}`, `{% block breadcrumbs %}` instead of `{% block block1 %}`.
- Use `{{ url_for() }}` for all links in `base.html` — never hardcode paths.
- Mark active nav links using `request.endpoint` — Flask provides this automatically.
- Use multi-level inheritance for admin panels or dashboard layouts that share structure but differ from the public site.
- Keep `base.html` focused on **structure only** — don't put page-specific logic there.

---

## 14. Mini Project — Complete Multi-Page Website

Extend the example above by adding:

1. **`/blog/<id>` for post IDs 1, 2, and 3** — add a third post to the `posts` dict in `app.py`
2. **`/admin`** — the admin dashboard page (using 3-level inheritance from this module)
3. **A `flash_messages` block in `base.html`** — so any page can show a notification banner (we cover Flask's `flash()` system in Module 16, but you can practice the block structure now by passing a `message` variable)
4. **Footer with working year** — use `{% set year = 2025 %}` in `base.html` to show the copyright year

---

## 15. Practice Exercises

**5 Easy Questions**
1. What is the first line of every child template that uses inheritance?
2. What Jinja2 tag defines a named block in a base template?
3. True/False: Content outside `{% block %}` tags in a child template is displayed normally.
4. What does `{{ super() }}` do inside a child block?
5. What is the conventional name for the main base template file?

**5 Medium Questions**
1. Why is template inheritance better than copy-pasting shared HTML across files?
2. How would you make a block in `base.html` have default content that child templates can optionally override?
3. How do you highlight the active navigation link without passing extra variables from Python?
4. What is the difference between `{% block content %}{% endblock %}` and `{% block content %}Default{% endblock %}`?
5. How would you add page-specific JavaScript only to one page while keeping a global JS file in `base.html`?

**5 Hard Questions**
1. Explain what Jinja2 does internally when it encounters `{% extends "base.html" %}` in a child template.
2. Why must `{% extends %}` be the first tag in a child template?
3. In 3-level inheritance (`base → admin_base → admin_dashboard`), if all three define `{% block title %}`, which one wins?
4. How would you create a "layout" template that itself inherits from `base.html` but adds a sidebar — without breaking child templates that want the sidebar?
5. Could you conditionally extend different base templates (e.g., a mobile vs desktop base)? How would you approach this?

**2 Debugging Questions**
1. A developer creates `home.html` that extends `base.html` but the page shows only the content with no navbar or footer. They check `base.html` and the navbar is there. What is the most likely cause?
2. A developer adds `<h1>Welcome</h1>` directly in `home.html` (outside any `{% block %}`) but it never shows on the page. Why?

**2 Interview Questions**
1. "Explain template inheritance in Jinja2. How does it work and why is it useful?"
2. "What is `{{ super() }}` and when would you use it?"

---

## 16. Summary

| Concept | Syntax | Where used |
|---|---|---|
| Define a block | `{% block name %}default{% endblock %}` | `base.html` |
| Inherit from base | `{% extends "base.html" %}` | Child template (first line) |
| Fill in a block | `{% block name %}content{% endblock %}` | Child template |
| Keep parent content | `{{ super() }}` | Inside a child block |
| Active nav link | `'active' if request.endpoint == 'x'` | `base.html` navbar |

### Block Naming Convention

| Block Name | What it holds |
|---|---|
| `title` | Page title in `<title>` tag |
| `extra_css` | Page-specific CSS styles |
| `content` | Main page content |
| `sidebar` | Sidebar content (if layout has one) |
| `extra_js` | Page-specific JavaScript |
| `breadcrumbs` | Breadcrumb navigation |
| `page_header` | Hero section or page title area |

### 💡 Memory Trick
**"Base template = a house blueprint with labelled rooms (blocks). Child templates = the interior designer who decides what furniture (content) goes in each room. The blueprint handles the structure — the designer handles the décor."**

---

**End of Module 13.**
