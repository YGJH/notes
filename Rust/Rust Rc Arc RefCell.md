在 Rust 中，`Rc<T>`（Reference Counted）和 `Arc<T>`（Atomically Reference Counted）都是用來實現「共享擁有權」的智慧指標（smart pointer），但它們在使用場景、執行緒安全性和效能上有明顯差異：

---

## 1. 基本概念

|特性|`Rc<T>`|`Arc<T>`|
|---|---|---|
|名稱|Reference Counted|Atomically Reference Counted|
|擁有權共用|多個擁有者共用同一份資料|多個擁有者共用同一份資料|
|執行緒安全|**非執行緒安全**（`!Send`、`!Sync`）|**執行緒安全**（實作了 `Send`、`Sync`）|
|計數器類型|非原子操作的整數|原子操作的整數（`AtomicUsize`）|
|適用場景|單執行緒內部結構共享|多執行緒間結構共享|
|效能|較快（計數操作是純整數加減）|慢一些（計數操作需要原子加減）|

---

## 2. 何時用 `Rc<T>`

- **單執行緒**：程式中只有一條執行緒，不需要考慮同步問題。
    
- **樹或圖結構**：例如樹、四叉樹、DAG 等，多個父節點共同擁有同一個子節點。
    
- **性能優先**：由於引用計數不是原子操作，呼叫 `clone()`／`drop()` 時只做單純的加減，性能開銷較低。
    

```rust
use std::rc::Rc;

#[derive(Debug)]
struct Node {
    value: i32,
    children: Vec<Rc<Node>>,
}

fn main() {
    let leaf = Rc::new(Node { value: 3, children: vec![] });
    let branch = Rc::new(Node { value: 5, children: vec![leaf.clone(), leaf.clone()] });
    println!("branch = {:?}, strong_count = {}", branch, Rc::strong_count(&leaf));
}
```

- `Rc::strong_count(&leaf)` 會顯示有多少個 `Rc` 實例指向同一個底層資料。
    

---

## 3. 何時用 `Arc<T>`

- **多執行緒**：程式有多條執行緒需要共同讀取或擁有同一份資料。
    
- **跨執行緒傳遞**：若你要將擁有者移到其他執行緒（例如放入 `std::thread::spawn` 的閉包），必須使用 `Arc<T>`。
    
- **配合鎖**：常見用法是 `Arc<Mutex<T>>` 或 `Arc<RwLock<T>>`，在多執行緒下進行共享與可變資料存取。
    

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let cnt = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = cnt.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }

    for h in handles {
        h.join().unwrap();
    }
    println!("Result: {}", *counter.lock().unwrap());
}
```

- 這裡 `Arc` 讓多個執行緒擁有同一個 `Mutex<i32>`，再透過鎖進行安全的可變存取。
    

---

## 4. `clone()` 與 計數減少

- **呼叫 `clone()`**：不會深度複製資料，只是複製智慧指標本身，並把引用計數加 1。
    
- **作用域結束**：當所有 `Rc<T>`／`Arc<T>` 都超出作用域或被 `drop` 時，引用計數歸零，底層資料才真正釋放。
    

```rust
let a = Rc::new(10);
{
    let b = a.clone();      // 計數器 +1
    println!("{}", *b);      // 10
}                           // b drop，計數器 -1
// 只有 a 留下，底層資料還在
```

---

## 5. `Weak<T>`：避免循環引用

- **問題**：若使用 `Rc<T>`／`Arc<T>` 在雙向結構（parent↔child）或圖結構時，容易產生「循環引用」導致記憶體永不釋放。
    
- **解法**：使用 `Weak<T>` 來表示「非擁有性」引用，不會增加強引用計數（strong count），只會增加弱引用計數（weak count）。可以透過 `.upgrade()` 嘗試把 `Weak<T>` 升級成 `Rc<T>`／`Arc<T>`，若底層已被釋放，則返回 `None`。
    

```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

#[derive(Debug)]
struct Node {
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}

fn main() {
    let leaf = Rc::new(Node { parent: RefCell::new(Weak::new()), children: RefCell::new(vec![]) });
    let branch = Rc::new(Node { parent: RefCell::new(Weak::new()), children: RefCell::new(vec![leaf.clone()]) });
    *leaf.parent.borrow_mut() = Rc::downgrade(&branch);
    // 使用 Weak 就不會和 branch 形成強循環，資料可以正常釋放
}
```

---

## 6. 小結

- **`Rc<T>`**：適合單執行緒，共享不可變資料，效能較佳。
    
- **`Arc<T>`**：適合多執行緒，共享資料並保證線程安全，但計數開銷略高。
    
- **搭配 Cell 類型**：若要在共享結構中修改內部狀態，可用 `RefCell<T>`（單執行緒）或 `Mutex<T>`／`RwLock<T>`（多執行緒）。
    
- **避免記憶體洩漏**：在可能形成環狀結構時，使用 `Weak<T>` 打破強引用循環。
    

掌握這些指標的特性與使用時機，能讓你在 Rust 中方便又安全地實現複雜的資料結構與並行邏輯。[]()