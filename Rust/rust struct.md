在 Rust 裡，用 `struct`（結構體）可以把多個欄位組合成一個自訂型別。基本語法、實例化、以及為它實作方法（`impl`）的範例如下：

```rust
// 1. 定義一個 struct
//    這個 Point 結構有兩個欄位：x 與 y，都是 f64
struct Point {
    x: f64,
    y: f64,
}

// 2. 為 Point 實作「關聯函式」與「成員方法」
impl Point {
    // 關聯函式（associated function），類似其他語言的「靜態方法」
    pub fn new(x: f64, y: f64) -> Self {
        Point { x, y }
    }

    // 成員方法（&self 是不可變借用）
    pub fn distance_from_origin(&self) -> f64 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }

    // 可變借用方法，改變自身狀態
    pub fn translate(&mut self, dx: f64, dy: f64) {
        self.x += dx;
        self.y += dy;
    }
}

fn main() {
    // 3. 建立一個實例
    let mut p = Point::new(3.0, 4.0);

    // 呼叫方法
    println!("p 在原點的距離是：{}", p.distance_from_origin()); // 5.0

    // 修改欄位（需可變變數）
    p.translate(-1.0, 2.0);
    println!("平移後的座標：({:.1}, {:.1})", p.x, p.y); // (2.0, 6.0)
}
```

---

### 補充：自動實作常見 trait

如果要讓你的 struct 支援列印 (`{:?}`)、複製、比較等功能，可以在定義前加上 `derive`：

```rust
#[derive(Debug, Clone, Copy, PartialEq)]
struct Color {
    r: u8,
    g: u8,
    b: u8,
}

fn main() {
    let c1 = Color { r: 255, g: 0, b: 128 };
    let c2 = c1;  // 因為實作了 Copy，可以直接複製
    println!("c1 = {:?}", c1); // c1 = Color { r: 255, g: 0, b: 128 }
}
```

- `Debug`：允許用 `{:?}` 列印
    
- `Clone`：支援 `.clone()` 深層複製
    
- `Copy`：讓型別具備「位元複製」的能力（只有當所有欄位也都是 `Copy`）
    
- `PartialEq`：支援 `==` / `!=` 比較
    

---

### 小結

1. 用 `struct Name { field1: Type1, field2: Type2, … }` 定義結構。
    
2. 透過 `impl Name { … }` 加方法與關聯函式。
    
3. 利用 `#[derive(...)]` 自動生成常用 trait。
    

掌握以上，便能在 Rust 中用 `struct` 有條理地封裝與操作複合資料了！

