---
title: "如何修改 RPG 遊戲 - Sequel Blight 角色的各項數值"
description: "使用 Cheat Engine 對 RPG 遊戲 Sequel Blight 進行逆向工程，修改角色數值與移動速度的完整記錄。"
publishDate: "2023-09-16"
tags: ["Reverse Engineering"]
coverImage:
  src: "https://files.sakana.tw/blog/Sequel-Blight-Reversing/Title.png"
  alt: "cover image"
---



## 前言
去年有段時間蠻閒的，划 FB 時看到有人推薦這款遊戲，就來玩玩看。
玩到一半發現這遊戲有點農，地圖還很大要走很久...但我只想趕快破完看劇情。
最後受不了了，決定直接開 CE 改數據。在逆向的過程中，意外的克服不少問題，學到了不少，於是決定記錄下來。

這篇文章會以這款遊戲的實際情境為例，講述我如何推測、找尋切入點，各種 CE 的操作小技巧，再到 CE 做不下去時，可以進行的其他選擇。
整個過程和操作細節上，應該還有很多能改進的點，若各路大神路過，有一些感覺能更好的做法，也懇請教教小弟...。

## 目標說明
### Sequel Blight
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-1.png)
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-3.png)
基本上遊戲過程就是解劇情，跑地圖打怪、升等打王，解隱藏地圖和道具等等的。而戰鬥機制的部分，就是常見的操控腳色跑地圖，撞到怪物時會進入戰鬥回合這樣。
還有不少沒提到，總之這遊戲的元素很豐富，像是可以轉的職業就有 10 幾個，當 RPG 玩的話確實也蠻耐玩的。

> 雖然遊戲本身是成人向，但這篇文章不含任何成人內容，可以放心瀏覽😂。

### 目標
我的訴求是盡快結束每一場戰鬥，並且減少跑圖的時間，故要做的事有以下幾點：
1. 修改角色數值，如攻擊力、血量
2. 調整地圖人物移動速度，讓移動快一點

## 修改角色數值
修改角色面板攻擊力、血量的步驟 (以下簡稱 atk、hp)
1. 取得這些角色數值的 memory address
2. 然後就可以修改了

### 取得角色數值的 address

好像就這麼簡單。但要怎麼這些取得角色數值呢？直接搜尋會發現找不到這些值，於是猜測這些值存到 memory 前有處理過。(如下圖，搜尋攻擊力 302，過濾幾次後會發現搜尋結果一片空)
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-4.png) 

推測這類型的數值，應該是由角色基礎數值、職業加成、裝備加成等等加總起來的，目標放在角色基礎數值的話，由於沒辦法知道確切數值為何，搜尋的方法會變成：搜尋 unknown value -> 回遊戲變動角色能力值 -> 過濾出變動的 value -> 回遊戲變動能力值 -> ... (無限 loop)。
但是，這流程有個 bug，我想不到要怎麼變動角色基礎能力值，話說能想變就變的話我也不用 CE 了。
卡了一段時間後，突然想到，在遊戲進行的過程中我有取得一些提升能力值的藥水，運氣不錯，這邊直接拿來試試。
以活力之水(效果是sp+5)為例，流程是先取得這道具數量的 address，然後一口氣把活力之水數量調大，再慢慢試角色能力值。
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-5.png)
(過濾方法基本上就是 unknown value -> decreased value -> decreased value ...)

把幾個看起來很像的 address 抓下來，隨便改個數字測試一下，確認 `0x0E481A8` 就是活力藥水的數量，同時還能發現一件事，記憶體中實際的數字是 `2x+1`，例如我藥水剩 4 瓶時，記憶體內容為 9，推測攻擊力、血量也是同理。
現在拿活力藥水狂加在拉比身上，過濾出 `0x14F53A24` 是人物基礎 sp 的位置。
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-6.png)

### 使用 Poninter Scan 找出指向 address 的 pointer
接下來有幾條路可以走
1. 由於人物屬性(hp, sp, atk 等等)應該會宣告在同一個 stuct 內，故 hp、atk 的 address 應該在 sp 附近。對這附近的值亂改一遍，看看哪個是 hp, atk。
2. 還沒想到...(單純覺得還能想到其他條路)

為了進行第一個方法，必須先把 sp 的 pointer 存下來，避免改壞 struct 造成遊戲 crash，遊戲重開後各物件的 address 通常都會被重分配一次，有存下實際的 pointer 才不會要再重找一遍 sp 的 address。
這時就要用到 Cheat Engine 的屌功能了 - **Pointer Scan** (在 Memory view 視窗的 Tools 選單中)，

簡單來說，就是掃一遍`0x14F53A24`，CE 會找出所有指向到這位置的 pointer，接著把遊戲關掉重開，Rescan memory 出還是有效的 pointer，如下圖
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-8.png)
重開遊戲後，跑一次 Rescan memory，過濾出正確的 pointer。
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-7.png)

