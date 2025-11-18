
---

## 1. 排序（Sorting）與自訂比較函式

Rust 標準庫提供 `sort`、`sort_unstable` 在切片（slice）上排序：

```rust
let mut v = vec![3, 1, 4, 1, 5, 9];

// 預設升冪排序（需要元素實作 Ord）
v.sort();
println!("{:?}", v); // [1, 1, 3, 4, 5, 9]

// 不穩定排序（速度稍快，但不保證相等元素原本相對順序）
v.sort_unstable();

// 自訂比較：降冪
v.sort_by(|a, b| b.cmp(a));
println!("{:?}", v); // [9, 5, 4, 3, 1, 1]

// 更自由的 Comparator：根據數字的「餘數」排序
v.sort_by(|a, b| {
    let ra = a % 3;
    let rb = b % 3;
    ra.cmp(&rb)
});
println!("{:?}", v);
```

---

## 2. 運算子多載（Operator Overloading）

Rust 透過實作標準 trait（`std::ops` 裡的）來支援運算子多載。例如，要讓兩個自訂向量相加：

```rust
use std::ops::Add;

#[derive(Debug, PartialEq)]
struct Vec2 { x: f64, y: f64 }

// impl Add 以支援 `a + b`
impl Add for Vec2 {
    type Output = Vec2;
    fn add(self, other: Vec2) -> Vec2 {
        Vec2 {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

fn main() {
    let a = Vec2 { x: 1.0, y: 2.0 };
    let b = Vec2 { x: 3.0, y: 4.0 };
    let c = a + b;
    println!("{:?}", c); // Vec2 { x: 4.0, y: 6.0 }
}
```

你也可以實作 `Sub`、`Mul`、`Div` 等等。

---

## 3. 閉包（Lambda）與函式指標

- **簡單閉包**
    
    ```rust
    let nums = vec![1, 2, 3, 4, 5];
    // map 每個元素都 +1
    let plus_one: Vec<_> = nums.iter().map(|x| x + 1).collect();
    ```
    
- **帶環境捕獲**
    
    ```rust
    let factor = 10;
    let mut apply = |x: i32| x * factor;
    println!("{}", apply(3)); // 30
    ```
    

---

## 4. 自訂 HashMap 的哈希演算法

標準的 `HashMap<K,V>` 底層使用預設的 `RandomState`（SipHash），若要換成自己的：

1. 為你的有鍵型別實作 `Hash`、`Eq`。
    
2. 實作一個 `BuildHasher`，並在建立 `HashMap` 時指定。
    

```rust
use std::collections::HashMap;
use std::hash::{BuildHasher, Hasher, Hash};

/// 一個非常簡單（但絕對不安全／不好用）的例子：把數值直接當 hash
struct SimpleHasher(u64);

impl Hasher for SimpleHasher {
    fn finish(&self) -> u64 { self.0 }
    fn write(&mut self, bytes: &[u8]) {
        // 把前 8 個 byte 直接 interpret 成 u64
        if bytes.len() >= 8 {
            let mut arr = [0u8; 8];
            arr.copy_from_slice(&bytes[0..8]);
            self.0 = u64::from_ne_bytes(arr);
        }
    }
}

/// BuildHasher 工廠
#[derive(Default)]
struct SimpleBuild;

impl BuildHasher for SimpleBuild {
    type Hasher = SimpleHasher;
    fn build_hasher(&self) -> SimpleHasher {
        SimpleHasher(0)
    }
}

// 鍵型別必須實作 Hash + Eq
#[derive(Hash, Eq, PartialEq, Debug)]
struct Key(u64);

fn main() {
    let mut map: HashMap<Key, &str, SimpleBuild> = 
        HashMap::with_hasher(SimpleBuild::default());

    map.insert(Key(42), "答案");
    println!("{:?}", map.get(&Key(42))); // Some("答案")
}
```

---

## 5. 自訂兩個物件「相等」的方法

只要為你的型別實作或 `derive` 這兩個 trait：

```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u8,
}

// 自訂 PartialEq
impl PartialEq for Person {
    fn eq(&self, other: &Self) -> bool {
        // 只要 name 一樣，就算相同
        self.name == other.name
    }
}
impl Eq for Person {} // 空實作

fn main() {
    let p1 = Person { name: "小明".into(), age: 20 };
    let p2 = Person { name: "小明".into(), age: 30 };
    assert!(p1 == p2);
}
```

---

### 小結

1. **排序**：`.sort()` / `.sort_by()` / `.sort_unstable()`。
    
2. **比較函式**：在 `.sort_by(...)` 中傳入 closure。
    
3. **運算子**：實作 `std::ops::Add`、`Sub`… 來 overload。
    
4. **閉包**：用 `|args| expr`，捕獲外部變數。
    
5. **自訂 Hash**：實作 `Hasher` + `BuildHasher`，用 `HashMap::with_hasher`。
    
6. **自訂相等**：實作 `PartialEq` (+ `Eq`)，可決定哪個欄位比較。
    

掌握這些，Rust 幾乎可以滿足你在型別、演算法、效能上的所有客製化需求！