### constructor

這是一個關鍵在於辨識反編譯器 (Decompiler) 的錯誤，並回歸到最根本的組合語言來理解程式的真實行為。

#### **1. 初步分析 (Initial Analysis)**

挑戰提供了一個 C++ 原始碼檔案 `constructor.cpp` 和一個編譯好的執行檔 `constructor`。

首先查看 `constructor.cpp`：

````cpp
#include <iostream>

void __attribute__((constructor)) solve() {
    char a1[] = "/dev/urandom";
    char flag[42];
    int v3; // [rsp+2Ch] [rbp-14h]
    int fd; // [rsp+30h] [rbp-10h]
    int i; // [rsp+34h] [rbp-Ch]
    int v6; // [rsp+38h] [rbp-8h]
    int v7; // [rsp+3Ch] [rbp-4h]

    v6 = 0;
    v7 = 0;
    fd = open(a1, 0);
    if ( fd != -1 )
    {
        read(fd, flag, 0x2A);
        close(fd);
        for ( i = 0; i <= 41; ++i )
        {
            v3 = (char)(v6 ^ flag[i]);
            v6 += 31;
            flag[i] = (i >> 1) ^ v3 ^ 0x5A;
        }
    }
}

int main() {
    // ...
}
````

程式碼的亮點是 `__attribute__((constructor))`，這意味著 `solve()` 函數會在 `main` 函數執行之前被呼叫。

`solve()` 函數的邏輯看起來是：
1.  從 `/dev/urandom` 讀取 42 個隨機位元組到 `flag` 陣列。
2.  對這 42 個位元組進行一系列的 XOR 和加法運算。

如果這是程式的真實行為，那麼 flag 將會是隨機的，無法求解。這暗示：**原始碼是具有欺騙性的，真正的邏輯在編譯後的二進位檔案中。**

#### **2. 深入彙編 (Diving into Assembly)**

使用 IDA 工具來分析 `constructor` 執行檔。定位到 `solve` 函數 

* 反編譯結果的陷阱

    IDA 的反編譯結果可能與原始碼相似，但會出現一些奇怪的現象，例如對一個從未被初始化的陣列進行操作。這進一步證實了反編譯器可能沒有完全理解程式碼。

* 組合語言才是真相 (The Assembly is the Truth)

    必須直接閱讀組合語言來確定到底發生了什麼。以下是 `solve` 函數核心迴圈的組合語言：

    ```nasm
    .text:00000000004011D2                 mov     rax, [rbp+i]
    .text:00000000004011D6                 movzx   eax, byte ptr [rdx+rax] ; [1]
    .text:00000000004011DA                 mov     ecx, [rbp+v6]
    .text:00000000004011DD                 xor     eax, ecx                ; [2] v8 = v6 ^ data[i]
    .text:00000000004011DF                 mov     [rbp+v3], al
    .text:00000000004011E2                 add     [rbp+v6], 1Fh           ; [3] v6 += 31
    .text:00000000004011E6                 mov     rax, [rbp+i]
    .text:00000000004011EA                 shr     rax, 1                  ; [4] i >> 1
    .text:00000000004011EE                 movzx   ecx, [rbp+v3]
    .text:00000000004011F2                 xor     cl, al                  ; [5] (i >> 1) ^ v3
    .text:00000000004011F4                 xor     cl, 5Ah                 ; [6] ((i >> 1) ^ v3) ^ 0x5A
    .text:00000000004011F7                 mov     rax, [rbp+i]
    .text:00000000004011FB                 mov     [rbp+flag+rax], cl      ; [7] flag[i] = result
    ```

    關鍵的發現是 `[1]` 這一行：`movzx eax, byte ptr [rdx+rax]`。在迴圈開始前，`rdx` 被設定為一個記憶體位址：

    ```nasm
    .text:00000000004011C7                 mov     rdx, cs:a3_data ; "3!..."
    ```
    `a3_data` 指向 `.rodata` 區段的一個位址 `0x403040`。這表示程式**並不是**從 `/dev/urandom` 讀取資料，而是使用一個硬編碼在執行檔中的 42 位元組陣列！

