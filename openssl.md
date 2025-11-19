## Windows Exporter HTTPS Setup Guide
---
### Prerequisites
- Windows Exporter installed
- Administrator access on Windows machine
---
### Install OpenSSL
- Download OpenSSL
- Visit: https://slproweb.com/products/Win32OpenSSL.html
- Download the OpenSSL installer for your system (64-bit recommended)
---
### Installation Steps
Run the OpenSSL installer
During installation:
- Select "The OpenSSL binaries (/bin) directory"
- Skip the $10 donation (optional)
- Click "Finish" to complete installation
---
### Configure Environment Variables
- Navigate to: C:\Program Files\OpenSSL-Win64\bin
- Copy this path
- open Edit  Environment Variables:
- Click on Environment variables 
- Find "Path" and double-click
- Click "New" and paste the copied path
- Click "OK" to save
---
### Verify OpenSSL Installation
cmd
```bash
"C:\Program Files\OpenSSL-Win64\bin\openssl.exe" version
```
Expected Output:
OpenSSL 3.6.0 1 Oct 2025 (Library: OpenSSL 3.6.0 1 Oct 2025)
---
## Generate SSL Certificates
Run these commands in Command Prompt:

cmd
#### 1. Generate private key
``` bah 
"C:\Program Files\OpenSSL-Win64\bin\openssl.exe" genrsa -out windows-exporter.key 2048
```
#### 2. Generate certificate request
``` bash 
"C:\Program Files\OpenSSL-Win64\bin\openssl.exe" req -new -key windows-exporter.key -out windows-exporter.csr -subj "/CN=privateip"
```
#### 3. Generate self-signed certificate valid for 30 days
``` bash 
"C:\Program Files\OpenSSL-Win64\bin\openssl.exe" x509 -req -days 30 -in windows-exporter.csr -signkey windows-exporter.key -out windows-exporter.crt
```
---
#### Verify Certificate Generation
   - Navigate to C:\Users\[YourUsername]\
  - You should find three files:
  -  windows-exporter.key (private key)
  -  windows-exporter.csr (certificate request)
  -  windows-exporter.crt (SSL certificate)
---
##### Create SSL directory
 "C:\Program Files\windows_exporter\ssl"

#### Copy certificate files to secure location
- copy windows-exporter.key to this path "C:\Program Files\windows_exporter\ssl\"
- copy windows-exporter.crt to  this path "C:\Program Files\windows_exporter\ssl\"
- Create web-config.yml File
- Navigate to C:\Program Files\windows_exporter\
- Create a new file named web-config.yml
- Important: Ensure the file extension is .yml (not .txt)
- Add the following content:
``` bash 
tls_server_config:
  cert_file: C:\Program Files\windows_exporter\ssl\windows-exporter.crt
  key_file: C:\Program Files\windows_exporter\ssl\windows-exporter.key
  ```
  ---
### Update Windows Exporter Configuration
  - Update Registry/Service Configuration
  - Use this path in your Windows Exporter service configuration:
```bash 
"C:\Program Files\windows_exporter\windows_exporter-0.31.3-amd64.exe" --collectors.enabled="cpu,logical_disk,net,os,service,system,textfile,mssql" --web.listen-address=:9182 --web.config.file="C:\Program Files\windows_exporter\web-config.yml"
```
---
###  Start Windows Exporter Service
   - Press Windows + R
   - Type services.msc and press Enter
   - Find "windows_exporter" service
   - Right-click and select "Start"
---
###  Verify HTTPS Access
``` bash 
https://localhost:9182/metrics
```
---
