## Step 1: Prepare the Environment
### 1.1 Update and install dependencies
```bash
sudo apt update
```
```bash
sudo apt install python3-pip python3-venv nginx -y
```
### 1.2 Create project directory
```bash
mkdir -p /var/www/ai
```
```bash
cd /var/www/ai
```
### 1.3 Set up Python virtual environment
```bash
python3 -m venv venv
```
```bash
source venv/bin/activate
```
### 1.4 Install Flask and Gunicorn

```bash
pip install flask gunicorn
```
Or
```bash
pip install -r requirements.txt
```


## Step 2: Create Your Flask App

### 2.1 When your flask as example Flask app (`app.py`)

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello, Flask with Gunicorn and Nginx!"
```
## Test Gunicorn

Run Gunicorn manually to verify it works:

```bash
gunicorn --bind 0.0.0.0:8001 app:app
```

Visit `http://your_server_ip:8001` to test it.

---

### 2.2 When your flask as example flask app (`app.py`)

```python
from app import create_app
app = create_app()
```

## Test Gunicorn

Run Gunicorn manually to verify it works:

```bash
gunicorn --bind 0.0.0.0:8001 'app:create_app()'
```
Or
```bash
gunicorn --bind 0.0.0.0:8001 wsgi:app
```

Visit `http://your_server_ip:8001` to test it.

---

## Step 3: Set proper permissions
To the directory `/var/www/ai/`
* Change the ownership:
   ```bash
  sudo chown -R www-data:www-data /var/www/ai
  ```
* Or adjust the permissions (if you don't want to change ownership):
  ```bash
  sudo chmod 755 /var/www/ai
  sudo chmod 766 /var/www/ai/ai.sock
  ```

  ---

## Step 4: Create Gunicorn Systemd Service

### 4.1 Create the service file

```bash
sudo nano /etc/systemd/system/ai.service
```

### 4.2 Paste the following:

```ini
[Unit]
Description=Gunicorn instance to serve myflaskapp
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/ai
Environment="PATH=/var/www/ai/venv/bin"
ExecStart=/var/www/ai/venv/bin/gunicorn --workers 3 --bind unix:/var/www/ai/ai.sock wsgi:app
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```
or when using app=create_app in app/__init__.py
```ini
[Unit]
Description=Gunicorn instance to serve myflaskapp
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/ai
Environment="PATH=/var/www/ai/venv/bin"
ExecStart=/var/www/ai/venv/bin/gunicorn --workers 3 --bind unix:/var/www/ai/ai.sock 'app:create_app()'
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```
### 4.3 Start and enable the service

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl start ai 
sudo systemctl enable ai
```

---
## Step 5: Configure Nginx

### 5.1 Create Nginx config file

```bash
sudo nano /etc/nginx/sites-available/ai
```
### 5.2 Paste the following:

```nginx
server {
    listen 8001;
    server_name 192.168.2.201;

    location / {
        include proxy_params;
        proxy_pass http://unix:/var/www/ai/ai.sock;
    }
}
```

### 5.3 Enable the Nginx site

```bash
sudo ln -s /etc/nginx/sites-available/ai /etc/nginx/sites-enabled
sudo nginx -t
sudo systemctl restart nginx
```

---
## Step 6: Adjust Firewall (Optional)

If you have UFW enabled:

```bash
sudo ufw allow 'Nginx Full'
```
Or 
```bash
sudo ufw allow 8001
```


## Problems
### show error logs 
* nginx
  ```bash
  sudo tail -n 50 /var/log/nginx/error.log
  ```
* gunicorn
  ```bash
  sudo journalctl -u ai -xe
  ```

### this also require 
```bash
sudo mkdir -p /var/www/.cache
sudo chown -R www-data:www-data /var/www/.cache  # Replace with actual user if different
```
```bash
sudo chmod -R 777 /var/www/.cache
```

## set permissions
```bash
sudo chown www-data:www-data /var/www/ai/ai.sock
sudo chmod 660 /var/www/ai/ai.sock
```
### Set directory permissions
```bash
sudo find /var/www/ai -type d -exec chmod 755 {} \;
```

### Set file permissions
```bash
 sudo find /var/www/ai -path /var/www/ai/venv/bin -prune -o -type f -exec chmod 644 {} \;
```

#### if file not 
```bash
sudo chown -R www-data:www-data /var/www/ai/logs
sudo chmod -R 755 /var/www/ai/logs
```

#### if file already 
```bash
sudo chown www-data:www-data /var/www/ai/logs/app.log
sudo chmod 644 /var/www/ai/logs/app.log
```
## problem - 
```python
try:
    nltk.data.find('corpora/words')
except LookupError:
    nltk.download('words', quiet=True)
```

is good in development, but in production, it's fragile because:

It silently tries to download data at runtime, which:

Fails if the app has no write access

Slows down boot time

Breaks if there's no internet or restricted access 

```python
import nltk
nltk.data.path.append('/var/www/ai/nltk_data')

try:
    nltk.data.find('corpora/words')
except LookupError:
    nltk.download('words', download_dir='/var/www/ai/nltk_data', quiet=True)
```