#### **3. 演算法還原與求解**

從 `.rodata` 區段位址 `0x403040` 提取出這 42 個位元組的資料。

然後，根據組合語言的邏輯，可以編寫一個 Python 腳本來模擬解密過程：

1.  初始化一個變數 `v6 = 0`。
2.  遍歷 42 個位元組，索引為 `i`。
3.  `v8 = v6 ^ a3_data[i]`
4.  `v6 = v6 + 31` (注意保持在 32 位元範圍內)
5.  `result = (i >> 1) ^ v8 ^ 0x5A`
6.  將 `result` 的最低位元組 (`result & 0xFF`) 存為 flag 的第 `i` 個字元。

**Solver Script :**

````python
def solve():
    # 從 .rodata:0x403040 提取的 42 位元組資料
    a3_data = bytes([
        0x33, 0x21, 0x00, 0x6d, 0x5f, 0xab, 0x86, 0xb4,
        0xd4, 0x2d, 0x36, 0x3a, 0x4e, 0x90, 0x8c, 0xe3,
        0xcc, 0x2e, 0x09, 0x6c, 0x49, 0xb8, 0x8f, 0xf7,
        0xcc, 0x22, 0x4e, 0x4d, 0x5e, 0xb8, 0x80, 0xcb,
        0xd3, 0xda, 0x20, 0x29, 0x70, 0x02, 0xb7, 0xd1,
        0xb7, 0xc4
    ])

    flag = bytearray(42)
    v6 = 0
    
    for i in range(42):
        # 模擬組合語言邏輯
        v8 = v6 ^ a3_data[i]
        v6 = (v6 + 31) & 0xFFFFFFFF # 模擬32位元無號整數加法
        v9 = (i >> 1) ^ v8 ^ 0x5A
        
        # mov [rax], cl 只移動最低位元組
        flag[i] = v9 & 0xFF

    print(f"[*] Flag: {flag.decode('utf-8')}")

if __name__ == "__main__":
    solve()
````

#### **4. 獲取 Flag**

執行上述 Python 腳本，即可得到最終的 Flag。

```shell
$ python solve.py
[*] Flag: idek{he4rd_0f_constructors?_now_you_d1d!!}
```

#### **結論**

這個挑戰的核心教訓是：**不要完全相信原始碼或反編譯器的輸出**。它們是很好的起點，但組合語言才是程式行為的最終權威。

好的，這是一份針對此 CTF 題目的中文 Writeup。

---

### gacha-gate 

#### **題目分析**

