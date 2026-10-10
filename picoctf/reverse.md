# Easy

## Flag Hunters
1. Start by checking the provided `lyric-reader.py` file.
   - The flag is loaded at the beginning of the script: it reads a file named `flag.txt` and appends its contents to the `secret_intro` string. This `secret_intro` is then appended to the `song_flag_hunters` lyric string.
   - This means the flag is at the beginning of the `song_flag_hunters` lyrics. When `song.splitlines()` splits the lyrics into lines, the flag is in the first few lines (indices 0 to 4).
2. The main logic and vulnerability: `reader` is the core function. It processes the lyrics line by line and executes different actions depending on the content. The most important part is how it handles user input. There is a classic pattern here.

```python
import re
import time

flag = open('flag.txt', 'r').read()

secret_intro = \
'''Pico warriors rising, puzzles laid bare,
Solving each challenge with precision and flair.
With unity and skill, flags we deliver,
The ether’s ours to conquer, '''\
+ flag + '\n'

song_flag_hunters = secret_intro +\
'''

[REFRAIN]
We’re flag hunters in the ether, lighting up the grid,
No puzzle too dark, no challenge too hid.
With every exploit we trigger, every byte we decrypt,
We’re chasing that victory, and we’ll never quit.
CROWD (Singalong here!);
RETURN

[VERSE1]
Command line wizards, we’re starting it right,
Spawning shells in the terminal, hacking all night.
Scripts and searches, grep through the void,
Every keystroke, we're a cypher's envoy.
Brute force the lock or craft that regex,
Flag on the horizon, what challenge is next?

REFRAIN;

Echoes in memory, packets in trace,
Digging through the remnants to uncover with haste.
Hex and headers, carving out clues,
Resurrect the hidden, it's forensics we choose.
Disk dumps and packet dumps, follow the trail,
Buried deep in the noise, but we will prevail.

REFRAIN;

Binary sorcerers, let’s tear it apart,
Disassemble the code to reveal the dark heart.
From opcode to logic, tracing each line,
Emulate and break it, this key will be mine.
Debugging the maze, and I see through the deceit,
Patch it up right, and watch the lock release.

REFRAIN;

Ciphertext tumbling, breaking the spin,
Feistel or AES, we’re destined to win.
Frequency, padding, primes on the run,
Vigenère, RSA, cracking them for fun.
Shift the letters, matrices fall,
Decrypt that flag and hear the ether call.

REFRAIN;

SQL injection, XSS flow,
Map the backend out, let the database show.
Inspecting each cookie, fiddler in the fight,
Capturing requests, push the payload just right.
HTML's secrets, backdoors unlocked,
In the world wide labyrinth, we’re never lost.

REFRAIN;

Stack's overflowing, breaking the chain,
ROP gadget wizardry, ride it to fame.
Heap spray in silence, memory's plight,
Race the condition, crash it just right.
Shellcode ready, smashing the frame,
Control the instruction, flags call my name.

REFRAIN;

END;
'''

MAX_LINES = 100

def reader(song, startLabel):
  lip = 0
  start = 0
  refrain = 0
  refrain_return = 0
  finished = False

  song_lines = song.splitlines()

  for i in range(0, len(song_lines)):
    if song_lines[i] == startLabel:
      start = i + 1
    elif song_lines[i] == '[REFRAIN]':
      refrain = i + 1
    elif song_lines[i] == 'RETURN':
      refrain_return = i

  line_count = 0
  lip = start
  while not finished and line_count < MAX_LINES:
    line_count += 1
    for line in song_lines[lip].split(';'):
      if line == '' and song_lines[lip] != '':
        continue
      if line == 'REFRAIN':
        song_lines[refrain_return] = 'RETURN ' + str(lip + 1)
        lip = refrain
      elif re.match(r"CROWD.*", line):
        crowd = input('Crowd: ')
        song_lines[lip] = 'Crowd: ' + crowd
        lip += 1
      elif re.match(r"RETURN [0-9]+", line):
        lip = int(line.split()[1])
      elif line == 'END':
        finished = True
      else:
        print(line, flush=True)
        time.sleep(0.5)
        lip += 1

reader(song_flag_hunters, '[VERSE1]')
```

**Flag: picoCTF{70637h3r_f0r3v3r_75053bc3}**

## Transformation
1. Download the provided file and inspect it.

```bash
$ cat enc
灩捯䍔䙻ㄶ形楴獟楮獴㌴摟潦弸彥ㄴㅡて㝽
$ file enc
enc: Unicode text, UTF-8 text, with no line terminators
```

