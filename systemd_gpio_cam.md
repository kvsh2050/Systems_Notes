# Guide: Running Python Scripts at Startup using systemd on Raspberry Pi

This guide explains how to run Python scripts at startup using `systemd` with and without virtual environments (venv), for both GUI and non-GUI applications — including GPIO and camera support.

---

## Prerequisites

* Raspberry Pi OS (desktop or lite)
* Your Python script ready (e.g., `/home/pi/myscript.py`)

---

## Directory Layout Example

```
/home/pi/
  ├── myscript.py         # Your Python script
  └── myenv/               # (optional) Python virtual environment
```

---

## Concept

A `systemd` service will run your script at boot, ensuring correct permissions and environment setup.

---

## 1. With Virtual Environment (venv)

### Step 1: Create the shell wrapper

Create `/home/pi/run_script.sh`:

```bash
#!/bin/bash
export DISPLAY=:0
export XAUTHORITY=/home/pi/.Xauthority
sleep 10
/home/pi/myenv/bin/python /home/pi/myscript.py
```

Make it executable:

```bash
chmod +x /home/pi/run_script.sh
```

### Step 2: Create the systemd unit file

```bash
sudo nano /etc/systemd/system/my_script.service
```

Paste:

```ini
[Unit]
Description=Python Script in venv
After=graphical.target

[Service]
Type=simple
ExecStart=/home/pi/run_script.sh
WorkingDirectory=/home/pi
Restart=on-failure
RestartSec=5

[Install]
WantedBy=graphical.target
```

### Step 3: Enable and start the service

```bash
sudo systemctl daemon-reload
sudo systemctl enable my_script.service
sudo systemctl start my_script.service
```

---

## 2. Without Virtual Environment

### Shell script not needed (but still useful for environment setup):

```bash
#!/bin/bash
export DISPLAY=:0
export XAUTHORITY=/home/pi/.Xauthority
sleep 10
/usr/bin/python3 /home/pi/myscript.py
```

Service file:

```ini
[Unit]
Description=Python Script without venv
After=graphical.target

[Service]
ExecStart=/home/pi/run_script.sh
WorkingDirectory=/home/pi
Restart=on-failure
RestartSec=5

[Install]
WantedBy=graphical.target
```

---

## GUI Script (Tkinter, etc.)

* Requires: `DISPLAY=:0`, `.Xauthority`, `sleep` for desktop load
* Use `After=graphical.target`

---

## Headless / No GUI Script

* Omit `DISPLAY` and `.Xauthority`
* Replace `graphical.target` with `multi-user.target`

Example:

```ini
[Unit]
Description=Headless Python Script
After=network.target

[Service]
ExecStart=/usr/bin/python3 /home/pi/myscript.py
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

---

## Scripts with GPIO or Camera

### Notes:

* GPIO libraries (like RPi.GPIO or `gpiozero`) require root access
* Camera modules require proper permissions (`video` group)

### Best Practice:

1. Use `/usr/bin/python3` unless using venv
2. Add to `video` group:

```bash
sudo usermod -aG video pi
```

3. Use `After=multi-user.target` if no GUI is involved
4. If accessing GPIO or camera, run script as `pi`, not `root`

### Tip: Run with logging

Modify shell script:

```bash
#!/bin/bash
export DISPLAY=:0
export XAUTHORITY=/home/pi/.Xauthority
sleep 10
/usr/bin/python3 /home/pi/myscript.py >> /home/pi/script.log 2>&1
```

---

## Useful Commands

```bash
sudo systemctl daemon-reload
sudo systemctl enable my_script.service
sudo systemctl start my_script.service
sudo systemctl status my_script.service
journalctl -u my_script.service -b
```

---

## Summary Table

| Type            | GUI | Venv | Uses GPIO/Camera | Needs Shell Script? |
| --------------- | --- | ---- | ---------------- | ------------------- |
| Headless        | No  | No   | Yes/No           | Optional            |
| GUI (Tkinter)   | Yes | No   | Optional         | Yes                 |
| GUI + venv      | Yes | Yes  | Optional         | Yes                 |
| Headless + venv | No  | Yes  | Yes/No           | Optional            |

---

For advanced logging, multi-script orchestration, or background daemons, extend `systemd` with `Type=notify`, `TimeoutStartSec`, or log redirection.
