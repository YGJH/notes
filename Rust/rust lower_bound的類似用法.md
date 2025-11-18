
>Rust 標準庫並沒有直接叫 `lower_bound` 的方法，但可以用切片（`[T]`）的二分搜尋 API 來達到同樣效果：

1. `binary_search`
    - 回傳 `Result<idx, Err(idx)>`：找到就回 `Ok(idx)`，找不到就回 `Err(insertion_point)`。
    - `insertion_point` 就是「lower bound」的位置。
```rust
let a = vec![1, 3, 3, 5, 7];
// 想找第一個 ≥ 3 的位置

let target = 3;
let pos = match a.binary_search(&target) {
    Ok(idx) => idx,        // 找到 target，就回它第一次出現的位置
    Err(ins) => ins,       // 找不到，就回插入點
};
assert_eq!(pos, 1);
```


>2. `binary_search_by`（自訂比較）
```rust
let a = vec![1, 3, 3, 5, 7];
let target = 4;
let pos = a.binary_search_by(|&x| x.cmp(&target))
           .unwrap_or_else(|ins| ins);

assert_eq!(pos, 3);  // 第一個 ≥ 4 的位置
```



>3. （Rust 1.52+）`partition_point`
    - 直接回傳「predicate 為真」的最右邊界索引，也就是第一個不滿足 `pred` 的位置。
```rust
let a = vec![1, 3, 3, 5, 7];
// 想要第一個 ≥ 4 => predicate: x < 4
let pos = a.partition_point(|&x| x < 4);
assert_eq!(pos, 3);
```

這三種方式都可以拿到 STL `lower_bound` 式的插入點／第一個不小於（≥）目標值的位置。
