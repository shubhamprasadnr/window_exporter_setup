# window_exporter_setup

# Monitoring MSSQL on Windows Server  
### A Guide to Maximizing Database Performance  


---

## 🎯 **Objective**

The objective of this POC is to **monitor MSSQL performance metrics** on a Windows Server using **Prometheus** and **Grafana**.  
It includes complete setup instructions for:
- Installing MSSQL Server and SSMS  
- Configuring SQL Server Agent  
- Installing and running Windows Exporter  
- Setting up Prometheus and Grafana on an Ubuntu server  
- Visualizing MSSQL metrics in Grafana dashboards  

---

##  **Requirements**

Before starting, ensure the following prerequisites are met:

| Component | Description |
|------------|-------------|
|  **Windows Server** | Host machine where MSSQL Server is installed |
|  **MSSQL Server 2022 Developer Edition** | Database engine to monitor |
|  **SQL Server Management Studio (SSMS)** | For database administration |
|  **Prometheus** | Metrics collection tool (installed on Ubuntu) |
|  **Grafana** | Visualization tool (installed on Ubuntu) |
|  **Windows Exporter** | Exposes system and MSSQL metrics |
|  **Open Ports** | `9182` (Windows Exporter), `9090` (Prometheus), `3000` (Grafana) |

---

##  **Installation and Setup of MSSQL**

### **Steps:**

