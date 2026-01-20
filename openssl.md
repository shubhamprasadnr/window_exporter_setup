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
<img width="1113" height="93" alt="Screenshot 2025-11-19 170330" src="https://github.com/user-attachments/assets/429fe73c-b4c3-478f-bae5-94fb4b468f95" />
---
## Generate SSL Certificates
Run these commands in Command Prompt:
``` bash 
"C:\Program Files\OpenSSL-Win64\bin\openssl.exe" req -x509 -newkey rsa:2048 -nodes -keyout windows-exporter.key -out windows-exporter.crt -`<days 30>` -subj "/C=IN/ST=`<Maharashtra>`/L=`<Pune>`/O=`<Your Organization>`/CN=`<name>`" -addext "subjectAltName=IP:172.30.176.1" -addext "extendedKeyUsage=serverAuth"
```
---
#### Verify Certificate Generation
   - Navigate to C:\Users\[YourUsername]
  - You should find three files:
  -  windows-exporter.key (private key)
  -  windows-exporter.crt (SSL certificate)
    <img width="700" height="160" alt="Screenshot 2026-01-20 111753" src="https://github.com/user-attachments/assets/cb096b87-060c-4d7a-8837-40f64dc22754" />

    
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
"C:\Program Files\windows_exporter\windows_exporter-0.31.3-amd64.exe" --collectors.enabled="cpu,logical_disk,net,os,service,system,textfile,mssql" --web.listen-address=:9182 --web.config.file="C:\Program Files\windows_exporter\ssl\web-config.yml"
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

### Create a file windows-exporter.crt in wsl ubuntu  

     notepad windows-exporter.crt 

     copy the certificate key to this file windows-exporter.crt 
    
<img src="https://github.com/user-attachments/assets/a804592b-cd22-40de-b424-499585c027e8" alt="Screenshot 2026-01-20 110956" width="613" height="431" />

---
### Mounting SSL Certificate in vmagent
 Used in Production Environment

- Mounts your local certificate file into the vmagent container

```bash
 volumes:
      - ./scrape.yml:/etc/vmagent/scrape.yml:ro
      - /<path>/windows-exporter.crt:/etc/vmagent/ssl/windows-exporter.crt:ro
```
---
## Modify your scrape yaml file 

```bash 
scrape_configs:
  - job_name: "DESKTOP-6L786HE"
    scrape_interval: 15s
    scheme: https
    tls_config:
      ca_file: /etc/vmagent/ssl/windows-exporter.crt
    static_configs:
      - targets:
          - "172.30.176.1:9182"
 ```
 ---
      
