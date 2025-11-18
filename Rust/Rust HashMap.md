

## 1. `HashMap` 基本概念

- `HashMap` 是 Rust 標準函式庫（`std::collections`）提供的雜湊表結構，用於儲存「鍵—值」（key–value）對。
    
- 底層透過哈希演算法將鍵映射到儲存槽，提供平均 O(1) 的讀寫效率。
    

## 2. 引入與初始化

```rust
use std::collections::HashMap;

fn main() {
    // 空的 HashMap，鍵類型為 String，值類型為 i32
    let mut map: HashMap<String, i32> = HashMap::new();

    // 也可以用 collect() 從一組 tuple 建立
    let pairs = vec![("apple", 3), ("banana", 5)];
    let map2: HashMap<_, _> = pairs
        .into_iter()
        .map(|(k, v)| (k.to_string(), v))
        .collect();

    println!("{:?}", map2);
}
```

- `HashMap::new()`：建立空的雜湊表。
    
- 也可用 `HashMap::with_capacity(n)` 事先分配容量（避免頻繁重配置）。
    

## 3. 插入（insert）與取出（get、get_mut）

```rust
map.insert("key1".to_string(), 10);
map.insert("key2".to_string(), 20);

// 讀取
if let Some(&v) = map.get("key1") {
    println!("key1 = {}", v);
}

// 可變讀取，修改後直接作用於原 map
if let Some(v_mut) = map.get_mut("key2") {
    *v_mut += 5;
}
```

- `insert(key, value)`：插入或覆蓋舊值，回傳舊值的 `Option<V>`。
    
- `get(&key)`：回傳 `Option<&V>`，只讀。
    
- `get_mut(&key)`：回傳 `Option<&mut V>`，可修改。
    

## 4. 刪除與判斷存在性

```rust
// 判斷是否含有指定鍵
if map.contains_key("key1") {
    println!("存在 key1");
}

// 刪除元素
if let Some(v) = map.remove("key1") {
    println!("移除 key1，其值為 {}", v);
}
```

- `contains_key(&key)`：檢查鍵是否存在。
    
- `remove(&key)`：移除並回傳被移除的值 `Option<V>`。
    

## 5. 遍歷（iteration）

```rust
// 遍歷所有鍵值對
for (k, v) in &map {
    println!("{}: {}", k, v);
}

// 只遍歷鍵
for k in map.keys() {
    println!("key = {}", k);
}

// 只遍歷值
for v in map.values() {
    println!("value = {}", v);
}
```

## 6. Entry API（進階使用）

`Entry` API 可以更靈活地對不存在的鍵進行初始化、或在存在時修改：

```rust
use std::collections::hash_map::Entry;

match map.entry("key3".to_string()) {
    Entry::Vacant(e) => {
        e.insert(30);
        println!("插入 key3 = 30");
    }
    Entry::Occupied(mut e) => {
        *e.get_mut() += 1;
        println!("key3 已存在，遞增為 {}", e.get());
    }
}

// 或用更便捷的方法
map.entry("key4".to_string())
   .or_insert(40); // 如果不存在就插入 40，回傳 &mut V

map.entry("key4".to_string())
   .and_modify(|v| *v += 5); // 如果存在就修改
```

- `or_insert(v)`：若鍵不存在，插入 `v`；並回傳可變參考 `&mut V`。
    
- `or_insert_with(|| …)`：懶惰初始化，只有在鍵不存在時才呼叫 closure。
    
- `and_modify(|v| …)`：如果鍵已存在，就在原值上執行修改。
    

## 7. 自訂雜湊器與性能調優

- 預設使用 `RandomState`，提供抗 DoS 攻擊能力。
    
- 若需更高效的哈希（但犧牲隱私性），可使用第三方庫如 `hashbrown`、或自訂 `BuildHasher`。
    
- 可以呼叫 `map.reserve(n)` 提前分配空間，減少重配置次數。
    

## 8. 注意事項

1. **鍵須可雜湊**：`K: Eq + Hash`；值則無特殊限制。
    
2. **所有權**：`HashMap` 會取得鍵和值的所有權；若要使用借用鍵查詢，需確保借用類型可互轉（例如 `String` vs `&str`）。
    
3. **並行讀寫**：標準 `HashMap` 不支援多執行緒同時變更；可用 `std::sync::Mutex<HashMap<…>>` 或 `dashmap`。
    

---

以上就是從基礎到進階的 `HashMap` 使用方法。如果有更深入的問題，或想看更實際的範例，隨時告訴我！