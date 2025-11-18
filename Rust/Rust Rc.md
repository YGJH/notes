在 Rust 中，`Rc<T>`（Reference Counted）是一種智慧指標（smart pointer），用來在**單執行緒**環境中對同一份資料做多重擁有（shared ownership）：

1. **引用計數**
    
    - 每當你用 `.clone()` 複製一個 `Rc<T>`，底層都會將一個內部的計數器（reference count）加 1；當所有的 `Rc<T>` 都被丟棄後，計數器歸零，底層資料才真正釋放。
        
2. **單執行緒限制**
    
    - `Rc<T>` 只適用在單執行緒；如果你需要跨執行緒共享，就要用 `Arc<T>`（Atomic Reference Counted），它的計數器在多執行緒下也能安全變動。
        
3. **用途**
    
    - 在像四叉樹這種樹狀或圖狀結構裡，常常有多個 parent 指向同一份子樹；用 `Rc<Node>` 就可以讓多個節點共享子樹，而不用複製整棵樹，也不用手動管理記憶體。
        
4. **不可變借用**
    
    - `Rc<T>` 本身只提供不可變借用；如果要在多處共享之餘還要修改，通常會搭配 `RefCell<T>`（內部可變性）或其它 Cell 類型，像我們在 `Node` 裡就用 `RefCell<Option<Rc<Node>>>` 來快取結果。
        

```rust
use std::rc::Rc;

let a = Rc::new(5);
let b = a.clone();      // 計數器從 1 變成 2
println!("{}", *a);     // 解引用取得內部值 5
// a, b 都超出 scope 時，才會把底層記憶體釋放
```

總結：`Rc<T>` 就是 Rust 裡用來在單一執行緒裡，安全又自動地做「多重擁有」的指標類型。


下面用兩個簡單範例示範 `Rc<T>` 在 Rust 裡的實際用法，並同時給出 C++ 對應的 `std::shared_ptr<T>` 或 `std::weak_ptr<T>` 實現，幫助你對照理解。

---

## 範例一：單向樹（Tree）結構共享子節點

### Rust（使用 `Rc`）

```rust
use std::rc::Rc;

#[derive(Debug)]
struct Node {
    value: i32,
    children: Vec<Rc<Node>>,
}

fn main() {
    // 建立兩個 leaf 節點
    let leaf1 = Rc::new(Node { value: 1, children: vec![] });
    let leaf2 = Rc::new(Node { value: 2, children: vec![] });

    // 建立一個 branch，指向同樣的兩個 leaf
    let branch = Rc::new(Node {
        value: 10,
        children: vec![leaf1.clone(), leaf2.clone()],
    });

    println!("branch = {:?}\nleaf1 strong_count = {}\nleaf2 strong_count = {}",
        branch,
        Rc::strong_count(&leaf1),
        Rc::strong_count(&leaf2),
    );
}
```

**重點**

- `leaf1.clone()`、`leaf2.clone()` 只是複製指標；底下的 `Node` 只存一份。
    
- `Rc::strong_count(&leaf1)` 可以看到有幾個 `Rc` 持有它（這裡會是 2）。
    

### C++（使用 `std::shared_ptr`）

```cpp
#include <iostream>
#include <memory>
#include <vector>

struct Node {
    int value;
    std::vector<std::shared_ptr<Node>> children;
};

int main() {
    auto leaf1 = std::make_shared<Node>(Node{1, {}});
    auto leaf2 = std::make_shared<Node>(Node{2, {}});

    auto branch = std::make_shared<Node>(Node{
        10,
        std::vector<std::shared_ptr<Node>>{leaf1, leaf2}
    });

    std::cout << "branch->value = " << branch->value << "\n";
    std::cout << "leaf1 use_count = " << leaf1.use_count() << "\n";
    std::cout << "leaf2 use_count = " << leaf2.use_count() << "\n";
}
```

> **對照**：`Rc<T>` ↔ `std::shared_ptr<T>`；兩者都管理引用計數並自動釋放。

---

## 範例二：雙向連結避免記憶體洩漏（Cycle）——用 `Weak`

如果你有雙向結構（parent ↔ child），單純用強引用會造成循環持有，永遠不會釋放。這時要用弱引用。

### Rust（`Rc` + `Weak`）

```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

#[derive(Debug)]
struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}

fn main() {
    // 建立 parent 和 child
    let parent = Rc::new(Node {
        value: 100,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });
    let child = Rc::new(Node {
        value: 200,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    // child 持有 parent 的弱引用
    *child.parent.borrow_mut() = Rc::downgrade(&parent);
    // parent 持有 child 的強引用
    parent.children.borrow_mut().push(child.clone());

    println!("parent strong = {}, weak = {}",
        Rc::strong_count(&parent),
        Rc::weak_count(&parent),
    );
    println!("child strong = {}, weak = {}",
        Rc::strong_count(&child),
        Rc::weak_count(&child),
    );

    // 嘗試從 weak 升級
    if let Some(up) = child.parent.borrow().upgrade() {
        println!("child's parent value = {}", up.value);
    }
}
```

### C++（`shared_ptr` + `weak_ptr`）

```cpp
#include <iostream>
#include <memory>
#include <vector>

struct Node {
    int value;
    std::weak_ptr<Node> parent;
    std::vector<std::shared_ptr<Node>> children;
};

int main() {
    auto parent = std::make_shared<Node>();
    parent->value = 100;

    auto child = std::make_shared<Node>();
    child->value = 200;

    // child 持有 parent 的弱引用
    child->parent = parent;
    // parent 持有 child 的強引用
    parent->children.push_back(child);

    std::cout << "parent use_count = " << parent.use_count()
              << ", expired children count = " << parent->children.size() << "\n";
    
    if (auto up = child->parent.lock()) {
        std::cout << "child's parent value = " << up->value << "\n";
    }
}
```

> **對照**：
> 
> - `Rc::downgrade(&parent)` ↔ `std::weak_ptr<Node> parent = parent_shared_ptr;`
>     
> - `weak_ptr::upgrade()` ↔ `weak_ptr::lock()`
>     

---

### 總結

1. **單執行緒**→ 用 `Rc<T>`（或 C++ 的 `shared_ptr<T>`）
    
2. **多執行緒**→ Rust 用 `Arc<T>`（C++ 也可搭配 `std::atomic` 或直接用 `shared_ptr`，但要注意 thread safety）
    
3. **避免循環**→ 用弱引用 `Weak<T>` / `std::weak_ptr<T>`
    

這兩個範例示範了常見的樹狀與雙向結構，應該能幫助你判斷「什麼時候該用 `Rc`（或 `Arc`）」，以及「如何搭配 `Weak` 打破循環」。