2. The challenge mentions the encryption script:

```python
''.join([chr((ord(flag[i]) << 8) + ord(flag[i + 1])) for i in range(0, len(flag), 2)])
```

This logic reads two characters at a time (`flag[i]` and `flag[i + 1]`), shifts the first character left by 8 bits, and adds the second one. The result is combined into a 16-bit value, then converted back into a Unicode character with `chr()`.

3. To decrypt it, we reverse the process:
   - iterate over each Unicode character in the encoded string
   - read its 16-bit value with `ord()`
   - extract the upper byte via `>> 8`
   - extract the lower byte via `& 0xFF`
   - convert both back with `chr()`
   - concatenate them to reconstruct the original flag

```python
def solve():
    encoded_string = "灩捯䍔䙻ㄶ形楴獟楮獴㌴摟潦弸彥ㄴㅡて㝽"
    decoded_flag = ""

    for char in encoded_string:
        unicode_val = ord(char)
        char1 = chr(unicode_val >> 8)
        char2 = chr(unicode_val & 0xFF)
        decoded_flag += char1
        decoded_flag += char2

    print(decoded_flag)

solve()
```

**Flag: picoCTF{16_bits_inst34d_of_8_e141a0f7}**

# Medium

## packer
* Description: Reverse this Linux executable.
* Hint: What can we do to reduce the size of a binary after compilation?

1. Inspect the file.

```bash
$ file out
out: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, no section header
```

This indicates it is a 64-bit ELF executable, statically linked, with no section headers. The key clue is that the section header was removed.

2. The common way to reduce executable size is to use a packer such as UPX.

```bash
$ sudo apt install upx-ucl
```

3. Use `-d` to unpack the file.

```bash
$ upx -d out
```

4. Run the binary; it prompts for a password and denies access with any string.

```bash
$ ./out
Enter the password to unlock this file: test
You entered: test
Access denied
```

5. Analyze `main` in GDB.

`readelf -h out` reveals the entry point `0x401c40`.

```bash
(gdb) disas 0x401c40
...
```

The function loads the address of `main` and then calls `__libc_start_main`.

```bash
(gdb) disas 0x401d65
...
```

Inside `main`, the flag is stored in pieces, loaded as 64-bit hexadecimal values into the stack. Because the system is little endian, we reverse the bytes when reconstructing the flag.

| Address | Hex value (big-endian) | Reversed (little-endian) | ASCII |
| :--- | :--- | :--- | :--- |
| `0x401e3a` | `0x6636333639363037` | `0x3730363936333666` | `7069636f` |
| `0x401e44` | `0x6237363434353334` | `0x3433353434363762` | `4354467b` |
| `0x401e56` | `0x6635383539333535` | `0x3535333935383566` | `5539585f` |
| `0x401e60` | `0x3433303565363535` | `0x3535366535303334` | `556e5034` |
| `0x401e72` | `0x6534313362363336` | `0x3633366233313465` | `636b314e` |
| `0x401e7c` | `0x3133323466353633` | `0x3336356634323331` | `365f4231` |
| `0x401e8e` | `0x3936323534336536` | `0x3665333435323639` | `6e345269` |
| `0x401e98` | `0x3633663533353333` | `0x3333353335663336` | `33535f36` |
| `0x401eaa` | `0x3633393336363636` | `0x3636363633393336` | `66663936` |
| `0x401eb4` | `0x6437363635363433` | `0x3334363536363764` | `3465667d` |

Combined, these produce the flag.

**Flag: picoCTF{U9X_UnP4ck1N6_B1n4Ri3S_6ff964ef}**

## Bit-O-Asm-1
* Description: Can you figure out what is in the `eax` register?
* Format: `picoCTF{n}` where `n` is the contents of `eax` in decimal.

1. Open the disassembly dump.

```bash
$ cat disassembler-dump0_a.txt
<+0>:     endbr64
<+4>:     push   rbp
<+5>:     mov    rbp,rsp
<+8>:     mov    DWORD PTR [rbp-0x4],edi
<+11>:    mov    QWORD PTR [rbp-0x10],rsi
<+15>:    mov    eax,0x30
<+20>:    pop    rbp
<+21>:    ret
```

2. The instruction `<+15>: mov eax, 0x30` moves the value `0x30` into `eax`, which is `48` in decimal.

The difference between 64-bit and 32-bit registers can be seen by the naming convention: `R*` for 64-bit, `E*` for 32-bit.

![image](https://hackmd.io/_uploads/S1cumeUPxl.png)

