# 🌐 Deploy Django Project on Subdomain

Deploying a Django project on a subdomain can be complex, but with careful attention to detail, it becomes straightforward. This guide will walk you through all the necessary steps, ensuring your project runs smoothly on a subdomain in shared hosting. Let’s begin! 🚀

---

## 📋 Step 1: Generate `requirements.txt` File

To ensure all the necessary dependencies are installed on the server, you need to create a `requirements.txt` file. This file lists all the Python packages your Django project depends on.

### Command:
```bash
pip freeze > requirements.txt
```

### Example of `requirements.txt`:
```plaintext
asgiref==3.8.1
crispy-bootstrap5==2024.2
dj-database-url==2.2.0
dj-email-url==1.0.6
Django==5.0.7
django-cache-url==3.4.5
django-crispy-forms==2.2
environs==11.0.0
marshmallow==3.21.3
packaging==24.1
python-dotenv==1.0.1
sqlparse==0.5.1
typing_extensions==4.12.2
tzdata==2024.1
whitenoise==6.7.0
```

### Why This Step is Important:
The `requirements.txt` file allows you to install the exact versions of packages you used during development on the server with a single command:
```bash
pip install -r requirements.txt
```
This ensures consistency between your local development environment and the production server, minimizing issues due to version mismatches.

---

## 🛠️ Step 2: Configure Environment Variables in `settings.py`

Environment variables are essential for keeping sensitive information (like secret keys and debug settings) out of your codebase. We’ll use `environs` to manage environment variables.

### Update `settings.py`:
```python
from environs import Env

env = Env()
env.read_env()

SECRET_KEY = env("DJANGO_SECRET_KEY")  # Load SECRET_KEY from environment
DEBUG = env.bool("DJANGO_DEBUG")       # Load DEBUG mode from environment
ALLOWED_HOSTS = ['*']  # Use '*' for development, update this for production
```

### Why Use Environment Variables:
1. **Security:** Keeps sensitive information (like secret keys) out of your code, protecting it from being exposed.
2. **Flexibility:** Easily switch between different settings for development and production without changing code.
3. **Best Practices:** Managing environment variables separately makes your application more portable and easier to maintain.

---

## 🧰 Step 3: Serve Static Files Using `whitenoise`

In a production environment, serving static files (like CSS, JavaScript, and images) efficiently is crucial. `whitenoise` allows Django to serve static files directly, which simplifies the setup, especially on shared hosting.

### Update `settings.py`:
```python
STATIC_URL = "static/"
STATICFILES_DIRS = [BASE_DIR / "static"]  # Directory where your static files are stored during development
STATIC_ROOT = BASE_DIR / "staticfiles"    # Directory where collected static files will be placed

STORAGES = {
    "default": {
        "BACKEND": "django.core.files.storage.FileSystemStorage",
    },
    "staticfiles": {
        "BACKEND": "whitenoise.storage.CompressedManifestStaticFilesStorage",
    },
}
```

### Explanation:
- **`STATIC_URL`:** This sets the URL path where static files will be served. When a user visits your website, they can access static files via this URL.
- **`STATICFILES_DIRS`:** This tells Django where to find additional static files in your development environment.
- **`STATIC_ROOT`:** This is where Django will collect all static files for use in production. Running `python manage.py collectstatic` will gather files from various locations into this directory.
- **`whitenoise`:** Helps to compress, cache, and serve static files efficiently, enhancing performance.

---

## 🔧 Step 4: Modify `wsgi.py` for Static File Serving

Your `wsgi.py` file needs to be updated to include `whitenoise`. This ensures that static files are correctly handled by Django when running on the server.

### Update `wsgi.py`:
```python
import os
from django.core.wsgi import get_wsgi_application
from whitenoise import WhiteNoise

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'django_project.settings')

app = get_wsgi_application()
application = WhiteNoise(app)
```

### Explanation:
- **The `wsgi.py` file:** It acts as an entry point for your application. When your server starts, it reads this file to know how to serve your Django app.
- **Adding `WhiteNoise`:** This ensures your static files are compressed, cached, and served efficiently without needing a separate server like Nginx or Apache for static files.

---

## 🔑 Step 5: Create `.env` File for Environment Variables

The `.env` file is where you store sensitive information like the Django `SECRET_KEY` and `DEBUG` mode settings. This file is not included in version control, ensuring your secrets are kept safe.

### Example `.env` file:
```plaintext
DJANGO_SECRET_KEY=SBX9MViDhwT8Vytc3aBu1Kl1-aCcl49uyUgb_ws_jIHKlYYVYeWxKIMoyfHYrBuhs5U
DJANGO_DEBUG=False
```

### How to Generate a New Secret Key:
```bash
python -c 'import secrets; print(secrets.token_urlsafe(50))'
```

### Why This Matters:
- **Security:** By using a `.env` file, you can change the secret keys and configuration without modifying the source code.
- **Consistency:** Ensures that your development, testing, and production environments can be easily configured using the same settings.

