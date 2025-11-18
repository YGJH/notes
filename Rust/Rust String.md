Rust 中的 `String` 類型是標準庫中最常用來處理**可變 UTF-8 編碼文字**的類型。它與 `&str`（字串 slice）密切相關，但更靈活，也更佔記憶體資源。以下我會**詳細介紹 `String` 的設計、特性、與常用操作**，以及與 `&str` 的比較、效能考量和使用範例。

---

## 一、什麼是 `String`

- `String` 是 Rust 提供的「堆積（heap）上可變字串類型」，由 `std::string::String` 定義。
    
- 它擁有其所指向的記憶體，可以在執行期間動態成長或縮短。
    
- 資料格式為 **UTF-8 編碼**（每個字元佔的位元組數不一定相同）。
    

### 與 `&str` 的差別

| 型別       | 所屬權           | 可變性     | 記憶體位置      | 大小是否固定  |
| -------- | ------------- | ------- | ---------- | ------- |
| `&str`   | 不擁有（borrowed） | 不可變（預設） | stack (引用) | 是       |
| `String` | 擁有            | 可變      | heap       | 否，可動態增減 |

---

## 二、內部結構（底層原理）

```rust
pub struct String {
    vec: Vec<u8>
}
```

- `String` 實際上是一個包裝了 `Vec<u8>` 的結構體。
    
- 每個 `String` 儲存的是 UTF-8 編碼的位元組陣列。
    
- `String` 中的每個字元（`char`）可能佔用 1 到 4 個 bytes。
    

這就是為什麼你不能像處理陣列一樣直接使用 `s[0]` 來取得第 0 個字元，因為那可能是 multi-byte 的！

---

## 三、常見建立方式

```rust
let s1 = String::new(); // 建立空字串
let s2 = String::from("hello"); // 從字面值建立
let s3 = "world".to_string(); // 用 &str 轉成 String
```

---

## 四、常見操作（附範例）

### 1. **新增（append）**

```rust
let mut s = String::from("Hello");
s.push(' '); // 加入單一 char
s.push_str("world!"); // 加入字串
```

### 2. **插入**

```rust
let mut s = String::from("Hello Rust");
s.insert(5, ',');           // 在位置 5 插入一個字符
s.insert_str(6, " dear");   // 在位置 6 插入字串
```

### 3. **刪除**

```rust
let mut s = String::from("Hello world");
s.pop(); // 移除最後一個字元
s.remove(5); // 移除位置 5 的字元（要注意 UTF-8 邊界）
s.clear(); // 清空整個字串
```

### 4. **長度與容量**

```rust
let s = String::from("哈囉"); // 三個中文字，9 bytes
println!("{}", s.len()); // 9
println!("{}", s.chars().count()); // 3
```

### 5. **字串連接**

```rust
let s1 = String::from("Hello");
let s2 = String::from(" world");
let s3 = s1 + &s2; // 注意：s1 被 move 掉了
```

或使用 `format!`（推薦，不會 move 所有權）：

```rust
let s3 = format!("{}{}", s1, s2);
```

---

## 五、遍歷字串

```rust
let s = String::from("哈囉");

for c in s.chars() {
    println!("{}", c); // 正確地一個字元一個字元印
}

for b in s.bytes() {
    println!("{}", b); // UTF-8 bytes
}
```

---

## 六、與 `&str` 的轉換

```rust
// String -> &str
let s = String::from("Hi");
let s_ref: &str = &s;

// &str -> String
let owned = s_ref.to_string();
let owned2 = String::from(s_ref);
```

---

## 七、常用方法總覽（整理）

|方法|說明|
|---|---|
|`push(char)`|加一個字元|
|`push_str(&str)`|加一個字串|
|`insert(idx, char)`|插入一個字元|
|`insert_str(idx, &str)`|插入一段字串|
|`pop()`|移除最後一個字元|
|`remove(idx)`|移除指定位置的字元|
|`clear()`|清空字串|
|`len()`|傳回 byte 數|
|`is_empty()`|是否為空|
|`chars()`|逐字元 iterator|
|`bytes()`|逐 byte iterator|
|`capacity()`|堆積預先配置的容量|
|`reserve(n)`|預留至少 n 個 byte 容量|
|`truncate(n)`|砍掉後面 n byte|

---

## 八、效能與安全性注意事項

1. **UTF-8 編碼問題**：  
    `String` 的 index 操作不安全（如 `s[1]` 是非法的），必須透過 `char_indices()` 等 API 處理。
    
2. **不適合頻繁插入中間字元**：  
    因為是基於 `Vec<u8>` 的連續記憶體結構，中間插入成本高。
    
3. **與 `Cow<str>` 搭配使用**：  
    當設計 API 時，如果不確定是否需要擁有權，可以使用 `Cow<'a, str>`（Copy on Write）。
    

---

## 九、`String` vs `str` 總結

|特性|`String`|`&str`|
|---|---|---|
|可變性|✅|❌|
|擁有記憶體|✅|❌（是引用）|
|記憶體位置|heap|stack / 靜態段|
|UTF-8|✅|✅|
|適合用在|動態建構、儲存用戶輸入|常量、函數參數|

---

## 十、完整範例

```rust
fn main() {
    let mut s = String::from("Rust");
    s.push(' ');
    s.push_str("is awesome!");
    println!("長度: {}", s.len());
    println!("容量: {}", s.capacity());
    
    for (i, c) in s.chars().enumerate() {
        println!("第 {} 個字元: {}", i, c);
    }
    
    s.truncate(4);
    println!("截斷後: {}", s);
}
```