### 使用 dissect struct 觀察周遭變數
接下來就可以開始大膽的亂改 memory 了，觀察上一步很屌的 pointer scan 結果，可以注意到 sp (`0x0EF7A374`)似乎位於 pointer 指向的 address 再 +4 的位置，強烈懷疑這 address 是拉比的人物屬性 struct，丟去 dissect struct 看一下。
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-9.png)
因為角色狀態的呈現是一條 hp 一條 sp，若第二個 value (offset +4) 是 sp，那第一個 value (offset +0) 應該是 hp，依此類推並測試確認後，這 struct 的 member 應該如下：
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-10.png)
``` c++
struct character_status {
    int hp;
    int sp;
    int atk;
    int def;
    int matk;
    int mdef;
    int spd;
    int luk;
};
```

整理一下 Cheat Table(以下以 CT 代稱)，把 hp、sp 之類的記下來，這裡又要再提到 CE 的一個 feature，我們可以新增一個 address，address 寫 +0, +4 之類的 offset 描述，然後用滑鼠把它拉到做為 base address 的 pointer 下，這個 address 就會自動把 base address 加上來。
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-11.png)
現在我們獲得了代表 rabi 狀態的 address，能夠隨便改 rabi 的攻擊力和血量了。那其他角色呢？

再通靈一次，依據感覺，rabi、uula 等等角色會有一個角色物件，存有各種 member 代表狀態、道具、職業等等資訊。
然後會有玩家物件，member 包括遊戲進行時的各種數據、各個角色物件等等。在這邊假設角色物件會用指標去存，所以 uula 等角色物件的指標應該都存在 rabi 指標的旁邊，由於 rabi 是第一位角色，其他三位角色理論上是在 `+4`, `+8`, `+C` 的位置。(當然也有可能 member 是完整的角色物件，而不是指標，那這個假設就不成立了)。
把可以調的 offset 都試了一遍，還真的給我試成功，某一段 pointer `offset +18` 代表 rabi，改成 `offset +1B,+20,+24` 後，發現分別代表其他三位女角，現在有辦法任意修改 4 位女主角的基本數值了。
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-12.png)

運氣很好，給我猜中，省下了重新找指標的時間...
> 其實當時我並沒有通靈這麼多，找到一個角色後就沒做下去了，在寫這篇文章把流程重跑一遍後，才想到好像可以這樣試試。

## 修改移動速度
當人物在地圖上移動時，有兩種狀態，一般情況下是正常移動速度，按住 shift 時是緩慢移動速度。我的目標就是把正常的移動速度改快一點。
> **2st round 發現的事**
> 後來重新審視一遍，實際上是分成一般速度和衝刺，這遊戲通常都在衝刺狀態，按住 shift 時會回到一般速度。如果事先有釐清出這件事，說不定我接下來的逆向過程會直接順利好幾倍...🥹。

該完成的步驟有
1. 取得人物移動速度的 address
2. 然後就可以修改了

### 過濾出可能代表人物速度的 address
我們不知道移動速度的數字是多少，所以再來一次 unknown value 搜索大法：
unknown value-> shift 按住移動人物 -> changed value -> 放掉 shift 移動人物 -> changed value -> 隨便做幾個動作後移動人物 -> unchanged value -> ... (infinite loop)

找到了 3 個看起來很像是移動速度的值，正常時是 11，按下 shift 會變成 9。
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-13.png)
但是把這 3 個 address 的值鎖在 11，會發現一點效果都沒有。
推測鎖值無效的原因，是這不是正確的 address，可能只是處理人物速度時，暫時分配的 variable 而已，而 CE 的鎖值其實是每 100ms 鎖一次 (可從設定調整)，所以鎖不到。
我猜邏輯可能是這樣：
- 正常速度的 value 放在另外一個 address 上，人物移動時從這 address 拿值，取得人物速度。
- 當偵測到 shift 按下時，進入另一個邏輯，這邏輯有兩種可能
   - 可能是正常情況下的人物速度 -2
   - 或是 shift 的速度也是放在另外一個 address 存著。

### 追查存取這些 address 的代碼段
接下來會使用兩個 CE 的屌功能：
1. `Find out what access this address`
2. `Find out what writes to this address`

![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-14.png)

依據目前的狀況，我猜這 3 個 address 有兩種可能
1. 從真正的正常速度或 shift 速度的 variable 拿值，複製過來的，作為人物速度相關處理的 tmp value : 使用 Find out what writes to this address
2. 人物真的要移動時，真的會套用的實際速度的 value : 使用 Find out what access to this address

不如就先追第 2 種的狀況看看，追了發現都是同一段程式碼在處理 = =...不論我是否有按 shift。
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-15.png)
那第 1 種呢，也可發現 3 個 address 都是相同的幾段程式碼在存取...不論我是否有按 shift。
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-16.png)

追一下第 1 種的代碼段，看起來 eax 是指向某個結構，依據 ecx,edx 值判斷要將 ebp (caller 呼叫這 func 時傳入的參數) 塞到 eax 物件中的哪一個 member variable。
推測 eax 應該是地圖角色的 "當前瞬間狀態"，eax + 0xA * 4 是 "當前速度"。
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-17.png)

