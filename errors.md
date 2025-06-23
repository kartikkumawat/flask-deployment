## 🧾 Deployment Error Log & Solution Document

### 📌 **1. Package Installation Errors**

#### **Error**

```bash
Using legacy 'setup.py install' for inexactsearch, since package 'wheel' is not installed.
...
ERROR: Could not install packages due to an OSError: [Errno 13] Permission denied...
```

#### **Cause**

* Missing `wheel`, `setuptools`, and incorrect file permissions in virtualenv.

#### **Solution**

1. Fix permissions:

   ```bash
   sudo chown -R $(whoami):$(whoami) /var/www/ai/venv
   ```

2. Install required build tools:

   ```bash
   pip install --upgrade pip setuptools wheel
   ```

---

### 📌 **2. setuptools Missing for Legacy Package**

#### **Error**

```bash
ERROR: Can not execute `setup.py` since setuptools is not available in the build environment.
```

#### **Cause**

* `setuptools` not installed in the environment.

#### **Solution**

```bash
pip install setuptools
```

---

### 📌 **3. Flask App Crashes: AttributeError – 'Flask' object has no attribute 'db\_manager'**

#### **Error**

```python
db_manager = current_app.db_manager
AttributeError: 'Flask' object has no attribute 'db_manager'
```

#### **Cause**

* `db_manager` was not added to `app` before accessing it.

#### **Solution**

Initialize `db_manager` in app context:

```python
from app.models import DatabaseManager

with app.app_context():
    app.db_manager = DatabaseManager()
```

---

### 📌 **4. Flask App Crashes: AttributeError – 'Flask' object has no attribute 'chatbot\_engine'**

#### **Error**

```python
chatbot_engine = current_app.chatbot_engine
AttributeError: 'Flask' object has no attribute 'chatbot_engine'
```

#### **Cause**

* `chatbot_engine` was accessed before being initialized.

#### **Solution**

```python
from app.chatbot import EnhancedChatbotEngine

with app.app_context():
    app.chatbot_engine = EnhancedChatbotEngine(app.db_manager)
```

---

### 📌 **5. Incorrect Initialization of `DatabaseManager`**

#### **Error**

```python
AttributeError: 'str' object has no attribute 'db_manager'
```

#### **Cause**

* `db_manager = DatabaseManager` (missing parentheses) results in a reference to the class, not an instance.

#### **Solution**

```python
db_manager = DatabaseManager()  # Use parentheses to create instance
```

---

### 📌 **6. Wrong Parameter Handling in Method Call**

#### **Error**

```python
TypeError: DatabaseManager.get_last_message_timestamp() missing 1 required positional argument: 'session_id'
```

#### **Cause**

* Accessing method via the class instead of instance, or forgetting the required parameter.

#### **Solution**

Use instance method properly:

```python
last_message_timestamp = app.db_manager.get_last_message_timestamp(session_id)
```

---

### ✅ **Final Working Initialization Snippet**

```python
# app/__init__.py or equivalent startup file
from flask import Flask
from app.models import DatabaseManager
from app.chatbot import EnhancedChatbotEngine

app = Flask(__name__)

with app.app_context():
    app.db_manager = DatabaseManager()
    app.chatbot_engine = EnhancedChatbotEngine(app.db_manager)
```

---

### ✅ **Tips for Production Setup**

1. Always upgrade `pip`, `setuptools`, and `wheel`:

   ```bash
   pip install --upgrade pip setuptools wheel
   ```

2. Use `app.app_context()` when assigning custom attributes to Flask app.

3. Handle package installs via a `requirements.txt` and use:

   ```bash
   pip install -r requirements.txt
   ```

4. Monitor errors with:

   ```bash
   sudo journalctl -u <your-service-name> -f
   ```

