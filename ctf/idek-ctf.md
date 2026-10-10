### constructor

The key to this challenge is identifying decompiler mistakes and going back to raw assembly to understand the program's real behavior.

#### **1. Initial Analysis**

The challenge provides a C++ source file `constructor.cpp` and a compiled executable `constructor`.

First, inspect `constructor.cpp`:

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

The key detail is `__attribute__((constructor))`, meaning `solve()` is called before `main()`.

`solve()` appears to do the following:
1.  Read 42 random bytes from `/dev/urandom` into `flag`.
2.  Apply XOR and addition operations to those 42 bytes.

If this were true behavior, the flag would be random and unsolvable. This suggests the source is misleading, and the real logic is in the compiled binary.

#### **2. Diving into Assembly**

Use IDA to analyze `constructor` and locate `solve`.

* Decompiler Pitfall

    IDA's decompilation may resemble the source but shows odd behavior, like operating on an uninitialized array, further confirming decompiler misunderstanding.

* Assembly Is the Truth

    You must read assembly directly to determine what really happens. Core loop of `solve`:

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

    The key finding is line `[1]`: `movzx eax, byte ptr [rdx+rax]`. Before the loop, `rdx` is set to a memory address:

    ```nasm
    .text:00000000004011C7                 mov     rdx, cs:a3_data ; "3!..."
    ```
    `a3_data` points to `.rodata` at `0x403040`, meaning the program is **not** reading `/dev/urandom`; it uses a hardcoded 42-byte array in the binary.

#### **3. Algorithm Reconstruction and Solve**

Extract the 42 bytes from `.rodata` address `0x403040`.

Then, based on assembly logic, write a Python script to simulate decryption:

1.  Initialize `v6 = 0`.
2.  Iterate over 42 bytes with index `i`.
3.  `v8 = v6 ^ a3_data[i]`
4.  `v6 = v6 + 31` (keep within 32-bit range).
5.  `result = (i >> 1) ^ v8 ^ 0x5A`
6.  Store the lowest byte of `result` (`result & 0xFF`) as flag character `i`.

**Solver Script :**

````python
def solve():
    # 42-byte data extracted from .rodata:0x403040
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
        # Simulate assembly logic
        v8 = v6 ^ a3_data[i]
        v6 = (v6 + 31) & 0xFFFFFFFF # simulate 32-bit unsigned addition
        v9 = (i >> 1) ^ v8 ^ 0x5A
        
        # mov [rax], cl writes only the lowest byte
        flag[i] = v9 & 0xFF

    print(f"[*] Flag: {flag.decode('utf-8')}")

if __name__ == "__main__":
    solve()
````

#### **4. Retrieve the Flag**

Run the script above to obtain the final flag.

```shell
$ python solve.py
[*] Flag: idek{he4rd_0f_constructors?_now_you_d1d!!}
```

#### **Conclusion**

Core lesson: **do not fully trust source code or decompiler output**. They are good starting points, but assembly is the final authority on behavior.

Below is a write-up for this CTF challenge.

---

### gacha-gate 

#### **Challenge Analysis**

This is a Misc challenge. After connecting to the remote server, it asks you to play a game with these rules:
```shell
$ nc gacha-gate.chal.idek.team 1337
== proof-of-work: disabled ==
lets play a game!
((((iiiliIIlll & (~((IiIlIllIll | (~(~(1132183904 | iIllIIillI)))) | 1390768878))) & (((((((liilIliiil ^ liIllIiliI) | (~1601386568)) ^ ((~2099381271) ^ (2161683113 & 995798985))) & (~((~2945857245) & (~IIIliIIlll)))) & 1288124991) | (~((iIiiIIIiiI ^ ((3501406885 & IlIlllIIIi) & IlIiiliIiI)) & ((~1695738124) | (1570513183 & (~956341317)))))) | IllilIiIll)) | (((1770909462 & (((~(4126174599 ^ (1676452920 & 3107131465))) & (~(~(illIiiiIIi | illIIIlIll)))) | ((((IiiIiliIII ^ 3637989455) & (iIilIIiliI ^ 1829704191)) & ((2563237554 & illiliIlil) & (lIlllliiIl | 4091089967))) & IliIiIilIi))) ^ (((~(((2109240680 ^ IlllIIliIl) | IiiiIlilii) & (~(1799678504 ^ liiIiiIIll)))) | (~(~((2440472658 & iiiilIliiI) ^ lIiIIilIli)))) ^ (((((IIiiiIIill & 3913500427) | (liIlIiiiil ^ 2081685968)) | IiIlllilII) & (2533036646 ^ ((2309295369 ^ 3508589138) ^ (~lIiliIIliI)))) | ((((1475639815 | lliiIIilli) ^ (illilillII | IIIiIlIiIl)) & ((3994550317 & liIIliIlll) & (liIiIlIIli ^ 382928417))) & ((~1774792488) & ((1673281464 ^ llllIiIIlI) | (iIlIliiIii ^ IiilillIIi))))))) & (~lIllIIllil))) ^ ((~(((((~(~(IIIillliiI & IIlIilllll))) & (~1023425418)) & ((((174332488 | iIiiiIIiii) | (iiIillIlli & lliIIlIIli)) | ((2713375997 | 2413166229) ^ iilliiiiIi)) | 3179235054)) ^ ((lIIiIiIIii | ((~598600916) | ((IlIliIiIii | IiIlilliII) ^ (~3759091976)))) | ((~(~(liIliIlIlI ^ 3844785463))) ^ (~((~365937032) | (1538737890 ^ liIliilIlI)))))) & (1872100136 & IiiiiIlllI))) | (~(((((((~861427195) ^ (2918173908 ^ iilIiiliil)) ^ ((lllIlIiIlI & IIlliIIlil) | (iiiIIlIlIl | 3219018794))) ^ ((~(IlIiIiiIll | IliiiiillI)) & (~(illiiiIiil ^ 1431193537)))) | ((~2890447430) ^ liiilliliI)) & (~208761875)) & ((~2021829566) ^ (((((iiliiiiiil | IIlIIIlIIi) & (illillilil | 629457641)) ^ 2748695811) & (((1882401206 | IllIIllIlI) & (~2260514738)) | ((IiliiiIiiI & lililIIIIi) & (iliilliiii ^ iIIiIIiIll)))) ^ (((~(1570491372 & 3882850081)) ^ ((~2420135640) & (IiiilIilII ^ lillIIIIIi))) & (((IIiiiIIlil ^ 574879500) ^ (2253108475 | iilIiiilIi)) | ((~lliIilIlll) ^ (1123928645 | 535672692))))))))))
too slow!
```
1.  The server sends a very long bitwise expression in infix notation.
2.  Within 5 seconds, we must convert it to equivalent Reverse Polish Notation (RPN) and send it back.
3.  This must be done successfully 50 times to get the flag.
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
From `server.py`, expressions include 32-bit integers, variables made of `i`, `I`, `l`, and operators `~`, `&`, `^`, `|`. Because expressions are complex and time is strict, manual conversion is impractical; automation is required.

