`saturating_sub()` 是 Rust 中的一個方法，用於執行**飽和減法**（saturating subtraction）。

## 功能說明：

`saturating_sub()` 會執行減法運算，但如果結果會導致**下溢**（underflow），它會返回該型別的**最小值**，而不是發生 panic 或產生錯誤的結果。

## 語法：
```rust
let result = a.saturating_sub(b);
```

```rust
## 範例：

fn main() {

    // 正常情況

    let a: usize = 10;

    let b: usize = 5;

    println!("{}", a.saturating_sub(b)); // 輸出: 5

    // 會導致下溢的情況

    let a: usize = 5;

    let b: usize = 10;

    println!("{}", a.saturating_sub(b)); // 輸出: 0 (而不是 panic)

    // 對於有符號整數

    let a: i32 = -2000000000;

    let b: i32 = 2000000000;

    println!("{}", a.saturating_sub(b)); // 輸出: -2147483648 (i32::MIN)

}
```

## 在你的程式碼中的用途：

這樣可以確保迴圈範圍始終是有效的，避免索引越界或程式崩潰。

## 其他類似方法：

- `saturating_add()` - 飽和加法
- `saturating_mul()` - 飽和乘法
- `wrapping_sub()` - 環繞減法（會環繞到型別的最大值）
- `checked_sub()` - 檢查減法（返回 `Option`，下溢時返回 `None`）

`saturating_sub()` 是處理可能發生下溢情況的安全方法，特別適合在需要確保程式不會因為數值運算而崩潰的場景中使用。