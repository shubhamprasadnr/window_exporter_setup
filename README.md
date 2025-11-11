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

## **Mssql Metrics **

## 📊 MSSQL Monitoring Metrics Overview

| **Category**                     | **Metric Description**                                                                                                                | **Purpose / Insight**                                                                                           |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| **1. Networking**                | **Network Traffic & Hourly Usage**                                                                                                    | Tracks incoming and outgoing traffic to monitor network throughput and identify congestion or latency.          |
| **2. Storage**                   | **Used Space & Available Space**                                                                                                      | Monitors disk utilization and available storage capacity to prevent database outages due to full disks.         |
| **3. Lock Statistics**           | **Lock Wait Time & Lock Timeout**                                                                                                     | Indicates database contention; high lock waits may point to performance bottlenecks in queries or transactions. |
| **4. Database Log Used**         | **Transaction Log Usage**                                                                                                             | Tracks how much of the SQL transaction log file is currently used to avoid log file overflow.                   |
| **5. Database Logs**             | **Log Pool Activities & Log Bytes**                                                                                                   | Measures the rate of log generation and log buffer utilization, useful for understanding transaction volume.    |
| **6. Database Latency**          | **Fetch / DLC Latency & Peak DLC Latency**                                                                                            | Reflects how long database operations take to fetch or commit data; helps identify performance degradation.     |
| **7. Database Size Trend**       | **Database Row Size & Log Size**                                                                                                      | Displays growth trends of data and log files over time to assist in capacity planning.                          |
| **8. Performance Counters**      | **SQL Server Activity, Database Activity, Buffer Cache, Disk I/O, Memory Manager, Lock Requests, Ratios**                             | Provides deep insight into database engine performance, memory utilization, and buffer cache efficiency.        |
| **9. KPI - Memory**              | **Overall Memory Usage (Committed & Free)**                                                                                           | Monitors SQL Server memory allocation and helps identify memory pressure or leaks.                              |
| **10. KPI - CPU**                | **CPU Usage & CPU Load per Process**                                                                                                  | Evaluates CPU consumption by SQL Server and related services to identify overutilization or idle patterns.      |
| **11. SRV - General Statistics** | **Database Health, Deadlocks, Network, Storage, Logins/Logouts, Connection Reset, Lock Waits, TempDB Free Space, Active Temp Tables** | Offers a high-level operational view of server performance and stability, covering all major resource areas.    |
