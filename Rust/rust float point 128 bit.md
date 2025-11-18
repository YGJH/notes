Rust 本身（在標準庫 `std` 或 `core`）**目前沒有內建支援 `f128`（128-bit 浮點數）**，內建的浮點型態只有：

- `f32`（32-bit，單精度浮點數）
    
- `f64`（64-bit，雙精度浮點數）
    

---

## 一、為什麼沒有內建 `f128`？

1. **硬體支援問題**：
    
    - 主流 CPU（如 x86-64 架構）並沒有直接支援 IEEE 754 的 128-bit 浮點硬體指令（不像 `f64` 有 SSE/AVX 支援）。
        
    - 雖然某些架構（如 PowerPC、某些 ARM 架構）有「擴展精度」的類似能力，但不具通用性。
        
2. **穩定性與標準化考量**：
    
    - 雖然 LLVM（Rust 背後的編譯器框架）有支援 `fp128` 型態，但這仍屬於「目標平台特定」的、不穩定的支援。
        

---

## 二、替代方案

### 1. 使用 `f128` crate（由 `softfloat` 模擬）

你可以使用社群提供的 `f128` crate（在 crates.io 上可取得）來模擬 128-bit 浮點運算：

```toml
# Cargo.toml
[dependencies]
f128 = "0.2"
```

```rust
use f128::f128;

fn main() {
    let a = f128::from(1.0f64);
    let b = f128::from(3.0f64);
    let c = a / b;
    println!("1 / 3 = {}", c);
}
```

- **優點**：提供 IEEE 754 quadruple precision 運算
    
- **缺點**：效率低、無法硬體加速、無法與 `std` 中其他數值型態直接混用
    

### 2. 使用 `rug` crate（多精度浮點）

`rug` 提供的是基於 GMP/MPFR 的高精度浮點數（arbitrary precision）：

```toml
[dependencies]
rug = "1.24"
```

```rust
use rug::Float;

fn main() {
    let a = Float::with_val(128, 1);
    let b = Float::with_val(128, 3);
    let c = &a / &b;
    println!("1 / 3 (128-bit precision) = {:.40}", c);
}
```

- **優點**：任意精度，支援超過 128-bit 的浮點運算
    
- **缺點**：非固定精度、需要外部 C 函式庫（GMP、MPFR）
    

---

## 三、LLVM 層級（進階）

如果你真的要用 `f128` 且目標平台支援，你可以透過 **unsafe Rust + FFI + 自訂 LLVM IR**，或使用 `bindgen` 綁定 GCC/Clang 提供的 `_Float128` 或 `__float128`，例如：

### C side：

```c
#include <quadmath.h>

__float128 my_div(__float128 a, __float128 b) {
    return a / b;
}
```

### Rust side（搭配 bindgen）：

```rust
extern "C" {
    fn my_div(a: f128, b: f128) -> f128;
}
```

> 注意：這需要平台支援 `__float128` 且你得自己處理 ABI、alignment 等問題，這是進階應用。

---

## 四、總結

| 方法              | 是否可用 | 精度      | 效能     | 備註                    |
| --------------- | ---- | ------- | ------ | --------------------- |
| `f128` crate    | ✅    | 128-bit | 慢（純軟體） | IEEE 754 binary128 模擬 |
| `rug::Float`    | ✅    | 任意精度    | 慢      | 用於科學計算或精度要求高情境        |
| 原生 LLVM IR 支援   | ⚠️   | 128-bit | 有風險    | 高難度、非穩定、不通用           |
| Rust 標準庫 `f128` | ❌    | ✘       | ✘      | 尚未支援                  |


# 編譯時出現錯誤

error: failed to run custom build command for `gmp-mpfr-sys v1.6.5`

這個錯誤的關鍵訊息在這裡：

```
configure: error: No usable m4 in $PATH or /usr/5bin
```

意思是你的系統中 **找不到 `m4` 這個工具**，而 `gmp-mpfr-sys` 在編譯 GMP 和 MPFR 原始碼時需要用到 `m4`，這是一個 **宏處理器（macro processor）**，在自動化建構工具（像 `autoconf`）中很常用。

---

## ✅ 解法：安裝 `m4`

請依照你所使用的環境安裝 `m4`：

### 如果你用的是 WSL（Ubuntu 或其他 Debian 系）

請打開終端機，輸入：

```bash
sudo apt update
sudo apt install m4
```

### 如果你用的是 macOS（含 Apple M1/M2）

使用 Homebrew 安裝：

```bash
brew install m4
```

### 如果你用的是 Arch Linux 或 Manjaro

```bash
sudo pacman -S m4
```

---

## 📌 完成後請重新編譯

安裝完畢後，回到你的 Rust 專案目錄，執行：

```bash
cargo clean
cargo build
```

`cargo clean` 是為了避免之前編譯失敗的殘留檔案干擾。

---

## 🧠 額外補充：WSL 使用者注意事項

你是在 **Windows 路徑 `/mnt/c/...` 底下操作 Rust 專案**，這可能會導致：

- 非 POSIX 符號連結（`ln -s`）失敗或行為不穩定
    
- 某些 GNU 工具執行異常（尤其像 `m4`, `configure`, `make`）
    

### 解法建議：

請**將你的 Rust 專案移動到 WSL 原生目錄（如 `/home/你的帳號`）底下**：

```bash
mv /mnt/c/Users/kkasdasd/Documents/rust1 ~/rust1
cd ~/rust1/daPi
cargo build
```

這樣可以避免很多與跨平台檔案系統的問題。

---

## 🧪 如果你仍想避免編譯 GMP/MPFR

你可以嘗試在系統中直接安裝這兩個函式庫，然後強制使用系統安裝的版本（而不是 crate 編譯時去重新編譯）：

### Ubuntu 安裝：

```bash
sudo apt install libgmp-dev libmpfr-dev
```

然後加上以下環境變數：

```bash
export GMP_MPFR_SYS_USE_SYSTEM_LIBS=1
cargo build
```

這樣 `gmp-mpfr-sys` crate 會跳過 build-from-source 的部分。

---

## ✅ 總結解法步驟

1. 安裝 `m4`
    
    - Ubuntu/WSL: `sudo apt install m4`
        
2. 如果你在 `/mnt/c/...` 路徑編譯，請改到 Linux 原生目錄如 `/home/you/...`
    
3. 可選：直接安裝 `libgmp-dev` 和 `libmpfr-dev`，再設環境變數使用系統庫
    

---

若你需要我幫你寫個簡單 `rug::Float` 使用範例（例如：高精度 π 或 `sqrt(2)` 計算），也可以告訴我，我可以一步步示範。