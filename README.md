# inkflow
A production-ready multilingual blog platform built with Python, Django, and MySQL.

**Features:** User auth · Post CRUD · Draft/Publish workflow · Comment moderation · Profanity filtering · 5-language support (EN/FR/AR/ES/HI) · Dark mode · Responsive editorial UI

---

## Project Structure

```
inkflow/
├── config/                 # Django project settings & root URLs
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── blog/                   # Core blog app
│   ├── models.py           # Post, Comment, Tag, UserProfile
│   ├── views.py
│   ├── forms.py
│   ├── urls.py
│   ├── admin.py
│   ├── middleware.py       # Profanity filter initialiser
│   └── context_processors.py
├── accounts/               # Auth app
│   ├── views.py
│   ├── forms.py
│   ├── urls.py
│   └── signals.py          # Auto-create UserProfile on register
├── static/
│   ├── css/style.css       # Full editorial stylesheet + dark mode
│   └── js/main.js          # Interactivity: dark mode, reading bar, etc.
├── templates/
│   ├── base.html           # Master layout with navbar + footer
│   ├── blog/               # post_list, post_detail, post_form, my_posts, ...
│   ├── accounts/           # register, login, profile, profile_edit
│   └── partials/           # sidebar
├── locale/                 # Translation .po files (FR, AR, ES, HI, EN)
├── requirements.txt
├── .gitignore
├── .env.example
└── manage.py
```

---

## 1. Prerequisites

| Tool    | Version     |
|---------|-------------|
| Python  | 3.10+       |
| MySQL   | 8.0+        |
| pip     | latest      |
| git     | any         |

---

## 2. MySQL Setup

Open a MySQL shell as root:

```sql
-- Create the database with full Unicode support
CREATE DATABASE inkflow_db
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

-- Create a dedicated user
CREATE USER 'inkflow_user'@'localhost' IDENTIFIED BY 'your_secure_password_here';

-- Grant all privileges on the inkflow database only
GRANT ALL PRIVILEGES ON inkflow_db.* TO 'inkflow_user'@'localhost';

-- Apply changes
FLUSH PRIVILEGES;

-- Verify
SHOW DATABASES;
EXIT;
```

---

## 3. Local Development Setup

### 3.1 Clone & virtual environment

```bash
git clone https://github.com/YOUR_USERNAME/inkflow.git
cd inkflow

# Create and activate virtual environment
python -m venv venv

# macOS / Linux
source venv/bin/activate

# Windows
venv\Scripts\activate
```

### 3.2 Install dependencies

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

### 3.3 Environment variables

```bash
# Copy the example file
cp .env.example .env

# Edit .env with your MySQL credentials and a new secret key
nano .env        # or use any editor
```

Generate a secure secret key:
```bash
python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

Paste the output into `.env` as `DJANGO_SECRET_KEY`.

### 3.4 Load environment variables

Add this to the top of `config/settings.py` (already included):
```python
from dotenv import load_dotenv
load_dotenv()
```

Or export manually:
```bash
export DJANGO_SECRET_KEY="your-key-here"
export DB_PASSWORD="your_secure_password_here"
```

---

## 4. Django Migrations

```bash
# Create migration files from models (if not already present)
python manage.py makemigrations blog
python manage.py makemigrations accounts

# Apply all migrations to the database
python manage.py migrate

# Verify tables were created
python manage.py dbshell
# Inside MySQL shell:
SHOW TABLES;
EXIT;
```

---

## 5. Compile Translation Files

```bash
# Compile all .po files to binary .mo files Django can read
python manage.py compilemessages

# If you want to extract new strings from templates/code:
python manage.py makemessages --all
python manage.py compilemessages
```

> **Note:** Requires GNU `gettext` tools. Install with:
> - macOS: `brew install gettext`
> - Ubuntu: `sudo apt install gettext`
> - Windows: download from https://mlocati.github.io/articles/gettext-iconv-windows.html

---

## 6. Create Superuser

```bash
python manage.py createsuperuser
# Follow the prompts for username, email, password
```

---

## 7. Collect Static Files

```bash
# Only needed for production / when DEBUG=False
python manage.py collectstatic --noinput
```

---

## 8. Run Development Server

```bash
python manage.py runserver
```

Open **http://127.0.0.1:8000** in your browser.

| URL                        | Description              |
|----------------------------|--------------------------|
| `http://127.0.0.1:8000/`   | Home – post listing      |
| `http://127.0.0.1:8000/admin/` | Django admin panel   |
| `http://127.0.0.1:8000/accounts/register/` | Register  |
| `http://127.0.0.1:8000/accounts/login/`    | Login     |
| `http://127.0.0.1:8000/post/new/`          | Create post |
| `http://127.0.0.1:8000/my-posts/`          | Author dashboard |

---

## 9. Language Switching

Language URLs are prefixed automatically by `i18n_patterns`:

| Language | URL prefix | Example              |
|----------|-----------|----------------------|
| English  | (none)    | `/post/my-post/`     |
| French   | `/fr/`    | `/fr/post/my-post/`  |
| Arabic   | `/ar/`    | `/ar/post/my-post/`  |
| Spanish  | `/es/`    | `/es/post/my-post/`  |
| Hindi    | `/hi/`    | `/hi/post/my-post/`  |

Use the globe icon in the navbar to switch languages in the UI.

---

## 10. GitHub Deployment Steps

### 10.1 Initialise repository

```bash
cd inkflow
git init
git add .
git commit -m "Initial commit: InkFlow blog platform"
```

### 10.2 Create GitHub repository

1. Go to https://github.com/new
2. Name it `inkflow`
3. Leave it empty (no README, no .gitignore)
4. Click **Create repository**

### 10.3 Push to GitHub

```bash
git remote add origin https://github.com/YOUR_USERNAME/inkflow.git
git branch -M main
git push -u origin main
```

### 10.4 Subsequent pushes

```bash
git add .
git commit -m "Your commit message"
git push
```

---

## 11. Production Checklist

Before deploying to a live server:

```python
# In .env or environment:
DJANGO_DEBUG=False
DJANGO_ALLOWED_HOSTS=yourdomain.com www.yourdomain.com
DJANGO_SECRET_KEY=<long-random-key>
```

```bash
# Collect static files
python manage.py collectstatic --noinput

# Compile messages
python manage.py compilemessages

# Run with Gunicorn (production WSGI server)
gunicorn config.wsgi:application --bind 0.0.0.0:8000 --workers 3
```

Add WhiteNoise to `MIDDLEWARE` (after `SecurityMiddleware`) for static file serving:
```python
'whitenoise.middleware.WhiteNoiseMiddleware',
```

---

## 12. Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| **Soft delete** for posts | Preserves data; admin can restore |
| **Auto-approve** authenticated comments | Reduces friction for logged-in users |
| **Flag + censor** profane comments | Holds for review rather than silently dropping |
| **`better_profanity`** loaded once in middleware | Avoids reloading word list on every request |
| **UTF8MB4** MySQL charset | Full Unicode + emoji support |
| **`i18n_patterns`** with `prefix_default_language=False` | English URLs stay clean (`/post/slug/` not `/en/post/slug/`) |
| **CSS custom properties** for theming | One-line dark mode toggle via `data-theme` attribute |
| **Cormorant Garamond** display font | Distinctive editorial feel, contrasts with DM Sans body font |