這是一道 Misc 類型的題目。連接到遠端伺服器後，它會要求玩一個遊戲。遊戲內容與規則如下：
```shell
$ nc gacha-gate.chal.idek.team 1337
== proof-of-work: disabled ==
lets play a game!
((((iiiliIIlll & (~((IiIlIllIll | (~(~(1132183904 | iIllIIillI)))) | 1390768878))) & (((((((liilIliiil ^ liIllIiliI) | (~1601386568)) ^ ((~2099381271) ^ (2161683113 & 995798985))) & (~((~2945857245) & (~IIIliIIlll)))) & 1288124991) | (~((iIiiIIIiiI ^ ((3501406885 & IlIlllIIIi) & IlIiiliIiI)) & ((~1695738124) | (1570513183 & (~956341317)))))) | IllilIiIll)) | (((1770909462 & (((~(4126174599 ^ (1676452920 & 3107131465))) & (~(~(illIiiiIIi | illIIIlIll)))) | ((((IiiIiliIII ^ 3637989455) & (iIilIIiliI ^ 1829704191)) & ((2563237554 & illiliIlil) & (lIlllliiIl | 4091089967))) & IliIiIilIi))) ^ (((~(((2109240680 ^ IlllIIliIl) | IiiiIlilii) & (~(1799678504 ^ liiIiiIIll)))) | (~(~((2440472658 & iiiilIliiI) ^ lIiIIilIli)))) ^ (((((IIiiiIIill & 3913500427) | (liIlIiiiil ^ 2081685968)) | IiIlllilII) & (2533036646 ^ ((2309295369 ^ 3508589138) ^ (~lIiliIIliI)))) | ((((1475639815 | lliiIIilli) ^ (illilillII | IIIiIlIiIl)) & ((3994550317 & liIIliIlll) & (liIiIlIIli ^ 382928417))) & ((~1774792488) & ((1673281464 ^ llllIiIIlI) | (iIlIliiIii ^ IiilillIIi))))))) & (~lIllIIllil))) ^ ((~(((((~(~(IIIillliiI & IIlIilllll))) & (~1023425418)) & ((((174332488 | iIiiiIIiii) | (iiIillIlli & lliIIlIIli)) | ((2713375997 | 2413166229) ^ iilliiiiIi)) | 3179235054)) ^ ((lIIiIiIIii | ((~598600916) | ((IlIliIiIii | IiIlilliII) ^ (~3759091976)))) | ((~(~(liIliIlIlI ^ 3844785463))) ^ (~((~365937032) | (1538737890 ^ liIliilIlI)))))) & (1872100136 & IiiiiIlllI))) | (~(((((((~861427195) ^ (2918173908 ^ iilIiiliil)) ^ ((lllIlIiIlI & IIlliIIlil) | (iiiIIlIlIl | 3219018794))) ^ ((~(IlIiIiiIll | IliiiiillI)) & (~(illiiiIiil ^ 1431193537)))) | ((~2890447430) ^ liiilliliI)) & (~208761875)) & ((~2021829566) ^ (((((iiliiiiiil | IIlIIIlIIi) & (illillilil | 629457641)) ^ 2748695811) & (((1882401206 | IllIIllIlI) & (~2260514738)) | ((IiliiiIiiI & lililIIIIi) & (iliilliiii ^ iIIiIIiIll)))) ^ (((~(1570491372 & 3882850081)) ^ ((~2420135640) & (IiiilIilII ^ lillIIIIIi))) & (((IIiiiIIlil ^ 574879500) ^ (2253108475 | iilIiiilIi)) | ((~lliIilIlll) ^ (1123928645 | 535672692))))))))))
too slow!
```
1.  伺服器會傳送一個非常長的、使用中序表示法 (Infix Notation) 的位元運算式。
2.  我們必須在 5 秒的時限內，將這個運算式轉換成等價的逆波蘭表示法 (Reverse Polish Notation, RPN)，並回傳給伺服器。
3.  這個過程需要成功重複 50 次，才能獲得 Flag。
`server.py` :
```python
#!/usr/bin/env python3
import contextlib
import os
import random
import re
import signal
import sys

from z3 import ArithRef, BitVec, BitVecRef, BitVecVal, Solver, simplify, unsat

WIDTH = 32
OPS = ['~', '&', '^', '|']
MAX_DEPTH = 10
FLAG = os.getenv('FLAG', 'idek{fake_flag}')
VARS = set('iIl')


def rnd_const() -> tuple[str, BitVecRef]:
    v = random.getrandbits(WIDTH)
    return str(v), BitVecVal(v, WIDTH)


def rnd_var() -> tuple[str, BitVecRef]:
    name = ''.join(random.choices(tuple(VARS), k=10))
    return name, BitVec(name, WIDTH)


def combine(
    op: str,
    left: tuple[str, BitVecRef],
    right: tuple[str, BitVecRef] | None = None,
) -> tuple[str, ArithRef]:
    if op == '~':
        s_left, z_left = left
        return f'(~{s_left})', ~z_left
    s_l, z_l = left
    s_r, z_r = right
    return f'({s_l} {op} {s_r})', {
        '&': z_l & z_r,
        '^': z_l ^ z_r,
        '|': z_l | z_r,
    }[op]


def random_expr(depth: int = 0) -> tuple[str, ArithRef]:
    if depth >= MAX_DEPTH or random.random() < 0.1:
        return random.choice((rnd_var, rnd_const))()
    op = random.choice(OPS)
    if op == '~':
        return combine(op, random_expr(depth + 1))
    return combine(op, random_expr(depth + 1), random_expr(depth + 1))


TOKEN_RE = re.compile(r'[0-9]+|[iIl]+|[~&^|]')


def parse_rpn(s: str) -> ArithRef:
    tokens = TOKEN_RE.findall(s)
    if not tokens:
        raise ValueError('empty input')

    var_cache: dict[str, BitVecRef] = {}
    stack: list[BitVecRef] = []

    for t in tokens:
        if t.isdigit():
            stack.append(BitVecVal(int(t), WIDTH))
        elif re.fullmatch(r'[iIl]+', t):
            if t not in var_cache:
                var_cache[t] = BitVec(t, WIDTH)
            stack.append(var_cache[t])
        elif t in OPS:
            if t == '~':
                if len(stack) < 1:
                    raise ValueError('stack underflow')
                a = stack.pop()
                stack.append(~a)
            else:
                if len(stack) < 2:
                    raise ValueError('stack underflow')
                b = stack.pop()
                a = stack.pop()
                stack.append({'&': a & b, '^': a ^ b, '|': a | b}[t])
        else:
            raise ValueError(f'bad token {t}')

    if len(stack) != 1:
        raise ValueError('malformed expression')
    return stack[0]


def equivalent(e1: ArithRef, e2: ArithRef) -> tuple[bool, Solver]:
    s = Solver()
    s.set(timeout=5000)
    s.add(simplify(e1) != simplify(e2))
    return s.check() == unsat, s


def _timeout_handler(_: int, __) -> None:
    raise TimeoutError


def main() -> None:
    signal.signal(signal.SIGALRM, _timeout_handler)
    print('lets play a game!')

    for _ in range(50):
        random.seed()
        expr_str, expr_z3 = random_expr()
        print(expr_str, flush=True)

        signal.alarm(5)
        try:
            line = sys.stdin.readline()
            signal.alarm(0)
        except TimeoutError:
            print('too slow!')
            return

        try:
            rpn_z3 = parse_rpn(line.strip())
        except Exception as e:
            print('invalid input:', e)
            return

        print('let me see..')
        is_eq, s = equivalent(expr_z3, rpn_z3)
        if not is_eq:
            print('wrong!')
            with contextlib.suppress(BaseException):
                print('counter example:', s.model())
            return

    print(FLAG)


if __name__ == '__main__':
    main()
```
從 `server.py` 原始碼中可以看到，運算式包含了 32 位元的整數、由 `i`, `I`, `l` 組成的變數，以及 `~` (NOT), `&` (AND), `^` (XOR), `|` (OR) 這四種位元運算子。由於運算式結構複雜且有嚴格的時間限制，手動轉換是不可能的，必須撰寫自動化腳本來解決。

