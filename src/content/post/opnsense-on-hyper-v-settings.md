---
title: "OPNsense on Hyper-V 建立惡意樣本分析環境"
description: "在 Hyper-V 上使用 OPNsense 建立惡意樣本分析環境的設定記錄，包含 VPN 導流、Kill Switch、mDNS 及路由迴圈除錯。"
publishDate: "2026-03-28"
tags: ["Security", "Networking"]
---

這篇文章爲我使用 OPNsense 在 hyper V 上建立分析環境的設定記錄，以及途中踩到的一些坑及解法。如果有想在 Hyper-V 上做 malware analysis, 但煩惱網路設定的夥伴們也能參考看看。(有錯誤的地方各路大大鞭小力一點)

## 我的需求
- 動態機出去的流量必須接上 VPN
- 其他 VM 如靜態機 traffic 維持平常

<img src="https://files.sakana.tw/blog/opnsense-on-hyper-v-settings/image%207.png" alt="image 7" style="max-height: 500px;" />

## **初始環境設定**

### **前置準備**

**在 HyperV 設定以下 virtual switch**
1. **Default Switch (WAN)**：對接 OPNsense 的 WAN 端，提供網際網路存取, 走 NAT。
2. **Internal Switch (mgmt)**：管理網路，用於 Host 與 OPNsense 之間的溝通, 一般 VM 也都放這網段
3. **Private Switch (analysis)**：分析專用網路，受控 VM（如動態機）放在此區，完全由 OPNsense 隔離。

### **OPNsense 初始化**

#### **安裝 opnsense 及介面指派**
- (安裝過程這邊不贅述)
- WAN 對接 Default Switch，LAN 對接 mgmt switch, Opt1 對接 analysis switch

#### DHCP 設定
1. set a static IP for your interface
   - interface → your interface → set static IP
2. set your interface as DNS server
   - services → dnsmasq → set dhcp range

![image.png](https://files.sakana.tw/blog/opnsense-on-hyper-v-settings/image.png)
![image 1.png](https://files.sakana.tw/blog/opnsense-on-hyper-v-settings/image%201.png)


## VPN 設定

#### Set WireGuard on OPNsense
- 我目前使用免費的 proton VPN, peer 及 instance 的設定可參考[Set proton Wireguard](https://docs.opnsense.org/manual/how-tos/wireguard-client-proton.html)

#### Traffic to VPN
接下來就是導流 analysis 網段的流量至 VPN 的設定了, 這邊參考 [wireguard-selective-routing](https://docs.opnsense.org/manual/how-tos/wireguard-selective-routing.html) 後, 依照自身狀況做了微調

**將目的地非 analysis net 的全導向 wireguard**
![image 2.png](https://files.sakana.tw/blog/opnsense-on-hyper-v-settings/image%202.png)

#### Kill switch
避免 VPN shutdown 時, 分析流量直接從 WAN gateway 出去, 在 WAN 將 source 爲 analysis net 的全部 block
- TODO

#### 指定使用 VPN 提供的 dns server
這邊直接用 dhcp 派下去, 使 analysis net 內的主機指定使用 VPN 提供的 dns server
![image 3.png](https://files.sakana.tw/blog/opnsense-on-hyper-v-settings/image%203.png)

:::note
不指定的情況下，dns server 爲 analysis address，dns query 會經過 opnsense unbound dns 處理，unbound dns 會產生 query DNS 的流量, 由於這是 opnsense 自己產生的流量，這流量會通過 WAN，造成 dns query leak，因此要指定使用 vpn 提供的 dns server
:::

(TODO: 更徹底一點, anaylsis interface 同時需要關閉 DNS 解析



#### TODO: 驗證流量不會從 WAN 漏出去
都做好之後, 從動態機產生的流量應該只會通過 VPN 出去了. 這時就可以開始分析工作了, 以下爲我個人方便而做的一些額外設定.
#### TODO: 驗證 VPN ip
動態機開啟瀏覽器查 IP，確認顯示的是 VPN 的 IP。

## 進階設置
以下是幫助我操作 vm 或 OPNsense 更方便的設定

### Access to portal
**設定 static ip, route**
首先將 Host connect to LAN(mgmt), 並設定 static ip, route
```bash
# (以下指令也可透過 ncpa.cpl 界面來設定)
netsh interface ip set address "vEthernet (self-host)" static 192.168.1.2 255.255.255.0

route -p add 192.168.1.0 mask 255.255.255.0 <OPNsense mgmt IP>
route -p add 192.168.2.0 mask 255.255.255.0 <OPNsense mgmt IP>
```
" 不設定會導致 interface 直接使用 opnsense 給的 dhcp ip 及 routing，有機會觸發 infinite loop (見 appendix: )


### mDNS 設定
這樣 host 就可以在不知道動態機 vm ip 的狀況下, 透過動態機 hostname 找到其 ip.
1. install `os-mdns-repeater` 來跨 interface 轉發 mDNS 封包
2. analysis interface 要加規則放行 dest ip `224.0.0.251`

![image 4.png](https://files.sakana.tw/blog/opnsense-on-hyper-v-settings/image%204.png)
目的是在 L3 導流 VPN **之前**放行 mDNS multicast 封包，這樣軟體層的 mDNS repeater 才能處理到。
:::note
不放行的話, 封包會回不去, 如下圖, 有興趣可以在 **Interfaces: Diagnostics: Packet Capture** 錄 mgmt, analysis & port 5353 的封包來 debug)
![image 5.png](https://files.sakana.tw/blog/opnsense-on-hyper-v-settings/image%205.png)
:::


## 附錄

### 路由迴圈的除錯記錄
#### 問題說明
- OPNsense WAN 接上通過 NAT 的 Default switch
- Host 接上 mgnt switch 並且收到 OPNsense dhcp 設定, 設定了 gateway routing to OPNsense’s mgnt interface

:::note
當符合以上條件, OPNsense VM 啟動後，host 可能會無法上網。原因見下圖
![image 6.png](https://files.sakana.tw/blog/opnsense-on-hyper-v-settings/image%206.png)
:::

#### 診斷方式
以下是我診斷 routing loop 的做法, 如果遇到類似狀況可參考.
**使用 tracert 檢查**
```bash
> tracert 8.8.8.8

 1   192.168.87.1          ← Wi-Fi router（偶爾搶到）
 2   172.20.64.1           ← Host (Default Switch)
 3   192.168.1.1           ← OPNsense
 4   172.20.64.1           ← Host
 5   192.168.1.1           ← OPNsense
 ...
```

**確認 routing table**
```bash
> route print

0.0.0.0  0.0.0.0  192.168.87.1  192.168.87.33   50  ← Wi-Fi
0.0.0.0  0.0.0.0  192.168.1.1   192.168.1.135   15  ← OPNsense（metrics 更低, 優先）

```

**Wireshark 觀察 TTL exceeded**
```bash
# (ICMP Type 11 (Time Exceeded) 就是 TTL 歸零時送回的訊息。)
icmp.type == 11
```

#### Solution
**Solution 1**
host 完全不接上 mgmt switch 就解決了
**Soultion 2**
將接上 mgmt switch 的 interface 調爲 static ip + route
```bash
# (以下指令也可透過 ncpa.cpl 界面來設定)
netsh interface ip set address "vEthernet (self-host)" static 192.168.1.2 255.255.255.0

route -p add 192.168.1.0 mask 255.255.255.0 <OPNsense mgmt IP>
route -p add 192.168.2.0 mask 255.255.255.0 <OPNsense mgmt IP>
```
