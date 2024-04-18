# Project 2 L2 Network Setting

## Table of Contents

- [Project 2 L2 Network Setting](#project-2-l2-network-setting)
  - [Table of Contents](#table-of-contents)
  - [Switch 基本設定](#switch-基本設定)
  - [VLAN](#vlan)
  - [Switch IP Address \& Gateway](#switch-ip-address--gateway)
    - [連通性](#連通性)
  - [STP](#stp)
    - [連通性(STP)](#連通性stp)
  - [CS-Core](#cs-core)
    - [連通性(CS-Core)](#連通性cs-core)

## Switch 基本設定

- 每台 switch 皆需要
- 將 Hostname 設爲標籤上的名稱

  ```text
  switch(config)# hostname CSCC-Lab1-Sw
  ```

- 增加本地帳號 (local account)
  - 帳號: `ccna`
  - 密碼: `ccna`
  - 以 **md5** 形式存在 configuration 裏
  - CS-Core console 操作需登入本地帳號

  ```text
  CSCC-Lab1-Sw(config)# username ccna secret ccna
  CSCC-Lab1-Sw(config)# line console 0
  CSCC-Lab1-Sw(config-line)# login local
  ```

- 設定 enable 密碼
  - 密碼: `project2`
  - 以 **md5** 形式存在 configuration 裏

  ```text
  CSCC-Lab1-Sw(config)# enable secret project2
  ```

- 設定 ssh
  - 使用 `cs.nycu.edu.tw` 爲 domain name
  - modulus length 設為 `2048`
  - version 設為 `2`
  - 僅能使用 ssh 登入，關閉 telnet 登入 **（套用到所有 vtys）**

  ```text
  CSCC-Lab1-Sw(config)# ip domain-name cs.nycu.edu.tw
  CSCC-Lab1-Sw(config)# crypto key generate rsa
  CSCC-Lab1-Sw(config)# ip ssh version 2
  CSCC-Lab1-Sw(config)# line vty 0 15
  CSCC-Lab1-Sw(config-line)# transport input ssh
  CSCC-Lab1-Sw(config-line)# login local
  ```

- 關閉所有沒有在使用的 interface（沒有接線的 interface）

  ```text
  CSCC-Lab1-Sw(config)# interface range fastEthernet 0/3-24
  CSCC-Lab1-Sw(config-if-range)# shutdown
  ```

- Switch 不允許向終端 (edge) 發送 CDP
  - CSCC 內的 Switch 除了往上行 NYCU IT 走之外都算終端設備 (edge)
  - **請勿將不發送設為預設，僅將某些下行介面關閉就好。**

  ```text
  CSCC-Lab1-Sw(config)# interfaceEthernet 0/1
  CSCC-Lab1-Sw(config-if)# no cdp enable
  ```

## VLAN

- 命名為 `VLAN{number}`
- Lab1 使用 `VLAN101`
- Lab2-1 & Lab2-2 使用 `VLAN102`
- Management 使用 `VLAN30`
- 324 使用 `VLAN324`
- 316 使用 `VLAN316`
- 321 使用 `VLAN321`
  - 請注意他的前面有一台 Hub
  
  ```text
  CSCC-Lab1-Sw(config)# vlan 101
  CSCC-Lab1-Sw(config-vlan)# name VLAN101
  ```

- **系計中內的 switch 之間的線路都是 trunk mode**
  - **其他的線路都是 access mode**
  
  ```text
  CSCC-Lab1-Sw(config-if)# switchport mode access
  CSCC-Lab1-Sw(config-if)# switchport access vlan 101
  ```

- 對於每一條 trunk 只允許**必要的** VLAN
  
  ```text
  CSCC-Lab1-Sw(config-if)# switchport mode trunk
  CSCC-Lab1-Sw(config-if)# switchport trunk allowed vlan 101,102,30
  ```

- 如果有調整 Native VLAN 的需求，請做**最小程度**的調整，其餘維持預設 (VLAN 1)

  ```text
  CSCC-PC-Room-Sw(config-if)# switchport trunk native vlan 30
  ```

## Switch IP Address & Gateway

| Device        | IP Address       | Gateway        |
| :------------ | :--------------- | :------------- |
| CSCC-PCRoom   | 140.113.10.10/24 | 140.113.10.254 |
| CSCC-intranet | 140.113.10.11/24 | 140.113.10.254 |
| CSCC-Lab1     | 140.113.10.12/24 | 140.113.10.254 |
| CSCC-Lab2     | 140.113.10.13/24 | 140.113.10.254 |
| EC321-Sw      | 140.113.10.14/24 | 140.113.10.254 |

- Switch IP 請設置於 VLAN30
  
  ```text
  CSCC-Lab1-Sw(config)# interface vlan 30
  CSCC-Lab1-Sw(config-if)# ip address 140.113.10.12 255.255.255.0
  ```

- CS-Core 會擔任每一個 VLAN 的 gateway：
  - VLAN 30: 140.113.10.254/24
  - VLAN 101: 140.113.20.1/27
  - VLAN 102: 140.113.20.33/27
  - VLAN 324: 140.113.24.254/24
  - VLAN 316: 140.113.16.254/24
  - VLAN 321: 140.113.21.254/24
  
  ```text
  CS-Core(config)# interface vlan 101
  CS-Core(config-if)# ip address 140.113.20.1 255.255.255.224
  ```

### 連通性

- 至此的設定應該使相同 LAN 機器 (PC & Switch) 可以互相 ping 通，而不同 LAN 則不行 (CS-Core routing 尚未啟用)
- 確保 CS-Core、CSCC-Lab2、CSCC-intranet 之間任何一條 link 斷線都不會影響連通性

## STP

- 對於每臺 Switch STP Mode 設定爲 `rapid-pvst`

  ```text
  CSCC-Lab1-Sw(config)# spanning-tree mode rapid-pvst
  ```

- Switch 終端 (edge) 連線 Port 設定直接進到 Forwarding，使連線快速啟用，並防止 BPDU 封包進入這些界面，當偵測到，則 **Error Disable 掉該介面**
  - CSCC 內的 Switch 除了往上行 NYCU IT 走之外都算終端設備 (edge)
  - 不要將設定設爲 default
  - 注意，CSCC & 321 之間也需要，但也要確保該線路通暢
    - CSCC設BPDU guard，EC321關掉spanning-tree
  
  ```text
  CSCC-Lab1-Sw(config-if)# spanning-tree portfast
  CSCC-Lab1-Sw(config-if)# spanning-tree bpduguard enable
  ```

- 固定 CS-Core 爲所有 VLAN 的 spanning tree instance 的 root
  - 設爲 root primary
  - 包括 `VLAN 1,30,101,102,316,321,324`
  
  ```text
  CS-Core(config)# spanning-tree vlan 1 root primary
  ```

### 連通性(STP)

- 可能會使部分連線 Error Disable

## CS-Core

- 啟用 Routing
  
  ```text
  CS-Core(config)# ip routing
  ```

- 設定 default route 將流量丟往 140.113.1.1

  ```text
  CS-Core(config)# ip route 0.0.0.0 0.0.0.0 140.113.1.1
  ```

- 上行介面（Gig1/0/24）
  - L3 interface
  - IP: 140.113.1.2/30
  - Gateway: 140.113.1.1
  - 要發送與接收 LLDP 封包

  ```text
  CS-Core(config-if)# no switchport
  CS-Core(config-if)# ip address 140.113.1.2 255.255.255.252
  CS-Core(config-if)# lldp transmit
  CS-Core(config-if)# lldp receive
  CS-Core(config-if)# exit
  CS-Core(config)# ip default-gateway 140.113.1.1
  ```

### 連通性(CS-Core)

- 除因 STP 部分 Error Disable 連線外，若暫時取消 STP 特定設定，則所有機器 (PC & Switch) 彼此應該都能 ping 通（包括 ping 到 NYCU-IT）
