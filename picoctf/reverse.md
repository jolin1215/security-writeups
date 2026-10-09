# Easy
## Flag Hunters
1. 先看題目給的 lyric-reader.py
    * flag 的載入 : 腳本的開頭讀取一個名為 `flag.txt` 的檔案，並將其內容加到 `secret_intro` 字串中。這個 `secret_intro` 接著被加到 `song_flag_hunters` 歌詞字串的最前面。
    * 這表示旗標位於 song_flag_hunters 歌詞的最開頭。當歌詞被 `song.splitlines()` 分割成行時，旗標會位於 song_lines 列表的前幾行 (索引 0 到 4)。
2. 主要邏輯與漏洞 : `reader` 函式是程式的核心。它逐行讀取歌詞並根據內容執行不同操作。其中最關鍵的是處理使用者輸入的部分。這裡存在一個典型的指令注入 (Command Injection) 漏洞，當程式遇到包含 CROWD 的歌詞行時，它會等待使用者輸入。使用者的輸入 `crowd` 會被拼接到 ``"Crowd: "`` 字串後面，然後覆寫 `song_lines` 中當前行的內容。在下一次迴圈中，程式會處理這個被修改過的行。處理行的時候，程式會使用 `split(';')` 來分割指令。這意味著如果輸入包含分號 `;`，就可以注入一個新的指令。
    ```
    import re
    import time

    # Read in flag from file
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

      # Get list of lyric lines
      song_lines = song.splitlines()

      # Find startLabel, refrain and refrain return
      for i in range(0, len(song_lines)):
        if song_lines[i] == startLabel:
          start = i + 1
        elif song_lines[i] == '[REFRAIN]':
          refrain = i + 1
        elif song_lines[i] == 'RETURN':
          refrain_return = i

      # Print lyrics
      line_count = 0
      lip = start
      while not finished and line_count < MAX_LINES:
        line_count += 1
        for line in song_lines[lip].split(';'): # 注意這裡的 split(';')
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
**Flag:  picoCTF{70637h3r_f0r3v3r_75053bc3}**

## Transformation
1. 下載題目給的檔案看看
    ```
    $ cat enc
    灩捯䍔䙻ㄶ形楴獟楮獴㌴摟潦弸彥ㄴㅡて㝽
    $ file enc
    enc: Unicode text, UTF-8 text, with no line terminators
    ```
2. 題目提到的加密的 Python 程式碼：
    ```
    ''.join([chr((ord(flag[i]) << 8) + ord(flag[i + 1])) for i in range(0, len(flag), 2)])
    ```
>這段程式碼的邏輯是：
(1) 它從 flag 字串中一次讀取兩個字元（flag[i] 和 flag[i+1]）。
(2) 它取得第一個字元的 ASCII/Unicode 碼 ord(flag[i])，並將其向左移動 8 個位元（相當於乘以 256）。
(3) 然後它加上第二個字元的 ASCII/Unicode 碼 ord(flag[i+1])。
這樣就將兩個 8 位元的字元合併成一個 16 位元的整數。
(4) 最後，chr() 函式將這個 16 位元的整數轉換成一個新的（通常是更寬的）Unicode 字元。
>
3. 解密邏輯
    * 要反轉這個過程，我們需要遍歷加密字串中的每一個 Unicode 字元。
    * 使用 ord() 取得每一個字元的 16 位元數值。
    * 第一個原始字元的數值可以透過將 16 位元數值向右移動 8 個位元（>> 8）來取得。
    * 第二個原始字元的數值可以透過將 16 位元數值與 0xFF（也就是 255）進行位元 AND 運算（& 0xFF）來取得，這會分離出較低的 8 個位元。
    * 將這兩個數值分別用 chr() 轉換回字元。
    * 將所有還原的字元組合起來，就是原始的 flag。

    solve.py：
    ```
    def solve():
      """
      Decodes the string based on the provided encryption logic.
      """
      encoded_string = "灩捯䍔䙻ㄶ形楴獟楮獴㌴摟潦弸彥ㄴㅡて㝽"
      decoded_flag = ""

      for char in encoded_string:
        # Get the 16-bit Unicode value
        unicode_val = ord(char)

        # Extract the high byte (first character)
        char1 = chr(unicode_val >> 8)

        # Extract the low byte (second character)
        char2 = chr(unicode_val & 0xFF)

        # Append to the result
        decoded_flag += char1
        decoded_flag += char2

      print(decoded_flag)

    solve()
    ```
**Flag: picoCTF{16_bits_inst34d_of_8_e141a0f7}**

# Medium
## packer
*   **描述** : Reverse this linux executable?
*   **提示** : What can we do to reduce the size of a binary after compiling it.
1. 分析檔案
```
$ file out
out: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, no section header
```
>ELF 64-bit : 標準的 64 位元 Linux 執行檔。
statically linked : 所有函式庫都被編譯進檔案中。
no section header : 最重要的線索。區段頭被移除
>
2. 編譯後減小執行檔大小最常見、最有效的方法是使用封裝器 (Packer)，例如 UPX (Ultimate Packer for eXecutables)
3. 下載工具
````
$ sudo apt install upx-ucl
````
4. 用 `-d` 解壓縮檔案
````
$ upx -d out
````
5. 直接執行檔案，程式會要求輸入密碼，輸入任意字串後提示存取被拒
````
$ ./out
Enter the password to unlock this file: test
You entered: test
Access denied
````

6. **分析 `main` 函式**

    現在檔案已經乾淨，我們使用 GDB 來找到並分析其 `main` 函式。

    *   **找到入口點**: 使用 `readelf -h out` 找到解壓縮後的入口點位址 `0x401c40`。
    ```
    $ readelf -h out
    ELF Header:
      Magic:   7f 45 4c 46 02 01 01 03 00 00 00 00 00 00 00 00
      Class:                             ELF64
      Data:                              2's complement, little endian
      Version:                           1 (current)
      OS/ABI:                            UNIX - GNU
      ABI Version:                       0
      Type:                              EXEC (Executable file)
      Machine:                           Advanced Micro Devices X86-64
      Version:                           0x1
      Entry point address:               0x401c40
      Start of program headers:          64 (bytes into file)
      Start of section headers:          870040 (bytes into file)
      Flags:                             0x0
      Size of this header:               64 (bytes)
      Size of program headers:           56 (bytes)
      Number of program headers:         10
      Size of section headers:           64 (bytes)
      Number of section headers:         32
      Section header string table index: 31
    ```
    * 啟用 GDB
    ```
    $ gdb out
    ```
    *   **找到 `main`**: 在 GDB 中反組譯入口點 `disas 0x401c40`，發現它在呼叫 `__libc_start_main` 之前，將 `main` 函式的位址 `0x401d65` 載入了 `%rdi` 暫存器。
    ```
    (gdb) disas 0x401c40
    Dump of assembler code for function _start:
       0x0000000000401c40 <+0>:     endbr64
       0x0000000000401c44 <+4>:     xor    %ebp,%ebp
       0x0000000000401c46 <+6>:     mov    %rdx,%r9
       0x0000000000401c49 <+9>:     pop    %rsi
       0x0000000000401c4a <+10>:    mov    %rsp,%rdx
       0x0000000000401c4d <+13>:    and    $0xfffffffffffffff0,%rsp
       0x0000000000401c51 <+17>:    push   %rax
       0x0000000000401c52 <+18>:    push   %rsp
       0x0000000000401c53 <+19>:    mov    $0x403020,%r8
       0x0000000000401c5a <+26>:    mov    $0x402f80,%rcx
       0x0000000000401c61 <+33>:    mov    $0x401d65,%rdi
       0x0000000000401c68 <+40>:    addr32 call 0x402320 <__libc_start_main>
       0x0000000000401c6e <+46>:    hlt
    End of assembler dump.
    ```
    *   **反組譯 `main`**: 執行 `disas 0x401d65`，我們得到了程式的核心邏輯。
    ```
    (gdb) disas 0x401d65
    Dump of assembler code for function main:
       0x0000000000401d65 <+0>:     endbr64
       0x0000000000401d69 <+4>:     push   %rbp
       0x0000000000401d6a <+5>:     mov    %rsp,%rbp
       0x0000000000401d6d <+8>:     push   %rbx
       0x0000000000401d6e <+9>:     sub    $0x98,%rsp
       0x0000000000401d75 <+16>:    mov    %fs:0x28,%rax
       0x0000000000401d7e <+25>:    mov    %rax,-0x18(%rbp)
       0x0000000000401d82 <+29>:    xor    %eax,%eax
       0x0000000000401d84 <+31>:    mov    %rsp,%rax
       0x0000000000401d87 <+34>:    mov    %rax,%rbx
       0x0000000000401d8a <+37>:    movq   $0x64,-0x98(%rbp)
       0x0000000000401d95 <+48>:    mov    -0x98(%rbp),%rax
       0x0000000000401d9c <+55>:    mov    %rax,%rdx
       0x0000000000401d9f <+58>:    sub    $0x1,%rdx
       0x0000000000401da3 <+62>:    mov    %rdx,-0x90(%rbp)
       0x0000000000401daa <+69>:    mov    %rax,%r8
       0x0000000000401dad <+72>:    mov    $0x0,%r9d
       0x0000000000401db3 <+78>:    mov    %rax,%rsi
       0x0000000000401db6 <+81>:    mov    $0x0,%edi
       0x0000000000401dbb <+86>:    mov    $0x10,%edx
       0x0000000000401dc0 <+91>:    sub    $0x1,%rdx
       0x0000000000401dc4 <+95>:    add    %rdx,%rax
       0x0000000000401dc7 <+98>:    mov    $0x10,%esi
       0x0000000000401dcc <+103>:   mov    $0x0,%edx
       0x0000000000401dd1 <+108>:   div    %rsi
       0x0000000000401dd4 <+111>:   imul   $0x10,%rax,%rax
       0x0000000000401dd8 <+115>:   mov    %rax,%rdx
       0x0000000000401ddb <+118>:   and    $0xfffffffffffff000,%rdx
       0x0000000000401de2 <+125>:   mov    %rsp,%rcx
       0x0000000000401de5 <+128>:   sub    %rdx,%rcx
       0x0000000000401de8 <+131>:   mov    %rcx,%rdx
       0x0000000000401deb <+134>:   cmp    %rdx,%rsp
    --Type <RET> for more, q to quit, c to continue without paging--c
       0x0000000000401dee <+137>:   je     0x401e02 <main+157>
       0x0000000000401df0 <+139>:   sub    $0x1000,%rsp
       0x0000000000401df7 <+146>:   orq    $0x0,0xff8(%rsp)
       0x0000000000401e00 <+155>:   jmp    0x401deb <main+134>
       0x0000000000401e02 <+157>:   mov    %rax,%rdx
       0x0000000000401e05 <+160>:   and    $0xfff,%edx
       0x0000000000401e0b <+166>:   sub    %rdx,%rsp
       0x0000000000401e0e <+169>:   mov    %rax,%rdx
       0x0000000000401e11 <+172>:   and    $0xfff,%edx
       0x0000000000401e17 <+178>:   test   %rdx,%rdx
       0x0000000000401e1a <+181>:   je     0x401e2c <main+199>
       0x0000000000401e1c <+183>:   and    $0xfff,%eax
       0x0000000000401e21 <+188>:   sub    $0x8,%rax
       0x0000000000401e25 <+192>:   add    %rsp,%rax
       0x0000000000401e28 <+195>:   orq    $0x0,(%rax)
       0x0000000000401e2c <+199>:   mov    %rsp,%rax
       0x0000000000401e2f <+202>:   add    $0x0,%rax
       0x0000000000401e33 <+206>:   mov    %rax,-0x88(%rbp)
       0x0000000000401e3a <+213>:   movabs $0x6636333639363037,%rax
       0x0000000000401e44 <+223>:   movabs $0x6237363434353334,%rdx
       0x0000000000401e4e <+233>:   mov    %rax,-0x80(%rbp)
       0x0000000000401e52 <+237>:   mov    %rdx,-0x78(%rbp)
       0x0000000000401e56 <+241>:   movabs $0x6635383539333535,%rax
       0x0000000000401e60 <+251>:   movabs $0x3433303565363535,%rdx
       0x0000000000401e6a <+261>:   mov    %rax,-0x70(%rbp)
       0x0000000000401e6e <+265>:   mov    %rdx,-0x68(%rbp)
       0x0000000000401e72 <+269>:   movabs $0x6534313362363336,%rax
       0x0000000000401e7c <+279>:   movabs $0x3133323466353633,%rdx
       0x0000000000401e86 <+289>:   mov    %rax,-0x60(%rbp)
       0x0000000000401e8a <+293>:   mov    %rdx,-0x58(%rbp)
       0x0000000000401e8e <+297>:   movabs $0x3936323534336536,%rax
       0x0000000000401e98 <+307>:   movabs $0x3633663533353333,%rdx
       0x0000000000401ea2 <+317>:   mov    %rax,-0x50(%rbp)
       0x0000000000401ea6 <+321>:   mov    %rdx,-0x48(%rbp)
       0x0000000000401eaa <+325>:   movabs $0x3633393336363636,%rax
       0x0000000000401eb4 <+335>:   movabs $0x6437363635363433,%rdx
       0x0000000000401ebe <+345>:   mov    %rax,-0x40(%rbp)
       0x0000000000401ec2 <+349>:   mov    %rdx,-0x38(%rbp)
       0x0000000000401ec6 <+353>:   movq   $0x0,-0x30(%rbp)
       0x0000000000401ece <+361>:   movq   $0x0,-0x28(%rbp)
       0x0000000000401ed6 <+369>:   movl   $0x0,-0x20(%rbp)
       0x0000000000401edd <+376>:   lea    0x93124(%rip),%rdi        # 0x495008
       0x0000000000401ee4 <+383>:   mov    $0x0,%eax
       0x0000000000401ee9 <+388>:   call   0x410c00 <printf>
       0x0000000000401eee <+393>:   mov    0xbe7e3(%rip),%rdx        # 0x4c06d8 <stdin>
       0x0000000000401ef5 <+400>:   mov    -0x98(%rbp),%rax
       0x0000000000401efc <+407>:   mov    %eax,%ecx
       0x0000000000401efe <+409>:   mov    -0x88(%rbp),%rax
       0x0000000000401f05 <+416>:   mov    %ecx,%esi
       0x0000000000401f07 <+418>:   mov    %rax,%rdi
       0x0000000000401f0a <+421>:   call   0x418530 <fgets>
       0x0000000000401f0f <+426>:   mov    -0x88(%rbp),%rax
       0x0000000000401f16 <+433>:   mov    %rax,%rsi
       0x0000000000401f19 <+436>:   lea    0x93111(%rip),%rdi        # 0x495031
       0x0000000000401f20 <+443>:   mov    $0x0,%eax
       0x0000000000401f25 <+448>:   call   0x410c00 <printf>
       0x0000000000401f2a <+453>:   mov    -0x98(%rbp),%rdx
       0x0000000000401f31 <+460>:   lea    -0x80(%rbp),%rcx
       0x0000000000401f35 <+464>:   mov    -0x88(%rbp),%rax
       0x0000000000401f3c <+471>:   mov    %rcx,%rsi
       0x0000000000401f3f <+474>:   mov    %rax,%rdi
       0x0000000000401f42 <+477>:   call   0x4010d0
       0x0000000000401f47 <+482>:   test   %eax,%eax
       0x0000000000401f49 <+484>:   jne    0x401f65 <main+512>
       0x0000000000401f4b <+486>:   lea    0x930f6(%rip),%rdi        # 0x495048
       0x0000000000401f52 <+493>:   call   0x418c00 <puts>
       0x0000000000401f57 <+498>:   lea    -0x80(%rbp),%rax
       0x0000000000401f5b <+502>:   mov    %rax,%rdi
       0x0000000000401f5e <+505>:   call   0x418c00 <puts>
       0x0000000000401f63 <+510>:   jmp    0x401f71 <main+524>
       0x0000000000401f65 <+512>:   lea    0x93150(%rip),%rdi        # 0x4950bc
       0x0000000000401f6c <+519>:   call   0x418c00 <puts>
       0x0000000000401f71 <+524>:   mov    %rbx,%rsp
       0x0000000000401f74 <+527>:   mov    $0x0,%eax
       0x0000000000401f79 <+532>:   mov    -0x18(%rbp),%rbx
       0x0000000000401f7d <+536>:   xor    %fs:0x28,%rbx
       0x0000000000401f86 <+545>:   je     0x401f8d <main+552>
       0x0000000000401f88 <+547>:   call   0x454e20 <__stack_chk_fail_local>
       0x0000000000401f8d <+552>:   mov    -0x8(%rbp),%rbx
       0x0000000000401f91 <+556>:   leave
       0x0000000000401f92 <+557>:   ret
    End of assembler dump.
    ```

在 `main` 函式的組合語言中，我們發現了程式儲存和比較 flag 的完整過程。

7.  **Flag 的儲存**

    程式並未直接將 flag 字串儲存起來，而是將其拆分成多個 64 位元的十六進位數，並使用 `movabs` 指令將它們依序載入到堆疊中。這是一種反靜態分析的技巧。

    ```assembly
       0x0000000000401e3a:   movabs $0x6636333639363037,%rax
       0x0000000000401e44:   movabs $0x6237363434353334,%rdx
       ... (後續 movabs 指令)
    ```

8.  **Flag 的計算與組合**

    我們需要將這些十六進位數從記憶體中提取出來，並轉換為 ASCII 字元。由於 x86 架構是**小端序 (Little-Endian)**，讀取時需要將位元組反轉。

    | 位址 | 十六進位值 (Big-Endian) | 反轉後 (Little-Endian) | ASCII 字元 |
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

        將轉換後的 ASCII 字元拼接起來，得到最終的 Flag。

Flag: **picoCTF{U9X_UnP4ck1N6_B1n4Ri3S_6ff964ef}**

## Bit-O-Asm-1
* **描述** : 
Can you figure out what is in the eax register? Put your answer in the picoCTF flag format: picoCTF{n} where n is the contents of the eax register in the decimal number base. If the answer was 0x11 your flag would be picoCTF{17}.
1. 打開文檔
    ```
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
2. `<+15>: mov eax, 0x30` 將十六進位值 0x30 (十進位為 48) 放入 eax 暫存器。eax 通常用來存放函式的傳回值。所以這個函式準備要傳回 48。
* 如何分辨64位元與32位元的暫存器？透過下圖可以發現，最長的區塊是64位元的暫存器，它是 R 開頭。如果遇到是 E 開頭的名稱，就是32位元的暫存器。
![image](https://hackmd.io/_uploads/S1cumeUPxl.png)


