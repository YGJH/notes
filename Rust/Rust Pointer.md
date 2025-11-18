在 Rust 裡，「指標」(pointer) 的世界其實比 C/C++ 直覺，但要掌握「所有權」(ownership) 和「借用」(borrowing) 的精神。以下從最常見的引用（reference）講起，再帶到更底層的 raw pointer，最後以 `HashMap::entry` 的例子串起來。

---

## 1. 引用（Reference）：`&T` 與 `&mut T`

> **重點**：引用本質是「不擁有」資料的指標，借來用一用就還。

```rust
fn add_one(x: &i32) -> i32 {
    *x + 1  // 用 * 解引用取值
}

fn double(x: &mut i32) {
    *x *= 2; // 可變引用能改東西
}

fn main() {
    let a = 10;
    let b = add_one(&a);       // 傳入 &a，a 的所有權不移動
    println!("b = {}", b);     // b = 11

    let mut c = 5;
    double(&mut c);            // 傳入 &mut c，可在函式內改動
    println!("c = {}", c);     // c = 10
}
```

- `&T` 是 _不可變引用_，在借用期間，你不能改它。
    
- `&mut T` 是 _可變引用_，一次只能有一個可變引用（no aliasing + mutation）。
    

這樣的借用規則，讓你在編譯時就避免資料競爭(race condition)。

---

## 2. 傳遞機制：Move vs. Copy vs. Borrow

- **Move（搬移）**：將擁有權從 A 轉給 B，A 之後不能再用。
    
- **Copy**：對於小型、固定大小的類型（如整數、浮點數），Rust 幫你做「位元複製」，用完後 A、B 都還能繼續用。
    
- **Borrow（借用）**：用 `&` 或 `&mut`，不用移動所有權，也不複製資料。
    

```rust
fn take_ownership(v: Vec<i32>) { /* 拿走所有權 */ }
fn make_copy(x: i32) { /* i32: Copy trait */ }
fn borrow(v: &Vec<i32>) { /* 借用 */ }

fn main() {
    let v = vec![1, 2, 3];
    // take_ownership(v);   // v 的所有權被拿走，下面不能再用 v
    borrow(&v);             // 借用 v，v 還在
    make_copy(42);          // 複製 42
    println!("{:?}", v);    // OK
}
```

---

## 3. Raw Pointer：`*const T` / `*mut T`

> **何時用？** 幾乎只在跟 C 互操作(FFI)或極限效能場景時，才需要跳過 Rust 的借用檢查。

```rust
fn main() {
    let mut num = 5;
    let r1: *const i32 = &num as *const i32;
    let r2: *mut i32   = &mut num as *mut i32;

    unsafe {
        println!("r1 = {}", *r1);
        *r2 = 10;
        println!("num = {}", num);
    }
}
```

- **注意**：使用 raw pointer 內部就要 `unsafe {}`；得自己對齊、保證不違反別人的借用規則。
    

---

## 4. 智能指標（Smart Pointer）：`Box<T>`、`Rc<T>`、`Arc<T>`

- `Box<T>`：最簡單的 heap allocation。
    
- `Rc<T>`：單執行緒下的多重擁有(Reference Counted)。
    
- `Arc<T>`：跨執行緒的 `Rc<T>`。
    

```rust
use std::rc::Rc;

fn main() {
    let a = Rc::new(vec![1, 2, 3]);
    let b = Rc::clone(&a);
    println!("ref count = {}", Rc::strong_count(&a)); // 2
}
```

未來如果要寫 GUI、遊戲、Server，這些智能指標就派上用場啦。

---

## 5. `HashMap::entry` API：用指標拿到「鍵-值」的入口

`entry` 其實是讓你在一次借用裡，既可以查詢也可以插入，避免重複查找：

```rust
use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    let key = "apple";

    // 如果 key 不存在，就插入 3；否則拿到現有值的可變引用
    let count = map.entry(key).or_insert(3);
    *count += 1;  // 或者 *count = 5;

    // 下一次 entry 直接回傳已存在的 &mut i32
    let count = map.entry(key).or_insert(3);
    println!("{} appears {} times", key, count); // apple appears 4 times
}
```

