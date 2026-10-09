## Rust fixme1
![{7B196B4C-E686-4BD4-9FE5-31B5BC073E20}](https://hackmd.io/_uploads/HyA0fsRoye.png)
Fixed Code:
```
use xor_cryptor::XORCryptor;

fn main() {
    // Key for decryption
    let key = String::from("CSUCKS"); // End statements in Rust with a semicolon.

    // Encrypted flag values
    let hex_values = ["41", "30", "20", "63", "4a", "45", "54", "76", "01", "1c", "7e", "59", "63", "e1", "61", "25", "7f", "5a", "60", "50", "11", "38", "1f", "3a", "60", "e9", "62", "20", "0c", "e6", "50", "d3", "35"];

    // Convert the hexadecimal strings to bytes and collect them into a vector
    let encrypted_buffer: Vec<u8> = hex_values.iter()
        .map(|&hex| u8::from_str_radix(hex, 16).unwrap())
        .collect();

    // Create decryption object
    let res = XORCryptor::new(&key);
    if res.is_err() {
        return; // Return statement in Rust.
    }
    let xrc = res.unwrap();

    // Decrypt flag and print it out
    let decrypted_buffer = xrc.decrypt_vec(encrypted_buffer);
    println!(
        "Decrypted flag: {}", // Use {} to print variables.
        String::from_utf8_lossy(&decrypted_buffer)
    );
}
```
![{096915A3-BB28-477F-AFAE-03E777EA0B06}](https://hackmd.io/_uploads/S1GU4o0skx.png)

## Rust fixme2
![{571BE6E6-7603-4473-B09C-72054B4AAE8B}](https://hackmd.io/_uploads/rkBW4iRjJl.png)
Fixed Code:
```
use xor_cryptor::XORCryptor;

fn decrypt(encrypted_buffer: Vec<u8>, borrowed_string: &mut String) {
    // Key for decryption
    let key = String::from("CSUCKS");

    // Editing our borrowed value
    borrowed_string.push_str("PARTY FOUL! Here is your flag: ");

    // Create decryption object
    let res = XORCryptor::new(&key);
    if res.is_err() {
        return; // Proper Rust return statement
    }
    let xrc = res.unwrap();

    // Decrypt flag and append it to the borrowed string
    let decrypted_buffer = xrc.decrypt_vec(encrypted_buffer);
    borrowed_string.push_str(&String::from_utf8_lossy(&decrypted_buffer));
    println!("{}", borrowed_string);
}

fn main() {
    // Encrypted flag values
    let hex_values = ["41", "30", "20", "63", "4a", "45", "54", "76", "01", "1c", "7e", "59", "63", "e1", "61", "25", "0d", "c4", "60", "f2", "12", "a0", "18", "03", "51", "03", "36", "05", "0e", "f9", "42", "5b"];

    // Convert the hexadecimal strings to bytes and collect them into a vector
    let encrypted_buffer: Vec<u8> = hex_values.iter()
        .map(|&hex| u8::from_str_radix(hex, 16).unwrap())
        .collect();

    let mut party_foul = String::from("Using memory unsafe languages is a: "); // Mutable variable
    decrypt(encrypted_buffer, &mut party_foul); // Pass as mutable reference
}
```
![{B1475961-812E-439E-963D-E5ECF97FB7F4}](https://hackmd.io/_uploads/rJkdViRike.png)
## Rust fixme3
```
use xor_cryptor::XORCryptor;

fn decrypt(encrypted_buffer: Vec<u8>, borrowed_string: &mut String) {
    // Key for decryption
    let key = String::from("CSUCKS");

    // Editing our borrowed value
    borrowed_string.push_str("PARTY FOUL! Here is your flag: ");

    // Create decryption object
    let res = XORCryptor::new(&key);
    if res.is_err() {
        return; // Return if decryption initialization fails
    }
    let xrc = res.unwrap();

    // Decrypt the buffer directly without unnecessary unsafe operations
    let decrypted_buffer = xrc.decrypt_vec(encrypted_buffer);

    // Append the decrypted string to the borrowed string
    borrowed_string.push_str(&String::from_utf8_lossy(&decrypted_buffer));
    println!("{}", borrowed_string);
}

fn main() {
    // Encrypted flag values
    let hex_values = ["41", "30", "20", "63", "4a", "45", "54", "76", "12", "90", "7e", "53", "63", "e1", "01", "35", "7e", "59", "60", "f6", "03", "86", "7f", "56", "41", "29", "30", "6f", "08", "c3", "61", "f9", "35"];

    // Convert the hexadecimal strings to bytes and collect them into a vector
    let encrypted_buffer: Vec<u8> = hex_values.iter()
        .map(|&hex| u8::from_str_radix(hex, 16).unwrap())
        .collect();

    // Initialize the mutable string
    let mut party_foul = String::from("Using memory unsafe languages is a: ");
    decrypt(encrypted_buffer, &mut party_foul);
}
```
![{C4E39F3E-6A80-47B7-A5DA-00E87C0A865F}](https://hackmd.io/_uploads/H1ViNi0okx.png)
![{E31D8A32-ACD0-442A-805F-25E92F7BE214}](https://hackmd.io/_uploads/Bkz3EsRj1l.png)
## hashcrack
![{D8EDC3BA-C47F-447C-927A-A6DCB109943A}](https://hackmd.io/_uploads/BkAqajAiye.png)
## EVEN RSA CAN BE BROKEN???
![{D3F35E16-A9B7-4D8E-926A-F336524CD940}](https://hackmd.io/_uploads/BJJ2maCskl.png)
![{3CF712DD-FEC3-4DAD-97D7-7F0D2505CE6B}](https://hackmd.io/_uploads/rJe1VpAs1l.png)
![{EE3E968B-82BA-4C12-81EE-E58D615AE8E5}](https://hackmd.io/_uploads/SJkVEpRi1e.png)
寫一 decode.py 爆出私鑰
![image](https://hackmd.io/_uploads/BkGsbXHhke.png)
![image](https://hackmd.io/_uploads/SJHWz7r3kg.png)
最後 deactivate 退出 venv

## Cookie Monster Secret Recipe
獲得 Cookie
![image](https://hackmd.io/_uploads/By6W_Er2kg.png)
base64 decode
![image](https://hackmd.io/_uploads/Bk5G_VB2ke.png)

## Ph4nt0m 1ntrud3r
Hint: Timing is essential
故把時間欄位按照順序排序，並發現從這欄開始的base64解碼有意義
![image](https://hackmd.io/_uploads/rkYHYrB31x.png)
利用 cyberchef
![image](https://hackmd.io/_uploads/rkjotBrnJe.png)

## RED
![image](https://hackmd.io/_uploads/By5mewShke.png)
![image](https://hackmd.io/_uploads/HyHLlPH2Jx.png)

## headdump
![image](https://hackmd.io/_uploads/Sk1ENvrnyg.png)
進到網頁後根據題目點選 API Documentation
![image](https://hackmd.io/_uploads/ryYy4vH31e.png)
下載檔案後得到
![image](https://hackmd.io/_uploads/HksbNPH2yx.png)