#### **Approach**

The core is converting infix expressions to RPN. The classic algorithm is the **Shunting-yard algorithm**.

Shunting-yard algorithm principles:

1.  Iterate through each infix token (numbers, variables, operators, parentheses).
2.  Use an output queue and an operator stack.
3.  **Number/variable**: push directly to output queue.
4.  **Operator**: compare precedence with the stack top. If current precedence is lower/equal (for left-associative case), pop from stack to output until condition fails, then push current operator.
5.  **Left parenthesis `(`**: push onto operator stack.
6.  **Right parenthesis `)`**: pop operators to output until left parenthesis appears, then discard it.
7.  After all tokens, pop remaining operators to output queue.

Finally, the output queue is the converted RPN expression.

#### **Implementation**

We use Python `pwntools` for network interaction and implement the above Shunting-yard logic.

```python
import re
import sys
from pwn import *

# Shunting-yard algorithm: infix to RPN
def to_rpn(infix_str: str) -> str:
    # Operator precedence
    precedence = {'~': 3, '&': 2, '^': 1, '|': 0}
    # ~ is right-associative; others are left-associative
    associativity = {'~': 'right', '&': 'left', '^': 'left', '|': 'left'}
    
    output_queue = []
    operator_stack = []

    # Use regex to tokenize correctly
    # Match numbers, variables (i/I/l), operators, or parentheses
    tokens = re.findall(r'[0-9]+|[iIl]+|[~&^|()]', infix_str)

    for token in tokens:
        if token.isalnum():  # number or variable
            output_queue.append(token)
        elif token in precedence:  # operator
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
                operator_stack.pop() # pop left parenthesis
    
    # Pop remaining operators from the stack
    while operator_stack:
        output_queue.append(operator_stack.pop())
        
    return ' '.join(output_queue)

# --- Main ---
# Replace with your remote host and port
conn = remote('gacha-gate.chal.idek.team', 1337)
# For local testing, use process
# conn = process(['python', 'server.py'])


conn.recvuntil(b'lets play a game!\n')

for i in range(50):
    try:
        # Receive one line up to newline
        line = conn.recvline().decode().strip()
        print(f"[*] Round {i+1}/50")
        print(f"  [-] Received: {line}")

        if not line:
            print("[!] Received empty line, exiting.")
            break

        # Convert to RPN
        rpn_expr = to_rpn(line)
        print(f"  [+] Sending RPN: {rpn_expr}")

        # Send RPN with newline
        conn.sendline(rpn_expr.encode())
        
        # Receive server response
        response = conn.recvline().decode().strip()
        if 'wrong!' in response or 'invalid' in response:
            print(f"[!] Error: {response}")
            print(conn.recvall().decode())
            break

    except EOFError:
        print("[!] Connection closed by server.")
        break

# Receive final FLAG
flag = conn.recvall().decode()
print(f"\n[+] FLAG: {flag.strip()}")

conn.close()
```

#### **Getting the Flag**

Run the script to automatically complete 50 rounds; the server then returns the flag.

```shell
...
[+] Receiving all data: Done (37B)
[*] Closed connection to gacha-gate.chal.idek.team port 1337

[+] FLAG: idek{n4nds_r_funct10nally_c0mpl3t3!}
```

Successfully obtained flag: `idek{n4nds_r_funct10nally_c0mpl3t3!}`