所以 ebp 可能就是真正的移動速度，追一下 stack 看看 caller 是誰，有兩個方法
1. 下 condition 斷點，看一下斷點發生時的 call stack，找出 caller。
2. 剛剛 Find out what access this address 的介面中，有一個視窗會顯示 access 時的 stack 內容

放上兩種方法的截圖：
1. 設個準確的條件，避免斷點到除了分配人物速度時的其他狀況。
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-18.png)
2. 或是看一下 access 時的 stack 內容，因為 CE 很屌，我們可以一眼看出來哪些是指向程式碼段的，比較懶一點不去追 stack 多大的話，通常比較上面的那一個會是當前 func 的 return address(只要 caller 參數不是傳 func pointer 之類的)，於是我們就找到 caller 是誰了 (`RGSS300.dll + 0x2D36E`)。
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-19.png)

### 追查 caller 呼叫進入這代碼段時傳入的參數
看一下 caller 這邊 (`RGSS300.dll + 0x2D36E`) 怎麼呼叫的，反推回呼叫 func 時這些 register 的值。
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-20.png)
算了我好像推不回來各 register 的值，直接下斷點。這裡下斷點有兩個方式
1. 設 condition 斷點 (e.g. 像是 ECX == 0x2fc1 && EDI == 0xb 之類的，注意 register 要大寫如 EAX，不行小寫如 eax) 
2. 做 AOB injection，裡面寫幾個 cmp 條件，斷點下在符合條件時會走到的指令上

因為第 1 種的斷點下了之後，遊戲會瞬間 lag 到不可思議，我猜他的 softw/blogare bp 是用 lua 做的。所以我推薦第 2 種，作法如下
1. Tools -> Auto Assemble -> template 選 AOB injection
2. 開始寫 asm
``` asm
[ENABLE]
aobscanmodule(DebugNormalSpeed,RGSS300.dll,83 C0 FC 83 C2 0C 89 16 89 46 04 8B) // should be unique
alloc(newmem,$1000)

label(code)
label(return)

newmem:
  cmp edi, 0xB
  jne code
  cmp ecx, 0x2FC1 // 這裡可以下一個斷點
  jne code 
  nop // 或下在這裡也行

code:
  add eax,-04
  add edx,0C
  jmp return

DebugNormalSpeed:
  jmp newmem
  nop
return:
registersymbol(DebugNormalSpeed)

[DISABLE]

DebugNormalSpeed:
  db 83 C0 FC 83 C2 0C

unregistersymbol(DebugNormalSpeed)
dealloc(newmem)
```
3. 存到 cheat table 中

這樣就寫好了一個簡單的 injection 了，隨時可以開啟關閉 cheat table 上你剛寫好的 asm script。
開啟時，CE 會分配 asm 一塊 memory，並將符合 AOB 特徵的程式段改成 jmp 到自製 asm 的 address。
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-24.png)
大概就是這樣子，可以做很多有的沒的事情，總之下個斷點簡單檢查一下，可以知道
1. edi = 0xb 時 ecx 都是 0x2FC1
2. 這時 eax = 0x326010c (被傳入 func 的參數 - edi 值為 [eax-4])

![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-25.png)
所以 `0x326010c - 4` 是 normal speed 的 adr 嗎？看了一下這個值會一直亂跳，看來不是，馬的。他只有在 `edi = 0xb` 和 `ecx = 0x2fc1` 時才會是正常速度值 `0xB`。
這要怎麼追，我破防了。

### 從人物 x 軸 address 切入
因為暫時不知道怎麼往回追了，先改用其他方式試試。
假設地圖角色的各項狀態數值，是存在一個 struct 裡的，member 可能包括 xy 軸、可否移動、可否無視怪物、人物移動速度等等，那我們先抓出 xy 軸，然後觀察附近 memory 的值的變動，說不定能找出人物移動速度？！

首先人物隨便移動幾下，過濾出 x 軸 adr。然後看一下附近的值，還真的有跟我們剛剛看到一樣是 0xB 的 value，而且按下 shift 也會變成 0x9，可能真的跟速度有關，在這邊先幫他命名為 speed。
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-26.png)
但我們調整 speed 的值，人物的移動速度仍然不會變，而且會被改回 0xB。使用上一段的方法找出讀取寫入的代碼段，發現還是回到了相同的地方 `RGSS300.dll + 0x2D369`，破防...。

