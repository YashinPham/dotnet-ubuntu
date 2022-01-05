# Install .NET On Ubuntu
### Step 1: Add the Microsoft package signing.
Installing with APT can be done with a few commands. Before you install .NET, run the following commands to add the Microsoft package signing key to your list of trusted keys and add the package repository.

Open a terminal and run the following commands:
```bash
wget https://packages.microsoft.com/config/ubuntu/21.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
rm packages-microsoft-prod.deb
```
### Step 2: Install the SDK Or Runtime
- Install the .NET SDK
```bash
sudo apt-get update; \
  sudo apt-get install -y apt-transport-https && \
  sudo apt-get update && \
  sudo apt-get install -y dotnet-sdk-6.0
```
- Install the .NET Runtime
```bash
sudo apt-get update; \
  sudo apt-get install -y apt-transport-https && \
  sudo apt-get update && \
  sudo apt-get install -y aspnetcore-runtime-6.0
```

# Deploy .NET On Ubuntu Server with Nginx
### Step 1: Installing Nginx
```bash
sudo apt update
sudo apt install nginx
```
### Step 2 – Adjusting the Firewall
List the application configurations that ufw knows how to work with by typing:
```bash
sudo ufw app list
```
You should get a listing of the application profiles:
```bash
Available applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH
```
As demonstrated by the output, there are three profiles available for Nginx:
1. Nginx Full: This profile opens both port 80 (normal, unencrypted web traffic) and port 443 (TLS/SSL encrypted traffic)
2. Nginx HTTP: This profile opens only port 80 (normal, unencrypted web traffic)
3. Nginx HTTPS: This profile opens only port 443 (TLS/SSL encrypted traffic)

It is recommended that you enable the most restrictive profile that will still allow the traffic you’ve configured. Right now, we will only need to allow traffic on port 80.

You can enable this by typing:
```bash
sudo ufw allow 'Nginx HTTP'
```
You can verify the change by typing:
```bash
sudo ufw status
```
### Step 3 – Checking your Web Server
```bash
sudo systemctl status nginx
```
**You can also managing the Nginx Process**

To stop your web server, type:
```bash
sudo systemctl stop nginx
```
To start the web server when it is stopped, type:
```bash
sudo systemctl start nginx
```
To stop and then start the service again, type:
```bash
sudo systemctl restart nginx
```
If you are only making configuration changes, Nginx can often reload without dropping connections. To do this, type:
```bash
sudo systemctl reload nginx
```
By default, Nginx is configured to start automatically when the server boots. If this is not what you want, you can disable this behavior by typing:
```bash
sudo systemctl disable nginx
```
To re-enable the service to start up at boot, you can type:
```bash
sudo systemctl enable nginx
```
You have now learned basic management commands and should be ready to configure the site to host more than one domain.
### Step 4: Configure Nginx
To configure Nginx as a reverse proxy to forward HTTP requests to your ASP.NET Core app, modify **/etc/nginx/sites-available/default**. Open it in a text editor, and replace the contents with the following snippet:
```ngnix
server {
    listen 8080;
    server_name _;
    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection keep-alive;
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}```
### Step 5: Run .NET publish
Run dotnet publish from the development environment to package an app into a directory (for example, bin/Release/{TARGET FRAMEWORK MONIKER}/publish, where the placeholder {TARGET FRAMEWORK MONIKER} is the Target Framework Moniker/TFM) that can run on the server:
```bash
dotnet publish --configuration Release
```
### Step 6: Create service File
Create the service definition file:
```bash
sudo nano /etc/systemd/system/kestrel-APPLICATIONNAME.service
```
The following example is a service file for the app:
```ini
[Unit]
Description=Example .NET Web API App running on Ubuntu

[Service]
WorkingDirectory=/var/www/PUBLISH_DIRECTORY
ExecStart=/usr/bin/dotnet /var/www/MYAPP_DIRECTORY/STARUP_PROJECT_NAME.dll
Restart=always
# Restart service after 10 seconds if the dotnet service crashes:
RestartSec=10
KillSignal=SIGINT
SyslogIdentifier=dotnet-example
User=www-data
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false

[Install]
WantedBy=multi-user.target
```
Save the file and enable the service.
```bash
sudo systemctl enable kestrel-APPLICATIONNAME.service
```
Start the service.
```bash
sudo systemctl start kestrel-APPLICATIONNAME.service
```
If you stop the website application, you can stop the service using the command line.
```bash
sudo systemctl stop kestrel-APPLICATIONNAME.service
```
