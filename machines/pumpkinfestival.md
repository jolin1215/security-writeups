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
2. Scan the IP address.
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

    lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
            inet 127.0.0.1  netmask 255.0.0.0
            inet6 ::1  prefixlen 128  scopeid 0x10<host>
            loop  txqueuelen 1000  (Local Loopback)
            RX packets 8  bytes 480 (480.0 B)
            RX errors 0  dropped 0  overruns 0  frame 0
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
[...]