### 反向找出各 register 的內容和相關的代碼段
為方便，在這邊先放上`RGSS300.dll + 0x2D34D` ~ `RGSS300.dll + 0x2D369` 的代碼段
```
RGSS300.dll+2D348 - jmp RGSS300.dll+2D1D6
RGSS300.dll+2D34D - mov eax,[esi+04] // 段落的開始
RGSS300.dll+2D350 - mov ecx,[edx+04]
RGSS300.dll+2D353 - mov ebx,[edx+08]
RGSS300.dll+2D356 - mov edi,[eax-04]
RGSS300.dll+2D359 - add eax,-04
RGSS300.dll+2D35C - add edx,0C
RGSS300.dll+2D35F - mov [esi],edx
RGSS300.dll+2D361 - mov [esi+04],eax
RGSS300.dll+2D364 - mov esi,[esi+14]
RGSS300.dll+2D367 - push edi
RGSS300.dll+2D368 - push ecx
RGSS300.dll+2D369 - call RGSS300.dll+285E0
RGSS300.dll+2D36E - mov esi,[esp+3C]
RGSS300.dll+2D372 - add esp,08
RGSS300.dll+2D375 - jmp RGSS300.dll+2D1D6 // 段落的結束，跳回 RGSS300.dll+2D1D6
```
總之，目標是找出到底從哪邊進來 `RGSS300.dll + 0x2D34D` 的，這樣我就有線索找出 `edi = 0xB` 之前的 esi、edx 等等 register 是被誰分配的，經過一番快速的分析，大概就像下面，感覺類似做某件事時會呼叫對應的 func pointer(`RGSS300.dll + 0x2F598`) 這樣。

![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-27.png)

下記憶體寫入斷點，追寫入 esi+4 (0x3DCB114) 且 condition 是 `readInterger(0x3DCB114) == 0xB` 時的位置，找到了 `RGSS300.dll+2D26E`。
```
RGSS300.dll+2D241 - mov eax,[edx+08]
RGSS300.dll+2D244 - mov ecx,[edx+04] // 3. 想記錄下來的值
RGSS300.dll+2D247 - add edx,0C
RGSS300.dll+2D24A - mov [esi],edx
RGSS300.dll+2D24C - push eax
RGSS300.dll+2D24D - mov eax,[esi+18]
RGSS300.dll+2D250 - mov edx,ecx // 2. 在這下 AOB injection，看看 edx + 0x4 和 ecx 的值這時為何，如果是 0xB 就中了
RGSS300.dll+2D252 - mov ecx,[ebp+08]
RGSS300.dll+2D255 - call RGSS300.dll+28140 // 1. 裡面有一個 sar edx,1，表示取得 edx/2，推測 edx 是關鍵值
RGSS300.dll+2D25A - add esp,04
RGSS300.dll+2D25D - mov ecx,[esi+04]
RGSS300.dll+2D260 - lea edx,[ecx+000000B4]
RGSS300.dll+2D266 - cmp edx,esi
RGSS300.dll+2D268 - jae RGSS300.dll+2F48D
RGSS300.dll+2D26E - mov [ecx],eax // 這裡寫入了 0xB 的值給疑似是代表 normal speed 的暫存變數(0x3DCB114)
RGSS300.dll+2D270 - add dword ptr [esi+04],04
RGSS300.dll+2D274 - jmp RGSS300.dll+2D1D6

```
如下為針對 2. 的 AOB 腳本，這腳本做的事就是額外宣告兩個變數，將這瞬間 ecx 和 edx+4 的值存下來，以便斷點命中時可以確認，不然值會被後來的指令改掉。
- AOB 
```
[ENABLE]
aobscanmodule(HookSpeedAlloc_lv2,RGSS300.dll,8B D1 8B 4D 08 E8 E6) // should be unique
alloc(newmem,$1000)
alloc(variables, 0x100)

label(ecx_val)
label(edx_add_4_val)
label(code)
label(return)

variables:
ecx_val:
  dd 0
edx_add_4_val:
  dd 0

newmem:
  mov [ecx_val], ecx
  mov [edx_add_4_val], edx
  add [edx_add_4_val], 4
code:
  mov edx,ecx
  mov ecx,[ebp+08]
  jmp return

HookSpeedAlloc_lv2:
  jmp newmem
return:
registersymbol(HookSpeedAlloc_lv2)

[DISABLE]

HookSpeedAlloc_lv2:
  db 8B D1 8B 4D 08

unregistersymbol(HookSpeedAlloc_1v2)
dealloc(newmem)
dealloc(ecx_val,$4)
dealloc(edx_add_4_val,$4)

```
- (或者也可以用 CE 跑 lua 腳本，大概的寫法像下面這樣)
``` lua
// 這是我之前拿來追角色狀態寫的腳本
character = 0
function debugger_onBreakpoint()
    if (EIP == 0x1008D272 ) then
      character = ESI
    end

    if(EIP == 0x1008D28E) then

        if(ESI==0x1445dff8) then
            printf("tirma: 0x%x",character)
            return 1
        end        
    end
    return 0        
end
debug_setBreakpoint(0x1008D272)
debug_setBreakpoint(0x1008D28E)
```

但是，下斷點後我發現完全沒命中過，靠杯，原來 2. 這邊根本不會被走到。

