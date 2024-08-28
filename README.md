Here's a detailed guide without the optional steps:

### 1. **Setup on RHEL (`example.com`):**
   - **1.1 Install Dependencies:**
     - SSH into your RHEL server:
       ```bash
       ssh user@amp012765.nomura.com
       ```
     - **Install Python 3 and pip:**
       ```bash
       sudo yum install python3 -y
       sudo yum install python3-pip -y
       ```
     - **Create a Project Directory:**
       ```bash
       mkdir ~/flask_app && cd ~/flask_app
       ```
     - **Create `requirements.txt`:**
       - Add Flask and Gunicorn as dependencies:
         ```bash
         echo "Flask" > requirements.txt
         echo "Gunicorn" >> requirements.txt
         ```
     - **Upgrade pip and install the required dependencies:**
       ```bash
       pip3 install --upgrade pip
       pip3 install -r requirements.txt
       ```

### 2. **Develop the Flask Application:**
   - **2.1 Create `app.py`:**
     - In the `~/flask_app` directory, create the Flask application file:
       ```bash
       nano app.py
       ```
     - Add the following code:
       ```python
       from flask import Flask
       app = Flask(__name__)

       @app.route('/')
       def hello_world():
           return 'Hello, World!'
       ```
     - Save and exit (`CTRL + X`, then `Y`, then `Enter`).

### 3. **Configure Gunicorn for Production:**
   - **3.1 Create Gunicorn Configuration:**
     - Create a `gunicorn_config.py` file:
       ```bash
       nano gunicorn_config.py
       ```
     - Add the following configuration:
       ```python
       import os

       workers = int(os.environ.get('GUNICORN_PROCESSES', '2'))
       threads = int(os.environ.get('GUNICORN_THREADS', '4'))
       bind = os.environ.get('GUNICORN_BIND', '0.0.0.0:8080')
       forwarded_allow_ips = '*'
       secure_scheme_headers = { 'X-Forwarded-Proto': 'https' }
       ```
     - Save and exit (`CTRL + X`, then `Y`, then `Enter`).

### 4. **Run the Flask Application with Gunicorn:**
   - **4.1 Start Gunicorn:**
     - Run the Gunicorn server to serve the Flask application:
       ```bash
       gunicorn --config gunicorn_config.py app:app
       ```
     - The application will be running on `http://0.0.0.0:8080`, accessible from the server itself.

### 5. **Configure Firewall and SELinux (if applicable):**
   - **5.1 Open Port 8080:**
     - Open port 8080 to allow external access:
       ```bash
       sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent
       sudo firewall-cmd --reload
       ```
   - **5.2 Configure SELinux (if enforced):**
     - Allow Flask to bind to port 8080:
       ```bash
       sudo semanage port -a -t http_port_t -p tcp 8080
       ```
     - If `semanage` is not installed:
       ```bash
       sudo yum install policycoreutils-python-utils
       ```

### 6. **Make the Application Accessible via `example.com`:**
   - **6.1 Ensure the Server's Public IP is Mapped to `amp012765.nomura.com`:**
     - Make sure the DNS is correctly set up to point `example.com` to your server’s IP address.

   - **6.2 Access the Application via Browser:**
     - Open your browser and navigate to `http://example.com:8080/`.
     - You should see the output:
       ```
       Hello, World!
       ```

### 7. **Conclusion:**
   - Your Flask application is now up and running, accessible through the hostname `amp012765.nomura.com` on port `8080`.

Let me know if you need help with any specific steps!