---

## 🗂️ Step 6: Prepare Project Files for Deployment

Before uploading your project, organize your files. Exclude unnecessary items (like the virtual environment) and compress everything into a `.zip` file for easy transfer.

### Explanation:
- **Clean Project:** Ensures you’re only deploying what’s needed, which helps reduce server storage usage and potential conflicts.
- **Exclude Virtual Environment:** The server will have its own virtual environment, so there’s no need to upload your local one.

---

## 🌐 Step 7: Create a Subdomain in cPanel

1. **Login to cPanel.**
2. Navigate to **Domains** > **Create A New Domain**.
3. **Enter Subdomain** (e.g., `article.mrfaizi.com`).
4. Confirm that the physical path matches:
   ```plaintext
   /home/mrfaizi/public_html/article.mrfaizi.com
   ```

### Explanation:
Creating a subdomain allows you to deploy different applications under one main domain. For instance, `article.mrfaizi.com` is separate from `mrfaizi.com`, which can host another app or service.

---

## 🛠️ Step 8: Set Up Python Application on cPanel

1. **Go to `Setup Python App`** under **Software** in cPanel.
2. **Select Python Version:** Choose the Python version compatible with your project.
3. **Application Root:** Set it to:
   ```plaintext
   /home/mrfaizi/public_html/article.mrfaizi.com
   ```
4. Click **Create.**

### Explanation:
Setting up the Python application on cPanel ensures that your Django project runs in a controlled environment. This setup isolates your app from others, reducing conflicts and making it easier to manage dependencies.

---

## 📂 Step 9: Upload Project Files

1. Navigate to `/home/mrfaizi/public_html/article.mrfaizi.com`.
2. Upload the compressed `.zip` file you created earlier.
3. **Extract the contents.**

### Explanation:
By uploading the `.zip` file, you can transfer your project to the server in a neat package. Extracting it ensures the files are in the correct directory, ready for configuration and setup.

---

## ✏️ Step 10: Modify `passenger_wsgi.py`

Update `passenger_wsgi.py` so the server knows how to interface with your Django application.
```python
from django_project.wsgi import application
```

### Explanation:
The `passenger_wsgi.py` file acts as the interface between your Django project and the server. Setting it correctly ensures that the server starts and manages your Django app properly.

---

## 🔄 Step 11: Update `ALLOWED_HOSTS` in `settings.py`

Update the `ALLOWED_HOSTS` list to allow requests from your subdomain:
```python
ALLOWED_HOSTS = ['article.mrfaizi.com', 'www.article.mrfaizi.com']
```

### Why This Step is Crucial:
- **Security Measure:** Django uses `ALLOWED_HOSTS` to prevent HTTP Host header attacks.
- **Ensures Accessibility:** Only requests from the specified domains can reach your application.

---

## 📦 Step 12: Execute Python Script to Install Dependencies

1. **Go to `Setup Python App`** in cPanel.
2. **Edit the existing application.**
3. **In the `Execute Python Script` section, enter:**
   ```bash
   /home/mrfaizi/virtualenv/public_html/article.mrfaizi.com/3.11/bin/pip install -r /home/mrfa

izi/public_html/article.mrfaizi.com/requirements.txt
   ```

### Explanation:
This command will install all the required packages listed in `requirements.txt`, ensuring your project has all the necessary libraries to run. Double-check the path and filenames to avoid errors.

---

## ⚙️ Step 13: Activate the Virtual Environment

Use the following command to activate the virtual environment:
```bash
source /home/mrfaizi/virtualenv/public_html/article.mrfaizi.com/3.11/bin/activate && cd /home/mrfaizi/public_html/article.mrfaizi.com
```

### Why Activation is Necessary:
Activating the virtual environment allows you to run your Django project commands (like applying migrations or running the server) with all required dependencies, avoiding conflicts with other Python installations on the server.

---

## 🖥️ Step 14: Final Setup Using Terminal

1. Open **Advanced** > **Terminal** in cPanel.
2. **Activate the Environment:**
   ```bash
   source /home/mrfaizi/virtualenv/public_html/article.mrfaizi.com/3.11/bin/activate && cd /home/mrfaizi/public_html/article.mrfaizi.com
   ```
3. **Verify Installed Packages:**
   ```bash
   pip list
   ```
4. **Apply Migrations:**
   ```bash
   python manage.py migrate
   ```
5. **Collect Static Files:**
   ```bash
   python manage.py collectstatic
   ```

### Explanation:
- **Migrations:** Set up your database schema based on your models.
- **Static Files:** Gather all your CSS, JavaScript, and other static assets to be served in production.
- **Terminal Commands:** Make sure everything is configured correctly before accessing your subdomain.

---

## 🎉 Deployment Complete!

Congratulations! Your Django project is now live and running on `article.mrfaizi.com`. Open your browser, visit the subdomain, and check out your newly deployed application! 🎊