多下了幾次斷點反追代碼段後，感覺應該是下面這樣子，實際是從 `RGSS300.dll+2D25D` 開始。
```
RGSS300.dll+2D25D - mov ecx,[esi+04] // 從這邊開始會被走到
RGSS300.dll+2D260 - lea edx,[ecx+000000B4]
RGSS300.dll+2D266 - cmp edx,esi
RGSS300.dll+2D268 - jae RGSS300.dll+2F48D
RGSS300.dll+2D26E - mov [ecx],eax // 這裡寫入了 0xB 的值給疑似是代表 normal speed 的暫存變數(0x3DCB114)
RGSS300.dll+2D270 - add dword ptr [esi+04],04
RGSS300.dll+2D274 - jmp RGSS300.dll+2D1D6 // 回去一開始
```

### 靜態逆向
那`RGSS300.dll+2D25D`是從哪裡 jmp 過來的呢？翻了一遍 func pointers (`RGSS300.dll + 0x2F598`)的內容，沒有 pointer 的值是 `0x102D25D` ，看來是無法知道是哪裡來的了...只好召喚出好朋友 IDA 來看一下 cross reference 了。

首先用 x32dbg 的 Scylla 把 module(RGSS300.dll) 當前的記憶體 dump 下來。
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-38.png)

> 要特別從記憶體 dump 的原因是如果直接把 RGSS300.dll 丟給 IDA，會發現`RGSS300.dll+2D25D`這邊的指令都還沒被載入，這是 runtime 才會載入的實作內容。

餵給 IDA，得到這邊的 cross reference 了。

```
seg000:1002D25D loc_1002D25D:                           ; CODE XREF: sub_1002D1C0+1E3↓j
seg000:1002D25D                                         ; sub_1002D1C0+59D↓j ...
seg000:1002D25D                 mov     ecx, [esi+4]
seg000:1002D260                 lea     edx, [ecx+0B4h]
seg000:1002D266                 cmp     edx, esi
seg000:1002D268                 jnb     loc_1002F48D
seg000:1002D26E
seg000:1002D26E loc_1002D26E:                           ; CODE XREF: sub_1002D1C0+254↓j - 0x1002D409
seg000:1002D26E                                         ; sub_1002D1C0+2D2↓j - 100 - 0x1002D492
seg000:1002D26E                 mov     [ecx], eax
seg000:1002D270                 add     dword ptr [esi+4], 4
seg000:1002D274                 jmp     loc_1002D1D6

```
至於 IDA decompile 後的結果我就沒附上了，大概就是這似乎是個超長一串的 while + switch loop，因為實在太長一大串，我自認搞不懂就先不看了。

先對相關的代碼段下幾個條件斷點。
- 對 0x1002d409 下條件斷點 EAX == 0xB，斷不到。
- 對 0x1002d492 下條件斷點 EAX == 0xB，斷不到。
- 對 0x1002d268 下條件斷點 EAX == 0xB，斷不到。

wtf...這什麼靈異現象，怎麼都斷不到的 na？

### 從人物 x 軸 address 切入 (again)

![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-28.png)
因為追不出甚麼結果，我決定重新來，說不定 0xB 這個值根本就錯了，其實速度不是 0xB = =。
重新再找了一次 x 軸，這一次，先鎖值確認這個 x 軸真的 work，然後看了一下地址 `0x10FE5D94`，嗯?!對照 `RGSS300.dll` 的 module base `0x10000000`，這 address 好像是在 `RGSS300.dll` memory 段中耶，難道其實 x 軸之類的 info 是 module 的 global variable 嗎? 那我剛剛找到的 x 軸莫非是 tmp var？會不會之前找錯目標了，看一下周遭 memory 的變化，亂試一下。

結果發現 +1C 的變數代表速度...，改成 0xB、0xD 之類的值後，人物速度真的會變快。
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-29.png)

至此完成了人物速度位置的尋找。定位到實際的 pointer 後就可以加一些小功能了，例如設定 hotkey 方便隨時調整自己的移動速度之類的。

## 修改移動速度 (cont'd)
耐心看到這邊的各位，心裡應該在想："怎麼可能這麼會猜，還都給你猜中，通靈校隊喔？你這感覺很大一部分都像賽到的，運氣流的是不是？"。
沒錯，其實當初在上一章節中，我在反追 `0xB` 這個代表人物速度的值到底哪裡來的時候，就卡關卡到放棄了 (而且事實速度也不是 0xB...是 0x9)。放棄的原因是我對整個遊戲機制的實作處理真的沒有概念，除了不確定自己過濾出的值對不對，想要往回追也不知道要往哪裡追。逆向過的應該心有戚戚焉(吧？)，遇到動態分析時大海撈針的不確定感，以及靜態分析 (e.g. IDA) 那硬核的組語和 decompiled c++ 內容，我都在想我是不是在自虐。

所以我放棄硬剛，轉向 google 找尋其他出路。
1. 搜 `RGSS300 dll` 時，發現這個遊戲是用 RPG Maker VX Ace 做的
2. 搜 `RPG Maker VX Ace Cheat` 之類的，發現有人寫了一個 RMVA(RPG Maker VX Ace) 用的 Cheater ([RMVA Cheat Menu](https://f95zone.to/threads/rpg-maker-vx-ace-cheat-menu.20567/) ([載點](https://files.sakana.tw/blog/Sequel-Blight-Reversing/RPG%20VX%20ACE%20Cheat%20System.7z)))

### RMVA Cheat Menu
    這插件的本名應該叫 RMVA Cheat System 才對，純粹因為是他有 menu，所以我都叫他 cheat menu 或 cheater。
這個 Cheater 大概長這樣，第一次執行時會先進行安裝 Cheat Menu，接下來就可以 F8 叫出作弊選單了。
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-30.png)
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-31.png)

