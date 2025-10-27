# Splunk Configuration File Permissions & Integrity Report

**Author:** Afsheen Golanbar  
**Date:** May 13, 2025  
**Organization:** StackFull Software — Cybersecurity Division  
**Report Type:** Security Vulnerability Analysis  

---

## 🧭 Overview
This project documents a real-world Splunk configuration vulnerability that affected file permissions and integrity controls. The issue was identified during log analysis troubleshooting, revealing that a critical configuration file (`config.conf`) had insecure permissions that allowed any user to modify it.  
This report follows a structured cybersecurity analysis approach aligned with the **CIA Triad (Confidentiality, Integrity, Availability)** and provides details on discovery, remediation, and long-term mitigation.

---

## 🧩 Executive Summary
While troubleshooting Splunk log search failures, a misconfigured file was discovered:
```
/opt/splunk/etc/system/local/config.conf
```
Permissions were set to **world-writable and executable**, meaning any system user could modify the file. This created serious risks:
- **Confidentiality:** Sensitive configurations visible to all users  
- **Integrity:** Potential for unauthorized edits or file corruption  
- **Availability:** Risk of Splunk malfunction or data loss

The issue was resolved by restricting file access to administrators, verifying integrity with MD5 hashing, and implementing daily monitoring via cron jobs.

---

## ⚙️ Root Cause
The file `config.conf` was found with overly permissive access rights:
```
-rwxrwxrwx 1 root root ... config.conf
```
This occurred due to incorrect default permissions during installation or manual testing.  
Additionally, duplicate copies existed — one legitimate (active) and one test version in the user’s Documents directory.

---

## 🔍 CIA Triad Impact

| CIA Element | Risk Description |
|--------------|------------------|
| **Confidentiality** | All users could read Splunk’s configuration details, exposing sensitive settings. |
| **Integrity** | Unauthorized users could edit or corrupt operational files. |
| **Availability** | Incorrect edits could disable Splunk’s indexing or logging capabilities. |

---

## 🛠️ Remediation Steps

1. **Restricted Permissions**
   ```bash
   chmod 600 /opt/splunk/etc/system/local/config.conf
   ```
   Result:
   ```
   -rw------- 1 root root ... config.conf
   ```

2. **Integrity Verification**
   ```bash
   md5sum /opt/splunk/etc/system/local/config.conf > conf.doc.txt
   # After edits
   md5sum /opt/splunk/etc/system/local/config.conf > confedit.doc.txt
   diff conf.doc.txt confedit.doc.txt
   ```

3. **Backup Creation**
   ```bash
   cp /opt/splunk/etc/system/local/config.conf ~/config.edit.conf
   ```

4. **File Review**
   The configuration file was edited using `nano` to ensure correctness and stored securely with limited privileges.

---

## 🧠 Monitoring & Recommendations
To prevent future issues:

1. **Access Control**  
   Restrict modification rights to a trusted admin group only.

2. **File Integrity Monitoring**  
   Add a daily cron job to check MD5 hash integrity:
   ```bash
   md5sum /opt/splunk/etc/system/local/config.conf | diff - /path/to/baseline.md5 || echo "ALERT: Splunk config changed!" | mail -s "Splunk Integrity Alert" admin@domain.com
   ```

3. **Audit Logging**  
   Enable Linux audit logs to track access attempts:
   ```bash
   auditctl -w /opt/splunk/etc/system/local/config.conf -p war -k splunk-config-monitor
   ausearch -k splunk-config-monitor
   ```

4. **Periodic Review**  
   Schedule quarterly audits to review configuration permissions and hash baselines.

---

## 🧾 Conclusion
This investigation exposed a critical configuration vulnerability that compromised the security posture of Splunk’s operational environment.  
By enforcing least privilege permissions, establishing integrity verification through hashing, and implementing monitoring controls, the risk was fully mitigated.  
Ongoing monitoring and administrative controls are essential to maintain the long-term security and reliability of Splunk systems.

---

## 📁 File Summary
| File | Purpose |
|------|----------|
| `/opt/splunk/etc/system/local/config.conf` | Active Splunk configuration file |
| `~/config.edit.conf` | Secure backup |
| `conf.doc.txt` | Pre-fix MD5 hash record |
| `confedit.doc.txt` | Post-fix MD5 hash record |

---

## 🧩 Keywords
Splunk, Security Report, Configuration File, Permissions, Vulnerability, CIA Triad, Integrity Monitoring, MD5, File Access Control, Linux Security

---

📘 *Prepared as part of the Fullstack Academy Cybersecurity Bootcamp – StackFull Software Security Division, May 2025.*
