在 Rust 中，有幾種快速讀取標準輸入的方法。以下是從最快到較慢的排序：

## 方法 1：一次性讀取所有輸入（最快）

```rust
use std::io::{self, Read};
fn main() {
    let mut buf = String::new();
    io::stdin().read_to_string(&mut buf).unwrap();
    let mut it = buf.split_ascii_whitespace();
    let n: usize = it.next().unwrap().parse().unwrap();
    let m: usize = it.next().unwrap().parse().unwrap();
    let arr: Vec<i32> = it.take(n).map(|s| s.parse().unwrap()).collect();
    println!("n={}, m={}, arr={:?}", n, m, arr);
}
```

## 方法 2：使用 BufReader 一次性讀取（也很快）
```rust
use std::io::{self, BufReader, Read};
fn main() {
    let mut reader = BufReader::new(io::stdin());
    let mut buf = String::new();
    reader.read_to_string(&mut buf).unwrap();
    let mut it = buf.split_ascii_whitespace();
    let n: usize = it.next().unwrap().parse().unwrap();
    let m: usize = it.next().unwrap().parse().unwrap();
    let arr: Vec<i32> = it.take(n).map(|s| s.parse().unwrap()).collect();
    println!("n={}, m={}, arr={:?}", n, m, arr);
}
```

## 方法 3：使用 Vec<u8> 讀取字節（最快，但需要手動解析）


```rust
use std::io::{self, Read};
fn main() {

    let mut buf = Vec::new();

    io::stdin().read_to_end(&mut buf).unwrap();

    let input = String::from_utf8(buf).unwrap();

    let mut it = input.split_ascii_whitespace();

    let n: usize = it.next().unwrap().parse().unwrap();

    let m: usize = it.next().unwrap().parse().unwrap();

    let arr: Vec<i32> = it.take(n).map(|s| s.parse().unwrap()).collect();

    println!("n={}, m={}, arr={:?}", n, m, arr);
}
```

## 競程模板（推薦使用）
```rust
use std::io::{self, Read};
fn main() {
    let mut buf = String::new();
    io::stdin().read_to_string(&mut buf).unwrap();
    let mut it = buf.split_ascii_whitespace();
    // 讀取第一行：3 4
    let n: usize = it.next().unwrap().parse().unwrap();
    let m: usize = it.next().unwrap().parse().unwrap();
    // 讀取第二行：3 2 1
    let arr: Vec<i32> = (0..n).map(|_| it.next().unwrap().parse().unwrap()).collect();
    // 你的邏輯在這裡
    println!("n={}, m={}, arr={:?}", n, m, arr);
}
```
## 為什麼這些方法快？

1. **一次性讀取**：避免了多次系統調用，減少了 I/O 開銷
2. **預分配記憶體**：`String::new()` 和 `Vec::new()` 會根據需要擴展
3. **批量解析**：使用迭代器一次性處理所有數據
4. **避免逐行讀取**：不需要每次都等待換行符

## 效能比較

- 方法 1（`read_to_string`）：最推薦，簡潔且快速
- 方法 2（`BufReader`）：稍慢一點，但差異不大
- 方法 3（`Vec<u8>`）：理論上最快，但需要額外的 UTF-8 轉換

對於競程，**方法 1** 是最佳選擇，代碼簡潔且效能優異