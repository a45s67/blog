---
title: "DEVCORE-Wargame-2023"
description: "DEVCORE Wargame 2023 CTF writeup，解題紀錄包括 web 題 first-flag 以及 MBR 逆向題 redpill。"
publishDate: "2023-09-03"
tags: ["CTF"]
---


# first-flag(有點忘記，應該是叫這名子沒錯)
隨便輸入一串 user，登入後會要你輸入 flag，超過 5 分會給你這一題的 flag
![](https://files.sakana.tw/blog/DEVCORE-Wargame-2023/first-flag-1.png)
![](https://files.sakana.tw/blog/DEVCORE-Wargame-2023/first-flag-2.png)
(搞笑的是我一開始以為 flag 是 `FLAG{this_is_the_first_flag}`，拿去輸入想說怎麼是錯的)

看 session，感覺有點東西，於是 base64 decode，嗯？感覺 exp 可以用。
```
0 point
session=eyJ1c2VyIjoiZmlzaCIsImV4cCI6MTY5MjQyNDA3NX0-7e958082bb77eba7e19784c775d7c3cc1b558774
{"user":"fish","exp":1692424075}>yO6my{_{;{swվy

1point
session=eyJ1c2VyIjoiZmlzaCIsImV4cCI6MTY5MjQyNDI1NCwiZmxhc2giOiJZb3Ugc3VjY2Vzc2Z1bGx5IHN1Ym1pdCB0aGUgZmxhZy4ifQ--4ce6b8a80c4324ef0f19d27bb7147f18d631ce78
{"user":"fish","exp":1692424254,"flash":"You successfully submit the flag."}

session=eyJ1c2VyIjoiZmlzaCIsImV4cCI6MTY5MjQyNDI1NH0-a0e88fbd65c0446aa1fb418c97232a2b3115cbbe
{"user":"fish","exp":1692424254}
```
於是試著改 session 中的 exp，發現 seesion 會直接失效，推測是有驗 hmac。但我稍微試了一下，實在摸不出 hmac 怎麼算的。

於是我想說這題太難了吧，而且大家第一題都解這個，根本不是交了 5 個 flag 的樣子 = =。
![](https://files.sakana.tw/blog/DEVCORE-Wargame-2023/first-flag-3.png)

前前後後試了快 3 hr，突然靈機一動想到我是不是可以登這些解過的人的帳號，看他們的 flag，還真的有些可以登，結果這些人的帳號也都是 1 分 2 分，三小，那他們跟誰拿 flag 的？

我想到了，admin
![](https://files.sakana.tw/blog/DEVCORE-Wargame-2023/first-flag-4.png)

嗚嗚嗚終於，順便看了一下 session，感覺 exp 好像真的有用到。
```
0 point
session=eyJ1c2VyIjoiZmlzaCIsImV4cCI6MTY5MjQyNDA3NX0-7e958082bb77eba7e19784c775d7c3cc1b558774
{"user":"fish","exp":1692424075}>yO6my{_{;{swվy

1 point 
session=eyJ1c2VyIjoiZmlzaCIsImV4cCI6MTY5MjQyNDI1NH0-a0e88fbd65c0446aa1fb418c97232a2b3115cbbe
{"user":"fish","exp":1692424254}

admin:
eyJ1c2VyIjoiYWRtaW4iLCJleHAiOjE2OTI0MjgyNjV98727389e2f8e138d34d215e16affb60fb4b75b9d
{"user":"admin","exp":1692428265}^߇vחoo]

```
# redpill
題目給了一個沒任何說明的檔案 [redpill](https://files.sakana.tw/blog/DEVCORE-Wargame-2023/redpill)
依據經驗，我猜這是一個逆向題(??)，先收集一下資訊
``` bash
# wsl ubuntu
> file redpill
redpill: DOS/MBR boot sector

> xxd redpill| head -n 10
00000000: 8cc8 8ed8 8ec0 8ed0 8ee0 bc00 7cb8 00b8  ............|...
00000010: 8ee8 fab8 0006 bb00 07b9 0000 ba4f 18cd  .............O..
00000020: 1066 b801 0000 00bb 0060 b901 00e8 3000  .f.......`....0.
00000030: 66b8 0200 0000 bb00 62b9 0100 e821 0066  f.......b....!.f

> strings redpill
>Wffg
fg)P
fg9E
         oooooooooo.   oooooooooooo oooooo     oooo        
         `888'   `Y8b  `888'     `8  `888.     .8'         
          888      888  888           `888.   .8'          
          888      888  888oooo8       `888. .8'           
          888      888  888             `888.8'            
          888     d88'  888       o      `888'             
         o888bood8P'   o888ooooood8       `8'              
                                                           
                                                           
      .oooooo.     .oooooo.   ooooooooo.   oooooooooooo    
     d8P'  `Y8b   d8P'  `Y8b  `888   `Y88. `888'     `8    
    888          888      888  888   .d88'  888            
    888          888      888  888ooo88P'   888oooo8       
    888          888      888  888`88b.     888            
    `88b    ooo  `88b    d88'  888  `88b.   888       o    
     `Y8bood8P'   `Y8bood8P'  o888o  o888o o888ooooood8    
ACCESS CODE:               
          ACESS DENY          
 Welcome agent, processing.... 

```
依據 `file` `strings` 和把 `8cc8 8ed8` 餵狗的資訊，推測這確實是 MBR 的磁區，開頭是 boot loader 的初始化。
並且從 `strings` 看不出任何 flag 的資訊。嘗試進行更深入的資訊收集
1. 靜態分析
2. 把這個 MBR 檔案執行起來，動態分析

## 靜態分析 1
在這邊我使用 radare2 做逆向
> 為啥不用 IDA Pro？因為手邊剛好只有 IDA free...，開不了 PE 以外的東西🥹。
> 我的做人，我的作法，沒錢的人，沒錢的做法。爬了一下文，感覺 radare2 可以試試就拿來上了，
> 順便學一下別的 decompiler 怎麼用。(而且感覺拿 IDA Pro 會瞬間做完很多事，這題可能會秒殺，會很無聊)

![](https://files.sakana.tw/blog/DEVCORE-Wargame-2023/redpill-1.png)
可以看到，斷在 0xffffffff8966e402 不知道這什麼地方。(其實這邊我 decompiler 的設定有誤，後面會說明)

## 動態分析 - 載入 MBR
爬了超久，關於怎麼載入、debug MBR 的資訊，因為一開始不知道要下哪些關鍵字，找到的內容都很瑣碎，重複嘗試了三個晚上。
最後主要是透過下面這幾篇的說明，我才搞懂要怎麼使用模擬器載入一個 MBR。
- [Debugging MBR - IDA + Bochs Emulator (CTF example)](https://github.com/Dump-GUY/Malware-analysis-and-Reverse-engineering/blob/main/Debugging%20MBR%20-%20IDA%20+%20Bochs%20Emulator/Debugging%20MBR%20-%20IDA%20+%20Bochs%20Emulator.md)
- [How to make a simple disk image - Bochs User Manual](https://bochs.sourceforge.io/doc/docbook/user/diskimagehowto.html) 
  - Create flat image
  - Set `bochsrc` file ([Search order for the bochs config file](https://bochs.sourceforge.io/doc/docbook/user/search-order.html))
- [《操作系统真xiang还原》阅读实践问题记录 bochs [HD ] ata0-0: could not open hard drive image file ‘hd60M.img](https://blog.csdn.net/q2453303961/article/details/122897092)

個人載入的步驟如下，使用了 bochs 這套模擬器 (qemu emulator 似乎也行，但我沒研究)
1. 使用 bochs 的 `bximage.exe` 建立一個 10MB flat image，叫 `redpill.img`
2. 生個 bochs 設定檔，這裡叫 `bochsrc.bxrc`，看是要用 bochs GUI 還是自己寫一份都行，我是用 GUI。
```
megs: 512
romimage: file="C:\Program Files\Bochs-2.7\BIOS-bochs-latest"
vgaromimage: file="C:\Program Files\Bochs-2.7\VGABIOS-lgpl-latest"
boot: cdrom, disk
ata0-master: type=disk, path="redpill.img", mode=flat
mouse: enabled=0
cpu: ips=90000000
```
3. 將 redpill 的內容覆寫到 `redpill.img` 的開頭 
``` python
with open('redpill', 'rb') as f:
    mbr = f.read()
with open('redpill.img', 'r+b') as f:
    f.write(mbr)
```
4. 開始 debug image 的載入 `bochsdbg.exe -f bochsrc.bxrc`
![終於成功載入](https://files.sakana.tw/blog/DEVCORE-Wargame-2023/redpill-2.png)

## 靜態分析 2 
參考[Solving the Disobey 2020 puzzle bootloader using Unicorn and Ghidra](https://jarijaas.github.io/posts/disobey-2020/)
觀察 bochs dbg diasm 後的內容，確認了 "靜態分析 - 1" 中幾件我搞錯的事
- radare2 的 cpu 模式要設成 16 bits 才對，boot 的時候 CPU 是 real mode
- 最後 jmp 是跳躍到 0x6000

![](https://files.sakana.tw/blog/DEVCORE-Wargame-2023/redpill-3.png)
重新設定 r2 disam 後再從頭分析一次，可以知道 0x6000~0x6600 應該是 call 0x60 時做了某些操作後產生的 code。
來看看 0x60 做了甚麼事，原來其是將對應 LBA sector 的內容拷貝到對應的記憶體上。
![](https://files.sakana.tw/blog/DEVCORE-Wargame-2023/redpill-4.png)

回到 0x00 的 asm，0x6000 ~ 0x6600 應為 image 的 sector 1 ~ 4 的內容，每一 sector 大小為 0x200，對應 image offset 0x200 ~ 0x800。

參考資料
- [自己動手從零寫桌面作業系統GrapeOS系列教學——21.組合語言寫硬碟實戰](https://www.tw511.com/a/01/55190.html)
  - 0x1f0 ~ 0x1f7 是
- [Master boot record - Wiki](https://en.wikipedia.org/wiki/Master_boot_record)、[计算机是如何启动的？](http://www.ruanyifeng.com/blog/2013/02/booting.html)、- [[筆記] 為什麼在 x86，MBR 會被載入到 0x7C00？(完全版)](https://gist.github.com/snyiu100/b147b3175315b9e959e3a7a6281a82c1)
  - MBR：大小為 0x200 Bytes，offset 0x1FE 為 0x55 0xAA
  - MBR 會被載入至 memory address 0x7C00 執行
- [LBA and sector size - Superuser](https://superuser.com/questions/418211/lba-and-sector-size)
  - 1 sector = 0x200 Bytes
- [硬盤訪問概述](https://learningos.github.io/ucore_os_webdocs/lab1/lab1_3_2_3_dist_accessing.html)
- 延伸閱讀 - [硬碟心得 - CHS、LBA](http://darren.logdown.com/posts/171249-hard-drive-experiences-chs-lba)
- 延伸閱讀 - [邏輯區塊位址](https://zh.wikipedia.org/zh-tw/%E9%82%8F%E8%BC%AF%E5%8D%80%E5%A1%8A%E4%BD%8D%E5%9D%80)
- [分段架構: Segment Selectors 和分段暫存器](https://www.csie.ntu.edu.tw/~wcchen/asm98/asm/proj/b85506061/chap2/segment.html)


## 靜態分析 3
接下來進行 0x200 的分析，並嘗試快速定位 flag 的判斷位置。
![](https://files.sakana.tw/blog/DEVCORE-Wargame-2023/redpill-5.png)
當 flag 輸入錯誤時，會顯示 ACCESS DENY 的文字。而 ACCESS DENY 在 file offset 0x745 (應對應 memory 0x6545)。
翻翻看哪個指令使用了這個字串，可以推測重點的判斷邏輯就在附近。
快速瀏覽一遍 0x200 後，可以發現用了大量的中斷，查了後這些中斷都蠻好懂的，能很大的幫助理解邏輯。
例如 0x237 - 0x278 這邊的迴圈是通過 `int 0x16` 取得鍵盤輸入，並呼叫 `int 0x10` 將其輸出到螢幕上，以及將輸入記錄到 buf 中。
那我們就可以很直接的知道這邊是處理我們輸入 flag 的互動實作。
![](https://files.sakana.tw/blog/DEVCORE-Wargame-2023/redpill-6.png)

最後追到使用 `ACCESS DENY` 的 0x307，前面的 asm 則是使用 `0x6585`、`0x65af` 做了某種處理及比對。
![](https://files.sakana.tw/blog/DEVCORE-Wargame-2023/redpill-7.png)

先檢查一下這兩個 buf 執行前的內容
![](https://files.sakana.tw/blog/DEVCORE-Wargame-2023/redpill-9.png)
![](https://files.sakana.tw/blog/DEVCORE-Wargame-2023/redpill-8.png)

## 動態 + 靜態分析 4
為了確認 `0x6585`、`0x65af` 這兩個 buf 在 runtime 進行處理、比對時的內容，直接在進入比對前的 `0x6097` 下斷點。
檢查這時的 buf 內容，可見 `0x6585` 為使用者的輸入內容，`0x65af` 為 secret，跟靜態分析時的內容相同。
![](https://files.sakana.tw/blog/DEVCORE-Wargame-2023/redpill-10.png)

確定了輸入是與 secret 進行某種操作比對後，直接針對這邊的實作進行逆向，得到邏輯大概是長這樣
```
secret[i] = input[i]*3 - input[i+1]*2
secret[i+1] = input[i]*4 + input[i+1]*7
```
原來是要解個二元一次方程式阿...開始寫腳本

``` python
def decode(v1, v2):
    # v1 = c2*3 - c1*2
    # v2 = c2*4 + c1*7
    c1 = int((v2*3 - v1*4)/29)
    c2 = int((v2*2 + v1*7)/29)
    return [chr(c2), chr(c1),]

flag_reorder = [] 
flag_reorder += decode(0x42,0x2f3)
flag_reorder += decode(0x7c,0x32d)
flag_reorder += decode(0x49,0x37a)
flag_reorder += decode(0xd9-0x100,0x471)
flag_reorder += decode(0xb8-0x100,0x3ee)
flag_reorder += decode(0xdb-0x100,0x365)
flag_reorder += decode(0xff,0x341)
flag_reorder += decode(0x5b,0x423)
flag_reorder += decode(0xe2,0x307)
flag_reorder += decode(0x45,0x470)
flag_reorder += decode(0x4f,0x4de)
flag_reorder += decode(0xb7-0x100,0x3c6)
flag_reorder += decode(0x77,0x435)
flag_reorder += decode(0x5f,0x4a6)
flag_reorder += decode(0x35,0x4a8)
flag_reorder += decode(0xd2,0x305)
flag_reorder += decode(0x43,0x477)
flag_reorder += decode(0xb5,0x305)
flag_reorder += decode(0x84,0x4a7)
flag_reorder += decode(0x42,0x29c)
flag_reorder += decode(0xc3-0x100,0x467)

print(''.join(flag_reorder))
```
得到 flag `DEVCORE{4r3_w3_al1_liv1ng_in_th3_ma7ri><?}`
(最後面那個 word 到底是甚麼我想了很久，想到題目是 redpill，借助 google 的協助才知道是 matrix...)




# 其他題
沒解到就關了，88