#### **解題思路**

解題的核心是將中序運算式轉換為逆波蘭表示法。這個問題最經典的演算法是 **調度場演算法 (Shunting-yard algorithm)**。

調度場演算法的原理是：

1.  遍歷中序運算式的每一個 token（數字、變數、運算子、括號）。
2.  使用一個佇列 (queue) 作為輸出，以及一個堆疊 (stack) 來暫存運算子。
3.  **數字或變數**：直接放入輸出佇列。
4.  **運算子**：與堆疊頂端的運算子比較優先級。如果當前運算子的優先級小於或等於堆疊頂端的運算子，就將堆疊頂端的運算子彈出至輸出佇列，直到條件不滿足為止，最後再將當前運算子推入堆疊。
5.  **左括號 `(`**：直接推入運算子堆疊。
6.  **右括號 `)`**：將運算子堆疊中的運算子不斷彈出至輸出佇列，直到遇到左括號為止。最後將左括號從堆疊中彈出並丟棄。
7.  遍歷完所有 token 後，將堆疊中剩餘的所有運算子依次彈出至輸出佇列。

最終，輸出佇列中的內容就是轉換後的 RPN 運算式。

#### **程式碼實現**

我們使用 Python 的 `pwntools` 函式庫來處理與伺服器的網路通訊，並實現上述的調度場演算法。

