# 🚀 Azure DevOps → Ubuntu CI/CD Deployment Master Guide

> **"Zero deployment failures from now on!"**  
> *Complete step-by-step checklist with copy-paste commands*

---

## 🎯 **What This Guide Solves**
✅ Connect Azure DevOps to your Ubuntu server  
✅ Deploy apps automatically with zero permission errors  
✅ Keep apps **ALWAYS ALIVE** with systemd  
✅ **Never** use wrong IP/hostname again  

---

## 📋 **DEPLOYMENT CHECKLIST** *(Execute in exact order)*

### **Step 1: Create Secure Deployment User** 
```bash
sudo adduser buildsvc
sudo usermod -aG sudo buildsvc
```
 Pro Tip: Always use buildsvc (or similar) - never root!

 ### **Step 2: Get Your Server's Magic IP** 
 ```
 ip addr show
```
### **Step 3: Setup App Directory***
```
sudo mkdir -p /var/www/my-app
sudo chown -R buildsvc:buildsvc /var/www/my-app
sudo chmod -R 755 /var/www/my-app
```
### **Step 4: Passwordless Sudo Magic**
```
sudo visudo
```
Add this exact line at the bottom:
```
buildsvc ALL=(ALL) NOPASSWD: /bin/systemctl restart my-app.service, /bin/systemctl status my-app.service
```
### **Step 5: Create Systemd Service (App stays alive forever)**
```
sudo vi /etc/systemd/system/my-app.service
```
Copy-paste this service file:
```
[Unit]
Description=My React App
After=network.target

[Service]
User=buildsvc
WorkingDirectory=/var/www/my-app
ExecStart=/usr/bin/serve -s dist -l 3000
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```
Activate service:
```
sudo systemctl daemon-reload
sudo systemctl enable my-app.service
```
### **Step 6: MANUAL SSH TEST**
```
ssh buildsvc@192.168.1.50  # Your IP from Step 2
sudo -u buildsvc sudo systemctl status my-app.service
```
✅ Green = Ready for Azure | ❌ Red = Fix permissions

### **Step 7: Azure DevOps Service Connection
Project Settings → Service connections → New → SSH

🟢 Host: 192.168.1.50          (Step 2 IP!)
🟢 Username: buildsvc
🟢 Password: [from Step 1]
🟢 Port: 22

Click "Verify" → "Save"

### **Step 8: Pipeline YAML**
Add to your azure-pipelines.yml:

```
- task: SSH@0
  inputs:
    sshEndpoint: 'my-server-connection'  # Step 7 name
    runOptions: 'commands'
    commands: |
      rsync -avz $(Build.ArtifactStagingDirectory)/dist/ buildsvc@192.168.1.50:/var/www/my-app/
      sudo systemctl restart my-app.service
```
###**Daily Health Check Commands**
```
# Service status
sudo systemctl status my-app.service

# Watch deployment logs LIVE
sudo journalctl -u my-app.service -f

# Test app is running
curl http://192.168.1.50:3000
```
###**SEQUENCE**
👤 User → 🌐 IP → 📁 Folder → 🔑 Sudoers → ⚙️ Service → 🧪 SSH Test → ☁️ Azure → 🚀 Pipeline

###**Quick Copy Reference**
```
IP:        ip addr show | grep inet
User:      buildsvc
Folder:    /var/www/my-app
Service:   my-app.service
Port:      3000
SSH:       ssh buildsvc@192.168.1.50
Restart:   sudo systemctl restart my-app.service
```
