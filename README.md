# 🔍 File Integrity Monitoring with Bash + Cron — SOC Lab Project

**Project Type:** Blue Team Lab | Host-Based Monitoring | Scripting Automation
**Core Skills:** Bash Scripting • SHA-256 Hashing • Cron Automation • Log-Based Alerting • Linux Fundamentals

![Bash](https://img.shields.io/badge/Bash-4EAA25?style=flat&logo=gnubash&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=flat&logo=linux&logoColor=black)
![Cron](https://img.shields.io/badge/Automation-Cron-blue?style=flat)
![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat)

---

## 🧭 Summary

This project demonstrates a simple but effective **file integrity monitoring (FIM) system** built entirely with native Linux tools. By combining **Bash scripting**, **SHA-256 hashing**, and **cron automation**, it simulates how a SOC analyst or system administrator detects unauthorized file changes — without relying on any third-party software.

---

## 🧰 Tools & Technologies

| Category | Tools Used |
|---|---|
| **Operating System** | Ubuntu (Linux) |
| **Scripting Language** | Bash |
| **Hashing Utility** | `sha256sum` |
| **Automation** | `cron` |
| **Editor** | `nano` |

---

## 🎯 Objective

Simulate a real-world file monitoring workflow by:

- Establishing a baseline SHA-256 hash for a critical file
- Automatically re-checking that hash on a schedule via cron
- Detecting and logging any unauthorized modification
- Producing a simple audit trail of integrity checks

---

## 📁 Repository Structure

```
.
├── Integrity check    # Monitoring script (bash, no extension)
└── README.md
```

> The baseline hash and log file are *created by the script itself* on first run — they aren't checked into the repo.

---

## ⚙️ How It Works

The script (`Integrity check`) is self-initializing:

1. **First run** — if no baseline hash file exists yet, it creates one from the target file and logs an `INIT` entry, then exits.
2. **Every run after** — it recomputes the target file's SHA-256 hash and compares it against the stored baseline, logging `OK` if they match or `WARNING` if they don't.
3. **Missing target file** — logs an `ERROR` and exits with a non-zero status rather than failing silently.

**Hardening touches worth calling out:**
- `set -Eeuo pipefail` + a strict `IFS` — the script exits immediately on any error, unset variable, or failed pipe stage instead of silently continuing.
- An explicit `PATH` and fully-qualified binary calls (`/usr/bin/sha256sum`) — mitigates `$PATH`-hijacking, where a malicious binary earlier in a user's `$PATH` could be executed instead of the real one.

---

## 🧱 Implementation Steps

### 1️⃣ Review the Script

```bash
#!/usr/bin/env bash
# File Integrity Monitoring Script
# Purpose: Compare a file's current SHA-256 hash to a saved baseline and log the result.

set -Eeuo pipefail
IFS=$'\n\t'
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# === Config (absolute paths recommended) ===
TARGET_FILE="/home/user/important.txt"
HASH_FILE="/home/user/baseline.hash"
LOG_FILE="/home/user/integrity_log.txt"

timestamp() { date "+%Y-%m-%d %H:%M:%S %Z"; }
log() { echo "$(timestamp): $1" >> "$LOG_FILE"; }

# --- Safety checks ---
if [[ ! -f "$TARGET_FILE" ]]; then
    log "ERROR - Target file not found: $TARGET_FILE"
    exit 1
fi

# If baseline doesn't exist yet, create it once
if [[ ! -f "$HASH_FILE" ]]; then
    /usr/bin/sha256sum "$TARGET_FILE" > "$HASH_FILE"
    log "INIT - Baseline created at $HASH_FILE"
    exit 0
fi

# Calculate current vs original hashes (1st column of sha256sum output)
CURRENT_HASH=$(/usr/bin/sha256sum "$TARGET_FILE" | awk '{print $1}')
ORIGINAL_HASH=$(awk '{print $1}' "$HASH_FILE")

if [[ "$CURRENT_HASH" == "$ORIGINAL_HASH" ]]; then
    log "OK - No change detected."
else
    log "WARNING - $TARGET_FILE has been modified!"
fi
```

> ⚠️ Update `TARGET_FILE`, `HASH_FILE`, and `LOG_FILE` to real paths on your system — `/home/user/...` is a placeholder as written.

---

### 2️⃣ Make It Executable and Create the Baseline

```bash
chmod +x "Integrity check"
./"Integrity check"
```

This creates `baseline.hash` and logs an `INIT` entry to `integrity_log.txt`.

> Consider renaming the script to `integrity_check.sh` for portability — the current filename has a space in it, so every reference to it must be quoted (as done above).

---

### 3️⃣ Automate with Cron

```bash
crontab -e
```

Add a line to run the check on a schedule, for example every 5 minutes:

```
*/5 * * * * /full/path/to/"Integrity check"
```

> **TODO:** This cron entry isn't committed to the repo yet — add your actual crontab line once finalized, and consider checking in a `crontab.txt` so a reader doesn't have to guess the schedule used.

---

### 4️⃣ Simulate Tampering

```bash
echo "unauthorized change" >> important.txt
```

On the next run (scheduled or manual), the script detects the mismatch and logs a `WARNING`.

---

## 📜 Sample Log Output

> **TODO:** No `integrity_log.txt` is committed to the repo yet. Paste real log lines here once you've run it for real — it's the single most convincing piece of evidence in a project like this. Based on the script's `log()` format, entries will look like:

```
2026-08-26 14:05:01 UTC: INIT - Baseline created at /home/user/baseline.hash
2026-08-26 14:10:01 UTC: OK - No change detected.
2026-08-26 14:15:01 UTC: WARNING - /home/user/important.txt has been modified!
```

---

## 🧠 What This Demonstrates

- Practical understanding of file integrity monitoring (FIM) concepts used in SOC and blue-team workflows
- Defensive bash scripting: strict error handling (`set -Eeuo pipefail`), input/state validation, and `$PATH`-hijack mitigation via fully-qualified binary paths
- Experience automating recurring security checks with cron
- Basic log-based alerting, a foundational concept behind SIEM tooling

---

## 🚀 Possible Improvements

- Monitor multiple files/directories instead of a single file
- Email or Slack notification on `WARNING`, in addition to the log entry
- Store baseline hashes for a whole directory tree using `find` + `sha256sum`
- Integrate with `syslog` for centralized logging
- Rename the script file to remove the space and add a `.sh` extension

---

## 👤 Author

**Luis Vega**
[GitHub](https://github.com/Luisv-Cyber)

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.
