# Deploying a Django Application using Nginx and uWSGI

Here is a comprehensive guide to setting up your Django application with Nginx and uWSGI (as the WSGI server):

---

## Prerequisites

Before you begin, make sure you have:

- A VPS running a Linux distribution (Ubuntu 20.04 or similar).
- SSH access to the VPS.
- A domain name (optional but recommended).

## Step 1: Setting Up the Server

1. **Connect to Your VPS**:

   Use SSH to connect to your server. Replace `your_username` and `your_ip_address` with your actual username and IP address.

   ```bash
   ssh your_username@your_ip_address
   ```

2. **Update the Package List**:

   Run the following command to update your server’s package list:

   ```bash
   sudo apt update
   ```

## Step 2: Installing Required Packages

Run the following commands to:

Install Nginx
```bash
sudo apt install -y nginx
```

Install Python and pip (if not already installed)
```bash
sudo apt install -y python3 python3-pip python3-venv
```

Install Python and pip (if not already installed)
```bash
sudo pip install uwsgi
```

## Step 3: Creating a Django Project

1. **Navigate to Your Desired Directory**:

Choose a directory for your Django project. In your case:

```bash
cd /home/imjatinx/Documents/
```

2. **Create a Virtual Environment**:

Create and activate a virtual environment:

```bash
python3 -m venv venv
```

```bash
source venv/bin/activate
```

3. **Install Django**:

   With your virtual environment activated, install Django:

   ```bash
   pip install django
   ```

4. **Create a New Django Project**:

   Use the Django management command to create a new project. You can skip this step if your project already exists.

   ```bash
   django-admin startproject example_uwsgi
   ```

5. **Navigate into Your Project Directory**:

   Change into your newly created project directory:

   ```bash
   cd example_uwsgi
   ```


## Step 4: Configuring uWSGI

1. **Create a uWSGI Configuration File**:

   Create a file named `example_uwsgi.ini` in your project directory:

   ```bash
   nano example_uwsgi.ini
   ```

2. **Add the Following Configuration**:

   Paste the following content into `example_uwsgi.ini`, using a TCP port for communication:

   ```ini
   [uwsgi]
   module = example_uwsgi.wsgi:application
   home = /home/imjatinx/Documents/example_uwsgi/venv
   chdir = /home/imjatinx/Documents/example_uwsgi
   http = 127.0.0.1:8000
   chmod-socket = 660
   vacuum = true
   die-on-term = true
   ```

3. **Save and Exit**:

   Save the file (CTRL + O, then Enter) and exit (CTRL + X).

5. **Run uWSGI Test**:

   Run uWSGI using the configuration file you created:

```bash
uwsgi --ini example_uwsgi.ini
```

## Step 5: Installing and Configuring Nginx

1. **Create a New Nginx Configuration File**:

   Create a new configuration file for your Django project:

   ```bash
   sudo nano /etc/nginx/sites-available/example_uwsgi
   ```

2. **Add the Following Configuration**:

   Paste the following content into the Nginx configuration file, ensuring that the `server_name` directive matches your domain or IP address:

   ```nginx
   server {
       listen 80;  # Listen on HTTP port 80
       server_name your_domain_or_ip;  # Replace with your domain name or server IP

       location = /favicon.ico { access_log off; log_not_found off; }
       location /static/ {
           root /home/imjatinx/Documents/example_uwsgi;  # Adjust this if you have a different static directory
       }

       location / {
           include uwsgi_params;  # Include standard uwsgi parameters
           uwsgi_pass 127.0.0.1:8000;  # Use TCP for uWSGI
       }
   }
   ```

3. **Enable the Nginx Configuration**:

   Link your new configuration file to the sites-enabled directory and test the Nginx configuration:

   ```bash
   # Enable the new site
   sudo ln -s /etc/nginx/sites-available/example_uwsgi /etc/nginx/sites-enabled/

   # Test for syntax errors
   sudo nginx -t
   ```

4. **Restart Nginx**:

   If there are no errors, restart Nginx to apply the changes:

   ```bash
   sudo systemctl restart nginx
   ```

## Step 6: Collecting Static Files

If your project uses static files, collect them using the following command:

```bash
cd /home/imjatinx/Documents/example_uwsgi
python manage.py collectstatic
```

## Step 7: Configuring uWSGI as a Service (Optional)

To run uWSGI in the background and have it start automatically on boot, create a systemd service file:

1. **Create the Systemd Service File**:

   ```bash
   sudo nano /etc/systemd/system/uwsgi.service
   ```

2. **Add the Following Content**:

   Paste the following content into the `uwsgi.service` file:

   ```ini
   [Unit]
   Description=uWSGI instance to serve example_uwsgi
   After=network.target

   [Service]
   User=your_username  # Replace with your username
   Group=www-data
   WorkingDirectory=/home/imjatinx/Documents/example_uwsgi
   Environment="PATH=/home/imjatinx/Documents/example_uwsgi/venv/bin"
   ExecStart=/home/imjatinx/Documents/example_uwsgi/venv/bin/uwsgi --ini /home/imjatinx/Documents/example_uwsgi/example_uwsgi.ini

   [Install]
   WantedBy=multi-user.target
   ```

3. **Start and Enable the uWSGI Service**:

   Start and enable the uWSGI service:

   ```bash
   sudo systemctl start uwsgi
   sudo systemctl enable uwsgi
   ```

## Step 8: Testing Your Application

Open a web browser and navigate to your server's domain name or IP address. You should see the default Django welcome page.

## Step 9: Conclusion

Congratulations! You have successfully deployed a Django application using Nginx and uWSGI on your VPS with TCP configuration. You can further customize your application as needed.

If you encounter any issues or have questions, feel free to seek help!

---

This updated documentation uses TCP for uWSGI. Let me know if you need any further adjustments or additional information!
