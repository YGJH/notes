在 Rust 裡，「vtable」（Virtual Method Table，虛擬方法表）是實作「動態分派」（dynamic dispatch）背後的關鍵機制。當你使用 trait 物件（`dyn Trait`）時，編譯器會把它表現成一個「fat pointer」——一個指向實際資料的指標＋一個指向對應 vtable 的指標，呼叫方法時就透過 vtable 來找到正確的函式指針。

---

## 1. 為什麼需要 vtable？

Rust 的 trait 可以用靜態分派或動態分派來呼叫方法：

- **靜態分派**（static dispatch）：泛型 `fn foo<T: Trait>(x: T)`，呼叫時編譯期就知道 `T` 的具體型別，會把方法直接「內聯」或以普通函式呼叫。
    
- **動態分派**（dynamic dispatch）：使用 trait 物件 `&dyn Trait`、`Box<dyn Trait>` 等，此時在編譯期不知道底層是什麼型別，只知道要呼叫 Trait 中的方法。這時候就得在執行期再決定，編譯器使用 vtable 來實作。
    

---

## 2. vtable 的結構

對於每一個具體型別 `S` 實作了某個 trait `Trait`，編譯器都會幫它產生一張 vtable，裡面大致包含：

1. **drop_in_place**：指向 `S` 的解構（`Drop`）函式，用於在結束生命週期時呼叫。
    
2. **size**：型別 `S` 的大小（`usize`），讓 runtime 知道要釋放多少記憶體。
    
3. **align**：型別 `S` 的對齊（`usize`），釋放或配置記憶體時要考慮的對齊。
    
4. **Trait 方法指針**：對應 trait 每一個方法的函式指標，簽名均被「擦除」成接受一個未指定型別的裸指標（`*const ()` 或 `*mut ()`）並回傳等同形狀的值。
    

```text
┌─────────────────────────────┐
│ vtable for S as dyn Trait   │
├──────────────┬──────────────┤
│ drop         │ fn(*mut ())  │
│ size         │ usize        │
│ align        │ usize        │
│ method1      │ fn(*mut ())  │
│ method2      │ fn(*mut (), args…) -> Ret │
│ …            │ …            │
└──────────────┴──────────────┘
```

---

## 3. fat pointer：資料指標＋vtable 指標

當你宣告 `let obj: &dyn Trait = &concrete;`，在記憶體中存的其實是兩個指標：

```rust
// 內部等同於：
struct TraitObject {
    data: *const (),       // 指向 concrete 的實際資料
    vtable: *const Vtable,  // 指向上面那張 vtable
}
```

呼叫 `obj.method1()` 時，實際執行：

1. 從 `obj.vtable` 取得 `method1` 的函式指標。
    
2. 把 `obj.data`（轉成 `*mut ()`）以及其他參數傳給該函式指標。
    
3. 執行真正屬於 `concrete` 的方法實作。
    

---

## 4. 使用範例

```rust
trait Draw {
    fn draw(&self);
}

struct Circle { /* ... */ }
struct Square { /* ... */ }

impl Draw for Circle {
    fn draw(&self) {
        println!("畫一個圓形");
    }
}
impl Draw for Square {
    fn draw(&self) {
        println!("畫一個正方形");
    }
}

fn render(shape: &dyn Draw) {
    // 這裡是動態分派，呼叫時靠 vtable 決定要執行哪個 draw
    shape.draw();
}

fn main() {
    let c = Circle { /* ... */ };
    let s = Square { /* ... */ };

    let shapes: Vec<&dyn Draw> = vec![&c, &s];
    for sh in shapes {
        render(sh);
    }
}
```

每個 `&dyn Draw` 底下都有自己的 vtable，分別指向 `Circle::draw` 與 `Square::draw`，執行時就正確呼叫。

---

## 5. 效能考量

- **額外的間接呼叫成本**：每次動態分派都要先從 vtable 拿函式指標再呼叫，無法內聯，也就是比靜態分派慢一點。
    
- **程式碼大小**：靜態分派會為每個 `T` 產生一份函式實作；動態分派只要一張 vtable，多型實例共用一份程式碼。
    

---

### 總結

> **vtable** 是 Rust 在動態分派（`dyn Trait`）時負責「找方法」的工具。它把具體型別對 trait 的每個方法實作，以函式指標的形式收進一張表上，fat pointer（資料＋vtable）讓你在執行期也能呼叫到正確的函式。

掌握 vtable 概念，就能在使用 trait 物件時，作出效能與靈活度之間的平衡選擇！