來了，這個 Cheater 就是我的研究目標了，至少研究這個東西怎麼做，應該比逆向遊戲本身還簡單吧。
分成幾個階段
1. 初步理解這 Cheater 內各檔案的作用
2. 研究第一次執行的安裝機制，抓出安裝的內容
3. 理解這個 Cheater 如何實作的


### Cheater 中各檔案的作用

```
.
├── Data_Cheat
│   ├── 002_Cheat_All.txt
│   ├── 002_Cheat_Text_Cheat.txt
│   └── Scripts.rvdata2
├── Game_Cheat.exe
├── Game_cheat.ini
└── System
    └── RGSS301.dll
```
光看檔案名稱，可以感受到 RGSS301.dll 以及 Data_Cheat 下的檔案 002_Cheat_All.txt、002_Cheat_Text_Cheat.txt、Scripts.rvdata2 應該是放作弊功能的實作。
而 Game_cheat.exe、Game_cheat.ini、不排除是重新包裝過的遊戲入口點，是為了成功載入 Data_Cheat 的內容而附上的。


#### Game_cheat.exe、Game_cheat.ini
Game_cheat.ini 內容
``` ini
[Game]
RTP=RPGVXAce
Library=System\RGSS301.dll
Scripts=Data_Cheat\Scripts.rvdata2
Title= RPG VX ACE Cheat System : dldb.info/blog/506
```

剛好遊戲本身也有一個 Game.ini，對比一下
``` ini
[Game]
RTP=RPGVXAce
Library=System\RGSS300.dll
Scripts=Data\Scripts.rvdata2
Title=SEQUEL blight ver.2.10
Description=(no description)
CreationDate=1401520961
```
看起來 Game_cheat.ini 是用來指定 `Game_Cheat.exe` 載入 Cheater 相關模組的。(e.g. `RGSS301.dll`、`Scripts.rvdata2`)
咦？感覺 Game_Cheat.exe 是不是沒有用啊？試試將遊戲自帶的 Game.exe 改名成 `Game_Cheat.exe` 執行看看，作弊的效果確實沒有影響。

總結一下，Game_Cheat.exe == Game.exe，而載入設定檔的行為如下，看起來是依據主程式的檔名來決定的。
- 執行 Game_Cheat.exe 會載入 Game_Cheat.ini
- 執行 Game.exe 會載入 Game.ini
- Game.exe 改名成 Game123.exe 會直接開不了

