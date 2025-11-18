在 Rust 中使用隨機數（random number generation）主要依賴 `rand` crate。以下是一步一步的使用方式與範例教學。

---

## 🔧 1. 加入 `rand` 依賴

在你的 `Cargo.toml` 中加入：

```toml
[dependencies]
rand = "0.8"
```

---

## 🧪 2. 基本用法範例

```rust
use rand::Rng;

fn main() {
    let mut rng = rand::thread_rng(); // 建立一個隨機數生成器

    // 隨機產生一個介於 1 到 100 的整數（包含上下限）
    let num: i32 = rng.gen_range(1..=100);
    println!("隨機整數：{}", num);

    // 產生一個 0 到 1 之間的浮點數
    let float_num: f64 = rng.gen();
    println!("隨機浮點數：{}", float_num);

    // 產生布林值
    let toss: bool = rng.gen_bool(0.5); // 50% 機率是 true
    println!("投擲結果：{}", toss);
}
```

---

## 🎲 常見功能整理

|功能|說明|
|---|---|
|`rng.gen()`|產生介於 0～1 的隨機浮點數或整數（依型別而定）|
|`rng.gen_range(a..b)`|產生介於 a（含）到 b（不含）的數字|
|`rng.gen_range(a..=b)`|產生介於 a 到 b（包含 b）的數字|
|`rng.gen_bool(p)`|以機率 `p` 產生布林值（例如 `0.7` → 70% 是 true）|

---

## 🧪 範例：隨機選一個元素

```rust
use rand::seq::SliceRandom;

fn main() {
    let choices = ["Rock", "Paper", "Scissors"];
    let mut rng = rand::thread_rng();

    if let Some(choice) = choices.choose(&mut rng) {
        println!("我選擇了：{}", choice);
    }
}
```

---

## 🚨 提醒：`rand::thread_rng()` 是安全的

這是最常用的隨機數生成器，具備：

- 快速
    
- 隨執行緒變化
    
- 自動初始化種子
    

如果你需要完全可重現的隨機序列（例如單元測試），可以考慮用 `StdRng::seed_from_u64(seed)`。

---

## 🧠 小彩蛋

如果你從 C++ 或 Python 過來，可能會想找 `srand()` 或 `random.seed()` 的對應物。Rust 的 `rand::rngs::StdRng` 搭配 `.seed_from_u64(seed)` 正是它！

```rust
use rand::{SeedableRng, Rng};
use rand::rngs::StdRng;

fn main() {
    let mut rng = StdRng::seed_from_u64(42); // 可重現的序列
    println!("{}", rng.gen_range(0..10));    // 每次都是相同結果
}
```

---

想玩得更瘋狂？還有亂數打亂 `Vec` 順序、亂數取樣 `Itertools::choose_multiple()` 這類進階玩法可以探索 😏


你中肯地命中競程（競技程式設計）玩家的痛點了！🎯  
Rust 沒有內建 `std::random()`，在競賽場上真的會讓人急著 Ctrl+C `rand` 範例或自己寫 LCG。

---

## 💡 總結：在競程中用 random，**你有三個選擇**

---

### ✅ **選項 1：手刻一個簡單的 PRNG（線性同餘生成器 LCG）**

快速、無需外部 crate，也能種子控制，常見寫法：

```rust
struct LCG {
    state: u64,
}

impl LCG {
    fn new(seed: u64) -> Self {
        Self { state: seed }
    }

    fn next(&mut self) -> u64 {
        // constants 可微調
        self.state = self.state.wrapping_mul(6364136223846793005).wrapping_add(1);
        self.state
    }

    fn next_range(&mut self, low: u64, high: u64) -> u64 {
        low + (self.next() % (high - low))
    }
}

fn main() {
    let mut rng = LCG::new(42);
    for _ in 0..5 {
        println!("{}", rng.next_range(1, 100));
    }
}
```

這種 LCG 在 **Codeforces / AtCoder / LeetCode** 都很夠用，快速又可控制。

---

### ✅ **選項 2：貼上 rand crate 的最小精簡版**

有些比賽平台允許使用 extern crate，只是不能 `cargo add`。  
這時你可以手動貼進：

```rust
use rand::Rng;
use rand::SeedableRng;
use rand::rngs::SmallRng;

fn main() {
    let mut rng = SmallRng::seed_from_u64(12345);
    let val: i32 = rng.gen_range(1..=10);
    println!("{}", val);
}
```

然後在最上面加：

```toml
// Cargo.toml
[dependencies]
rand = { version = "0.8", features = ["small_rng"] }
```

但這通常不適用「只給你 main.rs」的線上 OJ。

---

### ✅ **選項 3：直接用 `std::time` 當種子，然後 LCG**

如果你連 `rand` 都不給，那就混合用：

```rust
use std::time::{SystemTime, UNIX_EPOCH};

fn get_seed() -> u64 {
    SystemTime::now().duration_since(UNIX_EPOCH).unwrap().as_nanos() as u64
}
```

搭配剛剛的 `LCG::new(get_seed())`，就有一個簡易的 pseudo-random source 了！

---

## 🧠 附加建議

- 🚀 如果是「**打 seed 解法（亂數打表 / 隨機化演算法）**」，建議用固定 seed + LCG，不要用系統時間，方便重現。
    
- 🧪 想做平滑亂數範圍 → 用 `x % n` 可能會有偏差，建議改寫成：
    

```rust
fn next_range(&mut self, n: u32) -> u32 {
    let mut r = self.next() as u32;
    while r >= u32::MAX - (u32::MAX % n) {
        r = self.next() as u32;
    }
    r % n
}
```

---

要我幫你寫一個「競程專用的 Rust Random 工具模組」嗎？輸入種子、可重播、簡單使用那種 😎