Great question. File and directory **permissions** are crucial for security and correct functionality when deploying a Flask app with **Gunicorn** and **Nginx**.

Here’s how you should set permissions and ownership to ensure everything works **securely and correctly**:

---

## ✅ 1. Ownership and User Context

### Important Users:

* **Gunicorn** runs under the user defined in your systemd service file (commonly `www-data`).
* **Nginx** also runs under the `www-data` user.

### Change ownership of your project to `www-data`:

```bash
sudo chown -R www-data:www-data /var/www/myflaskapp
```

This allows both **Gunicorn** and **Nginx** to read/write as needed.

---

## ✅ 2. Directory and File Permissions

### General Rules:

* **Directories**: `755` — readable and executable by everyone, writable by owner.
* **Files**: `644` — readable by everyone, writable by owner.

### Apply these permissions:

```bash
# Set directory permissions
sudo find /var/www/myflaskapp -type d -exec chmod 755 {} \;

# Set file permissions
sudo find /var/www/myflaskapp -type f -exec chmod 644 {} \;
```

---

## ✅ 3. Special: The `.sock` File (Unix Socket)

Gunicorn creates a socket file, e.g., `/var/www/myflaskapp/myflaskapp.sock`, for communication with Nginx.

* Make sure **Gunicorn** has permission to create and write to it.
* Make sure **Nginx** can read from and write to it.

By setting the correct ownership (`www-data:www-data`) and running both services as `www-data`, this usually works without extra steps.

If needed:

```bash
sudo chown www-data:www-data /var/www/myflaskapp/myflaskapp.sock
sudo chmod 660 /var/www/myflaskapp/myflaskapp.sock
```

---

## ✅ Summary of Permissions

| Path                        | Owner      | Permission | Notes                         |
| --------------------------- | ---------- | ---------- | ----------------------------- |
| `/var/www/myflaskapp/`      | `www-data` | `755`      | Project directory             |
| Files inside (`*.py`, etc.) | `www-data` | `644`      | App code and configs          |
| `myflaskapp.sock`           | `www-data` | `660`      | Communication socket          |
| `venv/`                     | `www-data` | `755/644`  | Allow execution of virtualenv |

---

Let me know if you’re also dealing with multiple users or want stricter permission models (e.g., separating deploy and runtime roles).
