在 **Ubuntu 22.04 / 22.10** 上安裝 **V2Ray** 有幾種方式，以下我會用最穩定、最乾淨、最容易維護的方式來說明，並附上完整步驟與驗證方法。

---

## 🧩 一、安裝方式總覽

你可以選擇下列其中一種：

|安裝方式|特點|推薦程度|
|---|---|---|
|✅ 官方安裝腳本（推薦）|一鍵安裝、更新方便、包含 systemd|★★★★★|
|⬛ 手動安裝（進階用）|適合客製化、教學用途|★★★☆☆|
|🐳 Docker 方式|隔離性高，適合伺服器多實例|★★★★☆|

以下我先介紹最簡單、最常用的 **官方腳本安裝法**。

---

## 🛠️ 二、使用官方腳本一鍵安裝 V2Ray

### 1️⃣ 安裝必要工具

```bash
sudo apt update
sudo apt install -y curl unzip
```

### 2️⃣ 下載並執行官方安裝腳本

```bash
bash <(curl -L -s https://install.direct/go.sh)
```

> ⚠️ 若該腳本無法存取（部分地區被牆），請使用下面這個社群維護版（v2fly 官方繼承版本）：

```bash
bash <(curl -L -s https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh)
```

---

## 📦 三、檔案路徑與結構說明

安裝完成後，主要目錄如下：

|路徑|用途|
|---|---|
|`/usr/local/bin/v2ray`|主執行檔|
|`/usr/local/etc/v2ray/config.json`|設定檔|
|`/var/log/v2ray/`|日誌檔案|
|`/etc/systemd/system/v2ray.service`|Systemd 啟動服務|

---

## ⚙️ 四、編輯設定檔

你可以用以下指令打開設定檔：

```bash
sudo nano /usr/local/etc/v2ray/config.json
```

### ✅ 範例：客戶端配置（使用 VMess）

```json
{
  "inbounds": [
    {
      "port": 1080, 
      "listen": "127.0.0.1",
      "protocol": "socks",
      "settings": { "udp": true }
    }
  ],
  "outbounds": [
    {
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "你的伺服器IP或域名",
            "port": 443,
            "users": [
              {
                "id": "UUID格式的ID",
                "alterId": 0,
                "security": "auto"
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "tcp",
        "security": "tls"
      }
    }
  ]
}
```

> 💡 `UUID` 可使用：
> 
> ```bash
> cat /proc/sys/kernel/random/uuid
> ```

---

## 🚀 五、啟動與自動開機設定

```bash
sudo systemctl enable v2ray
sudo systemctl start v2ray
```

檢查運行狀態：

```bash
sudo systemctl status v2ray
```

若想查看日誌：

```bash
sudo journalctl -u v2ray -e
```

---

## 🧪 六、測試連線是否成功

### SOCKS5 測試（本地端）

```bash
curl -x socks5h://127.0.0.1:1080 https://www.google.com -v
```

若顯示 `200 OK` 或看到 Google 首頁的 HTML，就代表連線成功。

---

## 🧰 七、其他進階用法

### 1️⃣ 更新 V2Ray

```bash
sudo bash /usr/local/bin/v2ray-update.sh
```

（如果用 v2fly 安裝腳本的話，可以直接重新執行它，它會自動更新）

### 2️⃣ 移除 V2Ray

```bash
sudo bash <(curl -L -s https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh) --remove
```

### 3️⃣ 切換為 VLESS 協議

只需在 `outbounds` 把 `"protocol": "vmess"` 改成 `"vless"`，並根據伺服器要求調整 `settings` 格式。

---

## 🧭 八、常見問題整理

|問題|解決方式|
|---|---|
|啟動失敗 (`Failed to start V2Ray service`)|檢查 `/usr/local/etc/v2ray/config.json` 是否有 JSON 格式錯誤，用 `jq . config.json` 驗證|
|無法上網但服務正常|檢查防火牆（`ufw`）是否放行端口；或確認 socks5 代理設定正確|
|想讓瀏覽器自動走代理|安裝插件如 **SwitchyOmega**，設 socks5 代理到 `127.0.0.1:1080`|
|想要全局代理|可搭配 `v2rayA` 或 `Qv2ray` 圖形化介面（Ubuntu 有 snap 版）|

---

是否希望我幫你一併說明「如何在 Ubuntu 上安裝 V2Ray 的圖形化管理介面（例如 v2rayA 或 Qv2ray）」？  
這樣就可以免手動改 JSON、點幾下就能連線。