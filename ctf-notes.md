# Phase 1: Reconnaissance & Advanced Enumeration
Objective
The goal of this phase was to identify the target's attack surface, discover open services, and perform deep web enumeration to find hidden entry points.

1. Service Discovery (Nmap)
The initial scan revealed two primary open ports:

Port 80 (HTTP): Standard web service.

Port 443 (HTTPS): Secure web service.

Manual Inspection: A manual review of the web interface and "View Page Source" was performed, but no immediate visual clues or hardcoded credentials were found in the frontend code.

2. Directory Brute Forcing (DirBuster)
To uncover hidden directories, a brute force attack was conducted. Several paths returned a 200 OK status code, indicating that the server hosts content not directly linked from the homepage. This confirmed the necessity of automated discovery tools.

3. Vulnerability Scanning (Nikto Analysis)
A comprehensive scan using Nikto provided critical insights into the server's configuration and software stack:

CMS Identification: Multiple entries confirmed a WordPress installation (e.g., /wp-login.php, /wp-admin/, /blog/).

Server Misconfigurations: * Apache mod_negotiation: Enabled with MultiViews, allowing for easier brute-forcing of file names.

Missing Headers: The X-Content-Type-Options header is missing, which is a common security oversight.

Sensitive Files & Information Leakage:

/#wp-config.php#: A temporary or backup version of the core configuration file was detected. This is a critical finding as it often contains database credentials in plaintext.

/license.txt & /readme: These files often reveal specific software versions, aiding in the search for version-specific exploits.

wp-links-opml.php: Another script that leaks version information.

Conclusion of Phase 1
The reconnaissance phase successfully identified the target as a WordPress-based system with several information disclosure vulnerabilities. The discovery of potential backup configuration files and multiple login portals sets the stage for the next phase: Initial Access and Credential Exploitation.



# Phase 2: Initial Access & Web Exploitation
Objective
The primary focus of this phase was to exploit the information gathered during enumeration to gain initial access to the system. This involved discovering sensitive files, performing user enumeration, and launching a brute-force attack against the CMS.

1. Discovering Sensitive Files (The Robots.txt Lead)
By analyzing the http://[Target_IP]/robots.txt file, two critical resources were identified:

key-1-of-3.txt: Successfully retrieved the first flag of the CTF.

fsocity.dic: A custom dictionary file containing a large list of potential usernames and passwords.

Data Optimization: The dictionary contained numerous duplicates. I used Linux CLI tools to clean the wordlist for efficiency:

sort -u fsocity.dic -o fsocity_cleaned.dic

2. Proxy Configuration (Burp Suite & FoxyProxy)
To analyze and manipulate the authentication requests, I configured a proxy environment:

FoxyProxy: Used to toggle browser traffic redirection to the local proxy.

Burp Suite: Set as an Interception Proxy on 127.0.0.1:8080.

Purpose: By intercepting the login request to /wp-login.php, I identified the specific POST parameters (log for username and pwd for password) required for automated attacks.

3. User Enumeration (Hydra)
With the custom wordlist and the request structure identified, I performed User Enumeration to find a valid WordPress account.

Tool: Hydra

Mechanism: Exploiting the application's verbose error messages.

Command Logic:
hydra -L fsocity_cleaned.dic -p test [Target_IP] http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Invalid username"

Logic: The F=Invalid username flag tells Hydra that any response containing this string is a failed attempt. The absence of this string indicates a valid username.

4. Credential Brute Forcing
After successfully identifying a valid username (e.g., Elliot), the attack shifted to password cracking:

Attack Profile: Single username (-l) vs. Cleaned wordlist (-P).

Success: By matching the valid user with the correct entry from the fsocity dictionary, I gained administrative access to the WordPress dashboard.



# Phase 3: Exploitation & Initial Access (Reverse Shell)
Objective
The objective of this phase was to transition from web-based administrative access to a direct command-line interface (CLI) on the target's operating system. This is achieved by injecting a malicious payload into the CMS theme files.

1. Gaining Entry (Administrative Access)
Using the credentials discovered in the previous phase (Elliot : ER28-0652), I successfully logged into the WordPress Dashboard.

Target URL: http://[Target_IP]/wp-login.php

