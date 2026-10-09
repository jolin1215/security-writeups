# Easy
## Rust fixme 1
1. `wget https://challenge-xxx` 下載檔案
2. `tar zxvf fixme1.tar.gz` 解壓縮檔案
![image](https://hackmd.io/_uploads/HkqoBArllg.png)
3. 進 fixme1 執行`cargo run` 發現 `/src/main.rs` 程式碼有錯誤
![image](https://hackmd.io/_uploads/H1hnX1Ilxl.png)
修改程式碼:
```
use xor_cryptor::XORCryptor;

fn main() {
    // Key for decryption
    let key = String::from("CSUCKS"); // How do we end statements in Rust?//加 ; 做結尾

    // Encrypted flag values
    let hex_values = ["41", "30", "20", "63", "4a", "45", "54", "76", "01", "1c", "7e", "59", "63", "e1", "61", "25", "7f", "5a",>

    // Convert the hexadecimal strings to bytes and collect them into a vector
    let encrypted_buffer: Vec<u8> = hex_values.iter()
        .map(|&hex| u8::from_str_radix(hex, 16).unwrap())
        .collect();

    // Create decrpytion object
    let res = XORCryptor::new(&key);
    if res.is_err() {
        return; // How do we return in rust? //將 ret 改為 return
    }
    let xrc = res.unwrap();

    // Decrypt flag and print it out
    let decrypted_buffer = xrc.decrypt_vec(encrypted_buffer);
    println!(
        "{}", // How do we print out a variable in the println function? //將;改為{}
        String::from_utf8_lossy(&decrypted_buffer)
    );
}
```
**Flag: picoCTF{4r3_y0u_4_ru$t4c30n_n0w?}**

## Rust fixme 2
1. `wget https://challenge-xxx` 下載檔案
2. `tar zxvf fixme2.tar.gz` 解壓縮檔案
![image](https://hackmd.io/_uploads/SycgIkLxgx.png)
3. 進 fixme2 執行cargo run 發現 /src/main.rs 程式碼有錯誤
![image](https://hackmd.io/_uploads/BycV8kIege.png)修改程式碼:
```
use xor_cryptor::XORCryptor;

fn decrypt(encrypted_buffer:Vec<u8>, borrowed_string: &mut String){ // How do we pass values to a function that we want to change?
//在 Rust 透過 mut 關鍵字來使用可變性
//故這邊加一個 mut

    // Key for decryption
    let key = String::from("CSUCKS");

    // Editing our borrowed value
    borrowed_string.push_str("PARTY FOUL! Here is your flag: ");

    // Create decrpytion object
    let res = XORCryptor::new(&key);
    if res.is_err() {
        return; // How do we return in rust?
    }
    let xrc = res.unwrap();

    // Decrypt flag and print it out
    let decrypted_buffer = xrc.decrypt_vec(encrypted_buffer);
    borrowed_string.push_str(&String::from_utf8_lossy(&decrypted_buffer));
    println!("{}", borrowed_string);
}


fn main() {
    // Encrypted flag values
    let hex_values = ["41", "30", "20", "63", "4a", "45", "54", "76", "01", "1c", "7e", "59", "63", "e1", "61", "25", "0d", "c4",>

    // Convert the hexadecimal strings to bytes and collect them into a vector
    let encrypted_buffer: Vec<u8> = hex_values.iter()
        .map(|&hex| u8::from_str_radix(hex, 16).unwrap())
        .collect();

    let mut party_foul = String::from("Using memory unsafe languages is a: "); // Is this variable changeable?
//第二個 mut
    decrypt(encrypted_buffer, &mut party_foul); // Is this the correct way to pass a value to a function so that it can be change>}
//第三個 mut

```

**Flag: picoCTF{4r3_y0u_h4v1n5_fun_y31?}**

## Rust fixme3
1. `wget https://challenge-xxx` 下載檔案
2. `tar zxvf fixme3.tar.gz` 解壓縮檔案
![image](https://hackmd.io/_uploads/ry5u9J8lxl.png)
3. 進 fixme3 執行cargo run 發現 /src/main.rs 程式碼有錯誤
![image](https://hackmd.io/_uploads/BkDi9yLele.png)
修改程式碼:
```
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

    // Did you know you have to do "unsafe operations in Rust?
    // https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html
    // Even though we have these memory safe languages, sometimes we need to do things outside of the rules
    // This is where unsafe rust comes in, something that is important to know about in order to keep things in perspective       
    //某些函數可以聲明為unsafe，這意味著確保正確性是程式設計師的責任，而不是編譯器的責任。
    unsafe {
        // Decrypt the flag operations
        let decrypted_buffer = xrc.decrypt_vec(encrypted_buffer);

        // Creating a pointer
        let decrypted_ptr = decrypted_buffer.as_ptr();
        let decrypted_len = decrypted_buffer.len();

        // Unsafe operation: calling an unsafe function that dereferences a raw pointer
        let decrypted_slice = std::slice::from_raw_parts(decrypted_ptr, decrypted_len);

        borrowed_string.push_str(&String::from_utf8_lossy(decrypted_slice));
    }
    println!("{}", borrowed_string);
}

fn main() {
    // Encrypted flag values
    let hex_values = ["41", "30", "20", "63", "4a", "45", "54", "76", "12", "90", "7e", "53", "63", "e1", "01", "35", "7e", "59", "60", "f6", "03", "86", "7f", "56", "41", "29", "30", "6f", "08", "c3", "61", "f9", "35"];

    // Convert the hexadecimal strings to bytes and collect them into a vector
    let encrypted_buffer: Vec<u8> = hex_values.iter()
        .map(|&hex| u8::from_str_radix(hex, 16).unwrap())
        .collect();

    let mut party_foul = String::from("Using memory unsafe languages is a: ");
    decrypt(encrypted_buffer, &mut party_foul);
}
```
**Flag: picoCTF{n0w_y0uv3_f1x3d_1h3m_411}**

## Time Machine
1. 下載並解壓縮 challenge.zip 發現內容為 Git repository
![image](https://hackmd.io/_uploads/HyezheIexl.png)
2. 嘗試使用一些 Git command ，git log 可查看過去commit的紀錄，-p 則可以查看更詳細的內容
![image](https://hackmd.io/_uploads/rkcing8xlx.png)

**Flag: picoCTF{t1m3m@ch1n3_5cde9075}**

## endianness
1. 為了找到小端和大端，我們需要先反轉字串，然後將其轉換為十六進位 (Hex) 形式
2. Little:
![image](https://hackmd.io/_uploads/ByMT8WLxgl.png)
Big:
![image](https://hackmd.io/_uploads/rJ7kDW8eeg.png)
![image](https://hackmd.io/_uploads/rkF4IZ8glg.png)
**Flag: picoCTF{3ndi4n_sw4p_su33ess_91bc76a4}**

## Commitment Issues
1. 下載並解壓縮 `challenge.zip`，發現檔案為 Git repository
![image](https://hackmd.io/_uploads/ByUwt-8eeg.png)
2. 輸入不同的 command 試圖查看內容
![image](https://hackmd.io/_uploads/S1IJ5-Lexl.png)

**Flag: picoCTF{s@n1t1z3_cf09a485}**

## Collaborative Development
1. 下載並解壓縮 `challenge.zip`，發現檔案為 Git repository
2. 輸入不同的 command 查看內容，發現有不同的 branch 共同寫 flag。main: 初始 flag printer 設置 ; feature/part-1: Contains part 1 of the flag ; feature/part-2: Contains part 2 of the flag ; feature/part-3: Contains part 3 of the flag (branch 就像是程式碼的平行版本。可以將其視為在保證原始程式碼安全的同時建立一條新的開發路徑。)
![image](https://hackmd.io/_uploads/BJGLfLwgeg.png)
3. (1)推測需要查看每次修改之間的不一樣，故寫一 script
```
import subprocess
import sys

def run_git_command(command):
    """Execute git command and return output"""
    result = subprocess.run(command.split(), capture_output=True, text=True)
    return result.stdout.strip()

def analyze_branches():
    # Get all feature branches
    branches = ['feature/part-1', 'feature/part-2', 'feature/part-3']
    main_branch = 'main'
    
    print("# Analyzing differences between branches")
    print("\n## Branch Differences:")
    
    for branch in branches:
        print(f"\n### {branch} vs {main_branch}")
        # Show diff between main and feature branch
        diff = run_git_command(f"git diff {main_branch}..{branch}")
        print(diff)
        
        # Show commit info for the branch
        print("\nCommit info:")
        commit_info = run_git_command(f"git log {branch} -n 1")
        print(commit_info)

if __name__ == "__main__":
    analyze_branches()
```
3. (2)其實也等同於輸入 git diff command
![image](https://hackmd.io/_uploads/SkyqrLDegx.png)
**Flag: picoCTF{t3@mw0rk_m@k3s_th3_dr3@m_w0rk_4c24302f}**

## Blame Game
1. 下載並解壓縮 challenge.zip 發現內容為 Git repository，並且有非常多的 commits(可以將 commits 視為保存項目的一個版本，並附上一條描述更改內容的訊息)。
![image](https://hackmd.io/_uploads/SybljUDeel.png)
2. 根據 Hint: How can you see the changes within one file?。推測要查看 message.py 的修改紀錄
![image](https://hackmd.io/_uploads/SyuGAUwlxl.png)
**Flag: picoCTF{@sk_th3_1nt3rn_cfca95b2}**

## binhexa
1. nc 連線後開始解題
![螢幕擷取畫面 2025-05-07 103546](https://hackmd.io/_uploads/BJk4vHuxxe.png)
Number 1 計算後為205，Number 計算後為 18
2. 第一題 'multi' 計算方式為 205*18=3690，再轉換為 binary
3. 第二題 'or' 只要其中一位元為 1，則 result 就為 1
4. 第三題 'left shift' 將所有位元往左移，並在最右位加 0
5. 第四題 'and' 要兩個數字同一位元皆為 1 十 result 才為 1
6. 第五題 'right shift' 將所有位元往右移
7. 第六題 'add' 將兩數相加

## repetitions
1. `wget`下載檔案後查看文件類型與內容
![image](https://hackmd.io/_uploads/rJqxJIdeel.png)
2. 開始解碼並得到 flag
![image](https://hackmd.io/_uploads/rJuIkUueee.png)
**Flag: picoCTF{base64_n3st3d_dic0d!n8_d0wnl04d3d_9b59b35c}**

## Big Zip
1. `wget`下載完檔案後發現有巨量資料
2. 用`grep`直接搜尋所有檔案看有沒有藏 flag
![image](https://hackmd.io/_uploads/SJJCbI_xlx.png)
**Flag: picoCTF{gr3p_15_m4g1c_ef8790dc}**

## First Find
1. `wget`下載檔案看到題目提到的 uber-secret.txt
![image](https://hackmd.io/_uploads/SyjSFIOexl.png)
2. 直接搜尋關鍵字看看
![image](https://hackmd.io/_uploads/r1MtYUdexe.png)
3. 題外話放個`find`找尋檔案
![image](https://hackmd.io/_uploads/HyghFUulxl.png)

## PW Crack 2
1. 執行 python 後看到要輸入密碼
2. `cat`查看程式碼發現密碼等於 chr(0x64) + chr(0x65) + chr(0x37) + chr(0x36) 函數
3. 接著就轉換為其對應的 ASCII 字符 
![image](https://hackmd.io/_uploads/ryXkiLOlle.png)
**Flag: picoCTF{tr45h_51ng1ng_489dea9a}**

## Magikarp Ground Mission
1. ssh 連線後先看到第一段 flag
2. 到上一層後看到第三段 flag
![image](https://hackmd.io/_uploads/ryM9G_ugee.png)
3. 持續往上一層走，找到第二段 flag
![image](https://hackmd.io/_uploads/Hkb8Qduell.png)
**Flag: picoCTF{xxsh_0ut_0f_\/\/4t3r_1118a9a4}**

## Python Wrangling
1. 根據題目得到解密的密碼`pw.txt`、加密資料`flag.txt.en`、加密解密腳本`ende.py`

ende.py:
```
import sys
import base64
from cryptography.fernet import Fernet



usage_msg = "Usage: "+ sys.argv[0] +" (-e/-d) [file]"
help_msg = usage_msg + "\n" +\
        "Examples:\n" +\
        "  To decrypt a file named 'pole.txt', do: " +\
        "'$ python "+ sys.argv[0] +" -d pole.txt'\n"



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
    print("Unrecognized first argument: "+ sys.argv[1])
    print("Please use '-e', '-d', or '-h'.")
```
2. 根據 script 內容推測用`-d`來解開加密資料
![image](https://hackmd.io/_uploads/SyvK__dlxg.png)
![image](https://hackmd.io/_uploads/r1xEa_delg.png)

**Flag: picoCTF{4p0110_1n_7h3_h0us3_aa821c16}**

# Medium
## dont-you-love-banners
* 前情提要 : 符號連結（symbolic link 或 symlink）是 Linux 中一種文件類型，它指向計算機上的另一个文件或文件夹。語法為`ln -s <path to the file/folder to be linked> <the path of the link to be created>`。ln 是連結命令 ; -s 標記表示这个連結是軟連結 ; -s 也可以寫成 -symbolic。`<path to the file/folder to be linked>`為想要連結的文件，`<the path of the link to be created>`為連結本身（快捷方式）。
1. 先連接 54401 port 拿到密碼，再連接 53017 port 回答問題進入 shell
![image](https://hackmd.io/_uploads/Sy8Sv5Ogxx.png)
2. 查看 root 資料夾底下的 flag.txt，發現沒有權限，且加權也沒用。再接著查看 script.py，發現`with open("/home/player/banner", "r") as f:
        print(f.read())`
![image](https://hackmd.io/_uploads/B1GTw9Oxgg.png)
3. 再加上題目的`Hint:Do you know about symlinks?`，推測要用符號連結來使用 banner 訪問 /root/flag.txt
![image](https://hackmd.io/_uploads/BJSeD5Olgx.png)

**Flag: picoCTF{b4nn3r_gr4bb1n9_su((3sfu11y_a0e119d4}**

## useless
1. `ssh picoplayer@saturn.picoctf.net -p 63124`
2. 查看 script，看到`Read the manual`
![image](https://hackmd.io/_uploads/HJZC2cuxle.png)
3. Linux/Unix 系統中的`man`指令用於顯示系統的手冊頁。執行 man useless 時，是在要求查看 useless 的手冊文件。
![image](https://hackmd.io/_uploads/SyX8aquxll.png)
**Flag: picoCTF{us3l3ss_ch4ll3ng3_3xpl0it3d_8504**

## Permissions
* 前情提要:
    1. sudo -i (Initialize):
(1)建立一個新的 root 權限 shell
(2)載入 root 的環境變數
(3)切換到 root 的家目錄
(4)模擬 root 的初始登入環境
    2. sudo -l (List):
(1)列出目前使用者可執行的 sudo 指令
(2)顯示特殊的權限限制
(3)不會改變權限
(4)保持在當前使用者環境
1. ssh 連線後進入查看檔案權限，發現 challenge 檔案被限制很多
![image](https://hackmd.io/_uploads/rJi7wjdegg.png)
2. 查看 sudo 權限有甚麼，發現`(ALL) /usr/bin/vi`
![image](https://hackmd.io/_uploads/HkLgPsOlee.png)
3. 輸入`sudo vi /etc/passwd`以 root 權限執行 vi 編輯器，此時 vi 進程擁有 root 權限
4. 按 etc 後輸入`:!/bin/bash`。`:!`是 vim 的外部命令執行語法，當執行`:!/bin/bash`時，新的 shell 繼承了 vim 的 root 權限
![image](https://hackmd.io/_uploads/S1V-Oj_ggg.png)
![image](https://hackmd.io/_uploads/Hy8hui_lle.png)
**Flag: picoCTF{uS1ng_v1m_3dit0r_1cee9dcb}**

## Serpentine
serpentine.py:
```
import random
import sys

def str_xor(secret, key):
    #extend key to secret length
    new_key = key
    i = 0
    while len(new_key) < len(secret):
        new_key = new_key + key[i]
        i = (i + 1) % len(key)        
    return "".join([chr(ord(secret_c) ^ ord(new_key_c)) for (secret_c,new_key_c) in zip(secret,new_key)])


flag_enc = chr(0x15) + chr(0x07) + chr(0x08) + chr(0x06) + chr(0x27) + chr(0x21) + chr(0x23) + chr(0x15) + chr(0x5c) + chr(0x01) + chr(0x57) + chr(0x2a) + chr(0x17) + chr(0x5e) + chr(0x5f) + chr(0x0d) + chr(0x3b) + chr(0x19) + chr(0x56) + chr(0x5b) + chr(0x5e) + chr(0x36) + chr(0x53) + chr(0x07) + chr(0x51) + chr(0x18) + chr(0x58) + chr(0x05) + chr(0x57) + chr(0x11) + chr(0x3a) + chr(0x56) + chr(0x0e) + chr(0x5d) + chr(0x53) + chr(0x11) + chr(0x54) + chr(0x5c) + chr(0x53) + chr(0x14)


def print_flag():
  flag = str_xor(flag_enc, 'enkidu')
  print(flag)


def print_encouragement():
  encouragements = ['You can do it!', 'Keep it up!', 
                    'Look how far you\'ve come!']
  choice = random.choice(range(0, len(encouragements)))
  print('\n-----------------------------------------------------')
  print(encouragements[choice])
  print('-----------------------------------------------------\n\n')



def main():

  print(
'''
    Y
  .-^-.
 /     \      .- ~ ~ -.
()     ()    /   _ _   `.                     _ _ _
 \_   _/    /  /     \   \                . ~  _ _  ~ .
   | |     /  /       \   \             .' .~       ~-. `.
   | |    /  /         )   )           /  /             `.`.
   \ \_ _/  /         /   /           /  /                `'
    \_ _ _.'         /   /           (  (
                    /   /             \  \\
                   /   /               \  \\
                  /   /                 )  )
                 (   (                 /  /
                  `.  `.             .'  /
                    `.   ~ - - - - ~   .'
                       ~ . _ _ _ _ . ~
'''
  )
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
      print('\nI did not understand "' + choice + '", input only "a", "b" or "c"\n\n')



if __name__ == "__main__":
  main()
```
* 這段程式碼中，flag_enc 是加密的旗標，使用了 str_xor 函數進行解密。解密的關鍵是字串 'enkidu'。以下是解密 flag_enc 的程式碼：
```
def str_xor(secret, key):
    # Extend key to match the length of the secret
    new_key = key
    i = 0
    while len(new_key) < len(secret):
        new_key = new_key + key[i]
        i = (i + 1) % len(key)        
    return "".join([chr(ord(secret_c) ^ ord(new_key_c)) for (secret_c, new_key_c) in zip(secret, new_key)])

flag_enc = chr(0x15) + chr(0x07) + chr(0x08) + chr(0x06) + chr(0x27) + chr(0x21) + chr(0x23) + chr(0x15) + chr(0x5c) + chr(0x01) + chr(0x57) + chr(0x2a) + chr(0x17) + chr(0x5e) + chr(0x5f) + chr(0x0d) + chr(0x3b) + chr(0x19) + chr(0x56) + chr(0x5b) + chr(0x5e) + chr(0x36) + chr(0x53) + chr(0x07) + chr(0x51) + chr(0x18) + chr(0x58) + chr(0x05) + chr(0x57) + chr(0x11) + chr(0x3a) + chr(0x56) + chr(0x0e) + chr(0x5d) + chr(0x53) + chr(0x11) + chr(0x54) + chr(0x5c) + chr(0x53) + chr(0x14)

# 解密
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

## flag_shop
store.c:
```
#include <stdio.h>
#include <stdlib.h>
int main()
{
    setbuf(stdout, NULL);
    int con;
    con = 0;
    int account_balance = 1100;
    while(con == 0){
        
        printf("Welcome to the flag exchange\n");
        printf("We sell flags\n");

        printf("\n1. Check Account Balance\n");
        printf("\n2. Buy Flags\n");
        printf("\n3. Exit\n");
        int menu;
        printf("\n Enter a menu selection\n");
        fflush(stdin);
        scanf("%d", &menu);
        if(menu == 1){
            printf("\n\n\n Balance: %d \n\n\n", account_balance);
        }
        else if(menu == 2){
            printf("Currently for sale\n");
            printf("1. Defintely not the flag Flag\n");
            printf("2. 1337 Flag\n");
            int auction_choice;
            fflush(stdin);
            scanf("%d", &auction_choice);
            if(auction_choice == 1){
                printf("These knockoff Flags cost 900 each, enter desired quantity\n");
                
                int number_flags = 0;
                fflush(stdin);
                scanf("%d", &number_flags);
                if(number_flags > 0){
                    int total_cost = 0;
                    total_cost = 900*number_flags;
                    printf("\nThe final cost is: %d\n", total_cost);
                    if(total_cost <= account_balance){
                        account_balance = account_balance - total_cost;
                        printf("\nYour current balance after transaction: %d\n\n", account_balance);
                    }
                    else{
                        printf("Not enough funds to complete purchase\n");
                    }             
                    
                }
                
            }
            else if(auction_choice == 2){
                printf("1337 flags cost 100000 dollars, and we only have 1 in stock\n");
                printf("Enter 1 to buy one");
                int bid = 0;
                fflush(stdin);
                scanf("%d", &bid);
                
                if(bid == 1){
                    
                    if(account_balance > 100000){
                        FILE *f = fopen("flag.txt", "r");
                        if(f == NULL){

                            printf("flag not found: please run this on the server\n");
                            exit(0);
                        }
                        char buf[64];
                        fgets(buf, 63, f);
                        printf("YOUR FLAG IS: %s\n", buf);
                        }
                    
                    else{
                        printf("\nNot enough funds for transaction\n\n\n");
                    }}

            }
        }
        else{
            con = 1;
        }

    }
    return 0;
}
```
* 看以上過程式流程，可以得知金額變動的地方只有購買knockoff Flags 的區段，也相當於漏洞有可能出現的地方。於是試著輸入非常大的數字，最終得到 Flag

![截圖 2025-05-09 上午10.32.34](https://hackmd.io/_uploads/r1bwFyiglg.png)

**Flag: picoCTF{m0n3y_bag5_68d16363}**

## Special
1. ssh 連線進入後輸入密碼進到 shell，發現無法運行正常命令
![截圖 2025-05-09 上午11.45.53](https://hackmd.io/_uploads/BkJY5eoxee.png)
2. `${parameter='command_here'}`為一參數擴展的語法，要擴展的參數名稱或符號可以用括號括起來，括號是可選的，但可以保護要擴展的變數免受其後緊接的字元的影響
![截圖 2025-05-09 上午11.50.58](https://hackmd.io/_uploads/rJe2ogsegx.png)

**Flag: picoCTF{5p311ch3ck_15_7h3_w0r57_6a2763f6}**

## plumbing
1. `nc`連線進入伺服器跳出大量的資料不斷重複
![截圖 2025-05-09 下午4.14.06](https://hackmd.io/_uploads/BJoUKEselx.png)
2. 使用`grep`擷取關鍵字
![截圖 2025-05-09 下午4.15.06](https://hackmd.io/_uploads/rJLqFNjlxe.png)
**Flag: picoCTF{digital_plumb3r_06e9d954}**

## mus1c
lyrics.txt:
```
Pico's a CTFFFFFFF
my mind is waitin
It's waitin

Put my mind of Pico into This
my flag is not found
put This into my flag
put my flag into Pico


shout Pico
shout Pico
shout Pico

My song's something
put Pico into This

Knock This down, down, down
put This into CTF

shout CTF
my lyric is nothing
Put This without my song into my lyric
Knock my lyric down, down, down

shout my lyric

Put my lyric into This
Put my song with This into my lyric
Knock my lyric down

shout my lyric

Build my lyric up, up ,up

shout my lyric
shout Pico
shout It

Pico CTF is fun
security is important
Fun is fun
Put security with fun into Pico CTF
Build Fun up
shout fun times Pico CTF
put fun times Pico CTF into my song

build it up

shout it
shout it

build it up, up
shout it
shout Pico
```
1. 把以上歌詞丟到 rockstar online Interpreter 得到一串 ASCII
![截圖 2025-05-09 下午4.33.10](https://hackmd.io/_uploads/SJE0pNilel.png)
2. 解密完得到 rrrocknrn0113r

**Flag: picoCTF{rrrocknrn0113r}**

## specialer
1. ssh 連線進伺服器後先按兩下 tab 檢查有什麼可用的指令
![截圖 2025-05-09 下午5.52.32](https://hackmd.io/_uploads/BJyceIsgex.png)
2. 查看路徑與檔案
![image](https://hackmd.io/_uploads/SJD9_1y-xx.png)
3. `echo`: 顯示文本
`$()`: 命令替換(執行指令並回傳輸出)
`<`: 輸入重定向(`<abra/xxx.txt`將檔案內容重新導向為輸入)
![image](https://hackmd.io/_uploads/H1B1tkk-le.png)
**Flag: picoCTF{y0u_d0n7_4ppr3c1473_wh47_w3r3_d01ng_h3r3_838b49d1}**

## SansAlpha
1. ssh 連線進去伺服器後發現只能使用數字與符號下指令
![image](https://hackmd.io/_uploads/S19eMEk-xl.png)
2. 使用`./*/*`看路徑，發現 flag.txt
![image](https://hackmd.io/_uploads/Hk6mz4JZlx.png)
3. 用萬用的 ? 來做到以`/???`代替`/bin`執行指令
![image](https://hackmd.io/_uploads/SJ6uzNy-lx.png)
4. 原本想用`cat`，但系統會預設使用`awk`，於是決定使用`base64`讀檔案，為防止太多符號引發錯亂，將`/bin/base64`指定為_1
![image](https://hackmd.io/_uploads/HJF974y-lg.png)
5. 最後拿 base64 加密後的 flag 去 cyberchef 解密
![image](https://hackmd.io/_uploads/BkIGV41Wgx.png)
**Flag:　picoCTF{7h15_mu171v3r53_15_m4dn355_36a674c0}**