# Mr. Robot: 1 CTF Walkthrough

## 🚩 Project Overview
This repository contains a detailed, step-by-step walkthrough of the **Mr. [cite_start]Robot: 1** Capture The Flag (CTF) challenge from VulnHub[cite: 1, 14]. The objective of this machine is to find three hidden keys while navigating through a series of vulnerabilities inspired by the *Mr. Robot* series[cite: 16, 38, 54].

[cite_start]The walkthrough covers the entire penetration testing lifecycle, from initial reconnaissance to full root privilege escalation[cite: 55].

## 🛠️ Tools & Technologies Used
* [cite_start]**Reconnaissance:** Nmap, Nikto, DirBuster [cite: 2, 4, 6]
* **Proxy & Traffic Analysis:** Burp Suite, FoxyProxy [cite: 19]
* [cite_start]**Exploitation:** Hydra, WPScan, Netcat [cite: 21, 31]
* [cite_start]**Cryptography:** Hashcat / John the Ripper (MD5 Cracking) [cite: 41]
* **Languages:** Python (TTY Spawning), PHP (Reverse Shell) [cite: 28, 43]

## 📝 Methodology

### Phase 1: Reconnaissance
Identified the attack surface by scanning ports 80 (HTTP) and 443 (HTTPS)[cite: 2, 3]. Performed directory brute-forcing and vulnerability scanning with Nikto, which revealed a **WordPress** installation and sensitive backup files like `wp-config.php`[cite: 4, 6, 9].

### Phase 2: Initial Access
Discovered a custom dictionary (`fsocity.dic`) and the first key via `robots.txt`[cite: 16, 17]. Used **Hydra** to perform user enumeration and credential brute-forcing against the WordPress login portal[cite: 20, 23].

### Phase 3: Web Exploitation & Reverse Shell
Gained administrative access to WordPress and exploited the **Theme Editor** to inject a PHP reverse shell into the `footer.php` file[cite: 26, 27, 28]. Successfully established a connection back to a Netcat listener as the `daemon` user[cite: 30, 34].

### Phase 4: Horizontal Privilege Escalation
Discovered an MD5 hash for the user `robot`[cite: 37, 40]. After cracking the hash, I stabilized the shell using a Python PTY spawn and switched to the `robot` account to retrieve the second key[cite: 41, 43, 44].

### Phase 5: Vertical Privilege Escalation (Root)
Analyzed **SUID binaries** and found that **Nmap** was misconfigured with root permissions[cite: 46, 48]. Exploited Nmap's interactive mode (`!sh`) to drop into a root shell and capture the final flag in the `/root` directory[cite: 50, 52, 54].

## 🎓 Key Learnings
* [cite_start]Mastering the use of custom wordlists for CMS-specific attacks[cite: 18, 22].
* [cite_start]Understanding the importance of shell stabilization for interactive commands[cite: 42].
* Identifying and exploiting SUID bit misconfigurations for privilege escalation[cite: 47, 55].

---
**Disclaimer:** *This walkthrough is for educational purposes only. Always obtain permission before testing any system.*