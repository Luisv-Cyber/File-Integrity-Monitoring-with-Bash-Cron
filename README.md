# 🔍 File Integrity Monitoring with Bash + Cron

### 🧠 Summary
This project demonstrates a simple but effective **file integrity monitoring system** using native Linux tools.  
By combining **Bash scripting**, **SHA-256 hashing**, and **cron job automation**, I simulated how a SOC or system administrator can detect unauthorized file changes in real time.

---

## ⚙️ Tools & Technologies
| Category | Tools Used |
|-----------|-------------|
| **Operating System** | Ubuntu (Linux) |
| **Scripting Language** | Bash |
| **Hashing Utility** | sha256sum |
| **Automation** | cron |
| **Editor** | nano |

---

## 🎯 Objective
Simulate a **real-world file monitoring task** by:
- Detecting if a file is modified without authorization  
- Using **SHA-256 hashes** to verify file integrity  
- Automating the process via a **cron job**  
- Logging alerts when tampering is detected  

---

## 🧱 Implementation Steps

### 1️⃣ Create and Hash the Target File
1. Create a file named `important.txt` and add a simple message:
   ```bash
   echo "This is a critical configuration file." > important.txt
