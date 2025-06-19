# Ubuntu 上建立 DHCP Server 的實作指南

## 適用版本
- Ubuntu 20.04 / 22.04
- 支援 IPv4 動態 IP 分配

## 安裝步驟

### 1. 更新系統
```bash
sudo apt update && sudo apt upgrade -y
```

### 2. 安裝 DHCP Server 套件
```bash
sudo apt install isc-dhcp-server -y
```

---

## 設定 DHCP Server

### 3. 編輯主設定檔 `/etc/dhcp/dhcpd.conf`
```bash
sudo nano /etc/dhcp/dhcpd.conf
```

範例設定：
```conf
# 禁用動態 DNS 更新
ddns-update-style none;

# 預設租約時間
default-lease-time 600;
max-lease-time 7200;

# 僅提供靜態設定範圍
authoritative;

# 網段設定
subnet 192.168.10.0 netmask 255.255.255.0 {
  range 192.168.10.100 192.168.10.200;
  option routers 192.168.10.1;
  option subnet-mask 255.255.255.0;
  option domain-name-servers 8.8.8.8, 1.1.1.1;
}
```

---

## 指定服務的網卡

### 4. 編輯 `/etc/default/isc-dhcp-server`
```bash
sudo nano /etc/default/isc-dhcp-server
```

設定：
```bash
INTERFACESv4="eth0"
INTERFACESv6=""
```
> `eth0` 請替換成實際提供 DHCP 功能的網卡名稱，可用 `ip a` 查詢。

---

## 啟動服務並設為開機啟動

```bash
sudo systemctl restart isc-dhcp-server
sudo systemctl enable isc-dhcp-server
```

檢查狀態：
```bash
sudo systemctl status isc-dhcp-server
```

---

## 驗證 DHCP 是否正常工作

1. 在區網中插入一台 Client（如筆電或手機）
2. 設定為自動取得 IP（DHCP）
3. 使用 `ipconfig`（Windows）或 `ip a`（Linux）確認 IP 是否為 `192.168.10.x` 範圍

---





