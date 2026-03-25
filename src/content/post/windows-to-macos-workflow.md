---
title: "分享我從 windows 轉換至 mac OS 後，維持相近的操作習慣、工作流程的方法"
description: "分享從 Windows 轉換至 macOS 後，透過 karabiner、yabai、raycast 等工具維持相近操作習慣與工作流程的方法。"
publishDate: "2025-01-30"
tags: ["workflow", "mac", "windows"]
---


## 我的操作環境及習慣
我上班工作時是 windows + wsl，下班回家則是 mac OS 或是 windows，日常工作都是能用鍵盤就用鍵盤完成，高度相依於鍵盤快捷鍵。

以下簡介我如何在使用同樣鍵盤同樣 layout 的前提下，在兩種系統間保持同樣的操作習慣，而不會由於快捷鍵不一樣等問題導致精神分裂，並且分享一些 mac 下的我覺得頂級的小工具


## 如何從 windows 過渡到 mac 並維持相近的操作習慣

一般來說，日常基本使用行為包括瀏覽器搜尋資料(chrome)、用筆記軟體記筆記(e.g., obsidian, heptabase)、開其他有的沒的小工具(e.g., todolist, amazon music)
這些行為的操作方式與以下關鍵元素息息相關
- keybinding、視窗間的切換方式
- 輸入法
只要這些元素差異不大，或是 mac 更直覺，就不會水土不服。不幸的是有些差異 (尤其是 keybinding) 反直覺到無法忽視，以下將說明我如何以個人舒適的角度調整這些元素的。

### 調整 keybinding
#### 兩個系統的 keybinding 有微妙的差異
大家若用過這兩個系統，應該知道在  mac os 的 command 就像是 windows 的 ctrl key，相當單純。
應該是這樣才對...

實際上是很像，痛苦的就是真的有些地方不像，跟 command, ctrl 鍵相關的 keybinding 我理不清一個邏輯，讓我狂搞混按錯，我花一整天的時間 tune 這邊的操作習慣

以下是我最常用的幾個操作，windows 跟 mac 的比較
- 視窗
	- 切換視窗： alt tab -> command(win) tab
		- 不過這按鍵 mac os 切換的不是視窗，是切換 app，要切換視窗更準確的對應按鍵是 ctrl f4...
	- fuzzy search 切換視窗 (switcheroo) alt space -> command(win) space (spotlight)
	- 關閉視窗：ctrl-w -> command(win)-w / command(win)-q (這是完全關閉 app)
	- 快速開啟：分成兩個部分
		- fuzzy search 應用程式(開始功能表) win -> command(win) space (spotlight)
- 文字操作
	- 複製貼上：ctrl c/v -> command(win) c/v
	- 刪除 word：ctrl backspace -> alt backspace
	- 切換輸入法：win(command) space -> command space
