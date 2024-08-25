# Hosting a Python Flask Application on RHEL with Apache Tomcat

This guide will walk you through the steps to host a Python Flask application on a Red Hat Enterprise Linux (RHEL) server using Apache Tomcat. Although Tomcat is primarily used for Java applications, you can proxy requests to a Python application using Gunicorn.

## Prerequisites

- A RHEL 7 or later server
- Apache Tomcat installed
- Python 3 installed
- Basic knowledge of Linux command-line usage

## Step 1: Install Apache Tomcat

First, install Apache Tomcat on your RHEL server.

```bash
sudo yum install tomcat
```

Start and enable the Tomcat service:

```bash
sudo systemctl start tomcat
sudo systemctl enable tomcat
```

## Step 2: Install Python and Flask

Make sure Python 3 is installed on your server. Then, set up a virtual environment and install Flask along with Gunicorn.

```bash
python3 -m venv myenv
source myenv/bin/activate
pip install flask gunicorn
```

## Step 3: Create Your Flask Application

Create a simple Flask application (e.g., `app.py`).

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello, World!"

if __name__ == "__main__":
    app.run()
```

## Step 4: Run Flask with Gunicorn

Use Gunicorn to run your Flask application. This command binds the app to port 8000:

```bash
gunicorn --bind 0.0.0.0:8000 app:app
```

## Step 5: Configure Apache Tomcat to Proxy Requests

You need to configure Tomcat to proxy HTTP requests to the Flask application running on Gunicorn. Modify the `server.xml` file of your Tomcat installation, usually found in `/etc/tomcat/server.xml` or `/opt/tomcat/conf/server.xml`.

Add the following configuration under the `<Host>` section:

```xml
<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
<Context path="" docBase="/path/to/your/app" />
<Valve className="org.apache.catalina.authenticator.BasicAuthenticator" />

<Proxy ajpHost="localhost" ajpPort="8009" protocol="AJP/1.3" />
<ProxyPass / ajp://localhost:8009/>
<ProxyPassReverse / ajp://localhost:8009/>
```

### Example Configuration

Here is an example configuration to add under your `<Host>` section:

```xml
<Host name="localhost"  appBase="webapps" unpackWARs="true" autoDeploy="true">
    <Context path="" docBase="/var/www/html/python" />
    <Proxy ajpHost="localhost" ajpPort="8009" protocol="AJP/1.3" />
    <ProxyPass / ajp://localhost:8009/>
    <ProxyPassReverse / ajp://localhost:8009/>
</Host>
```

## Step 6: Restart Tomcat

After making changes to `server.xml`, restart the Tomcat service to apply the new configuration.

```bash
sudo systemctl restart tomcat
```

## Step 7: Access Your Flask Application

Your Flask application should now be accessible through Tomcat. Open your web browser and navigate to:

```
http://<your-server-ip>:8080/
```

You should see the output from your Flask application, "Hello, World!"

## Conclusion

This guide covered how to host a Python Flask application on a RHEL server using Apache Tomcat. While Tomcat is more commonly used with Java applications, it can proxy requests to a Python-based application using Gunicorn and the mod_proxy modules.
