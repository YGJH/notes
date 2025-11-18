

## 1. 為什麼需要 `Result`？

在 Rust 裡，錯誤處理不是丟例外（exception），而是透過回傳一個 **可以是成功，也可以是失敗** 的值。  
這個值就是：

```rust
enum Result<T, E> {
    Ok(T),  // 成功，包著一個 T
    Err(E), // 失敗，包著一個 E (通常是錯誤訊息或錯誤型別)
}
```

好處是：**編譯器強迫你面對錯誤，不准裝死**。

---

## 2. 基本用法

### 成功與失敗

```rust
fn divide(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 {
        Err("不能除以零！".to_string())
    } else {
        Ok(a / b)
    }
}

fn main() {
    let result = divide(10.0, 2.0);
    match result {
        Ok(value) => println!("結果是 {}", value),
        Err(msg) => println!("出錯了: {}", msg),
    }
}
```

### 輸出

```
結果是 5
```

---

## 3. `unwrap` 與 `expect`

如果你 **懶得處理錯誤**（但不推薦在生產環境用），可以直接暴力拆開：

```rust
let x = divide(10.0, 2.0).unwrap(); 
let y = divide(10.0, 0.0).expect("分母不能是零！");
```

- `unwrap()`：若是 `Err`，直接 panic。
    
- `expect(msg)`：若是 `Err`，panic 並輸出自訂訊息。
    

⚠️ `unwrap` 是把程式炸掉的好朋友，但也是線上服務凌晨三點打電話找你的原因。

---

## 4. `?` 運算子：錯誤傳遞神器

比起 `match`，Rust 提供 `?` 來自動往外傳遞錯誤：

```rust
fn safe_divide(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 {
        Err("不能除以零！".to_string())
    } else {
        Ok(a / b)
    }
}

fn compute() -> Result<f64, String> {
    let x = safe_divide(10.0, 2.0)?; // 自動把錯誤往外丟
    let y = safe_divide(x, 0.0)?;   // 這裡會直接 return Err
    Ok(y)
}
```

好處：**讓錯誤傳遞程式碼變得超級乾淨**。

---

## 5. 常用方法

Rust 的 `Result` 有很多方法可以鏈式操作，不必每次都寫 `match`。

### `map`

只處理成功的情況：

```rust
let result = divide(10.0, 2.0).map(|v| v * 2.0);
println!("{:?}", result); // Ok(10.0)
```

### `map_err`

只處理錯誤：

```rust
let result = divide(10.0, 0.0).map_err(|e| format!("錯誤發生: {}", e));
println!("{:?}", result); // Err("錯誤發生: 不能除以零！")
```

### `and_then`

鏈式呼叫（像 `flatMap`）：

```rust
let result = divide(10.0, 2.0)
    .and_then(|x| divide(x, 0.0));

println!("{:?}", result); // Err("不能除以零！")
```

---

## 6. 與 `Option` 搭配

有時候錯誤不用訊息，只要「有或沒有」即可，這就是 `Option`。  
可以用 `.ok_or()` 或 `.ok_or_else()` 轉換：

```rust
fn get_username(id: i32) -> Option<&'static str> {
    match id {
        1 => Some("Alice"),
        2 => Some("Bob"),
        _ => None,
    }
}

fn main() -> Result<(), String> {
    let name = get_username(3).ok_or("查無此人".to_string())?;
    println!("使用者: {}", name);
    Ok(())
}
```

---

## 7. 總結

- `Result<T, E>` = 可能成功 (`Ok(T)`) 或失敗 (`Err(E)`)。
    
- `match`：完整處理兩種情況。
    
- `unwrap` / `expect`：偷懶用，但會 panic。
    
- `?`：自動傳遞錯誤，讓程式乾淨。
    
- 常用方法：`map`、`map_err`、`and_then`。
    
- 可以和 `Option` 轉換，方便處理。
    

---

