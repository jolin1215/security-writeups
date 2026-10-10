# Web Exploitation
## North Pole Admin Portal

### 1. Recon

After entering the site, the login page had no backend interaction. At the end of the HTML source, an external JavaScript file was loaded:

```html
<script src="assets/main.js"></script>
```

So the focus shifted to the frontend JavaScript.

---

### 2. Source Code Analysis

Directly access `assets/main.js`:

```bash
$ curl -s https://ch3.northpole.acucysctf.org/assets/main.js
```

The `tryLogin()` function always returns failure and does not actually validate credentials.

In the same file, there is an uncalled function:

```bash
$ curl -s https://ch3.northpole.acucysctf.org/assets/main.js | sed -n '1,200p'
// main.js - site logic

function tryLogin(e){
  e.preventDefault();
  const u = document.getElementById('username').value || "";
  const p = document.getElementById('password').value || "";
  const msg = document.getElementById('message');

  // Simple fake check: always failing on purpose
  msg.textContent = "Invalid credentials. If you are an admin contact Santa.";
  msg.style.color = "salmon";
  // never call adminBackdoor() so the flag remains hidden in source
  return false;
}

/*
  Developer helper (never invoked)
  The flag is hidden inside this function — it's left here for debugging but not used.
  Players solving the CTF should open the page source / developer tools and read this file.
*/

function adminBackdoor(){
  const username = "admin";
  const password = "letmein";
  if (username === "admin" && password === "letmein") {
    // The real flag (do NOT remove)
    alert("AUCTF{0ce073a567bcad87f2e5b37181f3e36d}");
  }
}

// extra decoy function to make it look authentic
function heartbeat(){
  console.log("heartbeat: all systems jolly");
}
```

The comments clearly indicate this is a leftover developer helper function, and the flag is hardcoded in the frontend.

**Flag: AUCTF{0ce073a567bcad87f2e5b37181f3e36d}**

Below is a **concise, standard CTF write-up** with no unnecessary explanation, ready to submit.


# Steganography
## Christmas Jingles

### 1. Recon

The challenge provides a Windows `.exe` and hints: "Run the .exe file artefact with the notes to get the flag".
After checking with `file` and `strings`, `PYINSTALLER`-related strings appeared, indicating it is a **Python program packaged with PyInstaller**.
```bash
$ file jinglebells.pyc 
jinglebells.pyc: data
```

---

### 2. Unpacking

Use `pyinstxtractor.py` to unpack the executable:

```bash
$ python3 pyinstxtractor.py ~/Desktop/JingleBells.exe 

[+] Processing /home/jolin/Desktop/JingleBells.exe
[+] Pyinstaller version: 2.1+
[+] Python version: 3.14
[+] Length of package: 8041399 bytes
[+] Found 24 files in CArchive
[+] Beginning extraction...please standby
[+] Possible entry point: pyiboot01_bootstrap.pyc
[+] Possible entry point: pyi_rth_inspect.pyc
[+] Possible entry point: jinglebells.pyc
[!] Warning: This script is running in a different Python version than the one used to build the executable.
[!] Please run this script in Python 3.14 to prevent extraction errors during unmarshalling
[!] Skipping pyz extraction
[+] Successfully extracted pyinstaller archive: /home/jolin/Desktop/JingleBells.exe

You can now use a python decompiler on the pyc files within the extracted directory

```

The `JingleBells/` directory was successfully extracted, containing the main program `jinglebells.pyc`.

---

### 3. Analysis

Run string analysis directly on `jinglebells.pyc`:

```bash
$ cd JingleBells.exe_extracted   
```
```bash
$ ls -la
total 17852
drwxrwxr-x 3 jolin jolin    4096 Dec 19 23:26 .
drwxrwxr-x 4 jolin jolin    4096 Dec 19 23:26 ..
-rw-rw-r-- 1 jolin jolin 1698735 Dec 19 23:26 PYZ.pyz
drwxrwxr-x 2 jolin jolin    4096 Dec 19 23:26 PYZ.pyz_extracted
-rw-rw-r-- 1 jolin jolin  120400 Dec 19 23:26 VCRUNTIME140.dll
-rw-rw-r-- 1 jolin jolin   87896 Dec 19 23:26 _bz2.pyd
-rw-rw-r-- 1 jolin jolin  290136 Dec 19 23:26 _decimal.pyd
-rw-rw-r-- 1 jolin jolin   69464 Dec 19 23:26 _hashlib.pyd
-rw-rw-r-- 1 jolin jolin  161112 Dec 19 23:26 _lzma.pyd
-rw-rw-r-- 1 jolin jolin   87384 Dec 19 23:26 _socket.pyd
-rw-rw-r-- 1 jolin jolin  184656 Dec 19 23:26 _ssl.pyd
-rw-rw-r-- 1 jolin jolin  503640 Dec 19 23:26 _zstd.pyd
-rw-rw-r-- 1 jolin jolin 1387635 Dec 19 23:26 base_library.zip
-rw-rw-r-- 1 jolin jolin     806 Dec 19 23:26 jinglebells.pyc
-rw-rw-r-- 1 jolin jolin 5229424 Dec 19 23:26 libcrypto-3.dll
-rw-rw-r-- 1 jolin jolin  794992 Dec 19 23:26 libssl-3.dll
-rw-rw-r-- 1 jolin jolin    2851 Dec 19 23:26 pyi_rth_inspect.pyc
-rw-rw-r-- 1 jolin jolin    1915 Dec 19 23:26 pyiboot01_bootstrap.pyc
-rw-rw-r-- 1 jolin jolin    5054 Dec 19 23:26 pyimod01_archive.pyc
-rw-rw-r-- 1 jolin jolin   32335 Dec 19 23:26 pyimod02_importers.pyc
-rw-rw-r-- 1 jolin jolin    6635 Dec 19 23:26 pyimod03_ctypes.pyc
-rw-rw-r-- 1 jolin jolin    1705 Dec 19 23:26 pyimod04_pywin32.pyc
-rw-rw-r-- 1 jolin jolin 6760792 Dec 19 23:26 python314.dll
-rw-rw-r-- 1 jolin jolin   33624 Dec 19 23:26 select.pyd
-rw-rw-r-- 1 jolin jolin     307 Dec 19 23:26 struct.pyc
-rw-rw-r-- 1 jolin jolin  758104 Dec 19 23:26 unicodedata.pyd
```
```bash
$ strings -n 4 jinglebells.pyc

AUCTF{CHRISTMAS_MELODY}z#Welcome to Jingle Bells Music Game!zDEnter the first 11 notes for the melody. Use letters C D E F G A B. z
Notes (Seperated by space): z.Well Done! The flag is AUCTF{CHRISTMAS_MELODY}z
Almost! Try again. z
Press [ENTER] to exit.N)
correct_sequence
flag
print
input
strip
upper
split
user_input
jinglebells.py
<module>r
```

The output reveals these key points:

* The program asks for the first 11 notes of Jingle Bells
* Note format: `C D E F G A B`
* The flag is hardcoded in plaintext in the program

**Flag: AUCTF{CHRISTMAS_MELODY}**
