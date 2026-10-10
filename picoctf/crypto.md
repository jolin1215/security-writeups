# Easy

## hashcrack
Use the website [crackstation](https://crackstation.net/)
MD5 -> SHA1 -> SHA256

```bash
user@Jolin:/mnt/c/Users/kuojo/Downloads$ nc verbal-sleep.picoctf.net 51759
Welcome!! Looking For the Secret?

We have identified a hash: 482c811da5d5b4bc6d497ffa98491e38
Enter the password for identified hash: password123
Correct! You've cracked the MD5 hash with no secret found!

Flag is yet to be revealed!! Crack this hash: b7a875fc1ea228b9061041b7cec4bd3c52ab3ce3
Enter the password for the identified hash: letmein
Correct! You've cracked the SHA-1 hash with no secret found!

Almost there!! Crack this hash: 916e8c4f79b25028c9e467f1eb8eee6d6bbdff965f9928310ad30a8d88697745
Enter the password for the identified hash: qwerty098
Correct! You've cracked the SHA-256 hash with a secret found.
The flag is: picoCTF{UseStr0nG_h@shEs_&PaSswDs!_ce730f64}
```

**Flag: picoCTF{UseStr0nG_h@shEs_&PaSswDs!_ce730f64}**

## interencdec
1. Decode twice with base64 to obtain the Caesar-encrypted text.

```bash
user@Jolin:/mnt/c/Users/kuojo/Downloads$ echo "YidkM0JxZGtwQlRYdHFhR3g2YUhsZmF6TnFlVG
wzWVROclh6YzRNalV3YUcxcWZRPT0nCg==" | base64 -d
b'd3BqdkpBTXtqaGx6aHlfazNqeTl3YTNrXzc4MjUwaG1qfQ=='
user@Jolin:/mnt/c/Users/kuojo/Downloads$ echo "d3BqdkpBTXtqaGx6aHlfazNqeTl3YTNrXzc4MjUwaG1qfQ==" | base64 -d
wpjvJAM{jhlzhy_k3jy9wa3k_78250hmj}
```

2. Use the [Caesar cipher decoder](https://www.dcode.fr/caesar-cipher) to decrypt it.

![image](https://hackmd.io/_uploads/Hy_YLvzBgl.png)

**Flag: picoCTF{caesar_d3cr9pt3d_78250afc}**

## Mod 26
Use the [ROT13 decoder](http://www.hiencode.com/rot13.html).

![image](https://hackmd.io/_uploads/B1iE-v7Bxx.png)

**Flag: picoctf{next_time_i'll_try_2_rounds_of_rot13_wqwosbkw}**

## The Numbers
1. The challenge provides the following image.

![the_numbers](https://hackmd.io/_uploads/SJaTpDQHxe.png)

2. I only figured out it was a simple A1Z26 substitution cipher after asking AI.
3. Use the [A1Z26 online decoder](https://zh.planetcalc.com/4884/) to decode it.

![image](https://hackmd.io/_uploads/S1sW0wQrxg.png)

**Flag: picoCTF{thenumbersmason}**
