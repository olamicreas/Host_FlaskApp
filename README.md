
# Hosting a Flask Application Using Gunicorn on RHEL

This guide provides detailed steps to host a Flask application using Gunicorn on RHEL (Red Hat Enterprise Linux) 8/9. 

## Prerequisites

- A server running RHEL 8 or 9
- Python 3.6 or higher installed
- Root or sudo privileges
- Basic knowledge of the terminal and command-line operations

## Step 1: Install Required Packages

First, you need to install Gunicorn, Nginx, and other dependencies.

```bash
sudo yum -y install python3 python3-pip nginx
```

## Step 2: Set Up Your Flask Application

1. **Create a Project Directory**: 

   ```bash
   mkdir ~/my_flask_app
   cd ~/my_flask_app
   ```

2. **Set Up a Virtual Environment**:

   ```bash
   python3 -m venv venv
   source venv/bin/activate
   ```

3. **Install Flask and Gunicorn**:

   ```bash
   pip install flask gunicorn
   ```

4. **Create Your Flask Application** (`app.py`):

   ```python
   from flask import Flask, render_template

   app = Flask(__name__)

   @app.route('/')
   def index():
       return render_template('index.html')

   if __name__ == "__main__":
       app.run()
   ```

5. **Create an HTML Template**:

   Create a `templates` folder and an `index.html` file:

   ```bash
   mkdir templates
   ```

   In `templates/index.html`:

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>Flask App</title>
   </head>
   <body>
       <h1>Hello, This is a Flask-powered webpage!</h1>
   </body>
   </html>
   ```

## Step 3: Test Gunicorn Locally

To make sure everything is working properly, you can test your Flask app locally using Gunicorn:

```bash
gunicorn --bind 0.0.0.0:8000 app:app
```

You should be able to visit `http://your_server_ip:8000` and see the "Hello, This is a Flask-powered webpage!" message.

## Step 4: Configure Nginx as a Reverse Proxy

1. **Create an Nginx Configuration File**:

   ```bash
   sudo vi /etc/nginx/conf.d/my_flask_app.conf
   ```

   Add the following configuration:

   ```nginx
   server {
       listen 80;
       server_name your_domain_or_IP;

       location / {
           proxy_pass http://127.0.0.1:8000;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```

2. **Test Nginx Configuration**:

   Check for syntax errors in the configuration:

   ```bash
   sudo nginx -t
   ```

   If everything is OK, restart Nginx:

   ```bash
   sudo systemctl restart nginx
   ```

## Step 5: Set Up Gunicorn as a Systemd Service

To ensure your Flask app starts with the system and is managed correctly, you can set up Gunicorn as a systemd service.

1. **Create a Gunicorn Service File**:

   ```bash
   sudo vi /etc/systemd/system/gunicorn.service
   ```

   Add the following content:

   ```ini
   [Unit]
   Description=Gunicorn instance to serve my_flask_app
   After=network.target

   [Service]
   User=your_username
   Group=nginx
   WorkingDirectory=/home/your_username/my_flask_app
   Environment="PATH=/home/your_username/my_flask_app/venv/bin"
   ExecStart=/home/your_username/my_flask_app/venv/bin/gunicorn --workers 3 --bind unix:my_flask_app.sock -m 007 app:app

   [Install]
   WantedBy=multi-user.target
   ```

   Replace `your_username` with your actual username.

2. **Start and Enable the Gunicorn Service**:

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl start gunicorn
   sudo systemctl enable gunicorn
   ```

   Check the status of the service:

   ```bash
   sudo systemctl status gunicorn
   ```

## Step 6: Finalize and Test

Visit your server's IP address or domain name. Your Flask app should be live and running, served by Gunicorn and Nginx.