> **底層原理**：
> 
> - `entry(key)` 回傳一個 `Entry` 枚舉：
>     
>     - `Entry::Occupied(OccupiedEntry)`（已存在）
>         
>     - `Entry::Vacant(VacantEntry)`（尚未存在）
>         
> - `or_insert`、`or_insert_with` 讓你優雅地處理兩種情況。
>     

---

### 小結

1. **借用**(`&T`, `&mut T`)：最常用的「安全指標」。
    
2. **搬移/複製**：所有權語意牢不可破，少踩雷。
    
3. **Raw pointer**：少用、只在特殊場合。
    
4. **智能指標**：Heap allocation、Reference Counting…為複雜資料結構服務。
    
5. **Entry API**：用 `&mut self` 拿到「可變借用」，即查又改，性能佳又語意清晰。
    

下次寫程式，想要更高階的指標玩法，還可以看看 `Pin<T>`、`Mutex<T>`、`RwLock<T>`…Rust 的指標世界，永遠都有新招等著你探索！ 🚀


可變借用（`&mut T`）和真正的「移動」（move）其實差得有點像隔壁鄰居和路過朋友的差別──看起來都把東西帶走了，但歸還機制不同：

1. **Move**：
    
    ```rust
    fn take_ownership(v: Vec<i32>) { /* v 的所有權被搬走 */ }
    let v = vec![1,2,3];
    take_ownership(v);
    // println!("{:?}", v); // ❌ v 已經不屬於你了
    ```
    
    把東西打包直接帶走，不會還回來。
    
2. **Mutable Borrow (`&mut`)**：
    
    ```rust
    fn tweak(x: &mut i32) {
        *x += 10;
    }
    let mut a = 5;
    tweak(&mut a);     // 借走、改一改
    println!("{}", a);  // ✅ 借完還你，新值是 15
    ```
    
    _所有權（ownership）一直留在原位_，只是假借「修改權」（mutation right）。借用結束後，原本的變數依然健在、可以再用。
    

> 💡 **比喻**：
> 
> - **Move** 就像你把自行車騎走，主人再也看不到那台車。
>     
> - **`&mut`** 就像你借朋友騎車兜風，定好時間、結束後歸還，車子仍屬於你。
>     

---

## `HashMap::entry` vs `get` / `get_mut` 的選擇

Rust 標準庫提供了幾種從 `HashMap` 取值或插入的途徑，選哪個要看你的需求：

|方法|回傳型態|適用場景|
|---|---|---|
|`get(&K)`|`Option<&V>`|**只讀**：你確定不會新增、也不會改值。|
|`get_mut(&K)`|`Option<&mut V>`|**單純修改**：鍵一定存在，或不存在就跳過。|
|`entry(K)`|`Entry<K, V>` 枚舉|**插入或修改**：根據是否存在，執行不同邏輯。|

### 1. 單純查值、確定存在就讀用 `get`

```rust
if let Some(count) = map.get(&key) {
    println!("Found: {}", count);
} else {
    println!("No such key");
}
```

### 2. 需要改現有值就用 `get_mut`

```rust
if let Some(count) = map.get_mut(&key) {
    *count += 1;
} else {
    // 要插入？還是直接忽略
}
```

### 3. 要「不存在就插入／存在就修改」時，用 `entry`

```rust
use std::collections::HashMap;

let mut map = HashMap::new();
let key = "rust";

map.insert(key, 1);

// 在同一次 Hash 查找裡，決定插入或修改
let entry = map.entry(key).or_insert(0);
*entry += 1;  // 如果 key 不在，先插入 0；再加一
```

- **優點**：只做一次 hash lookup（效能更好），語意也最清楚。
    
- **缺點**：語法稍微長一點，但可讀性通常不差。
    

---

## 小結與建議

- **當你只想「看」而不動它**，用 `get`。
    
- **當你只想「改」已存在的值**，用 `get_mut`。
    
- **當你同時有插入和修改的需求**，或希望只 lookup 一次就搞定，選 `entry`。
    

前瞻性觀點──隨著 Rust 生態發展，`HashMap` 也可能引入更靈活的 raw‑entry 或並行版本（`dashmap`、`chashmap` 等），但基本原理不變：**用最貼近需求的 API，就能最優雅、最高效地管理你的資料結構。**