
# 🚀 How to Auto-Start a Python Script at Boot Using systemd

This guide walks you through setting up a Python script to automatically run at boot using **systemd** on a Raspberry Pi or Linux system.

---

## 📝 1. Create Your Python Script

Save your Python script to `/home/pi/startup_test.py`:

```python
#!/usr/bin/env python3
import time

while True:
    with open("/home/pi/startup_output.log", "a") as f:
        f.write("Hello from startup!\n")
    time.sleep(5)
```

Make it executable:

```bash
chmod +x /home/pi/startup_test.py
```

---

## 🛠️ 2. Create the systemd Service Unit File

Create a new service file:

```bash
sudo nano /etc/systemd/system/startup_test.service
```

Paste the following content:

```ini
[Unit]
Description=Run Python script at boot
After=network.target

[Service]
ExecStart=/usr/bin/python3 /home/pi/startup_test.py
WorkingDirectory=/home/pi
User=pi
Restart=always
Environment=PYTHONUNBUFFERED=1

[Install]
WantedBy=multi-user.target
```

> 🔍 `PYTHONUNBUFFERED=1` ensures `print()` or file writes are not buffered.

Save and exit (`Ctrl+X`, then `Y`, then `Enter`).

---

## 🔄 3. Reload systemd and Start the Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable startup_test.service
sudo systemctl start startup_test.service
```

> 📝 `enable` makes it run at every boot.

---

## 🔁 4. Reboot and Test

Reboot your Pi:

```bash
sudo reboot
```

After reboot, check output:

```bash
cat /home/pi/startup_output.log
```

You should see:

```
Hello from startup!
Hello from startup!
...
```

---

## 🧪 5. Check Service Status and Logs

### View service status:

```bash
systemctl status startup_test.service
```

### View logs (from this boot):

```bash
journalctl -u startup_test.service -b
```

---

## 🛑 6. Stop or Disable the Service (if needed)

To stop the running service:

```bash
sudo systemctl stop startup_test.service
```

To disable it from starting on boot:

```bash
sudo systemctl disable startup_test.service
```

To remove it completely:

```bash
sudo rm /etc/systemd/system/startup_test.service
sudo systemctl daemon-reload
```

---

## ✅ Final Notes

* Use **absolute paths** in the Python script.
* Always check **file permissions** (log files must be writable by the user).
* Use **try-except blocks** inside the script to prevent crashes.


