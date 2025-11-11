# 🧠 Setting up App as a Systemd Service (Ubuntu EC2)

This guide explains how to run a Python 3.11 application as a background service using **systemd**, ensuring it auto-starts on reboot and uses your virtual environment.

---

## 📂 Project Structure

```
/home/ubuntu/examiner-ai/
|
├── other files...
├── app.py
├── requirements.txt
└── venv/
```

---

## ⚙️ 1. Create and Activate Virtual Environment

```bash
cd /home/ubuntu/examiner-ai
python3.11 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

✅ Verify correct Python version:
```bash
python --version
# should show Python 3.11.x
```

Deactivate after installing:
```bash
deactivate
```

---

## 🧾 2. Create a Systemd Service File

```bash
sudo nano /etc/systemd/system/examiner-ai.service
```

Paste the following configuration (update paths as needed):

```ini
[Unit]
Description=Examiner AI
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/examiner-ai
ExecStart=/home/ubuntu/examiner-ai/venv/bin/python3.11 /home/ubuntu/examiner-ai/app.py
Restart=always
Environment=PYTHONUNBUFFERED=1
StandardOutput=append:/var/log/examiner-ai.log
StandardError=append:/var/log/examiner-ai.log

[Install]
WantedBy=multi-user.target
```

> 💡 `StandardOutput` & `StandardError` will log everything to `/var/log/examiner-ai.log`.

---

## 🪄 3. Set Log File Permissions

```bash
sudo touch /var/log/examiner-ai.log
sudo chown ubuntu:ubuntu /var/log/examiner-ai.log
```

---

## 🔁 4. Enable and Start the Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable examiner-ai.service
sudo systemctl start examiner-ai.service
```

---

## 🧩 5. Verify Service Status

```bash
sudo systemctl status examiner-ai.service
```

Expected output:
```
● examiner-ai.service - Examiner AI
     Loaded: loaded (/etc/systemd/system/examiner-ai.service; enabled)
     Active: active (running)
   Main PID: 12345 (python3.11)
      Tasks: 3
```

---

## 📜 6. Check Logs

Using systemd logs:
```bash
journalctl -u examiner-ai.service -f
```

Or directly from your log file:
```bash
tail -f /var/log/examiner-ai.log
```

---

## 🔄 7. Managing the Service

| Action | Command |
|--------|----------|
| **Start service** | `sudo systemctl start examiner-ai.service` |
| **Stop service** | `sudo systemctl stop examiner-ai.service` |
| **Restart service** | `sudo systemctl restart examiner-ai.service` |
| **View logs** | `journalctl -u examiner-ai.service -n 20` |
| **Enable auto-start** | `sudo systemctl enable examiner-ai.service` |
| **Disable auto-start** | `sudo systemctl disable examiner-ai.service` |

---

## 🚀 8. Test Auto-Start on Reboot

Reboot your EC2 instance:
```bash
sudo reboot
```

After it restarts, check:
```bash
sudo systemctl status examiner-ai.service
```

It should show:
```
Active: active (running)
```

---

## 🧰 9. Common Troubleshooting

| Issue | Cause | Fix |
|--------|-------|-----|
| `ModuleNotFoundError` | System using global Python | Use venv’s `python3.11` in `ExecStart` |
| `Permission denied` | Wrong file ownership | `sudo chown -R ubuntu:ubuntu /home/ubuntu/examiner-ai` |
| No `.env` variables | Not loaded by systemd | Add `EnvironmentFile=/home/ubuntu/examiner-ai/.env` |
| Service keeps restarting | Crash on startup | Temporarily set `Restart=no` to debug |
| No logs visible | Not configured | Add `StandardOutput` and `StandardError` lines |

---

## ✅ Final Notes
- Uses **Python 3.11** explicitly.
- Runs in **virtual environment**.
- Automatically restarts on failure.
- Starts on **boot**.
- Logs stored in `/var/log/examiner-ai.log`.

---

### Example Verification Command
To confirm it’s running from the correct interpreter:
```bash
echo 'import sys; print(sys.executable)' | sudo tee /home/ubuntu/examiner-ai/app.py
sudo systemctl restart examiner-ai.service
tail -n 5 /var/log/examiner-ai.log
```
Output should include:
```
/home/ubuntu/examiner-ai/venv/bin/python3.11
```

---

🟢 **You’re done!** Ther app now runs as a reliable background service on Ubuntu EC2 using the virtual environment.