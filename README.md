# Ubuntu 上透過 NAT 模式 + SSH 設定 DHCP Server 的完整指南

## 環境說明

- 虛擬化平台：VirtualBox
- 作業系統：Ubuntu Server（20.04 或 22.04）
- 網路模式：NAT（設定階段）＋ Internal Network（測試階段）
- 連線方式：使用主機 SSH 連到虛擬機（埠轉發）

---

##  VirtualBox NAT + SSH 連線設定

###  1. 設定 NAT 模式
- VirtualBox → 選擇 VM → 設定 → 網路 → 附加到：NAT

###  2. 設定埠轉送
- 點「進階」→「埠轉送」
- 新增一條規則如下：

| 名稱     | 協定 | 主機埠 | 客體埠 | 客體 IP |
|----------|------|--------|--------|----------|
| ssh-nat  | TCP  | 2222   | 22     | （留空） |

###  3. 啟動 VM 並透過主機連線
```
ssh <使用者名稱>@127.0.0.1 -p 2222
```

---

## 安裝 DHCP Server 套件

```bash
sudo apt update
sudo apt install isc-dhcp-server -y
```
 
 

---

## 設定靜態 IP（為Internal Network 測試做準備）

### 1. 查詢網卡名稱
```bash
ip a
```
- ![image](https://github.com/user-attachments/assets/e2f3c1fc-7c23-4e89-848f-b442af0892fa)

 
###  2. 編輯 Netplan 設定檔
```
sudo nano /etc/netplan/01-netcfg.yaml
```


```
network:
  version: 2
  ethernets:
    enp0s3:  # 替換為實際網卡名稱
      dhcp4: no
      addresses:
        - 192.168.10.1/24
```
- ![image](https://github.com/user-attachments/assets/fe97d61a-4736-4cd9-8d64-99f4a748fea1)

- ![image](https://github.com/user-attachments/assets/1befdb01-86a1-4f3a-a1ae-26cb6bc04c39)

 
### 套用設定
```bash
sudo netplan apply
```

---

## 設定 DHCP Server 分配範圍

編輯主設定檔 `/etc/dhcp/dhcpd.conf`：
```bash
sudo nano /etc/dhcp/dhcpd.conf
```

填入：
```conf
ddns-update-style none;
default-lease-time 600;
max-lease-time 7200;
authoritative;

subnet 192.168.10.0 netmask 255.255.255.0 {
  range 192.168.10.100 192.168.10.200;
  option routers 192.168.10.1;
  option subnet-mask 255.255.255.0;
  option domain-name-servers 8.8.8.8, 1.1.1.1;
}
```
- ![image](https://github.com/user-attachments/assets/db8f64c9-d145-4029-a5c1-52aa9a4e81a1)


---

## 指定啟用的網卡

編輯：
```bash
sudo nano /etc/default/isc-dhcp-server
```

設定：
```bash
INTERFACESv4="enp0s3"
```
- ![image](https://github.com/user-attachments/assets/e74dd631-2ec6-413e-afb9-8d174720ba24)

---

## 啟動並驗證服務

```bash
sudo systemctl restart isc-dhcp-server
sudo systemctl enable isc-dhcp-server
sudo systemctl status isc-dhcp-server
```
- ![image](https://github.com/user-attachments/assets/278c09eb-ea71-4736-aba1-e64caf6a8990)

---

## 關機，改為 Internal Network 模式測試

### 修改 VM 網路模式：
- 「附加到」：內部網路（Internal Network）
- 名稱：`dhcpnet`（Client VM 也要相同）
Dhcp server端網路介面設定
- ![image](https://github.com/user-attachments/assets/b2ff8e18-0c61-4cd0-8e6e-2e58e7b77681)

Dhcp client端網路介面設定
- ![image](https://github.com/user-attachments/assets/a1527d01-d080-42bb-a95f-7f59ba2c0f4b)

---

## Client VM 測試

Client 開機後執行：
```bash
sudo dhclient
ip a
```
- ![image](https://github.com/user-attachments/assets/b95c513c-47ac-466c-9f67-67f731617220)

 
Ping 192.168.10.1 成功ping到dhcp server
 
---
- ![image](https://github.com/user-attachments/assets/12573733-a923-456d-8917-ddd4338611f6)



