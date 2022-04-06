# picoCTF 2022
2022 picoCTF 的 Writeup 紀錄，這真的是我寫過最佛的 CTF 了，是個培養信心的好地方XDD
###### tags: `picoCTF`
[TOC]
## Web Exploitation
### Include
- 題目解釋了 Include file 是什麼，按下 say hello 會跟你說 code 在 separate file
- 很明顯要你去看看 include file(?

    ![](https://i.imgur.com/rralW9D.png)
- 於是打開 F12，看到 include 的 script，點開來看看，就發現註解的地方看起來像 flag 的屁股

    ![](https://i.imgur.com/WzYrTfg.png)
    
    ![](https://i.imgur.com/k9MhsGZ.png)
- 再開另一個 css 的 include file，結果還真的是 0.0
    
    ![](https://i.imgur.com/VGccZCM.png)

### Inspect HTML
- 題目都叫你檢查 HTML 了，不 F12 看一下 source code 說不過去吧 XD，水題萬歲~

    ![](https://i.imgur.com/KHrVT2S.png)

### Local Authority
- 進來要你輸入帳密，且限英文數字，隨便打個 test, test 進去。

    ![](https://i.imgur.com/as18gs3.png)
    
- 檢視原始碼可以看到有個 hidden 的 form 跟一個 script 的 include

    ![](https://i.imgur.com/v3IAdxA.png)
    
- script 前半在做英文數字的檢查，後面可以看到他通過檢查後再做一個 checkPassword 來檢查帳密正確性，但這裡看不到這個 function
- 上面還有一個 secure.js 的 script，裡面藏了正確的帳密 (不用懷疑就是這麼好心)

    ![](https://i.imgur.com/su4th3P.png)

- 登入後就可以拿到 FLAG 了

    ![](https://i.imgur.com/zBPT5GC.png)
### Search source
- 打開 link 是個瑜珈網站

    ![](https://i.imgur.com/l9h9C9x.png)

- 看一下 source code 發現一個 google map 的地圖標記用的圖片是 maps-and-flags.png，看起來很可疑
    
    ![](https://i.imgur.com/W6VxXFp.png)
    
- 但直接訪問存取不到圖片，題目提示是怎麼 mirror website 到 local 更好搜尋網頁資訊，於是我就用 HTTrack 把整個網站 mirror 下來了。
- 不過還是沒有這 png，只有留下 404 的紀錄，看來是被刪掉了?
    
    ![](https://i.imgur.com/8QOl7Dq.png)

- 後來用 vscode 打開，搜尋 `picoCTF{` 就在 css 找到了 ... 害我找那張圖片找超久 (╯°Д°)╯ ┻━┻
    
    ![](https://i.imgur.com/y6kBtqy.png)


### Forbidden Paths
- 題目說網頁檔案在 `/usr/share/nginx/html/`，flag 檔案在 `/flag.txt`，但有 filter 會去過濾絕對路徑，沒辦法直接從絕對路徑拿到檔案。
- 直覺想到用 `../` 繞過

    ![](https://i.imgur.com/0lNBpd5.png)
    
    ![](https://i.imgur.com/jvFTawb.png)
    
### Power Cookie
- 按個鍵可以以 guest 登入，但會告訴你沒有可提供的 service。

    ![](https://i.imgur.com/Cup0Hnr.png    
- 打開可以看到一個 isAdmin=0 的 cookie。

    ![](https://i.imgur.com/LKN4nnd.png)

- 設成 0 用 curl 重送 http request 就可以拿到 flag 了。
    
    ![](https://i.imgur.com/zNPSF45.png)

### Roboto Sans
- 點進去就是一個看起來很正常的瑜珈網站，題目的 Roboto 感覺就是在講 robots.txt。
- 打開可以看到有三行看起來就很像 base64 encoding。

    ![](https://i.imgur.com/oYO4xLW.png)
- 個別進行轉換後可以發現分別是 `flag1.txt` 跟 `js/myfile.txt` 兩個檔案

    ![](https://i.imgur.com/Gi3mL7l.png)

- `flag1.txt` 讀不到(ノ▼Д▼)ノ，`js/myfile.txt`就拿到 flag 了。
    
    ![](https://i.imgur.com/WmH1kyX.png)

### Secret
- 點開連結是一個粉正常的網頁，開 sorce code 看一下。
    
    ![](https://i.imgur.com/HDh81Kt.png)

- 題目的提示是 folder folder folder(很重要所以講三次XD)
- 看著看著覺得 secret 資料夾頗可疑。
- 加了一層 `secret/`，發現了一個新頁面。
`http://saturn.picoctf.net:58314/secret/index.html`
    ![](https://i.imgur.com/2HC5reR.png)
- 又看到一個 `hidden` 資料夾，進去又是一個新頁面。

    ![](https://i.imgur.com/bSpsiV3.png)

- 找到 flag 了~ (別問我怎麼知道每層資料夾都有 index.html，可能我通靈能力變強了XDD)

    ![](https://i.imgur.com/h7HGWyO.png)

### SQL Direct
- 這題是很簡單的 SQL，用平常 SQLi 查資料表的方式 `SELECT * FROM information_schema.tables;` 就可以找到 flag 了。

    ![](https://i.imgur.com/BC6RY42.png)
    
    ![](https://i.imgur.com/ZZC6Twe.png)

### SQLiLite
- 簡單的登入介面，username 輸入 `'or 1=1` 閉合`'`，並註解掉後面 username 後的輸入，執行 `SELECT * FROM users WHERE name='' or 1=1`

    ![](https://i.imgur.com/IDiNyEp.png)

- flag 在 source code

    ![](https://i.imgur.com/ijhodMZ.png)

## Reverse Engineering
### file-run1
- 給了一個 run 的 binary 檔
- chmod +x 後就拿到 flag 了

    ![](https://i.imgur.com/LilIwUn.png)

### file-run2
-  一樣 chmod +x 改執行權限
-  傳入 argv `Hello!`，就拿到 flag 了

    ![](https://i.imgur.com/Gl2Mtth.png)
    
### GDB Test Drive
- 這題是 GDB 教學(?，基本上照著題目給的打就可以拿到 flag 了

    ![](https://i.imgur.com/EsoMohD.png)
    
- `layout asm` 可以顯示 assembly 視窗，`*(main+99)` 是一個 sleep 100000 秒的函式，如果你直接執行要快一個月：)，所以才需要 jump 到他的下個指令 `*(main+104)`，跳過 sleep 繼續執行。

    ![](https://i.imgur.com/OQWPA4n.png)
    
    ![](https://i.imgur.com/JZiXjeL.png)

### patchme.py
- 題目給了一個加密 flag 檔案，還有以個解密的 py，但須要輸入正確密碼才能解密
- 打開 py 檔就可以看到他解密的地方就是比對輸入字串而已 (\是字串換行語法)，輸入`"ak98-=90adfjhgj321sleuth9000" ` 後就拿到 flag 了
    
    ![](https://i.imgur.com/lFVPS6F.png)

### Safe Opener
- 這題也是類似東西，可以看到有 3 次機會給你輸入密碼，每次也是去呼叫 openSafe() 看你輸入的密碼過 base64 後有沒有等於 `cGwzYXMzX2wzdF9tM18xbnQwX3RoM19zYWYz`，而這個東西拿來 base64 decode 後是 `pl3as3_l3t_m3_1nt0_th3_saf3`，看來就很 flag ~
```java=
public class SafeOpener {
    public static void main(String args[]) throws IOException {
        BufferedReader keyboard = new BufferedReader(new InputStreamReader(System.in));
        Base64.Encoder encoder = Base64.getEncoder();
        String encodedkey = "";
        String key = "";
        int i = 0;
        boolean isOpen;
        

        while (i < 3) {
            System.out.print("Enter password for the safe: ");
            key = keyboard.readLine();

            encodedkey = encoder.encodeToString(key.getBytes());
            System.out.println(encodedkey);
              
            isOpen = openSafe(encodedkey);
            if (!isOpen) {
                System.out.println("You have  " + (2 - i) + " attempt(s) left");
                i++;
                continue;
            }
            break;
        }
    }
    
    public static boolean openSafe(String password) {
        String encodedkey = "cGwzYXMzX2wzdF9tM18xbnQwX3RoM19zYWYz";
        
        if (password.equals(encodedkey)) {
            System.out.println("Sesame open");
            return true;
        }
        else {
            System.out.println("Password is incorrect\n");
            return false;
        }
    }
}
```
### unpackme.py
- 題目給了一個 py 檔，用了一個 str 過 base64 然後當作 Fernet 對稱式加密的 key，然後用這個 key 去解 payload。

```python=
import base64
from cryptography.fernet import Fernet

payload = b'gAAAAABiMD1Ju5_eZeZy7C03K_YcWGDGXfvy5A9b5HzV-uZIYN8syTFGHgLwoRonYtCS0WcDrufxRRXlvNKtyEMqMS0AADLcRNr6VYpLLbKaETF37L22GEg1ok8NutHXK6gy47sBLmxmWWU729b86rzK6IMc2Kg-CR0bMm_fzrbRrWEYSk0WRNnKxy7Juuy-Ss2RjbACKgbwL7HNGATu3hYuPflf3PCKztLRFXCBxijKncKZgt68wYhGnPAzYvUVrdhhtMg9ra7ZKIirltPfKC8iX2DqmR9vVA=='

key_str = 'correctstaplecorrectstaplecorrec'
key_str.encode()
key_base64 = base64.b64encode(key_str.encode())
print(key_base64)
f = Fernet(key_base64)
plain = f.decrypt(payload)
print(plain)
exec(plain.decode())
```

- 蠻神奇的，我想說奇怪他又沒寫 input，也沒寫比較的地方，怎麼執行會要我輸入密碼。

    ![](https://i.imgur.com/3DwbHMl.png)

- 將 plain print 出來才發現原來他的 payload 解出來是 python 程式碼。

    ![](https://i.imgur.com/sd6poIe.png)

### bloat.py
- code 如題目，真的很"腫"，這題基本上沒什麼特別技巧，就是想辦法看懂程式碼，從最下面 main 開始執行的地方，透過 vscode 把 arg[xxx] 都給 replace 成比較人性化的名字，慢慢轉就會知道他在幹嘛了。

```python=
import sys

a = "!\"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ"+ \

            "[\\]^_`abcdefghijklmnopqrstuvwxyz{|}~ "

def arg133(arg432):

  if arg432 == a[71]+a[64]+a[79]+a[79]+a[88]+a[66]+a[71]+a[64]+a[77]+a[66]+a[68]:

    return True

  else:

    print(a[51]+a[71]+a[64]+a[83]+a[94]+a[79]+a[64]+a[82]+a[82]+a[86]+a[78]+\

a[81]+a[67]+a[94]+a[72]+a[82]+a[94]+a[72]+a[77]+a[66]+a[78]+a[81]+\

a[81]+a[68]+a[66]+a[83])

    sys.exit(0)

    return False

def arg111(arg444):

  return arg122(arg444.decode(), a[81]+a[64]+a[79]+a[82]+a[66]+a[64]+a[75]+\

a[75]+a[72]+a[78]+a[77])

def arg232():

  return input(a[47]+a[75]+a[68]+a[64]+a[82]+a[68]+a[94]+a[68]+a[77]+a[83]+\

a[68]+a[81]+a[94]+a[66]+a[78]+a[81]+a[81]+a[68]+a[66]+a[83]+\

a[94]+a[79]+a[64]+a[82]+a[82]+a[86]+a[78]+a[81]+a[67]+a[94]+\

a[69]+a[78]+a[81]+a[94]+a[69]+a[75]+a[64]+a[70]+a[25]+a[94])

def arg132():

  return open('flag.txt.enc', 'rb').read()

def arg112():

  print(a[54]+a[68]+a[75]+a[66]+a[78]+a[76]+a[68]+a[94]+a[65]+a[64]+a[66]+\

a[74]+a[13]+a[13]+a[13]+a[94]+a[88]+a[78]+a[84]+a[81]+a[94]+a[69]+\

a[75]+a[64]+a[70]+a[11]+a[94]+a[84]+a[82]+a[68]+a[81]+a[25])

def arg122(arg432, arg423):

    arg433 = arg423

    i = 0

    while len(arg433) < len(arg432):

        arg433 = arg433 + arg423[i]

        i = (i + 1) % len(arg423)        

    return "".join([chr(ord(arg422) ^ ord(arg442)) for (arg422,arg442) in zip(arg432,arg433)])

arg444 = arg132()

arg432 = arg232()

arg133(arg432)

arg112()

arg423 = arg111(arg444)

print(arg423)

sys.exit(0)
```
- 最終轉出來可以看到重點是那個 cmp_pwd() 的 function，把 cmp 的字串給印出來就知道密碼，拿到 flag 了。
```python=
import sys

a = "!\"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ"+ \

            "[\\]^_`abcdefghijklmnopqrstuvwxyz{|}~ "

def cmp_pwd(pwd):

  if pwd == a[71]+a[64]+a[79]+a[79]+a[88]+a[66]+a[71]+a[64]+a[77]+a[66]+a[68]:

    return True

  else:

    print(a[51]+a[71]+a[64]+a[83]+a[94]+a[79]+a[64]+a[82]+a[82]+a[86]+a[78]+\

a[81]+a[67]+a[94]+a[72]+a[82]+a[94]+a[72]+a[77]+a[66]+a[78]+a[81]+\

a[81]+a[68]+a[66]+a[83])

    sys.exit(0)

    return False

def decode_flag(enc_flag):

  return decode_alg(enc_flag.decode(), a[81]+a[64]+a[79]+a[82]+a[66]+a[64]+a[75]+\

a[75]+a[72]+a[78]+a[77])

def input_pwd():

  return input(a[47]+a[75]+a[68]+a[64]+a[82]+a[68]+a[94]+a[68]+a[77]+a[83]+\

a[68]+a[81]+a[94]+a[66]+a[78]+a[81]+a[81]+a[68]+a[66]+a[83]+\

a[94]+a[79]+a[64]+a[82]+a[82]+a[86]+a[78]+a[81]+a[67]+a[94]+\

a[69]+a[78]+a[81]+a[94]+a[69]+a[75]+a[64]+a[70]+a[25]+a[94])

def read_enc_flag():

  return open('flag.txt.enc', 'rb').read()

def print_something():

  print(a[54]+a[68]+a[75]+a[66]+a[78]+a[76]+a[68]+a[94]+a[65]+a[64]+a[66]+\

a[74]+a[13]+a[13]+a[13]+a[94]+a[88]+a[78]+a[84]+a[81]+a[94]+a[69]+\

a[75]+a[64]+a[70]+a[11]+a[94]+a[84]+a[82]+a[68]+a[81]+a[25])

def decode_alg(pwd, target):

    tmp = target

    i = 0

    while len(tmp) < len(pwd):

        tmp = tmp + target[i]

        i = (i + 1) % len(target)        

    return "".join([chr(ord(j) ^ ord(k)) for (j,k) in zip(pwd,tmp)])



print(a[71]+a[64]+a[79]+a[79]+a[88]+a[66]+a[71]+a[64]+a[77]+a[66]+a[68])

enc_flag = read_enc_flag()

pwd = input_pwd()

cmp_pwd(pwd)

print_something()

target = decode_flag(enc_flag)

print(target)

sys.exit(0)
```

![](https://i.imgur.com/xq6rfPE.png)

### Fresh Java
- 這題就是反編譯 java 執行檔而已，用線上反編譯工具就可以看到 flag 了。

    ![](https://i.imgur.com/fHWJLUO.png)
    
    ![](https://i.imgur.com/KV1ZVdw.png)

### Bbbbloat
- 這題其實我不太確定他要幹嘛，因為我用 ida 開起來，找到 main function，F5 可以看到他就只是比對輸入數字 == 549255 就拿到 flag。

    ![](https://i.imgur.com/5dVp8Kr.png)
    
- 然後我照著打進去，就拿到 flag 了 0.0。

    ![](https://i.imgur.com/lKB5Jd6.png)

### unpackme
- 簡單的 UPX 脫殼而已，檔案丟到 DIE 脫殼，用 IDA 開起來，就可以看到比對的數字。

    ![](https://i.imgur.com/ExWaaQP.png)
    
    ![](https://i.imgur.com/JorQcg4.png)
    
- 輸入後就拿到 flag 了。

    ![](https://i.imgur.com/8xkWbpW.png)
