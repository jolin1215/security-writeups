# Easy
## CanYouSee
1. 解壓縮之後得到一圖片，用`exiftool`查看，看到一串 base64
![image](https://hackmd.io/_uploads/B1Fzc4JZgx.png)
2. cyberchef 解開
![image](https://hackmd.io/_uploads/Sy9Z5V1Wgx.png)
**Flag: picoCTF{ME74D47A_HIDD3N_4dabddcb}**

## Secret of the Polyglot
1. 拿到 pdf 後打開發現 flag 的屁股
![image](https://hackmd.io/_uploads/HJteZH1bgx.png)
2. 用`exiftool`查看，看到 file type 為 png
![image](https://hackmd.io/_uploads/BkeVlB1Wgl.png)
3. 把檔案轉為 png 看看發現 flag 的頭部
![image](https://hackmd.io/_uploads/ByKNZHJWxg.png)
**Flag: picoCTF{f1u3n7_1n_pn9_&_pdf_249d05c0}**

## Verify
1. ssh 連線進伺服器
![image](https://hackmd.io/_uploads/H1iIaSy-xg.png)
2. 找到檔案
`for f in files/*`: 針對 files 資料夾中的每一個檔案 f。
`sha256sum "$f"`: 計算該檔案的 SHA-256 雜湊值。
`done`: 結束 for 迴圈。
`| grep <雜湊值>`：從一堆檔案中，找出雜湊值（SHA-256）與 checksum.txt 一致的那一個檔案，這才是真正的 flag 檔案。
3. 最後執行`./decrypt.sh/<file>`解密
![image](https://hackmd.io/_uploads/H1_Rnryble.png)
**Flag: picoCTF{trust_but_verify_87590c24}**

# Medium
## shark on wire 1What Lies Within
1. 下載 buildings.png 後
![image](https://hackmd.io/_uploads/rk1BhTZZle.png)

* Hint: What are streams?
3. 下載完後用`strings`查看 pcap 檔案，發現沒有特別的
![image](https://hackmd.io/_uploads/Sk07O6-blg.png)
2. 看到 Hint 後，用 wireshark 打開並用搜尋欄搜尋 streams
![image](https://hackmd.io/_uploads/ryiYdabble.png)
3. Analyze -> Follow -> UDP Stream
![image](https://hackmd.io/_uploads/S136_6-Weg.png)
4. 於是一直改 stream 數直到 6 發現 flag
![image](https://hackmd.io/_uploads/r1OzK6Z-le.png)
**Flag: picoCTF{StaT31355_636f6e6e}**

## extensions
1. 下載 flag.txt 檔案後用`strings`與`cat`查看後都沒甚麼東西
2. 於是用`file`檢查，發現檔案類型居然為 png
3. 轉檔案為 flag.png
![image](https://hackmd.io/_uploads/Hke0cpbZel.png)
4. 在檔案總管打開圖片得到 flag
**Flag: picoCTF{now_you_know_about_extensions}**

## What Lies Within
* 下載 buildings.png 後用`exiftool`、`binwalk`、`zsteg`輪著查看發現 flag
![image](https://hackmd.io/_uploads/rk1BhTZZle.png)
**Flag: picoCTF{h1d1ng_1n_th3_b1t5}**

## like1000
1. 解壓縮 1000.tar 後發現還有好多層...
![image](https://hackmd.io/_uploads/r1Mi0p-Wlx.png)
2. 於是寫一腳本自動解壓縮到最後一層拿到 flag
extract.sh: 
```
#!/bin/bash

# 初始檔案名稱
current_tar="1000.tar"

# 解壓縮循環
while [ -f "$current_tar" ]; do
    tar xvf "$current_tar" > /dev/null  # 解壓縮並隱藏輸出
    next_tar=$(ls *.tar 2>/dev/null | grep -v "$current_tar")  # 找到下一個 .tar 文件
    rm -f "$current_tar"  # 刪除已解壓縮的 .tar 文件
    current_tar="$next_tar"  # 更新為下一個 .tar 文件
done

echo "所有檔案已解壓縮完成！"
```
**Flag: picoCTF{l0t5_0f_TAR5}**
