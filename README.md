# guide-django-deploy-uwsgi-nginx-linux

This is a complete quick guide to deploy a Django application with Nginx web server using uWSGI interface.

To deploy a **Django** application on your VPS, you typically use **uWSGI** as the application server and **Nginx** as the reverse proxy. Here's a step-by-step guide on deploying Django on your VPS, assuming you're using **Nginx** as the web server and want to set it up with a domain and SSL.

### Prerequisites:
- A VPS with root access (DigitalOcean in your case).
- Nginx installed.
- A domain name pointing to your VPS.
- SSL with Certbot (Let's Encrypt) installed.

---

### 1. **Install Python and Virtual Environment**
Make sure you have Python installed, along with `pip` and `virtualenv` to isolate your project dependencies.

```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv
```

---

### 2. **Clone Your Django App to the VPS**
SSH into your VPS and navigate to the directory where you want to deploy your application, such as `/var/www/your-django-app`:

```bash
cd /var/www
git clone https://github.com/your-username/your-django-app.git
cd your-django-app
```

---

### 3. **Set Up a Virtual Environment**
Create and activate a virtual environment:

```bash
python3 -m venv venv
source venv/bin/activate
```

Install the necessary dependencies (including Django, uWSGI, and any other requirements for your app):

```bash
pip install -r requirements.txt
```

---

### 4. **Configure Django Settings for Production**
- **Update `settings.py`**: Modify your `ALLOWED_HOSTS` to include your domain name and server IP:

```python
ALLOWED_HOSTS = ['your-domain.com', 'www.your-domain.com', 'your-server-ip']
```

- **Set up Static and Media files**:
  
In `settings.py`, configure the paths for serving static files:
```python
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static/')
```

Collect the static files:
```bash
python manage.py collectstatic
```

---

### 5. **Set Up uWSGI (WSGI Server)**
uWSGI is a WSGI server that will serve your Django application.

1. Install uWSGI:

```bash
pip install uwsgi
```

2. Create a `uwsgi.ini` configuration file in your Django project directory:

```ini
[uwsgi]
chdir = /var/www/your-django-app
module = your_django_project.wsgi:application

# Enable socket for Nginx
socket = /run/uwsgi/your_project.sock
chmod-socket = 664

# Processes and threads
workers = 4
threads = 2

# Master process
master = true
vacuum = true

# Log output
logto = /var/log/uwsgi/uwsgi.log

# Static and media files
static-map = /static=/var/www/your-django-app/static
static-map = /media=/var/www/your-django-app/media
```

3. Test uWSGI with your Django app:

```bash
uwsgi --ini uwsgi.ini
```

If the app runs successfully, you can proceed to create a systemd service for uWSGI.

---

### 6. **Set Up uWSGI as a Systemd Service**
To keep your Django app running, create a systemd service file for uWSGI:

1. **Create a uWSGI service file**:

```bash
sudo nano /etc/systemd/system/uwsgi.service
```

2. **Add the following configuration**:

```ini
[Unit]
Description=uWSGI service for Django application
After=network.target

[Service]
User=user
Group=user
WorkingDirectory=/var/www/your-django-app
ExecStart=/var/www/your-django-app/venv/bin/uwsgi --ini /var/www/your-django-app/uwsgi.ini
Restart=always

[Install]
WantedBy=multi-user.target
```

3. **Start and enable uWSGI**:

```bash
sudo systemctl start uwsgi
sudo systemctl enable uwsgi
```

Check the status of uWSGI to ensure it's running:

```bash
sudo systemctl status uwsgi
```

---

### 7. **Configure Nginx as a Reverse Proxy**
Nginx will act as a reverse proxy to uWSGI.

1. **Create an Nginx Server Block**:

```bash
sudo nano /etc/nginx/sites-available/your_domain
```

Add the following configuration:

```nginx
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;

    location / {
        include uwsgi_params;
        uwsgi_pass unix:/run/uwsgi/your_project.sock;
    }

    location /static/ {
        alias /var/www/your-django-app/static/;
    }

    location /media/ {
        alias /var/www/your-django-app/media/;
    }

    # Optional: Redirect HTTP to HTTPS
    # return 301 https://$server_name$request_uri;
}
```

2. **Enable the Nginx site**:

```bash
sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/
```

3. **Test Nginx for syntax errors**:

```bash
sudo nginx -t
```

4. **Restart Nginx**:

```bash
sudo systemctl restart nginx
```

---

### 8. **Set Up SSL with Certbot (Optional)**
If you want to enable HTTPS using Let's Encrypt, use Certbot to automatically configure SSL for your domain.

1. Install Certbot (if not already installed):

```bash
sudo apt install certbot python3-certbot-nginx
```

2. Obtain an SSL certificate and configure Nginx:

```bash
sudo certbot --nginx -d your-domain.com -d www.your-domain.com
```

Certbot will automatically update your Nginx configuration to serve your Django app securely over HTTPS.

---

### 9. **Verify Deployment**
Visit your domain (`http://your-domain.com` or `https://your-domain.com` if using SSL) and verify that your Django app is running correctly.

---

### Summary:
- **uWSGI**: Use it as the WSGI server for your Django app.
- **Nginx**: Configure it as a reverse proxy to serve your app.
- **Certbot**: Use it to set up SSL for secure HTTPS access.
- **Static Files**: Serve them using Nginx while routing dynamic requests to uWSGI.
