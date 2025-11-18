在 Ubuntu（或其他 systemd 系統）中，要設定一個服務開機自動啟動，有幾種方法，最常見的做法是透過 **`systemd` 服務單元（service unit）** 來實現。以下是詳細步驟：

---

## 🔧 方法 1：使用 `systemctl enable` 讓現有的服務開機啟動

如果你要啟動的是一個已經存在的服務，例如 `ssh`、`nginx`、`openvpn` 等：

```bash
sudo systemctl enable <service-name>
```

例如：

```bash
sudo systemctl enable ssh
```

這會建立一個 symbolic link，讓它在開機時自動啟動。

你可以使用以下指令啟動或查看狀態：

```bash
sudo systemctl start <service-name>       # 立即啟動
sudo systemctl status <service-name>      # 查看狀態
```

---

## 🛠️ 方法 2：自訂一個 systemd 服務單元檔（.service）

如果你有自訂腳本或程式想在開機時執行，可以這樣做：

### 📁 第一步：建立 systemd 服務檔案

以 root 權限建立檔案（以下假設你的程式是 `/home/user/myscript.sh`）：

```bash
sudo nano /etc/systemd/system/myscript.service
```

填入內容：

```ini
[Unit]
Description=My Startup Script
After=network.target

[Service]
ExecStart=/home/user/myscript.sh
WorkingDirectory=/home/user
StandardOutput=journal
StandardError=journal
Restart=on-failure
User=<你的使用者名稱>
Group=<你的使用者名稱>

[Install]
WantedBy=multi-user.target
```

> 請把 `<你的使用者名稱>` 改成你目前登入的使用者名稱，例如 `ubuntu` 或 `yourname`。

---

### 🧪 第二步：設定腳本權限

```bash
chmod +x /home/user/myscript.sh
```

---

### 🔄 第三步：啟用並啟動服務

```bash
sudo systemctl daemon-reexec        # 重載 systemd
sudo systemctl daemon-reload        # 如果 service 有改內容
sudo systemctl enable myscript.service
sudo systemctl start myscript.service
```

---

### ✅ 第四步：確認是否開機啟動

```bash
systemctl is-enabled myscript.service
```

如果輸出是 `enabled`，代表會開機啟動。

---

## 🧹 方法 3：使用 crontab 的 `@reboot`（較簡單，但較不穩定）

如果只是想在開機執行某個指令，也可以用 cron：

```bash
crontab -e
```

加上一行：

```bash
@reboot /home/user/myscript.sh
```

這種方式適合非長駐型任務（non-daemon），例如開機後執行一次的腳本。但它的錯誤處理、輸出紀錄較差，不建議用來做背景服務。

---

## 🧭 檢查開機流程中有沒有被執行

你可以查看 log：

```bash
journalctl -u myscript.service
```

或者用 `dmesg`/`journalctl -b` 查看整個開機記錄。

---

## 補充：user-level systemd（不用 sudo）

也可以設定使用者層級的 systemd 單元：

1. 把 `.service` 放在：
    
    ```
    ~/.config/systemd/user/
    ```
    
2. 使用 `systemctl --user` 來操作：
    

```bash
systemctl --user daemon-reload
systemctl --user enable myscript.service
systemctl --user start myscript.service
```

要讓這種方式開機就能生效，還需要設定「登入時啟動 user systemd」的服務（詳解可另開話題）。

---

如你有具體要設定哪一個服務或腳本，也可以提供內容，我可以幫你寫出完整 service 設定。