具體可參考 [RGSS 规格](http://miaowm5.github.io/RMVA-F1/RPGVXAcecn/rgss/rgss.html)，這篇對 RGSS3 包括 `Game.ini` 做了簡單的介紹。

#### RGSS301.dll
遊戲本身自帶的為 RGSS300.dll，跟這個 RGSS301.dll 有啥差別呢，直接餵狗似乎只差在語言而已。
- 300 = 日文版和汉化日文版
- 301 = 英文版

將 Game_cheat.ini 的 Library 改為 `System\RGSS300.dll` 後，仍然能正常運作，看起來 dll 也是不影響的。甚至直接使用 [RMVA](https://www.rpgmakerweb.com/products/rpg-maker-vx-ace) 產生一個 demo 遊戲，將產生的遊戲包當中的 Game.exe 和 RGSS301.dll 拿來用都可以。
在使用 RMVA 產生遊戲的同時，我也確定了這個遊戲引擎的支援語言是 ruby，推測 RGSS300.dll 就是用來載入 ruby 腳本之類的 dll 模組。

參考資料：[[已经解决] 关于RGSS301.dll 以及更高版本的dll](https://rpg.blue/thread-411720-1-1.html)

#### 002_Cheat_All.txt、002_Cheat_Text_Cheat.txt、Scripts.rvdata2
參考 [RGSS 规格](http://miaowm5.github.io/RMVA-F1/RPGVXAcecn/rgss/rgss.html)，`Scripts.rvdata2` 是加密後的程式邏輯。至於`002_Cheat_All.txt`、`002_Cheat_Text_Cheat.txt`，快速喵了一下應該是作弊選單邏輯，語言沒意外是 ruby。

#### 重點整理
1. Game_Cheat.exe 和 RGSS301.dll 跟遊戲本體內容沒有差別，而 Game_Cheat.ini 則是用來指定載入模組，跟作弊選單本身無關。
2. `Scripts.rvdata2`、`002_Cheat_All.txt`、`002_Cheat_Text_Cheat.txt` 應是作弊選單的核心實作

### 拆解安裝機制及內容
由先前整理的資訊，接下來重點將會放在分析 `Scripts.rvdata2`、`002_Cheat_All.txt`、`002_Cheat_Text_Cheat.txt` 上。

第一次執行的安裝操作應該是 `Scripts.rvdata2` 的行為。
解密 rvdata2 的內容可以使用
1. 使用人家寫的 vim plugin
	- [RGSS Script Editor Plugin - An RGSS Editor Plugin : vim online](https://www.vim.org/scripts/script.php?script_id=4367)
    - 網頁好像掛了，最近看是沒辦法抓
2. 自己寫一個 RMVA 的 ruby 腳本進行讀取
	- (這是逆向 Cheater 中 `Scripts.rvdata2` 的實作後發現的，使用 RGSS::load_data + Zlib::inflate)
	- [Method: RGSS#load_data — Documentation for openrgss (0.1.5) (rubydoc.info)](https://www.rubydoc.info/gems/openrgss/RGSS:load_data)
3. 使用人家寫的專案 [rvpacker](https://github.com/Solistra/rvpacker)
    - 我弄了一個小時就是灌不起來，放棄。推測是這個專案支援的 ruby 版本太舊。
4. 拿 RMVA 內建的編輯器來用
    - 建立遊戲專案後，直接把 `Scripts.rvdata2` 複製一份取代遊戲專案中的版本，把腳本編輯器打開後就能見到解密後的 `Scripts.rvdata2` 內容了。
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-32.png)

以上幾種方法我都試過，我覺得 4. 是最方便的，最不用掃雷除錯 = =。附上拆出來的腳本，
``` ruby
if FileTest.exist?("002_Cheat.txt")
  change_script_name = File.read("002_Cheat.txt")
else
  change_script_name = false
end

change_script_text_1 = false
change_script_text_2 = false

if FileTest.exist?("Data_Cheat/002_Cheat_All.txt")
  file = File.open("Data_Cheat/002_Cheat_All.txt")
  change_script_text_1 = file.read
  file.close
end

if FileTest.exist?("Data_Cheat/002_Cheat_Text_Cheat.txt")
  file = File.open("Data_Cheat/002_Cheat_Text_Cheat.txt")
  change_script_text_2 = file.read
  file.close
end

change_text = []
change_text[0] = ["Data/Items.rvdata2","Data_Cheat/Items.rvdata2"]

  def inflate(string)
    zstream = Zlib::Inflate.new()
    buf = zstream.inflate(string)
    zstream.finish
    zstream.close
    buf
  end
  def deflate(string, level = Zlib::BEST_COMPRESSION)
    z = Zlib::Deflate.new(level)
    dst = z.deflate(string, Zlib::FINISH)
    z.close
    dst
  end

  $cheat_scripts = load_data("Data/Scripts.rvdata2")
  k = 0
  k2 = 0
  $cheat_save_scripts = []
  
  $cheat_scripts.each do |unScript |
    k = k + 1
    nom = unScript[1]
    code = inflate(unScript[2])
      
    code.gsub!("\x0D\n") { "\n" }
    @file = File.open("009_script.txt","w")
    @file.write(code)
    @file.close
    @file = File.open("009_script.txt","r")
    code = @file.read
    @file.close
    for i in change_text
      code.gsub!( i[0] ) { i[1] }
    end
    if change_script_name
      if unScript[1] == change_script_name
        msgbox(change_script_name + " Cheat Add")
        $cheat_save_scripts.push(["","",deflate(change_script_text_1)]) if change_script_text_1
        $cheat_save_scripts.push(["","",deflate(change_script_text_2)]) if change_script_text_2
      end
    else
      code.gsub(/[   ]*rgss_main \{ SceneManager.run \}/) do
        msgbox(unScript[1] + " Cheat Add")
        $cheat_save_scripts.push(["","",deflate(change_script_text_1)]) if change_script_text_1
        $cheat_save_scripts.push(["","",deflate(change_script_text_2)]) if change_script_text_2
      end
    end
    
    $cheat_save_scripts.push(unScript)
  end
      
  save_data($cheat_save_scripts, "Data_Cheat/Scripts.rvdata2")
```

簡單來說，這部分做的事情是將 `Data_Cheat/002_Cheat_All.txt` 和 `Data_Cheat/002_Cheat_Text_Cheat.txt` 的內容串接到遊戲原本的 `Data/Scripts.rvdata2` 後，變成一個新的 `Data_Cheat/Scripts.rvdata2`。
不得不說，我覺得這個方法蠻高明的，無痛的在遊戲本身的內容上加入自己的新東西。若使用者後來不想用這作弊功能，改為執行 Game.exe 就可以了。

### 作弊選單的實作
總算到理解實作的環節了，雖然一開始的目標是修改人物速度，但我同樣很好奇他一些功能是怎麼實現的，所以多訂了幾個目標
1. 如何把作弊選單載入到遊戲中
2. 看看人物速度怎麼改的

#### 將選單載入到遊戲中
搜尋叫出作弊選單的按鍵 `F8`，可知當按鍵按下時，會交由 `Scene_Map` 處理，大概是做了這幾件事，
1. 初始化作弊選單的視窗 (e.g. `Scene_Cheat`, `Window_CheatCommand`)
2. 設定每個選項個別的 handle，當選擇對應的選項時，其 handle 會創建新的子選單並套用設定。 (e.g. `Scene_Cheat`, `set_handler`)

![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-33.png)
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-34.png)

不過，當去比對遊戲本體的腳本內容時，會發現 `Scene_Map` 是本來就有的 class。
如果跟我一樣只有寫過 c++、python 之類的朋友，應該會黑人問號，重複定義 class 會噴錯吧？其實這個操作在 ruby 叫做 [open class](https://stackoverflow.com/questions/3822471/are-you-allowed-to-redefine-a-class-in-ruby-or-is-this-just-in-irb)，可以往已經定義好的 class 中打 patch，太淫蕩了...。
儘管身為 c++ 開發者的我無法適應這個特性，甚至還想忘記我看了什麼，但也確實是有了這個特性，這個作弊腳本才能在保留遊戲原先邏輯的前提下加東西，牛逼。

#### 調整人物速度的選項
翻了一遍跟 speed 選單有關的實作，基本上流程是這樣的
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-35.png)
1. 開啟 "Move Speed" 選單，呼叫 edit_move command (此 command ext = "move")。
2. edit_move 呼叫 Window_ChaatMove 處理設定速度 (4~9) 的選項，在這邊選項是 4。
3. Window_CheatMove 呼叫 cheat_change("move", 4)，設定速度值。
4. cheat_change 跟據傳入的 ext 來設定對應的屬性值，在這邊設定 `@move = 4`。

大致了解選單的邏輯後，開始追 move 值的改動會如何套用到遊戲人物上，這部分蠻單純的，就是覆寫原本的 `real_move_speed`，改為使用作弊選單設定的 move 值。(下圖左為作弊選單，圖右為原本邏輯)
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-36.png)
(原本的`real_move_speed`跟我在使用 CE 逆向時的猜測之一蠻接近的，還真的是人物速度 +1，再細追一下會發現這個值主要影響每一幀更新時人物的移動距離。)

### 回到 CE
目前已經有兩個方法可以達成這件事了
1. CE
2. Cheat Menu

跟一開始不一樣的是，我們手上有 source 了，在使用 CE 逆向的階段時，有很多不確定的 struct member，現在可以無恥的跟 src 比對， 一個一個找出這些值到底是啥。
直接來比對 `Game_CharacterBase` 和 CE 看到的 struct 內容。
![](https://files.sakana.tw/blog/Sequel-Blight-Reversing/sequel-blight-37.png)
可見數字是以 2x + 1 去存，而 bool 是以 2x 去存。並且把這些值玩一遍之後，我感覺比較實用的有 move speed、through mode (不會撞怪、不受障礙的移動)。

## 總結
以上就是逆向這遊戲的過程。
通過這篇文，不專業的記錄了用 CE 逆向遊戲的各個階段
1. 使用 CE 抓出記憶體位置
2. 使用 CE 反追寫入讀取的程式段
3. 使用 IDA、x64dbg 等動態、靜態工具輔助分析
4. 放棄分析，使用通靈，猜到賺到

以及通過蒐集資訊，從其他角度切入，盡量避開硬剛記憶體的方式
1. 研究其他人寫好的工具
2. 熟析對應的遊戲引擎的大致運作方式
3. 通過 1、2 的資訊，嘗試用較高的層次來分析遊戲

## 心得
一開始只是想說記錄一下，將近一年前逆這個遊戲逆的很痛苦，不停撞牆的過程。

雖然通過 CE 純逆向的成果並不是很好，但過程我覺得還算有趣，像是變數追到一半邏輯就斷掉了怎麼辦，如何通過 CE 強大的工具找到寫入點、找到變數真正的指標之類的，有不少實際做之前都不會想到的騷操作。

但逆向的過程要解釋清楚真的比想像中還困難，就我自己的感覺，逆向時會一直有不同支線冒出來，還會不停的做假設，有時候追不同地方的邏輯時，突然就發現接起來了...這種看似毫無邏輯的事也會發生。要用一直線的文章思路記錄下來真的是難倒我，寫到一半就感覺自己開錯坑了。

不過寫文章的過程中也意外獲得了一些收穫，因為這篇是一邊寫一邊逆的，其實會花更多的時間審視每一步的目的，比較確定在做什麼，因為想得比較久，也會想到之前分析時沒想到的操作，蠻不錯的。

想起以前想學外掛，逛外掛論壇時看到有些人問大神，人物無敵的地址到底怎麼找到的，大神說這是一個感覺，很難解釋，然後看起來也沒打算解釋XD。我心裡就在想 "靠北阿，到底是什麼感覺，天生的感覺嗎...我真的感覺不到任何的感覺耶..."。
寫這篇文章時突然想起這事，如果有跟我一樣找不到感覺的朋友，希望看完這篇後能找到一點感覺。



