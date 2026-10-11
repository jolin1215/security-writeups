1. `sudo netdiscover -i eth0` 
![image](https://hackmd.io/_uploads/r14CwEiLex.png)
    - 偵測在同一個網段內有哪些裝置在線
2. `nmap -sV 192.168.101.131`
![image](https://hackmd.io/_uploads/B1ZMONjUex.png)
3. `gobuster dir -u http://192.168.101.131 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php , html , txt`
![image](https://hackmd.io/_uploads/ByUwGPo8gx.png)
4. 打開網頁
![image](https://hackmd.io/_uploads/ByNrYEoUle.png)
5. 註冊帳號
![image](https://hackmd.io/_uploads/rkHBfwo8gx.png)
6. 釐清漏洞類型
    - "Tabnapping" (或稱為 Reverse Tabnabbing)，這是一個與 target="_blank" 相關的釣魚攻擊漏洞。
    - 什麼是 Tabnapping？
當一個網頁 A 包含一個連結指向網頁 B，並且該連結使用了 target="_blank" 屬性時，網頁 B（新開的分頁）可以透過 window.opener 這個 JavaScript 物件來存取和控制網頁 A（原始分頁）。
```
<!-- 網頁 A (http://trusted-site.com) 上的連結 -->
<a href="http://malicious-site.com" target="_blank">點擊這裡查看有趣的內容</a>
```
攻擊者可以在 malicious-site.com 這個頁面上放置惡意 JavaScript 程式碼。

。 攻擊流程
使用者點擊連結：使用者在受信任的網站 A 上，點擊了一個您提交的、指向您惡意網站 B 的連結。
。 新分頁開啟：惡意網站 B 在一個新的瀏覽器分頁中開啟。
。 惡意程式碼執行：網站 B 上的 JavaScript 程式碼被執行。這段程式碼會使用 window.opener.location 將原始分頁（網站 A）重新導向到一個假的登入頁面。
```
// 在 malicious-site.com 上的程式碼
if (window.opener) {
  window.opener.location.replace('http://fake-login-page.com');
}
```
使用者返回原始分頁：當使用者看完新分頁的內容，關閉它並返回原始分頁時，他們會看到一個看起來一模一樣但卻是假的登入頁面（例如，偽裝成 "您的連線已逾時，請重新登入"）。
竊取憑證：使用者不疑有他，在假的登入頁面上輸入他們的帳號密碼。這些憑證就會被傳送到攻擊者的伺服器。
