# Easy

## Rust fixme 1
1. Download the file with `wget https://challenge-xxx`.
2. Extract the archive with `tar zxvf fixme1.tar.gz`.

![image](https://hackmd.io/_uploads/HkqoBArllg.png)

3. Go into `fixme1` and run `cargo run`. The program in `/src/main.rs` has an error.

![image](https://hackmd.io/_uploads/H1hnX1Ilxl.png)

Modify the code:

```rust
use xor_cryptor::XORCryptor;

fn main() {
    // Key for decryption
    let key = String::from("CSUCKS"); // How do we end statements in Rust?

    // Encrypted flag values
    let hex_values = ["41", "30", "20", "63", "4a", "45", "54", "76", "01", "1c", "7e", "59", "63", "e1", "61", "25", "7f", "5a", "60", "50", "11", "38", "1f", "3a", "60", "e9", "62", "20", "0c", "e6", "7d", "4d", "77", "4b", "35", "5b", "0f", "4d", "56", "14", "63", "16", "2d", "1d", "44", "67", "1e", "4e", "7b", "6a", "31", "72", "15", "1d", "57", "50", "5e", "3d", "6c", "2d", "1a", "6f", "56", "4f", "51", "47", "17", "5b", "26", "6d", "74", "4d", "3d", "4c", "19", "1b", "2b", "75"];

    // Convert the hexadecimal strings to bytes and collect them into a vector
    let encrypted_buffer: Vec<u8> = hex_values.iter()
        .map(|&hex| u8::from_str_radix(hex, 16).unwrap())
        .collect();

    // Create decryption object
    let res = XORCryptor::new(&key);
    if res.is_err() {
        return; // How do we return in Rust?
    }
    let xrc = res.unwrap();

    // Decrypt flag and print it out
    let decrypted_buffer = xrc.decrypt_vec(encrypted_buffer);
    println!(
        "{}", // How do we print a variable in the println function?
        String::from_utf8_lossy(&decrypted_buffer)
    );
}
```

**Flag: picoCTF{4r3_y0u_4_ru$t4c30n_n0w?}**

## Rust fixme 2
1. Download the file with `wget https://challenge-xxx`.
2. Extract the archive with `tar zxvf fixme2.tar.gz`.

![image](https://hackmd.io/_uploads/SycgIkLxgx.png)

3. Go into `fixme2` and run `cargo run`. The program in `/src/main.rs` has an error.

![image](https://hackmd.io/_uploads/BycV8kIege.png)

Modify the code:

```rust
use xor_cryptor::XORCryptor;

fn decrypt(encrypted_buffer: Vec<u8>, borrowed_string: &mut String) {
    // Key for decryption
    let key = String::from("CSUCKS");

    // Editing our borrowed value
    borrowed_string.push_str("PARTY FOUL! Here is your flag: ");

    // Create decryption object
    let res = XORCryptor::new(&key);
    if res.is_err() {
        return; // How do we return in Rust?
    }
    let xrc = res.unwrap();

    // Decrypt flag and print it out
    let decrypted_buffer = xrc.decrypt_vec(encrypted_buffer);
    borrowed_string.push_str(&String::from_utf8_lossy(&decrypted_buffer));
    println!("{}", borrowed_string);
}

fn main() {
    // Encrypted flag values
    let hex_values = ["41", "30", "20", "63", "4a", "45", "54", "76", "01", "1c", "7e", "59", "63", "e1", "61", "25", "0d", "c4", "60", "f2", "12", "a0", "18", "03", "51", "03", "36", "05", "0e", "f9"];

    // Convert the hexadecimal strings to bytes and collect them into a vector
    let encrypted_buffer: Vec<u8> = hex_values.iter()
        .map(|&hex| u8::from_str_radix(hex, 16).unwrap())
        .collect();

    let mut party_foul = String::from("Using memory unsafe languages is a: ");
    decrypt(encrypted_buffer, &mut party_foul);
}
```

**Flag: picoCTF{4r3_y0u_h4v1n5_fun_y31?}**

## Rust fixme3
1. Download the file with `wget https://challenge-xxx`.
2. Extract the archive with `tar zxvf fixme3.tar.gz`.

![image](https://hackmd.io/_uploads/ry5u9J8lxl.png)

3. Go into `fixme3` and run `cargo run`. The program in `/src/main.rs` has an error.

![image](https://hackmd.io/_uploads/BkDi9yLele.png)

Modify the code:

```rust
use xor_cryptor::XORCryptor;

fn decrypt(encrypted_buffer: Vec<u8>, borrowed_string: &mut String) {
    // Key for decryption
    let key = String::from("CSUCKS");

    // Editing our borrowed value
    borrowed_string.push_str("PARTY FOUL! Here is your flag: ");

    // Create decryption object
    let res = XORCryptor::new(&key);
    if res.is_err() {
        return;
    }
    let xrc = res.unwrap();

    // Did you know you need to use "unsafe operations" in Rust?
    unsafe {
        let decrypted_buffer = xrc.decrypt_vec(encrypted_buffer);
        let decrypted_ptr = decrypted_buffer.as_ptr();
        let decrypted_len = decrypted_buffer.len();
        let decrypted_slice = std::slice::from_raw_parts(decrypted_ptr, decrypted_len);
        borrowed_string.push_str(&String::from_utf8_lossy(decrypted_slice));
    }
    println!("{}", borrowed_string);
}

fn main() {
    let hex_values = ["41", "30", "20", "63", "4a", "45", "54", "76", "12", "90", "7e", "53", "63", "e1", "01", "35", "7e", "59", "60", "f6", "03", "86", "7f", "56", "41", "29", "30", "6f", "08", "c3"];
    let encrypted_buffer: Vec<u8> = hex_values.iter()
        .map(|&hex| u8::from_str_radix(hex, 16).unwrap())
        .collect();

    let mut party_foul = String::from("Using memory unsafe languages is a: ");
    decrypt(encrypted_buffer, &mut party_foul);
}
```

**Flag: picoCTF{n0w_y0uv3_f1x3d_1h3m_411}**

## Time Machine
1. Download and extract `challenge.zip`; it contains a Git repository.

![image](https://hackmd.io/_uploads/HyezheIexl.png)

2. Try some Git commands. `git log` shows the commit history, and `-p` shows more detailed changes.

![image](https://hackmd.io/_uploads/rkcing8xlx.png)

**Flag: picoCTF{t1m3m@ch1n3_5cde9075}**

## endianness
1. To find the endianness, reverse the string and convert it to hex.
2. Little:

![image](https://hackmd.io/_uploads/ByMT8WLxgl.png)

Big:

![image](https://hackmd.io/_uploads/rJ7kDW8eeg.png)
![image](https://hackmd.io/_uploads/rkF4IZ8glg.png)

**Flag: picoCTF{3ndi4n_sw4p_su33ess_91bc76a4}**

## Commitment Issues
1. Download and extract `challenge.zip`; it contains a Git repository.

![image](https://hackmd.io/_uploads/ByUwt-8eeg.png)

2. Try different commands to inspect the contents.

![image](https://hackmd.io/_uploads/S1IJ5-Lexl.png)

**Flag: picoCTF{s@n1t1z3_cf09a485}**

## Collaborative Development
1. Download and extract `challenge.zip`; it contains a Git repository.
2. Inspect different commands and find that multiple branches were used to build the flag. `main`: initial flag printer; `feature/part-1`: contains part 1; `feature/part-2`: contains part 2; `feature/part-3`: contains part 3.

![image](https://hackmd.io/_uploads/BJGLfLwgeg.png)

3. (1) We need to compare differences between commits; a script can help.

```python
import subprocess
import sys

def run_git_command(command):
    result = subprocess.run(command.split(), capture_output=True, text=True)
    return result.stdout.strip()

def analyze_branches():
    branches = ['feature/part-1', 'feature/part-2', 'feature/part-3']
    main_branch = 'main'

    print('# Analyzing differences between branches')
    print('\n## Branch Differences:')

    for branch in branches:
        print(f'\n### {branch} vs {main_branch}')
        diff = run_git_command(f'git diff {main_branch}..{branch}')
        print(diff)

        print('\nCommit info:')
        commit_info = run_git_command(f'git log {branch} -n 1')
        print(commit_info)

if __name__ == "__main__":
    analyze_branches()
```

3. (2) This is essentially equivalent to running a `git diff` command.

![image](https://hackmd.io/_uploads/SkyqrLDegx.png)

**Flag: picoCTF{t3@mw0rk_m@k3s_th3_dr3@m_w0rk_4c24302f}**

## Blame Game
1. Download and extract `challenge.zip`; it contains a Git repository with many commits.

![image](https://hackmd.io/_uploads/SybljUDeel.png)

2. Based on the hint—"How can you see the changes within one file?"—we inspect the history of `message.py`.

![image](https://hackmd.io/_uploads/SyuGAUwlxl.png)

**Flag: picoCTF{@sk_th3_1nt3rn_cfca95b2}**

## binhexa
1. Connect via `nc` and start the challenge.

![螢幕擷取畫面 2025-05-07 103546](https://hackmd.io/_uploads/BJk4vHuxxe.png)

The first number computes to 205, and the second to 18.
2. The first question, `multi`, is calculated as `205 * 18 = 3690`, then converted to binary.
3. The second question, `or`, is `1` if either bit is `1`.
4. The third, `left shift`, shifts all bits left and adds `0` at the rightmost position.
5. The fourth, `and`, returns `1` only if both numbers have `1` in the same bit.
6. The fifth, `right shift`, shifts all bits right.
7. The sixth, `add`, adds the two numbers.

## repetitions
1. Download the file with `wget`, then inspect its type and contents.

![image](https://hackmd.io/_uploads/rJqxJIdeel.png)

2. Decode it and recover the flag.

![image](https://hackmd.io/_uploads/rJuIkUueee.png)

**Flag: picoCTF{base64_n3st3d_dic0d!n8_d0wnl04d3d_9b59b35c}**

## Big Zip
1. Download the file and find that it contains a huge amount of data.
2. Use `grep` across all files to search for the hidden flag.

![image](https://hackmd.io/_uploads/SJJCbI_xlx.png)

**Flag: picoCTF{gr3p_15_m4g1c_ef8790dc}**

## First Find
1. Download the file and notice the challenge mentions `uber-secret.txt`.

![image](https://hackmd.io/_uploads/SyjSFIOexl.png)

2. Search directly for the keyword.

![image](https://hackmd.io/_uploads/r1MtYUdexe.png)

3. As a bonus, use `find` to locate the file.

![image](https://hackmd.io/_uploads/HyghFUulxl.png)

## PW Crack 2
1. Running the Python script prompts for a password.
2. Inspect the source with `cat`; the password is computed as `chr(0x64) + chr(0x65) + chr(0x37) + chr(0x36)`.
3. Convert those values to their ASCII characters.

![image](https://hackmd.io/_uploads/ryXkiLOlle.png)

**Flag: picoCTF{tr45h_51ng1ng_489dea9a}**

## Magikarp Ground Mission
1. Connect via SSH and the first part of the flag is visible.
2. Go up one level and find the third part of the flag.

![image](https://hackmd.io/_uploads/ryM9G_ugee.png)

3. Continue navigating upward until the second part is found.

![image](https://hackmd.io/_uploads/Hkb8Qduell.png)

**Flag: picoCTF{xxsh_0ut_0f_\/\/4t3r_1118a9a4}**

## Python Wrangling
1. Based on the challenge, we have the decryption password in `pw.txt`, the encrypted data in `flag.txt.en`, and the script `ende.py`.

`ende.py`:

```python
import sys
import base64
from cryptography.fernet import Fernet

usage_msg = "Usage: " + sys.argv[0] + " (-e/-d) [file]"
help_msg = usage_msg + "\n" + \
        "Examples:\n" + \
        "  To decrypt a file named 'pole.txt', do: " + \
        "'$ python " + sys.argv[0] + " -d pole.txt'\n"

if len(sys.argv) < 2 or len(sys.argv) > 4:
    print(usage_msg)
    sys.exit(1)

if sys.argv[1] == "-e":
    if len(sys.argv) < 4:
        sim_sala_bim = input("Please enter the password:")
    else:
        sim_sala_bim = sys.argv[3]

    ssb_b64 = base64.b64encode(sim_sala_bim.encode())
    c = Fernet(ssb_b64)

    with open(sys.argv[2], "rb") as f:
        data = f.read()
        data_c = c.encrypt(data)
        sys.stdout.write(data_c.decode())

elif sys.argv[1] == "-d":
    if len(sys.argv) < 4:
        sim_sala_bim = input("Please enter the password:")
    else:
        sim_sala_bim = sys.argv[3]

    ssb_b64 = base64.b64encode(sim_sala_bim.encode())
    c = Fernet(ssb_b64)

    with open(sys.argv[2], "r") as f:
        data = f.read()
        data_c = c.decrypt(data.encode())
        sys.stdout.buffer.write(data_c)

elif sys.argv[1] == "-h" or sys.argv[1] == "--help":
    print(help_msg)
    sys.exit(1)

else:
    print("Unrecognized first argument: " + sys.argv[1])
    print("Please use '-e', '-d', or '-h'.")
```

2. Based on the script, we can use `-d` to decrypt the data.

![image](https://hackmd.io/_uploads/SyvK__dlxg.png)
![image](https://hackmd.io/_uploads/r1xEa_delg.png)

**Flag: picoCTF{4p0110_1n_7h3_h0us3_aa821c16}**

# Medium

## dont-you-love-banners
* Background: a symbolic link (symlink) is a file type in Linux that points to another file or directory on the system. Syntax: `ln -s <path to file/folder>`.
1. Connect to port 54401 to get the password, then connect to port 53017 to answer a question and obtain a shell.

![image](https://hackmd.io/_uploads/Sy8Sv5Ogxx.png)

2. Check the `/root` directory and find `flag.txt` is inaccessible; the permissions also do not help. Then inspect `script.py` and notice:

```python
with open("/home/player/banner", "r") as f:
    print(f.read())
```

![image](https://hackmd.io/_uploads/B1GTw9Oxgg.png)

3. Based on the hint, "Do you know about symlinks?", we use a symlink to access `/root/flag.txt` through `banner`.

![image](https://hackmd.io/_uploads/BJSeD5Olgx.png)

**Flag: picoCTF{b4nn3r_gr4bb1n9_su((3sfu11y_a0e119d4}**

## useless
1. `ssh picoplayer@saturn.picoctf.net -p 63124`
2. Inspect the script and see `Read the manual`.

![image](https://hackmd.io/_uploads/HJZC2cuxle.png)

3. In Linux/Unix, the `man` command displays system manuals. Running `man useless` means we need to read the manual page for `useless`.

![image](https://hackmd.io/_uploads/SyX8aquxll.png)

**Flag: picoCTF{us3l3ss_ch4ll3ng3_3xpl0it3d_8504}**

## Permissions
* Background:
    1. `sudo -i` (Initialize):
       - starts a new root shell
       - loads the root environment
       - changes to the root home directory
       - simulates a root login shell
    2. `sudo -l` (List):
       - lists the commands the current user can run as root
       - shows any restrictions
       - does not change permissions
       - keeps the current user's environment
1. Connect via SSH and inspect the file permissions; the challenge file is heavily restricted.

![image](https://hackmd.io/_uploads/rJi7wjdegg.png)

2. Check `sudo` privileges and see `ALL /usr/bin/vi`.

![image](https://hackmd.io/_uploads/HkLgPsOlee.png)

3. Run `sudo vi /etc/passwd` to open the file as root.
4. Once inside vi, type `:!/bin/bash` to launch a root shell.

![image](https://hackmd.io/_uploads/S1V-Oj_ggg.png)
![image](https://hackmd.io/_uploads/Hy8hui_lle.png)

**Flag: picoCTF{uS1ng_v1m_3dit0r_1cee9dcb}**

## Serpentine
`serpentine.py`:

```python
import random
import sys

def str_xor(secret, key):
    new_key = key
    i = 0
    while len(new_key) < len(secret):
        new_key = new_key + key[i]
        i = (i + 1) % len(key)
    return "".join([chr(ord(secret_c) ^ ord(new_key_c)) for (secret_c, new_key_c) in zip(secret, new_key)])

flag_enc = chr(0x15) + chr(0x07) + chr(0x08) + chr(0x06) + chr(0x27) + chr(0x21) + chr(0x23) + chr(0x15) + chr(0x5c) + chr(0x01) + chr(0x57) + chr(0x2a) + chr(0x17) + chr(0x5e) + chr(0x5f) + chr(0x5c)

def print_flag():
    flag = str_xor(flag_enc, 'enkidu')
    print(flag)

def print_encouragement():
    encouragements = ['You can do it!', 'Keep it up!', 'Look how far you\'ve come!']
    choice = random.choice(range(0, len(encouragements)))
    print('\n-----------------------------------------------------')
    print(encouragements[choice])
    print('-----------------------------------------------------\n\n')


def main():
    print('''
    Y
  .-^-.
 /     \      .- ~ ~ -.
()     ()    /   _ _   `.                     _ _ _
 \_   _/    /  /     \   \                . ~  _ _  ~ .
    | |     /  /       \   \             .' .~       ~-. `.
    | |    /  /         )   )           /  /             `.`.
    \ \_ _/  /         /   /           /  /                `'
     \_ _ _.'         /   /           (  (
                     /   /             \  \
                    /   /               \  \
                   /   /                 )  )
                  (   (                 /  /
                   `.  `.             .'  /
                     `.   ~ - - - - ~   .'
                        ~ . _ _ _ . ~
''')
    print('Welcome to the serpentine encourager!\n\n')

    while True:
        print('a) Print encouragement')
        print('b) Print flag')
        print('c) Quit\n')
        choice = input('What would you like to do? (a/b/c) ')

        if choice == 'a':
            print_encouragement()
        elif choice == 'b':
            print('\nOops! I must have misplaced the print_flag function! Check my source code!\n\n')
        elif choice == 'c':
            sys.exit(0)
        else:
            print('\nI did not understand "' + choice + '"\n\n')

if __name__ == "__main__":
    main()
```

This code uses `flag_enc`, the encrypted flag, and `str_xor` to decrypt it. The key is the string `enkidu`. Here is the decryption code:

```python
def str_xor(secret, key):
    new_key = key
    i = 0
    while len(new_key) < len(secret):
        new_key = new_key + key[i]
        i = (i + 1) % len(key)
    return "".join([chr(ord(secret_c) ^ ord(new_key_c)) for (secret_c, new_key_c) in zip(secret, new_key)])

flag_enc = chr(0x15) + chr(0x07) + chr(0x08) + chr(0x06) + chr(0x27) + chr(0x21) + chr(0x23) + chr(0x15) + chr(0x5c) + chr(0x01) + chr(0x57) + chr(0x2a) + chr(0x17) + chr(0x5e) + chr(0x5f) + chr(0x5c)

flag = str_xor(flag_enc, 'enkidu')
print(flag)
```

**Flag: picoCTF{7h3_r04d_l355_7r4v3l3d_8e47d128}**

## PW Crack 5
level5.py:
```
import hashlib

### THIS FUNCTION WILL NOT HELP YOU FIND THE FLAG --LT ########################
def str_xor(secret, key):
    #extend key to secret length
    new_key = key
    i = 0
    while len(new_key) < len(secret):
        new_key = new_key + key[i]
        i = (i + 1) % len(key)        
    return "".join([chr(ord(secret_c) ^ ord(new_key_c)) for (secret_c,new_key_c) in zip(secret,new_key)])
###############################################################################

flag_enc = open('level5.flag.txt.enc', 'rb').read()
correct_pw_hash = open('level5.hash.bin', 'rb').read()


def hash_pw(pw_str):
    pw_bytes = bytearray()
    pw_bytes.extend(pw_str.encode())
    m = hashlib.md5()
    m.update(pw_bytes)
    return m.digest()


def level_5_pw_check():
    user_pw = input("Please enter correct password for flag: ")
    user_pw_hash = hash_pw(user_pw)
    
    if( user_pw_hash == correct_pw_hash ):
        print("Welcome back... your flag, user:")
        decryption = str_xor(flag_enc.decode(), user_pw)
        print(decryption)
        return
    print("That password is incorrect")

level_5_pw_check()
```
* 以上這段程式碼的目的是檢查使用者輸入的密碼是否正確，並使用正確的密碼解密加密的旗標檔案 level5.flag.txt.enc。以下是解密的步驟：

    1. 檢查密碼的 hash 值：
使用者輸入密碼後，程式會將密碼轉換為 MD5 哈希值，並與檔案 level5.hash.bin 中的正確哈希值進行比較。
    2. 解密旗標：
如果密碼正確，程式會使用 str_xor 函數和密碼來解密 level5.flag.txt.enc 的內容，並輸出解密後的旗標。

decode.py:
```
import hashlib

def hash_pw(pw_str):
    pw_bytes = bytearray()
    pw_bytes.extend(pw_str.encode())
    m = hashlib.md5()
    m.update(pw_bytes)
    return m.digest()

def str_xor(secret, key):
    # Extend key to match the length of the secret
    new_key = key
    i = 0
    while len(new_key) < len(secret):
        new_key = new_key + key[i]
        i = (i + 1) % len(key)        
    return "".join([chr(ord(secret_c) ^ ord(new_key_c)) for (secret_c, new_key_c) in zip(secret, new_key)])

# 讀取正確的哈希值
correct_pw_hash = open('level5.hash.bin', 'rb').read()

# 讀取加密的旗標
flag_enc = open('level5.flag.txt.enc', 'rb').read()

# 使用字典檔案進行暴力破解
with open('dictionary.txt', 'r') as f:
    for line in f:
        password = line.strip()  # 去除換行符號
        if hash_pw(password) == correct_pw_hash:
            print(f"找到正確密碼: {password}")
            
            # 使用正確密碼解密旗標
            decrypted_flag = str_xor(flag_enc.decode(), password)
            print(f"解密後的旗標: {decrypted_flag}")
            break
    else:
        print("未能找到正確的密碼")
```
**Flag: picoCTF{h45h_sl1ng1ng_40f26f81}**
