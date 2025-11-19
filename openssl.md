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
<img width="1366" height="768" alt="Screenshot (45)" src="https://github.com/user-attachments/assets/e839415f-6e91-4bac-b4a8-d3bd42ffaf6c" />

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
<img width="635" height="581" alt="Screenshot 2025-11-19 164336" src="https://github.com/user-attachments/assets/5877585d-694f-4666-becc-55feb633a334" />

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
<img width="910" height="137" alt="Screenshot 2025-11-19 164624" src="https://github.com/user-attachments/assets/1e0084be-8e44-441d-bb24-ce063838a393" />
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
   - Navigate to C:\Users\[YourUsername]
  - You should find three files:
  -  windows-exporter.key (private key)
  -  windows-exporter.csr (certificate request)
  -  windows-exporter.crt (SSL certificate)
    <img width="823" height="209" alt="Screenshot 2025-11-19 164918" src="https://github.com/user-attachments/assets/52959b16-455d-442c-8246-be83119d6f41" />
    
---

#### Create SSL directory

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
## Modify your scrape yaml file 

```bash 
scrape_configs:
  - job_name: "windows-exporter"
    static_configs:
      - targets: ["172.20.0.1:9182"]
    scheme: https
    tls_config:
      insecure_skip_verify: true
 ```
      
