# 🐾 Petstagram — Azure Deployment Guide

A Django-based photo sharing app for pets, deployed to Azure App Service with PostgreSQL and automated CI/CD via GitHub Actions.

---

## 🛠 Tech Stack

- **Backend:** Django (Python 3.13)
- **Database:** PostgreSQL (Azure Database for PostgreSQL)
- **Hosting:** Azure App Service
- **Static Files:** WhiteNoise
- **CI/CD:** GitHub Actions

---

## 🚀 Azure Deployment

### Step 1: Create Azure Resources

1. Go to [portal.azure.com](https://portal.azure.com)
2. Click **Create a resource** → search for **Web App + Database**
3. Fill in the required fields:
   - **Subscription** – your Azure subscription
   - **Resource Group** – create new or use existing
   - **Region** – choose the closest to your users
   - **Runtime stack** – Python 3.13
   - **Database** – PostgreSQL Flexible Server
4. Click **Review + Create** → **Create**

---

### Step 2: Configure Environment Variables

In the Azure Portal go to your **App Service → Settings → Environment variables** and add:

| Variable | Description |
|---|---|
| `SECRET_KEY` | Django secret key |
| `DEBUG` | `False` in production |
| `ALLOWED_HOSTS` | `petstagram.azurewebsites.net` |
| `CSRF_TRUSTED_ORIGINS` | `https://petstagram.azurewebsites.net` |
| `DB_NAME` | PostgreSQL database name |
| `DB_USER` | PostgreSQL username |
| `DB_PASS` | PostgreSQL password |
| `DB_PORT` | `5432` |
| `DP_HOST` | PostgreSQL host |
| `EMAIL_HOST_USER` | SMTP email address |
| `EMAIL_HOST_PASSWORD` | SMTP email password |
| `COMPANY_EMAIL` | Company contact email |
| `CLOUDINARY_CLOUD_NAME` | Cloudinary cloud name |
| `CLOUDINARY_API_KEY` | Cloudinary API key |
| `CLOUDINARY_API_SECRET` | Cloudinary API secret |

> 💡 Database connection details: **Azure Portal → your PostgreSQL resource → Connection strings**  
> 💡 Cloudinary credentials: **cloudinary.com → Dashboard**

---

### Step 3: Set Up `.env` for Local Development

Create a `.env` file in the root of the project (never commit this to Git):

```env
SECRET_KEY=your-secret-key
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1
CSRF_TRUSTED_ORIGINS=http://localhost:8000
DB_NAME=petstagram
DB_USER=your-db-user
DB_PASS=your-db-password
DB_PORT=5432
DP_HOST=localhost
EMAIL_HOST_USER=your-email@gmail.com
EMAIL_HOST_PASSWORD=your-email-password
COMPANY_EMAIL=company@email.com
CLOUDINARY_CLOUD_NAME=your-cloud-name
CLOUDINARY_API_KEY=your-api-key
CLOUDINARY_API_SECRET=your-api-secret
```

Make sure `.env` is in your `.gitignore`:

```
.env
```

---

### Step 4: Update `settings.py`

Add these to your `settings.py` to read hosts dynamically from environment variables:

```python
import os

ALLOWED_HOSTS = [host for host in os.getenv('ALLOWED_HOSTS').split(',') if host]
CSRF_TRUSTED_ORIGINS = [host for host in os.getenv('CSRF_TRUSTED_ORIGINS').split(',') if host]
```

---

### Step 5: Handle Static Files with WhiteNoise

**Install WhiteNoise:**

```bash
pip install whitenoise
pip freeze > requirements.txt
```

**Add WhiteNoise to `MIDDLEWARE` in `settings.py`** (must be second, right after `SecurityMiddleware`):

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware',   # 👈 add this
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

**Add `STATIC_ROOT` to `settings.py`:**

```python
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
```

**Collect static files:**

```bash
python manage.py collectstatic
```

---

## STEP 6: 🚀 Deployment

1. In the Azure Portal, go to your **App Service → Deployment Center**
2. Under **Source**, choose **GitHub**
3. Authorize and fill in:
   - **Organization** – your GitHub username
   - **Repository** – your repo
   - **Branch** – `main`
4. Under **Workflow option**, select **Add a workflow**
5. Click **Save**

Azure will automatically create a GitHub Actions workflow file in your repo (`.github/workflows/`).

> ✅ After saving, go to your GitHub repo → **Actions** tab to monitor the deployment. Once it goes green, your app is live!


---

##  STEP 7:🗄️ Run Migrations

1. In the Azure Portal, go to your **App Service → SSH**
2. Click **Go →**
3. In the terminal, run:

```bash
python manage.py migrate
```

##  STEP 8:☁️ Bonus: Cloudinary Setup

**Install packages:**

```bash
pip install cloudinary django-cloudinary-storage
pip freeze > requirements.txt
```

**In `settings.py`, add to `INSTALLED_APPS`:**

```python
INSTALLED_APPS = [
    ...
    'cloudinary',
    'cloudinary_storage',
]
```

**Remove `MEDIA_URL` / `MEDIA_ROOT` and replace with:**

```python
import cloudinary

CLOUDINARY_STORAGE = {
    'CLOUD_NAME': os.getenv('CLOUDINARY_CLOUD_NAME'),
    'API_KEY': os.getenv('CLOUDINARY_API_KEY'),
    'API_SECRET': os.getenv('CLOUDINARY_API_SECRET'),
}

DEFAULT_FILE_STORAGE = 'cloudinary_storage.storage.MediaCloudinaryStorage'
```



## ✅ Deployment Checklist

- [ ] Azure Web App + Database resource created
- [ ] Environment variables set in Azure Portal
- [ ] `.env` file created locally (not committed to Git)
- [ ] `ALLOWED_HOSTS` and `CSRF_TRUSTED_ORIGINS` reading from environment
- [ ] WhiteNoise installed and added to `MIDDLEWARE`
- [ ] `STATIC_ROOT` set in `settings.py`
- [ ] `python manage.py collectstatic` run
- [ ] GitHub Actions secrets added
- [ ] Push to `main` triggers successful deployment