- in chrome
	- 切分頁：ctrl tab -> ctrl tab
	- 關閉分頁：ctrl w -> command(win) w
	- (這是最吐血的，應該不少人按錯過...切分頁按成 command tab...為什麼關分頁是用 cmd，切換分頁卻是 ctrl ?
- in terminal(e.g., iterm2, wezterm)
	- ctrl 相關操作 -> 一樣 map 對應 ctrl
- 切換虛擬桌面：win(command) 左右方向鍵 -> ctrl 左右方向鍵

其實問題最大的就是 chrome 而已，對我來說真的反直覺。總之知道哪些不一樣了，可以開始調整了。

#### 將同類型的按鍵統一
可以發現
1. mac command 基本上跟 windows ctrl 差不多
2. mac ctrl 跟 windows win 差不多
3. mac alt(option) 跟 windows alt 完全他媽不一樣

第一步：對調 cmd-ctrl
- 進 settings -> keyboard shortcut，將 command ctrl 對調 (或是使用 karabiner 也可以)
第二步：spotlight 改成 alt space，輸入法切換改成 ctrl space
第三步：調整視窗切換 (cmd-tab) 及瀏覽器 tab 切換 (ctrl-tab) 的 mapping


第一、二步可從 settings 調整
第三步
- alt tab 加入 alttab 來做視窗切換
- cmd-tab 是 system wide 的 general shortcut，無法透過設定拿掉，需要使用 karabiner 調整
	- 將 cmd-tab 映射到 ctrl-map，從此再也不存在 cmd-map
```
{
    "description": "Change command-tab to control-tab",
    "manipulators": [
        {
            "from": {
                "key_code": "tab",
                "modifiers": {
                    "mandatory": ["command"],
                    "optional": ["any"]
                }
            },
            "to": [
                {
                    "key_code": "tab",
                    "modifiers": "control"
                }
            ],
            "type": "basic"
        }
    ]
}
```


以下是調整後的 mapping 表，還有一些小缺點，在工具介紹的章節會再微調

|          |                       | windows                | mac (微調後)        | mac (微調前)             |
| -------- | --------------------- | ---------------------- | ---------------- | --------------------- |
| 視窗       | 切換視窗 (一般)             | alt-tab                | alt-tab (alttab) | cmd-tab               |
|          | 切換視窗 (fuzzy search)   | alt-space (switcheroo) | -                | cmd-space (spotlight) |
|          | 關閉視窗、分頁               | ctrl-w                 | ctrl-w/q         | cmd-w / cmd-q         |
|          | 開啟應用程式 (fuzzy search) | win                    | -                | cmd-space (spotlight) |
| 文字操作     | 複製貼上                  | ctrl-c/v               | ctrl-c/v         | cmd-c/v               |
|          | 刪除 word               | alt-backspace          | -                | alt-backspace         |
|          | 切換輸入法                 | win-space              | cmd-space        | cmd-space             |
| chrome   | 切換分頁                  | ctrl-tab               | ctrl-tab         | ctrl-tab              |
|          | 關閉分頁                  | ctrl-w                 | ctrl-w           | cmd-w                 |
| terminal | 任何 ctrl 相關操作          | ctrl-?                 | cmd-? (x)        | ctrl-?                |
| 視窗管理     | 切換 workspace          | win-左右                 | cmd-左右           | ctrl-左右               |

#### Reference
- [AltTab - Windows alt-tab on macOS](https://alt-tab-macos.netlify.app/)
- 如何 disable cmd-tab
	- [macos - Change Command-Shift-Tab on OSX - Super User](https://superuser.com/questions/808078/change-command-shift-tab-on-osx)
		- [Karabiner-Elements](https://karabiner-elements.pqrs.org/)
	- [macos - Is there any program or way to make Mac OS X's ⌘-Tab behave like Windows' Alt-Tab? - Super User](https://superuser.com/questions/193922/is-there-any-program-or-way-to-make-mac-os-xs-tab-behave-like-windows-alt-ta)
	- [keyboard - Override the command+tab shortcut in macOS Catalina when using Chrome - Ask Different](https://apple.stackexchange.com/questions/401942/override-the-commandtab-shortcut-in-macos-catalina-when-using-chrome)
		- `According to Apple: "[...] you cannot define keyboard shortcuts for general purpose tasks such as opening an app **or switching between apps**."`
		- [Use macOS keyboard shortcuts - Apple Support (EG)](https://support.apple.com/en-eg/guide/mac-help/mchlp2262/mac)
	- 使用 karabiner-elements 把 command tab 給取代成 ctrl tab，實測可行

### 輸入法
Mac OS 輸入法

個人覺得原生的中文輸入法還不差，屌打微軟，可惜的是按着 shift 輸入符號時，都是全形，可以是半形就更好了。
且我的問題是沒有caps lock，因此切換要按ctrl space，但 ctrl-space 與 IDE 中自動補全的 shortcut 衝突，只好爬文找其他輸入法方案
- 小麥、rime、Yahoo
由於近幾年來許多人都使用 rime，影片教學多且許多開源字典，實際使用手感也不錯，且支援 shift 切換中英，就決定用這個了。

#### 拼音、注音？
> Rime 自帶的輸入法清單：[UserGuide · rime/home Wiki](https://github.com/rime/home/wiki/UserGuide)

在研究 rime 設定的過程中，發現了許多輸入方案可以使用，機會難得就順便研究，因爲我被微軟注音的爛選字搞得很痛苦，決定趁這機會試試看把這卡住的點給打通。
拼音
- 分全拼跟雙拼
	- 雙拼要背按鍵，雖然它概念跟注音很接近，但學習成本有點高，略過。
- 最有名的霧凇拼音跟預設的拼音
	- 霧凇雖然有內建整合了不同詞庫還能不切換的拼英文單字，但實際用起來對我來說這些額外的東西其實不常使用，有點雞肋
	- 可能預設的拼音就夠了
注音 (bopomofo)
- 這個注音，是真的 dope，是像手機那樣子，打聲母就對應出詞組，個人感覺這樣子的打字體驗順暢了應該有兩倍左右...
- 我用過兩套注音，rime 預設的和洋蔥拼音，個人認為洋蔥拼音選字比較準 (比較適合台灣人)，rime 預設注音有時候會找不到字，但換一個奇怪的念法卻可以找到那個字(暫時沒想到例子)，感覺是因為詞庫使用中國拼音。
其他
- 無蝦米跟倉頡，參考大部分使用者的心得，應該是最快的輸入法，但後來想想還是放棄，我連雙拼都不敢碰了，就別想拆部首搞聯想這一套了

所以在 rime 這個輸入法核心的基礎上，不論使用拼音或注音的方案，我認爲熟練之後速度不會差太多，但是拼音因爲沒有聲調，可能會花更多時間在選詞上面。

#### Rime 安裝及 config 設定
Rime 安裝
- [下載及安裝 | RIME | 中州韻輸入法引擎](https://rime.im/download/)
- 我習慣用 homebrew: `brew install --cask squirrel`

安裝洋蔥拼音
- step1. 下載後 python 執行 `python3 sort_rime_file.py`，產生 電腦RIME方案_20250123 這檔名資料夾
	- 下載包 [Releases · oniondelta/Onion_Rime_Files](https://github.com/oniondelta/Onion_Rime_Files/releases)
- step2. 進入"電腦RIME方案_XXXXXX/注音洋蔥純注音版"資料夾，把裡面檔案全部丟到 `~/Library/Rime/`

設定洋蔥拼音
- 複製這邊的設定 [bopomo_onion.schema.yaml](https://github.com/a45s67/dotfiles/blob/main/private_Library/Rime/bopomo_onion.schema.yaml "bopomo_onion.schema.yaml")、[bopomo_onion_symbols.yaml](https://github.com/a45s67/dotfiles/blob/main/private_Library/Rime/bopomo_onion_symbols.yaml "bopomo_onion_symbols.yaml") 貼上取代 step2 複製的其中這兩個檔案的內容
- 相關快捷鍵：
	- F4 選方案和全形半形之類的
	- shift 中英切換
	- 方括號`[,]`，用以選字時上下移動
	- 等號減號 `-,=`，用以切換上下頁
- (這是我的個人設定，我覺得比較好用，有興趣折騰的朋友可以用預設的設定試試看...)

#### 其它小知識
安裝 rime 後如何 customize
- (這邊參照 rime ice 的教學
- 打 patch: squirrel.custom.yaml、default.custom.yaml
	- [Rime 配置：雾凇拼音 - Dvel's Blog#打補丁的示例](https://dvel.me/posts/rime-ice/#%E5%87%A0%E4%B8%AA%E6%89%93%E8%A1%A5%E4%B8%81%E7%9A%84%E7%A4%BA%E4%BE%8B)
	- [CustomizationGuide · rime/home Wiki](https://github.com/rime/home/wiki/CustomizationGuide#rime-%E5%AE%9A%E8%A3%BD%E6%8C%87%E5%8D%97)
- 遵循這個慣例：squirrel.custom.yaml 只取代 squirrel.yaml 的設定，default.custom.yaml 一樣，不然兩邊容易打架，發生明明有設定卻怎麼好像沒套用到的問題。

輸入法字體
- 霞鶩文楷
	- 繁體版 [lxgw/LxgwWenkaiTC: The Traditional Chinese Edition of LXGW WenKai.](https://github.com/lxgw/LxgwWenkaiTC)
	- [lxgw/LxgwWenKai: An open-source Chinese font derived from Fontworks' Klee One. 一款开源中文字体，基于 FONTWORKS 出品字体 Klee One 衍生。](https://github.com/lxgw/LxgwWenKai)

vim mode
- 參考別人的設定時，通常會看到 vim_mode: true 這個設定，這是 no document 的設定，來自[[提案] app_options 参数设定追加 esc 自动切换 ascii_mode 选项 · Issue #124 · rime/squirrel](https://github.com/rime/squirrel/issues/124)。

shortcuts
- 設置 vim_mode，esc 鍵回到英文模式
- 左 shift 會 commit 當前選詞並切換成英文
- 右 shift 會進入 inline ascii 將當前輸入內容切換爲英文
- 左右方括號 （［）進行選字

#### Reference
- [Mac 電腦三方中文輸入法好選擇：小麥注音輸入法](https://www.kocpc.com.tw/archives/539100)
- [[軟體] 關於嘸蝦米輸入法設定 - 看板 MAC - 批踢踢實業坊](https://www.ptt.cc/bbs/MAC/M.1689557088.A.300.html)
- [山景答問 | RIME | 中州韻輸入法引擎](https://rime.im/blog/2016/04/14/qna-in-mtvu/)
> ——「RIME」這個名字，還有「中州韻」「小狼毫」和「鼠鬚管」這些有趣的稱呼都是怎麼想出來的呢？方便的話可否講講這裏面的故事？
> ——不起個好名，寫碼興致索然。 #Quote 
- [推薦一個神級輸入法——Rime](https://byvoid.com/zht/blog/recommend-rime/)
- [[求救] 注音輸入下臨時切換英文輸入不要變全形 - 看板 MAC - 批踢踢實業坊](https://www.ptt.cc/bbs/MAC/M.1683219177.A.4B1.html)
- 建議是直接使用 capslock
	- 結論: 先使用 cmd space 試試看吧


## 優化工作流及推薦軟體
### Window Management： yabai+skhd
#### 安裝
參考 yabai 文件
- 安裝文件
	- [Disabling System Integrity Protection · koekeishiya/yabai Wiki](https://github.com/koekeishiya/yabai/wiki/Disabling-System-Integrity-Protection)
	- [Installing yabai (latest release) · koekeishiya/yabai Wiki](https://github.com/koekeishiya/yabai/wiki/Installing-yabai-(latest-release))

#### 如何設定
1. yabai 對 workspace 的操作優化我覺得蠻不錯的，爲了使用這類功能，需要調整一些 root 等級的設定
    - SIP部分設定要關掉
    - 要將 yabai 加入 sudoers.d 中
2. 爲保持 OS 的界面乾淨，進入 setting 將 menu bar 及 dock 都設定爲自動隱藏
3. 附上我的 skhd, yabai 設定
    - [.skhdrc](https://github.com/a45s67/dotfiles/blob/main/dot_skhdrc)
    - [.yabairc](https://github.com/a45s67/dotfiles/blob/main/dot_yabairc)

#### 我的操作習慣
通常我會常駐 2 個 workspace
space 1: terminal, IDE 等開發工具
space 2: chrome, obsidian
space 3,4,5: 任意軟體

alt-1,2 or alt-= 切換到對應的 space，alt-w 在 space 下不同視窗互相切換，透過這方法我能用相當快的速度切換到對應的視窗。

我的使用範例: space 1 下 IDE 寫到一半，alt-w 切換到 terminal 推 commit，alt-2 切換到 chrome 找資料，再 alt-w 切換到 obsidian 記筆記，最後 alt-1 alt-w 切換回 IDE 繼續開發。


#### 除錯
```
# debug yabai
tail -f /tmp/yabai_$USER.err.log /tmp/yabai_$USER.out.log

# debug skhd
# 套路: 用 echo 大法看執行 shortcut 時的相關指令的結果
tail -f /tmp/skhd_$USER.err.log /tmp/skhd_$USER.out.log
```

#### 需要注意的點
使用 yabai 跳轉 workspace 不會有漸變動畫，不知道是不是爲了更迅速的 focus 而刻意爲之的，不過 mac 切換畫面的效果雖然蠻漂亮的，但整個切換的過程又會因此卡住，沒動畫也是不錯。

#### References
- 設定參考
	- [Blazing Fast Window Management on macOS - YouTube](https://www.youtube.com/watch?v=fYsCAOfGjxE&t=804s)
	- [yabai/doc](https://github.com/koekeishiya/yabai/blob/master/doc/yabai.asciidoc#space)


### App Launcher, switcher：Raycast
raycast 使用下來的感想
- 單就 launcher 上，比 spotlight 好用
- 切換 app 視窗的部分
	- 比 alttab 順手，因為我在 windows 都用 alt-space (switcheroo) 來移動到目標視窗，且 raycast 針對 search 的推薦排序還做了優化
	- 不過我後期切換視窗都靠 yabai 了
- 總之蠻全能的

### Terminal
#### 近期流行 terminal 的選擇
原先是直接抓 iterm2 來用，因爲我在 windows 時最常聽到的 terminal 就是 iterm，本來也沒想過會換，不過在做設定的爬文過程中看到了一堆我在 windows 下用過的幾個 terminal 也有不少 macOS 用戶推薦，所以就一併試用看看了。

我的需求很簡單，terminal 有基本的樣式設定(e.g., 透明化、可設定背景圖片)，並且能夠 remapping key 就好

總之從 iterm2 -> warp -> alacirtty -> wezterm 試了一遍，最後依照顏值，決定用 wezterm。
- iterm2
	- 老牌 terminal，沒什麼好挑剔的
- warp
	- 要使用還要登入帳號，很怪
	- 小白能無痛上手，預設就有很不錯的 terminal 界面可以用了，還有一堆 fancy 的輔助功能，只是我用不到
- alacirtty
	- 也沒啥好挑剔
- wezterm
	- 也沒啥好挑剔
	- 他的 glyph 比較好看 (glyph 是指一些特殊符號如 emoji，render 在螢幕上的樣子，不專業說明，有錯鞭小力一點...)
	- reddit 上比較多人推薦
#### Wezterm 及 keybinding 設定參考
安裝 [macOS - Wez's Terminal Emulator](https://wezfurlong.org/wezterm/install/macos.html#installing-on-macos)
- 我習慣使用 homebrew `brew install --cask wezterm`

設定：主要就是修正 keybinding

因為在 keybinding 章節中，對調了 cmd, ctrl，會導致 shell, vim 下的 ctrl 相關指令變成要按 cmd，因此也要作對應的處理。
實際上在 terminal 視窗工作時，需要調整、保留哪些指令呢，以及會要繞過什麼坑？
- 目標 keybinding 大致方向
	- cmd-tab 保留原樣，能夠切換 tab
	- cmd-v/cmd-l 等等組合會 send ^V ^L 進入 visual mode 或清空命令列這類的功能
- 大部分 ctrl 的按鍵組合，會先被 OS 攔截，執行 OS 設置好的shortcut，例如
	- ctrl-L 爲 OS 的 show launchpad
	- 在 wezterm 下，直接按 ctrl-L 會 show launchpad
- 這挺頭痛的，但 cmd 的按鍵組合，大部分跟 os shortcut 無關，因此我們能將這類按鍵送到 wezterm 後，再交由 wezterm remapping 成實際想解析成的指令。
	- 這個還帶來意外的好處，我能夠在 wezterm 透過 remap 讓 cmd 針對 terminal 執行我習慣的 ctrl 快捷鍵操作，同時不影響到原本 OS 與 ctrl 相關的 shortcut 設定
	
因此在這裡設定 `.wezterm.lua` 將 cmd+[0123...xyz[]] 等等基本按鍵映射到 ctrl+ [0123...xyz[]]，具體設定參考 [dotfiles/dot_wezterm.lua at main · a45s67/dotfiles](https://github.com/a45s67/dotfiles/blob/main/dot_wezterm.lua)

### Reference
Reddit 上的討論
- [Wezterm or Tmux/Alacritty : r/neovim](https://www.reddit.com/r/neovim/comments/16kjpxn/wezterm_or_tmuxalacritty/)
wezterm 設置
- docs
	- [Configuration - Wez's Terminal Emulator](https://wezfurlong.org/wezterm/config/files.html)
	- [Colors & Appearance - Wez's Terminal Emulator](https://wezfurlong.org/wezterm/config/appearance.html)
- 其他人的設定
	- [Configure Wezterm terminal emulator in Lua with me [ASMR Coding] - YouTube](https://www.youtube.com/watch?v=I3ipo8NxsjY)

## 最後的 key mapping

|          |                       | windows                | mac (微調後)           | mac (微調前)             |
| -------- | --------------------- | ---------------------- | ------------------- | --------------------- |
| 視窗       | 切換視窗 (一般)             | alt-tab                | alt-tab (alttab)    | cmd-tab               |
|          | 切換視窗 (fuzzy search)   | alt-space (switcheroo) | alt-space (raycast) | cmd-space (spotlight) |
|          | 關閉視窗、分頁               | ctrl-w                 | ctrl-w/q            | cmd-w / cmd-q         |
|          | 開啟應用程式 (fuzzy search) | win                    | ctrl-space(raycast) | cmd-space (spotlight) |
| 文字操作     | 複製貼上                  | ctrl-c/v               | ctrl-c/v            | cmd-c/v               |
|          | 刪除 word               | alt-backspace          | alt-backspace       | alt-backspace         |
|          | 切換輸入法                 | win-space              | cmd-space           | cmd-space             |
| chrome   | 切換分頁                  | ctrl-tab               | ctrl-tab            | ctrl-tab              |
|          | 關閉分頁                  | ctrl-w                 | ctrl-w              | ctrl-w                |
| terminal | 任何 ctrl 相關操作          | ctrl-?                 | ctrl-?              | ctrl-?                |
| 視窗管理     | 切換 workspace          | win-左右                 | cmd-左右              | ctrl-左右               |


## 其他
### 螢幕調效
我是用 M27q 這顆螢幕，在 windows 下顏色不知道爲何比較鮮豔，mac 輸出到螢幕上的色彩明顯比較暗淡。
看了一堆影片，總結下來推測是因爲 gamma 值之類的色彩輸出沒有跟螢幕對齊的緣故，那反正我最後是開啓 HDR，色彩就基本上都對了

#### Reference
獲得和MacBook一樣色彩的真正方法 feat.2023 Mac用荧幕推薦 - YouTube
- https://www.youtube.com/watch?v=uSg5tYe3OAE


## 心得分享
### 進入 macOS 的契機
從以前就耳聞 mac OS 的流暢及操作性，對這套生態系的印象就是彷彿只要一旦進入，就能提升 3 倍的工作效率，但礙於 mac 那蔑視窮人(?)的價格，一直都沒機會體驗這個操作系統。
直到 2024 初注意到 mac mini m2，各個 youtuber 開始發各種測評，評價爲 apple 史上 cp 值最高的設備之一，且效能匹敵甚至勝過同等價位的自組桌機。看到大家吹的這麼牛皮我也被燒到了，於是用學生方案直接衝了一發，也買了一個觸控板。(嘿嘿，卑鄙源之助)

### 一年下來簡單的心得以及遇到的坑
用了快一年了，我相當喜歡在 mac 上作開發及日常使用，我認為 windows 及 mac 在視窗上的工作流有不同的體驗，除了系統介面帶來的差異，使用的軟體工具也有影響，我覺得兩種系統在使用體驗上各有優秀的地方，我在 mac 下最喜歡的大概是介面的簡約感，以及 yabai 切換視窗的絲滑體驗。

不過有個軟體開發者需要注意的點，M 系列晶片的 mac 是 arm 架構，偶而會遇到 x86 下支援的套件沒有支援 arm 的狀況，我曾經在 mac 上跑一些開源的模型，結果裡面使用的 python 套件沒有 arm 版本的狀況...，除了 hack 一下把套件拿掉，或是在 arm 下跑個 x86 的 docker image (很折騰且速度很慢)，遇到這情況時我基本建議退回在 x86 主機上開發...。