```python
import re
import sys
from pwn import *

# 中序 (infix) 轉 逆波蘭表示法 (RPN) 的 Shunting-yard 演算法
def to_rpn(infix_str: str) -> str:
    # 運算子優先級
    precedence = {'~': 3, '&': 2, '^': 1, '|': 0}
    # ~ 是右結合，其他是左結合
    associativity = {'~': 'right', '&': 'left', '^': 'left', '|': 'left'}
    
    output_queue = []
    operator_stack = []

    # 使用正則表達式來正確分割 token
    # 匹配數字、變數(由i,I,l組成)、運算子或括號
    tokens = re.findall(r'[0-9]+|[iIl]+|[~&^|()]', infix_str)

    for token in tokens:
        if token.isalnum():  # 數字或變數
            output_queue.append(token)
        elif token in precedence:  # 運算子
            while (operator_stack and operator_stack[-1] != '(' and
                   (precedence[operator_stack[-1]] > precedence[token] or
                    (precedence[operator_stack[-1]] == precedence[token] and associativity[token] == 'left'))):
                output_queue.append(operator_stack.pop())
            operator_stack.append(token)
        elif token == '(':
            operator_stack.append(token)
        elif token == ')':
            while operator_stack and operator_stack[-1] != '(':
                output_queue.append(operator_stack.pop())
            if operator_stack and operator_stack[-1] == '(':
                operator_stack.pop() # 彈出左括號
    
    # 將剩餘的運算子從堆疊中彈出
    while operator_stack:
        output_queue.append(operator_stack.pop())
        
    return ' '.join(output_queue)

# --- 主程式 ---
# 替換成你的遠端主機和端口
conn = remote('gacha-gate.chal.idek.team', 1337)
# 如果在本機測試，請使用 process
# conn = process(['python', 'server.py'])


conn.recvuntil(b'lets play a game!\n')

for i in range(50):
    try:
        # 接收一行，直到換行符
        line = conn.recvline().decode().strip()
        print(f"[*] Round {i+1}/50")
        print(f"  [-] Received: {line}")

        if not line:
            print("[!] Received empty line, exiting.")
            break

        # 轉換為 RPN
        rpn_expr = to_rpn(line)
        print(f"  [+] Sending RPN: {rpn_expr}")

        # 發送 RPN 並加上換行符
        conn.sendline(rpn_expr.encode())
        
        # 接收伺服器的回應
        response = conn.recvline().decode().strip()
        if 'wrong!' in response or 'invalid' in response:
            print(f"[!] Error: {response}")
            print(conn.recvall().decode())
            break

    except EOFError:
        print("[!] Connection closed by server.")
        break

# 接收最後的 FLAG
flag = conn.recvall().decode()
print(f"\n[+] FLAG: {flag.strip()}")

conn.close()
```

#### **取得 Flag**

執行上述腳本，它會自動完成 50 回合的問答，最終伺服器會回傳 Flag。

```shell
...
[+] Receiving all data: Done (37B)
[*] Closed connection to gacha-gate.chal.idek.team port 1337

[+] FLAG: idek{n4nds_r_funct10nally_c0mpl3t3!}
```

成功取得 Flag：`idek{n4nds_r_funct10nally_c0mpl3t3!}`