1. Go to the [Microsoft SQL Server official download page](https://www.microsoft.com/en-us/sql-server/sql-server-downloads)  
2. Under **SQL Server 2022 Developer**, click **Download now**  

During installation:

- Choose **Basic** installation type  
- Accept license terms  
- Click **Install**  
- Wait for **10–20 minutes** (installation runs silently)  
- Keep the **default instance name** or rename if desired  
- Once done, click **Connect**  

---

##  **Install SQL Server Management Studio (SSMS)**

### **Steps:**

1. Download SSMS from the [official Microsoft website](https://learn.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms)  
2. Download **SSMS 21** and install it normally  

---

##  **Enable SQL Server Agent in SSMS**

To allow external collectors (e.g., Prometheus) to query the SQL Server engine, you need to **enable the SQL Server Agent**.

### **Steps:**

1. Open **SQL Server Management Studio (SSMS)**  
2. In the **Object Explorer**, locate **SQL Server Agent**  **Right click** and Start the Agent. 
<img width="1366" height="768" alt="Screenshot (12)" src="https://github.com/user-attachments/assets/db511a01-c630-4294-981f-b58821a813ee" />


###  Enabling Automatic Start for SQL Server Agent

####  Using Windows Services Manager

Follow the steps below to configure the **SQL Server Agent** service to start automatically when Windows boots:

1. Press **`Windows + R`** to open the Run dialog.  
2. Type **`services.msc`** and press **Enter** to open the **Services** window.  
3. Locate the service named **SQL Server Agent (<InstanceName>)**, where `<InstanceName>` is the name of your SQL Server instance.  
4. Double-click the **SQL Server Agent** service to open its **Properties** window.  
5. In the **Startup type** dropdown menu, select **Automatic**.  
   - You can also choose **Automatic (Delayed Start)** if you want other services to start first.  
   <img width="1366" height="768" alt="Screenshot (10)" src="https://github.com/user-attachments/assets/c13b2527-8396-4bb6-b7e4-e4c696439df2" />

6. Click **Apply**, and then **OK** to save the changes.To verify it check startup type changes to automatic. 
<img width="1366" height="768" alt="Screenshot (11)" src="https://github.com/user-attachments/assets/2e8d58c9-d36e-46f5-b162-4cc57d99d54e" />

---

##  **Install Prometheus Windows Exporter**

Prometheus Windows Exporter helps expose system and MSSQL metrics for monitoring.

### **Steps:**

1. **Download Windows Exporter**  
   Visit the [official releases page](https://github.com/prometheus-community/windows_exporter/releases) to download the latest version of **Windows Exporter**.

2. **Download the Binary File**  
   Select and download the binary file:

   
3. **Open Command Prompt as Administrator**  
- Press **Start → Search “cmd” → Right-click → Run as Administrator**.

4. **Run the Exporter Manually**  
Execute the following command to start the Windows Exporter and fetch SQL Server statistics:

```bash
"C:\Program Files\windows_exporter\windows_exporter-0.31.3-amd64.exe" --collectors.enabled="cpu,logical_disk,net,os,service,system,textfile,mssql"
```

5.**Verify the Metrics Endpoint**
 - Press Open your browser and navigate to:
[http://PublicIp:9182/metrics](http://PublicIp:9182/metrics)

---

##  Run Windows Exporter Automatically After System Restart
 To ensure the Windows Exporter service starts automatically after a system reboot, follow these steps:

**Open Registry Editor**

- Press Press Start → Search “regedit” → Open Registry Editor

2. Navigate to the Following Path:
**HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\windows_exporter**
Modify the ImagePath Value,Double-click on ImagePath.

3. Replace the existing value with the following (adjust if your installation path differs):
```bash
"C:\Program Files\windows_exporter\windows_exporter-0.31.3-amd64.exe" --collectors.enabled="cpu,logical_disk,net,os,service,system,textfile,mssql"
```
<img width="1366" height="768" alt="Screenshot (13)" src="https://github.com/user-attachments/assets/8726918c-d1fb-48eb-9659-baa839d93c0e" />

---

##  Prometheus and Grafana Setup on Ubuntu

This Proof of Concept (POC) demonstrates the installation and setup of **Prometheus** and **Grafana** for system monitoring and visualization on an Ubuntu machine.

---

###  Step 1: Update Your System

```bash
sudo apt update -y && sudo apt upgrade -y
```

### Step 2: **Install Prometheus**

2.1 **Download Prometheus**
```bash
wget https://github.com/prometheus/prometheus/releases/download/v3.5.0/prometheus-3.5.0.linux-amd64.tar.gz
```
2.2 **Extract and Move Binaries**
```bash
tar xvf prometheus-3.5.0.linux-amd64.tar.gz
mv prometheus-3.5.0.linux-amd64 /home/ubuntu/prometheus
```

### Step 3: **Configure Prometheus**

- Edit the Prometheus configuration file:
```bash
sudo nano prometheus/prometheus.yml
```
- Add the following configuration:

```bash
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'Mssql_metrics'
    static_configs:
      - targets: ['<privateip>:9090']

```
- Command to run Prometheus 
```bash
 ./prometheus --config.file=prometheus.yml
 ```

  - Press Open your browser and navigate to:
 [http://PublicIp:9090/targets](http://PublicIp:9090/targets)

<img width="1366" height="768" alt="Screenshot (14)" src="https://github.com/user-attachments/assets/375314ac-7ba4-484e-83c6-b425f92b7654" />

<img width="1366" height="768" alt="Screenshot (15)" src="https://github.com/user-attachments/assets/13b6203e-9416-4ded-9185-838d22e5a3f5" />


## Install Grafana

### 1. **Install Grafana from Official Repository**
``` bash
sudo apt install -y apt-transport-https software-properties-common wget
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
sudo apt update
sudo apt install grafana -y
```

### 2. **Start and Enable Grafana**
```bash
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
sudo systemctl status grafana-server
```

✅ Access Grafana:
👉 http://<publicip>:3000

**Default Credentials:
Username: admin
Password: admin**

### 3. **Add Prometheus as a Data Source in Grafana**

   - Go to Grafana → Connections → Data Sources → Add data source

   - Choose Prometheus

   - Set URL: http://publicip:9090

   - Click Save & Test

### 4. **Import a Dashboard**

   - Press You can import a prebuilt Grafana dashboard for Prometheus (e.g., Dashboard ID 15024):

   - Press Go to Dashboard → Import

   - Press Enter the ID (e.g., 1860)

   - Press Click Load

   - Press Select Prometheus as the data source

   - Press Click Import

✅ You’ll now see CPU, memory, and network and Mssql metrics visualized in Grafana.

---

