1. Scan the network segment.
    ```command
    $ sudo netdiscover -i eth0
    ```
    ![image](https://hackmd.io/_uploads/r1THU7yOxe.png)
2. Scan the IP address.
    ```linux
    $ nmap -sV 192.168.163.136
    Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-05 02:34 EDT
    Nmap scan report for 192.168.163.136
    Host is up (0.00033s latency).
    Not shown: 989 filtered tcp ports (no-response), 10 filtered tcp ports (host-prohibited)
    PORT   STATE SERVICE VERSION
    80/tcp open  http    Apache httpd 2.2.15 ((CentOS) DAV/2 PHP/5.3.3)
    MAC Address: 08:00:27:A5:A6:76 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 11.41 seconds
    ```
3. Directory traversal.
    ```linux
    $ dirb http://192.168.163.136 
    ```
    ![image](https://hackmd.io/_uploads/S1lWsXy_ex.png)
Visit `robots.txt`.
![image](https://hackmd.io/_uploads/HJZmom1dll.png)
However, all three paths return `301`, and adding `/` at the end redirects to another page that says we went to the wrong place.
![image](https://hackmd.io/_uploads/BkkTimkuxl.png)
Visit `http://192.168.163.136/images/` and view the directory listing.
![image](https://hackmd.io/_uploads/ry4uTQJulg.png)
4. Use `nikto` for a more thorough website vulnerability scan.
    ```linux
    $ nikto -h http://192.168.163.136
    ```
    ![image](https://hackmd.io/_uploads/HyhTTQJOgg.png)
5. This really is a “Happy New Year” vibe — this directory name is too on the nose.
    ```
    http://192.168.163.136/fristi/
    ```
    ![image](https://hackmd.io/_uploads/ByyEhVJ_ll.png)
Find a base64 string in the page source.
![image](https://hackmd.io/_uploads/rJkWoVg_xg.png)
Since the string starts with `iVBORw0KGgo...`, it clearly represents a PNG image encoded in base64, so we decode it with a [base64 to image](https://base64.guru/converter/decode/image) converter and reveal an image.
![image](https://hackmd.io/_uploads/rkxqsNgOlg.png)
The source also contains a hidden hint from the author.
![image](https://hackmd.io/_uploads/HJI1GHxuxg.png)
We then try logging in with the credentials `eezeepz/keKkeKKeKKeKkEkkEk`.
![image](https://hackmd.io/_uploads/H1g2ZBeuxx.png)
6. After entering the upload page, upload a normal image first.
![image](https://hackmd.io/_uploads/SynM1dedeg.png)
[...]
