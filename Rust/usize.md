`usize` 是 Rust 中的一個**無符號整數型別**，代表「unsigned size」。

## 基本特性：

### 1. **無符號整數**

- 只能表示非負整數（0, 1, 2, 3, ...）
- 不能表示負數

### 2. **平台相關的大小**

- 在 32 位系統上：`usize` = 32 位（4 字節）
- 在 64 位系統上：`usize` = 64 位（8 字節）
- 大小與指標相同

### 3. **數值範圍**

**32** 位系統**
-  usize 範圍：0 到 4,294,967,295 (2^32 - 1)
64 位系統  
 = usize 範圍：0 到 18,446,744,073,709,551,615 (2^64 - 1)

## 主要用途：

### 1. **陣列/向量索引**
```rust

let vec = vec![1, 2, 3, 4, 5];

let index: usize = 2;

println!("{}", vec[index]); // 輸出: 3
```
### 2. **長度和大小**
```rust
let vec = vec![1, 2, 3];

let len: usize = vec.len(); // 長度是 usize 型別

println!("長度: {}", len);
```

### 3. **迴圈計數**
```rust
for i in 0..10usize {

    println!("{}", i);

}
```

## 在你的程式碼中：
```rust

let n: usize = it.next().unwrap().parse().unwrap();

let k: usize = it.next().unwrap().parse().unwrap();

let _x: usize = it.next().unwrap().parse().unwrap();
```
這裡使用 `usize` 是因為：

- [n], [k] [_x] 都表示數量或索引
- 後續會用來作為陣列索引或迴圈範圍
- 不會是負數

## 相關型別：

- `isize` - 有符號版本，可以是正數或負數
- `u32`, `u64` - 固定大小的無符號整數
- `i32`, [i64] - 固定大小的有符號整數

## 轉換範例：
```rust
let a: usize = 10;

let b: i32 = a as i32;    // 轉換為 i32

let c: u64 = a as u64;    // 轉換為 u64
```

`usize` 是 Rust 中處理索引、長度和大小的標準型別，確保了在不同平台上的正確性和效能。

