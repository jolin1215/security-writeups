# Easy

## CanYouSee
1. After extracting the archive, we get an image. Use `exiftool` to inspect it and we find a base64 string.

![image](https://hackmd.io/_uploads/B1Fzc4JZgx.png)

2. Decode it in CyberChef.

![image](https://hackmd.io/_uploads/Sy9Z5V1Wgx.png)

**Flag: picoCTF{ME74D47A_HIDD3N_4dabddcb}**

## Secret of the Polyglot
1. After opening the PDF, the flag is at the bottom.

![image](https://hackmd.io/_uploads/HJteZH1bgx.png)

2. Inspect the file with `exiftool` and determine that the file type is PNG.

![image](https://hackmd.io/_uploads/BkeVlB1Wgl.png)

3. Convert the file to PNG and inspect it; the flag header appears.

![image](https://hackmd.io/_uploads/ByKNZHJWxg.png)

**Flag: picoCTF{f1u3n7_1n_pn9_&_pdf_249d05c0}**

## Verify
1. Connect to the server via SSH.

![image](https://hackmd.io/_uploads/H1iIaSy-xg.png)

2. Find the file.

`for f in files/*`: iterate over each file in the `files` directory.
`sha256sum "$f"`: compute the SHA-256 hash of the file.
`done`: end the loop.
`| grep <hash>`: find the file whose SHA-256 hash matches the value in `checksum.txt`; that is the real flag file.

3. Run `./decrypt.sh/<file>` to decrypt it.

![image](https://hackmd.io/_uploads/H1_Rnryble.png)

**Flag: picoCTF{trust_but_verify_87590c24}**

# Medium

## shark on wire 1What Lies Within
1. Download `buildings.png`.

![image](https://hackmd.io/_uploads/rk1BhTZZle.png)

* Hint: What are streams?

3. After downloading it, use `strings` on the pcap file and find nothing special.

![image](https://hackmd.io/_uploads/Sk07O6-blg.png)

2. After reading the hint, open the file in Wireshark and search for `streams`.

![image](https://hackmd.io/_uploads/ryiYdabble.png)

3. Go to `Analyze -> Follow -> UDP Stream`.

![image](https://hackmd.io/_uploads/S136_6-Weg.png)

4. Keep changing the stream number until stream 6 reveals the flag.

![image](https://hackmd.io/_uploads/r1OzK6Z-le.png)

**Flag: picoCTF{StaT31355_636f6e6e}**

## extensions
1. Download the `flag.txt` file and inspect it with `strings` and `cat`; there is nothing obvious.
2. Use `file` to check it and discover that it is actually a PNG file.
3. Convert the file to `flag.png`.

![image](https://hackmd.io/_uploads/Hke0cpbZel.png)

4. Open the image in the file manager to view the flag.

**Flag: picoCTF{now_you_know_about_extensions}**

## What Lies Within
* Download `buildings.png` and inspect it with `exiftool`, `binwalk`, and `zsteg` until the flag is found.

![image](https://hackmd.io/_uploads/rk1BhTZZle.png)

**Flag: picoCTF{h1d1ng_1n_th3_b1t5}**

## like1000
1. Extract `1000.tar` and find there are many layers inside.

![image](https://hackmd.io/_uploads/r1Mi0p-Wlx.png)

2. Write a script to automatically extract until the final layer is reached and the flag is revealed.

`extract.sh`:

```bash
#!/bin/bash

# Initial filename
current_tar="1000.tar"

# Extraction loop
while [ -f "$current_tar" ]; do
    tar xvf "$current_tar" > /dev/null    # Extract and hide output
    next_tar=$(ls *.tar 2>/dev/null | grep -v "$current_tar")    # Find the next .tar file
    rm -f "$current_tar"    # Remove the extracted .tar file
    current_tar="$next_tar"    # Move to the next tarball
done

echo "Extraction complete!"
```

**Flag: picoCTF{l0t5_0f_TAR5}**
