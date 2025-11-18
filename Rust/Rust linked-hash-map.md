在 Rust 中，`linked-hash-map` 是一個 **第三方 crate（外部套件）**，其目的是提供一個**保有插入順序的 HashMap**。它與標準函式庫中的 `std::collections::HashMap` 類似，但主要差異是它維持了鍵值對的**插入順序**。

---

## 📌 基本概念

### HashMap 是什麼？

Rust 的 `std::collections::HashMap<K, V>` 是一種 **雜湊表資料結構**，用來把「鍵 `K`」對應到「值 `V`」，但是它**不保證插入順序**。例如：

```rust
use std::collections::HashMap;

let mut map = HashMap::new();
map.insert("apple", 3);
map.insert("banana", 2);
map.insert("cherry", 5);

for (key, value) in &map {
    println!("{key}: {value}");
}
```

輸出的順序可能不是你插入的順序，因為雜湊表是根據 hash 值排的。

---

### `linked-hash-map` 是什麼？

`linked-hash-map` 是一個 crate，其提供的 `LinkedHashMap<K, V>`：

- ✅ 保有插入順序
    
- ✅ 支援快速存取與修改（O(1) 複雜度）
    
- ✅ 可用於需要有序資料的情境，例如 LRU Cache、JSON 解析（需要維持鍵順序）
    

它內部使用**雙向鍊結串列 + 雜湊表**的混合結構：

- 雜湊表提供 O(1) 的查找
    
- 雙向串列提供插入順序的追蹤
    

---

## 🛠️ 如何使用 `linked-hash-map`

### 1. 加入 Cargo.toml

```toml
[dependencies]
linked-hash-map = "0.5"
```

### 2. 範例程式碼

```rust
use linked_hash_map::LinkedHashMap;

fn main() {
    let mut map = LinkedHashMap::new();

    map.insert("apple", 3);
    map.insert("banana", 2);
    map.insert("cherry", 5);

    for (key, value) in &map {
        println!("{key}: {value}");
    }

    // 順序就是 apple, banana, cherry
}
```

---

## 🧠 特殊功能

### 1. 插入相同 key 會更新順序

```rust
map.insert("banana", 10);
// "banana" 會被視為最新插入的，排到最後
```

### 2. 支援 `get_refresh()`：存取時更新順序（例如 LRU Cache）

```rust
map.get_refresh(&"apple");
// 把 "apple" 移動到最後，表示剛使用過
```

這很適合用來做 LRU（Least Recently Used）快取系統。

---

## 📦 內部實作（進階）

`LinkedHashMap` 的內部實作使用：

- 一個標準的 HashMap：存 `K -> Entry`（其中 Entry 包含指向雙向鏈表節點的指標）
    
- 一個雙向鏈表：追蹤鍵的插入順序
    
- 當你插入時，它會把新節點加入到鏈尾
    
- 查詢更新時，可以透過 hash 快速找到位置，再調整鏈表順序（如果使用 `get_refresh`）
    

---

## 🚀 與其他類似資料結構比較

|特性|std::HashMap|std::BTreeMap|linked_hash_map|
|---|---|---|---|
|查找速度|O(1)|O(log n)|O(1)|
|是否排序|❌ 無順序|✅ 鍵排序|✅ 插入順序|
|是否更新順序|❌ 無此特性|❌|✅（`get_refresh`）|

---

## 🔚 結語

`linked-hash-map` 是一個在 Rust 中非常實用的資料結構，當你需要：

- 儲存 key-value 組
    
- 維持插入順序
    
- 或想實作類似 LRU 快取
    

它會是非常理想的選擇。由於標準庫中的 `HashMap` 並不保證順序，所以 `linked-hash-map` 彌補了這個空缺。

如果你未來使用 Rust 寫 JSON 處理器、設定檔讀寫、或快取機制等應用，`LinkedHashMap` 是一個很值得學習和使用的工具。

---

需要我進一步幫你示範 LRU cache 的實作嗎？或者你想知道它跟 `indexmap` 的差異？