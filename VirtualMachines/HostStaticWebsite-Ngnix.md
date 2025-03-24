## ON a new Azure VM from scratch, assuming you already have the VM and domain (example.centralindia.cloudapp.azure.com) ready.

FOR THIS TUTORIAL WE ARE DEPLOYING THE FLASK APP [Link](https://github.com/vedantterse/AIO-PDF-tools-web)

### 🚀 Full Deployment Guide for AIO-PDF on Azure VM

1️⃣ SSH into the VM
On your local machine, run:

```bash
ssh your-user@your-vm-ip

# Replace your-user with your VM username and your-vm-ip with the VM's public IP.
```
2️⃣ Update & Install Required Packages
Run these commands to update and install necessary dependencies:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y python3 python3-venv python3-pip git nginx curl
```
3️⃣ Clone the GitHub Repository
Navigate to the desired directory and clone your project:
```bash
cd /var/www/
sudo git clone https://github.com/your-repo/AIO-PDF-tools-web.git
sudo chown -R $USER:$USER AIO-PDF-tools-web
cd AIO-PDF-tools-web
```
4️⃣ Create and Activate a Virtual Environment
Inside the project directory:

```bash
python3 -m venv venv
source venv/bin/activate
```
5️⃣ Install Project Dependencies
```bash

pip install --upgrade pip
pip install -r requirements.txt
```
6️⃣ Run Initial Gunicorn Test
To check if the app runs with Gunicorn:

```bash
gunicorn --bind 0.0.0.0:8000 app:app
# If you see output like Running on http://0.0.0.0:8000, your app is working. Press CTRL+C to stop.
```
7️⃣ Set Up Gunicorn as a Systemd Service
Create a Gunicorn service file:

```bash

sudo nano /etc/systemd/system/gunicorn.service
```
Add the following content:

```ini
[Unit]
Description=Gunicorn instance for AIO-PDF
After=network.target

[Service]
User=your-user
Group=www-data
WorkingDirectory=/var/www/AIO-PDF-tools-web
Environment="PATH=/var/www/AIO-PDF-tools-web/venv/bin"
ExecStart=/var/www/AIO-PDF-tools-web/venv/bin/gunicorn --workers 3 --bind unix:/var/www/AIO-PDF-tools-web/app.sock app:app

[Install]
WantedBy=multi-user.target
```
Save and exit (CTRL+X, then Y, then ENTER).
Reload systemd and enable the service:

```bash
sudo systemctl daemon-reload
sudo systemctl start gunicorn
sudo systemctl enable gunicorn
```
Check the status:

```bash
sudo systemctl status gunicorn
```
You should see Active: running.

8️⃣ Set Up Nginx
Create an Nginx configuration file:

```bash
sudo nano /etc/nginx/sites-available/AIO-PDF
```

Add the following content:

```bash
server {
    listen 80;
    server_name example.centralindia.cloudapp.azure.com;

    location / {
        proxy_pass http://unix:/var/www/AIO-PDF-tools-web/app.sock;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
Save and exit.

Create a symlink to enable this configuration:

```bash
sudo ln -s /etc/nginx/sites-available/AIO-PDF /etc/nginx/sites-enabled/
```
Test Nginx for errors:

```bash
sudo nginx -t
```
Restart Nginx:

```bash
sudo systemctl restart nginx
```
9️⃣ Firewall Configuration
Ensure your firewall allows necessary ports:

```bash
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
```
🔟 Obtain SSL Certificate with Certbot
Install Certbot:

```bash
sudo apt install certbot python3-certbot-nginx -y
```
Run Certbot:

```bash
sudo certbot --nginx -d example.centralindia.cloudapp.azure.com
```
Follow the prompts and choose option to redirect HTTP to HTTPS.

Test renewal:

```bash
sudo certbot renew --dry-run
```
Restart Nginx to apply changes:

```bash
sudo systemctl restart nginx
```
1️⃣1️⃣ Final Checks
Open your browser and go to:
👉 https://example.centralindia.cloudapp.azure.com
Test with curl:
```bash
curl -I https://example.centralindia.cloudapp.azure.com
```
You should see HTTP/1.1 200 OK with SSL headers.
