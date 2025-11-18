[請直接看這篇文章](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/get-started/linux-macos-setup.html)

---

## 🧩 一、前置條件與概念

ESP32-S3 使用 **Xtensa LX7 架構**，但 Rust 官方工具鏈目前對 Xtensa 支援是非正式的。  
不過好消息是：有一個專案 **`esp-rs`** 已經幫我們把這些工具整合好，可以完全用 Rust 開發 ESP32、ESP32-S3、ESP32-C3 等晶片。

📚 官方文件：[https://esp-rs.github.io/book/](https://esp-rs.github.io/book/)

---

## ⚙️ 二、安裝開發環境

在 Ubuntu 22 上，請依序執行下列指令：

### 1️⃣ 安裝基本工具

```bash
sudo apt update
sudo apt install -y git curl pkg-config libudev-dev python3
```

---

### 2️⃣ 安裝 Rust 工具鏈

```bash
curl https://sh.rustup.rs -sSf | sh
source $HOME/.cargo/env
```

確認版本：

```bash
rustc --version
```

---

### 3️⃣ 安裝 ESP Rust 工具

官方提供一鍵安裝指令：

```bash
cargo install espup
espup install
```

這會安裝：

- Xtensa Rust toolchain
    
- ESP-IDF 開發框架
    
- 烏龜核心（Xtensa GCC）
    
- 並設定 `export` 變數
    

你完成後，請重新載入環境變數：

```bash
source $HOME/export-esp.sh
```

---

### 4️⃣ 建立一個新專案

我們使用官方提供的模板：

```bash
cargo install cargo-generate
cargo generate --git https://github.com/esp-rs/esp-idf-template.git
```

執行後會問你幾個問題：

```
project name? : hello-esp32
Which MCU?     : esp32s3
```

進入資料夾：

```bash
cd hello-esp32
```

---


先運行
```
cargo install ldproxy
```

### 5️⃣ 編譯測試（確認可編譯）

```bash
cargo build
```

第一次會花一點時間（要抓取 IDF 與 Rust crate）。  
如果成功，你會看到：

```
Finished dev [optimized + debuginfo] target(s) in 2m 30s
```

---

## 🔥 三、寫一個實際會閃燈的程式

打開 `src/main.rs`，改成：

```rust
use esp_idf_svc::hal::delay::Delay;
use esp_idf_svc::hal::gpio::PinDriver;
use esp_idf_svc::hal::prelude::*;


fn main() -> anyhow::Result<()> {
	// 初始化所有外設
	esp_idf_svc::sys::link_patches();
	let peripherals = Peripherals::take().unwrap();
	let pins = peripherals.pins;
	// 假設板上LED接在 GPIO2（大部分ESP32-S3板都是）
	let mut led = PinDriver::output(pins.gpio2)?;
	let delay = Delay::new_default();
	loop {
		led.set_high()?;
		delay.delay_ms(500);
		led.set_low()?;
		delay.delay_ms(500);
	}
}
```

這是一個最簡單的「閃爍LED」範例。

---

## 🔌 四、燒錄程式到 ESP32-S3

1. 插上 USB Type-C 線（你現在已經這樣做了）。
    
2. 確認裝置：
    
    ```bash
    ls /dev/tty*
    ```
    
    應該會看到：
    
    ```
    /dev/ttyUSB0
    或 /dev/ttyACM0
    ```
    
3. 安裝燒錄工具：
    
    ```bash
    cargo install espflash
    ```
    
4. 燒錄程式：
    
    ```bash
    espflash flash /dev/ttyUSB0
    ```
    
	若成功，你會看到：
    
    ```
    [success] Flashing has completed successfully!
    ```
    
5. 重新啟動板子後，LED 會開始閃爍 🔴✨。
    

---


# 如果燒錄出現以下錯誤
```
[2025-10-18T01:18:49Z INFO ] Serial port: '/dev/ttyACM0'
[2025-10-18T01:18:49Z INFO ] Connecting...
Error:   × Failed to open serial port /dev/ttyACM0
  ╰─▶ Error while connecting to device

```

這個狀況非常典型，這表示你的 **ESP32-S3** 已經被 Ubuntu 偵測到了（`/dev/ttyACM0`），但是在燒錄時因為權限或驅動問題導致無法開啟該序列埠。  
我們一步步解決 👇

---

## 🧩 一、確認目前裝置

請先執行：

```bash
ls -l /dev/ttyACM0
```

你應該會看到類似這樣的輸出：

```
crw-rw----+ 1 root dialout 166, 0 Oct 18 09:18 /dev/ttyACM0
```

這裡很關鍵——  
如果你看到：

- 擁有者是 `root`
    
- 群組是 `dialout`
    
- 權限是 `rw` 只給 `root` 和 `dialout`
    

那代表你需要屬於 `dialout` 群組才能操作這個埠。

---

## 🧠 二、加入 dialout 群組

執行以下指令讓自己有權限：

```bash
sudo usermod -aG dialout $USER
```

然後 **登出再登入**（或重啟電腦）以套用新的群組設定。  
你可以用以下指令確認是否生效：

```bash
groups
```

應該要看到裡面包含：

```
dialout
```

---

## ⚡ 三、再試一次燒錄

重新插拔你的 ESP32-S3，然後執行：

```bash
espflash flash /dev/ttyACM0
```

正常情況下你應該會看到：

```
[INFO] Connecting...
[INFO] Serial connected
[INFO] Flashing has completed successfully!
```

---

## 🧠 四、如果仍然連不上

請檢查以下幾項：

### 1️⃣ USB線材問題

有些 Type-C 線「只能充電、不能傳輸資料」。  
請務必使用「能傳資料」的線（例如手機傳輸線、非充電專用線）。

---

### 2️⃣ 嘗試進入 Boot 模式手動燒錄

ESP32-S3 上通常有兩顆按鈕：

- `BOOT`
    
- `RST`（或 `EN`）
    

👉 在插上電腦前：

1. 按住 **BOOT** 不放
    
2. 同時按一下 **RST**
    
3. 放開 **RST**，再放開 **BOOT**
    

這樣板子會進入 UART Download 模式。  
然後再執行：

```bash
espflash flash /dev/ttyACM0
```

---

### 3️⃣ 驗證權限是否真的生效

如果還是錯誤，可以手動確認權限：

```bash
sudo chmod 666 /dev/ttyACM0
```

這是臨時方案（重插 USB 會還原），但可以驗證是不是權限的問題。

---

## 🧰 五、檢查是否有正確的 USB 驅動

ESP32-S3 開發板通常內建這兩種 USB 晶片之一：

|晶片|裝置名稱|驅動|
|---|---|---|
|**CP210x**|`/dev/ttyUSB0`|由 `cp210x` 核心模組提供|
|**CH340**|`/dev/ttyUSB0`|由 `ch341` 模組提供|
|**原生USB (CDC)**|`/dev/ttyACM0`|Linux 內建驅動（不需額外安裝）|

你的是 `/dev/ttyACM0` → 屬於原生 USB CDC，所以 **不用額外驅動**，只要權限正確即可。

---

## ✅ 六、快速測試是否能通訊

你也可以測試板子是否能正確開啟串口：

```bash
screen /dev/ttyACM0 115200
```

如果能開啟（畫面空白但不報錯），就代表連線 OK。  
退出用：`Ctrl + A` → `K` → `Y`

---

## 🧭 總結

|問題|解法|
|---|---|
|沒權限開啟 /dev/ttyACM0|`sudo usermod -aG dialout $USER` 然後重登|
|線材無法傳資料|換條 Type-C 傳輸線|
|無法進入下載模式|同時按 `BOOT + RST` 進入 UART 模式|
|權限測試|`sudo chmod 666 /dev/ttyACM0`|
|驗證連線|`screen /dev/ttyACM0 115200`|

---

你想我幫你確認一下這塊 ESP32-S3 的預設 RGB LED 腳位（這樣你燒錄成功後可以讓燈閃爍驗證）嗎？  
不同版本板子的 LED 腳位可能是 `GPIO38`、`GPIO48` 或 `GPIO2`。












## 🧠 五、擴充與除錯

如果你要看 log 輸出，可以用：

```bash
espflash monitor /dev/ttyUSB0
```

或在程式裡加上：

```rust
use log::info;

fn main() -> anyhow::Result<()> {
    esp_idf_svc::sys::link_patches();
    esp_idf_svc::log::EspLogger::initialize_default();

    info!("ESP32-S3 started!");
    Ok(())
}
```

---

## ✅ 六、完整開發鏈總結

|工具|功能|
|---|---|
|`rustup`|管理 Rust 工具鏈|
|`espup`|安裝 ESP Rust 工具鏈|
|`cargo-generate`|建立 ESP 專案模板|
|`espflash`|將程式燒錄進晶片|
|`esp-idf-hal`|控制 GPIO、I2C、SPI、UART 等硬體介面|
|`esp-idf-svc`|提供 log、wifi、網路等高階功能|

---

如果你願意，我可以幫你：

1. 針對你這塊 **ESP32-S3-WROOM-1** 查出預設 LED 腳位是哪一個；
    
2. 幫你擴充範例，例如「按下 BOOT 鍵 LED 改變狀態」、「Wi-Fi 連線並印出 IP」。
    

你想先讓 LED 閃爍成功，還是直接做 Wi-Fi 範例？