Access Level: Administrative privileges within the CMS.

2. Weaponizing WordPress (Theme Editor Exploitation)
To execute arbitrary code on the server, I utilized the built-in Theme Editor feature, which allows administrators to modify the PHP source code of the website's templates.

Navigation: Appearance -> Theme Editor.

Target File: footer.php.

Action: I replaced the entire content of the footer.php file with a PHP Reverse Shell script (sourced from common cheat sheets like PentestMonkey).

Configuration: I modified the payload to point back to my Kali Linux machine's IP (LHOST) and a specific listening port (LPORT).

3. Catching the Connection (Netcat Listener)
Before triggering the payload, I set up a listener on my local machine to catch the incoming connection from the target server.

Tool: Netcat (nc)

Command: nc -lvnp 4444

-l: Listen mode.

-v: Verbose output.

-n: Do not perform DNS lookups.

-p: Specifies the port number.

4. Execution & Shell Stabilization
Once the listener was active, I triggered the shell by navigating to the modified file's URL or simply refreshing the homepage (since footer.php is included in almost every page load).

Result: The server executed the PHP code and initiated a connection back to my machine.

Status: Successfully obtained a shell as the www-data user.

Post-Exploitation Tip: To gain a more functional and interactive terminal, I used the following Python command to spawn a TTY shell:

python -c 'import pty; pty.spawn("/bin/bash")'



# Phase 4: Horizontal Privilege Escalation & TTY Shell
Objective
The goal of this phase was to elevate privileges from a low-level service account (daemon) to a more privileged user (robot) by discovering credentials and stabilizing the shell.

1. Initial System Enumeration
After gaining access to the system, I performed basic enumeration to identify the users on the machine.

Findings: The /home directory contained a folder for a user named robot.

Sensitive Files Found:

key-2-of-3.txt: The second flag (initially inaccessible due to permission restrictions).

password.raw-md5: A file containing a username and an MD5 hashed password.

Credential Leak: robot:c3fcd3d76192e4007dfb496cca67e13b

2. Hash Cracking (MD5)
The discovered string was identified as an MD5 hash. Using online databases or local tools (like John the Ripper/Hashcat), the hash was decrypted:

Decrypted Password: abcdefghijklmnopqrstuvwxyz

3. Shell Stabilization (TTY Spawning)
I attempted to switch users using su robot, but the initial shell was not interactive ("su: must be run from a terminal"). To overcome this, I used a Python-based PTY spawn to create a fully functional terminal environment.

Technique: python -c 'import pty; pty.spawn("/bin/bash")'

Mechanism: This command utilizes Python's built-in pty module to start a pseudo-terminal, allowing the system to handle interactive commands and password prompts correctly.

4. User Pivoting
With a stable TTY shell, I successfully switched to the robot user:

Command: su robot

Result: Successfully escalated privileges from daemon to robot. I can now access the second flag (key-2-of-3.txt).



# Phase 5: Vertical Privilege Escalation (Root Access)
Objective
The final objective was to escalate privileges from the robot user to the root user to gain full control over the target system and retrieve the final flag.

1. Enumerating SUID Binaries
I searched for files with the SUID (Set User ID) bit enabled. These files execute with the privileges of the file owner (in this case, root) rather than the user running them.

Command: find / -perm -u=s -type f 2>/dev/null

Findings: The scan identified /usr/local/bin/nmap as having the SUID bit set. This is a well-known misconfiguration in older versions of Nmap (specifically versions 2.02 to 5.21).

2. Exploiting Nmap (Interactive Mode)
Older versions of Nmap include an "interactive mode" which allows users to execute shell commands directly from within the Nmap console. Because the binary has the SUID bit, any shell spawned from it inherits root privileges.

Execution:

Launched interactive mode: nmap --interactive

Escaped to a shell: !sh

Verification: Running whoami confirmed that I had successfully elevated my privileges to root.

3. Retrieving the Final Flag
With root access, I navigated to the root user's home directory and secured the final key.

Path: /root/key-3-of-3.txt

Conclusion
The CTF was completed by following a structured methodology: Reconnaissance -> Web Exploitation -> User Pivoting -> Privilege Escalation. This experience highlighted the danger of misconfigured SUID bits and the importance of keeping system binaries updated.