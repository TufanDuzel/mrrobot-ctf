# Mr. Robot: 1 CTF Walkthrough

## 🚩 Project Overview
This repository contains a detailed, step-by-step walkthrough of the **Mr. Robot: 1** Capture The Flag (CTF) challenge from VulnHub. The objective of this machine is to find three hidden keys while navigating through a series of vulnerabilities inspired by the *Mr. Robot* series.

The walkthrough covers the entire penetration testing lifecycle, from initial reconnaissance to full root privilege escalation.

## 🛠️ Tools & Technologies Used
* **Reconnaissance:** Nmap, Nikto, DirBuster
* **Proxy & Traffic Analysis:** Burp Suite, FoxyProxy
* **Exploitation:** Hydra, WPScan, Netcat
* **Cryptography:** Hashcat / John the Ripper (MD5 Cracking)
* **Languages:** Python (TTY Spawning), PHP (Reverse Shell)

## 📝 Methodology

### Phase 1: Reconnaissance
Identified the attack surface by scanning ports 80 (HTTP) and 443 (HTTPS). Performed directory brute-forcing and vulnerability scanning with Nikto, which revealed a **WordPress** installation and sensitive backup files like `wp-config.php`.

### Phase 2: Initial Access
Discovered a custom dictionary (`fsocity.dic`) and the first key via `robots.txt`. Used **Hydra** to perform user enumeration and credential brute-forcing against the WordPress login portal.

### Phase 3: Web Exploitation & Reverse Shell
Gained administrative access to WordPress and exploited the **Theme Editor** to inject a PHP reverse shell into the `footer.php` file. Successfully established a connection back to a Netcat listener as the `daemon` user.

### Phase 4: Horizontal Privilege Escalation
Discovered an MD5 hash for the user `robot`. After cracking the hash, I stabilized the shell using a Python PTY spawn and switched to the `robot` account to retrieve the second key.

### Phase 5: Vertical Privilege Escalation (Root)
Analyzed **SUID binaries** and found that **Nmap** was misconfigured with root permissions. Exploited Nmap's interactive mode (`!sh`) to drop into a root shell and capture the final flag in the `/root` directory.

## 🎓 Key Learnings
* Mastering the use of custom wordlists for CMS-specific attacks.
* Understanding the importance of shell stabilization for interactive commands.
* Identifying and exploiting SUID bit misconfigurations for privilege escalation.

---
**Disclaimer:** *This walkthrough is for educational purposes only. Always obtain permission before testing any system.*