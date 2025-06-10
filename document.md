Deploying a Flask project on a Linux machine using **Gunicorn** and **Nginx** involves several clear steps. Below is a **step-by-step guide**, including example configuration files for **Gunicorn** (systemd service file) and **Nginx**.

---

## ✅ Step 1: Prepare the Environment

### 1.1 Update and install dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install python3-pip python3-venv nginx -y
```

### 1.2 Create project directory

```bash
mkdir -p /var/www/myflaskapp
cd /var/www/myflaskapp
```

### 1.3 Set up Python virtual environment

```bash
python3 -m venv venv
source venv/bin/activate
```

### 1.4 Install Flask and Gunicorn

```bash
pip install flask gunicorn
```

---

## ✅ Step 2: Create Your Flask App

### 2.1 Example Flask app (`app.py`)

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello, Flask with Gunicorn and Nginx!"
```

---

## ✅ Step 3: Test Gunicorn

Run Gunicorn manually to verify it works:

```bash
gunicorn --bind 0.0.0.0:8000 app:app
```

Visit `http://your_server_ip:8000` to test it.

---

## ✅ Step 4: Create Gunicorn Systemd Service

### 4.1 Create the service file

```bash
sudo nano /etc/systemd/system/myflaskapp.service
```

### 4.2 Paste the following:

```ini
[Unit]
Description=Gunicorn instance to serve myflaskapp
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/myflaskapp
Environment="PATH=/var/www/myflaskapp/venv/bin"
ExecStart=/var/www/myflaskapp/venv/bin/gunicorn --workers 3 --bind unix:/var/www/myflaskapp/myflaskapp.sock app:app

[Install]
WantedBy=multi-user.target
```

### 4.3 Start and enable the service

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl start myflaskapp
sudo systemctl enable myflaskapp
```

---

## ✅ Step 5: Configure Nginx

### 5.1 Create Nginx config file

```bash
sudo nano /etc/nginx/sites-available/myflaskapp
```

### 5.2 Paste the following:

```nginx
server {
    listen 80;
    server_name your_domain_or_ip;

    location / {
        include proxy_params;
        proxy_pass http://unix:/var/www/myflaskapp/myflaskapp.sock;
    }
}
```

### 5.3 Enable the Nginx site

```bash
sudo ln -s /etc/nginx/sites-available/myflaskapp /etc/nginx/sites-enabled
sudo nginx -t
sudo systemctl restart nginx
```

---

## ✅ Step 6: Adjust Firewall (Optional)

If you have UFW enabled:

```bash
sudo ufw allow 'Nginx Full'
```

---

## ✅ Step 7: Test Deployment

Go to `http://your_domain_or_ip/` and you should see:

```
Hello, Flask with Gunicorn and Nginx!
```

---

## 📁 Directory Structure Example

```
/var/www/myflaskapp/
│
├── app.py
├── venv/
├── myflaskapp.sock (created by Gunicorn)
```

---

Let me know if you'd like HTTPS (Let's Encrypt) setup included.
