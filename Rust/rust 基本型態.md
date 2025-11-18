Rust 的基本型態（primitive types）主要可以分成「標量型態」（scalar types）與「複合型態」（compound types）。下面先列出最常用的標量型態，特別是數值相關的部分：

---

## 一、數值型態（Numeric Types）

### 1. 整數（Integer）

|類別|符號|說明|位元數|
|---|---|---|---|
|有號整數|`i8`|範圍 –2⁷ … 2⁷-1|8|
||`i16`|範圍 –2¹⁵ … 2¹⁵-1|16|
||`i32`|範圍 –2³¹ … 2³¹-1|32|
||`i64`|範圍 –2⁶³ … 2⁶³-1|64|
||`i128`|範圍 –2¹²⁷ … 2¹²⁷-1|128|
||`isize`|指標大小（pointer-sized）；32 位元系統即 32 位，64 位元系統即 64 位|32/64|
|無號整數|`u8`|範圍 0 … 2⁸-1|8|
||`u16`|範圍 0 … 2¹⁶-1|16|
||`u32`|範圍 0 … 2³²-1|32|
||`u64`|範圍 0 … 2⁶⁴-1|64|
||`u128`|範圍 0 … 2¹²⁸-1|128|
||`usize`|指標大小（pointer-sized）|32/64|

- **註**：`isize`／`usize` 用於表示記憶體位址大小（如索引陣列時）；其位元寬度會隨目標平台（32-bit／64-bit）變化。
    

### 2. 浮點數（Floating-Point）

|類別|符號|說明|位元數|
|---|---|---|---|
|單精度|`f32`|約 6 位十進位有效數字|32|
|雙精度|`f64`|約 15 位十進位有效數字|64|

- **沒有**內建的 `f128`（128-bit）型態；若真的需要更高精度，可以考慮引入外部 crate（例如 [`rug`](https://crates.io/crates/rug) 或 [`decimal`](https://crates.io/crates/decimal)）來做多精度運算。
    

---

## 二、其他標量型態

- **布林（Boolean）**
    
    ```rust
    let flag: bool = true;
    ```
    
- **字元（Character）**
    
    ```rust
    let c: char = '中';  // 4 bytes, Unicode Scalar Value
    ```
    
- **單位型態（Unit）**
    
    ```rust
    let unit: () = ();
    ```
    

---

## 三、複合型態（Compound Types）

> 雖然不算「primitive numeric」，但也很常用：

1. **元組（Tuple）**
    
    ```rust
    let tup: (i32, f64, char) = (42, 3.14, '🙂');
    let (x, y, z) = tup;
    ```
    
2. **陣列（Array） & 切片（Slice）**
    
    ```rust
    let arr: [u8; 4] = [1, 2, 3, 4];
    let slice: &[u8] = &arr[1..3];
    ```
    
3. **字串切片（String Slice）**
    
    ```rust
    let s: &str = "Hello, 世界";
    ```
    

---

### 小結

- **整數**：`i8`/`i16`/`i32`/`i64`/`i128` 以及指標大小的 `isize`，對應無號版 `u8` … `u128`/`usize`。
    
- **浮點**：只有 `f32` 與 `f64`，沒有內建的 128 位元浮點。
    
- **其他**：`bool`、`char`、`()`，以及常見的複合型態（tuple、array、slice、&str）。
    

這些就是 Rust 最常用、可直接放在棧上且零成本的基本型態。希望對你有幫助！
