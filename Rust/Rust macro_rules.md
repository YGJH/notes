下面將從宏的基本概念、宣告式宏（`macro_rules!`）、過程式宏（Procedural Macros）、以及實務注意事項等面向，詳細介紹 Rust 的宏系統。

---

## 一、什麼是宏（Macro）？

- **編譯期元程式**：宏是在編譯階段展開（expand）的程式碼生成工具，可以生成任意合法的 Rust 代碼。
    
- **動機**：減少重複程式碼、實現 DSL（領域專屬語言），或在型別系統外實現無法用一般函式達成的語法擴充。
    
- **分類**：分為「宣告式宏（declarative macros）」，也就是常見的 `macro_rules!`，以及「過程式宏（procedural macros）」，包含三種形式：derive、屬性（attribute-like）、函式樣（function-like）。
    

---

## 二、宣告式宏：`macro_rules!`

### 1. 基本語法

```rust
macro_rules! my_vec {
    ( $( $x:expr ),* $(,)? ) => {
        {
            let mut v = Vec::new();
            $(
                v.push($x);
            )*
            v
        }
    };
}
```

- `$( ... )*`：重複匹配。
    
- `$x:expr`：匹配任意運算式。
    
- `$(,)?`：可選的結尾逗號。
    

### 2. 模式（Pattern）與重複（Repetition）

- **單一匹配**：`($a:ident)` 匹配一個標識符。
    
- **多重匹配**：`$( $name:ident ),+` 至少一個，`*` 表示可有可無、多個。
    
- **屬性捕獲**：可以在展開中使用捕獲變數（如 `$name`）生成程式碼。
    

### 3. 範例：`vec!` 實作簡化

```rust
macro_rules! simple_vec {
    ( $( $elem:expr ),* ) => {
        {
            let mut v = Vec::new();
            $(
                v.push($elem);
            )*
            v
        }
    };
}

// 使用：
let v = simple_vec![1, 2, 3];
```

### 4. 宣告式宏的優缺點

- **優點**：易讀、學習曲線平緩、不需要額外 crate。
    
- **缺點**：語法樹規格比較簡單，無法做複雜邏輯（例如任意高階操作、型別推導），調試錯誤訊息不直觀。
    

---

## 三、過程式宏（Procedural Macros）

過程式宏是用 Rust 程式撰寫的，接收並操作 Token Stream，最終輸出 Token Stream。

### 1. 三種類型

1. **Derive Macros**：自動為結構或列舉產生 trait 實作，常見如 `#[derive(Debug, Clone)]`。
    
2. **Attribute-like Macros**：用在宣告上，可自訂屬性。例如 `#[route(GET, "/")]`。
    
3. **Function-like Macros**：看起來像函式呼叫，例如 `sql!(...)`，可實現 SQL DSL。
    

### 2. 實作流程

1. **建立 proc-macro crate**
    
    ```toml
    [lib]
    proc-macro = true
    ```
    
2. **引入外部解析套件**，例如 `syn`（解析）與 `quote`（生成）：
    
    ```rust
    extern crate proc_macro;
    use proc_macro::TokenStream;
    use quote::quote;
    use syn;
    ```
    
3. **撰寫宏函式**
    
    ```rust
    #[proc_macro_derive(MyTrait)]
    pub fn my_trait_derive(input: TokenStream) -> TokenStream {
        let ast = syn::parse(input).unwrap();
        impl_my_trait(&ast)
    }
    
    fn impl_my_trait(ast: &syn::DeriveInput) -> TokenStream {
        let name = &ast.ident;
        let gen = quote! {
            impl MyTrait for #name {
                fn hello() {
                    println!("Hello from {}", stringify!(#name));
                }
            }
        };
        gen.into()
    }
    ```
    
4. **使用**
    
    ```rust
    use my_proc_macro_crate::MyTrait;
    #[derive(MyTrait)]
    struct Foo;
    // Foo::hello();
    ```
    

### 3. 優點與應用

- **靈活強大**：能操作完整的語法樹（AST），實現複雜邏輯。
    
- **常見應用**：自動產生序列化程式（Serde 的 `#[derive(Serialize)]`）、Web 框架路由、嵌入式 DSL、測試框架（如自動生成測試）。
    

---

## 四、宏與 DSL

- **內嵌 DSL**：利用宣告式或過程式宏打造領域專屬語言，例如 `sql!`、`html!`、`regex!`。
    
- **可讀性注意**：DSL 雖然方便，但過度使用可能讓程式碼對新手不友善。
    

---

## 五、實務注意事項

1. **展開階段錯誤**：宏錯誤多半在展開階段，錯誤訊息指向展開後的程式，閱讀時需先將宏「手動展開」思考。
    
2. **命名衝突**：宣告式宏會把變數插入呼叫方命名空間，使用前綴（`$crate::foo`）或全域路徑避免衝突。
    
3. **檔案組織**：過程式宏需要獨立 crate，且無法在同一 crate 中呼叫。
    
4. **編譯速度**：過度使用或複雜的過程式宏會影響編譯時間，請衡量效益。
    

---

### 小結

- **宣告式宏（`macro_rules!`）**：簡單、直觀，適合少量重複程式碼生成。
    
- **過程式宏（Procedural Macros）**：功能強大，可操作 AST，適用於大型框架與 DSL。
    

掌握宏的兩大體系後，即可根據需求在專案中靈活運用，減少樣板程式、提升開發效率。祝你在 Rust 世界裡宏用自如！

以下章節將深入解構 `macro_rules!` 的語法結構與特性，並搭配範例說明。

---

## 1. 基本結構

```rust
macro_rules! 名稱 {
    ( 模式1 ) => { 展開式1 };
    ( 模式2 ) => { 展開式2 };
    // …
}
```

- **名稱**：宏的識別符，呼叫時以 `名稱!(…)` 或 `名稱![…]` 語法使用。
    
- **模式（pattern）**：定義輸入 token 的結構，使用 metavariable（如 `$x:expr`）與重複語法。
    
- **展開式（expansion）**：匹配後生成的程式碼片段，可包含對應的 metavariable。
    

---

## 2. Metavariable 與 Fragment Specifiers

在模式裡，使用 `$變數:片段類型` 來捕獲呼叫方輸入的 token。常見片段類型有：

|片段類型（fragment）|意義|
|---|---|
|`expr`|任意運算式|
|`ident`|標識符（變數、函式名、struct 名等）|
|`tt`|一個 token tree（最基礎的單位）|
|`path`|Rust 路徑（如 `std::io::Result`）|
|`ty`|型別宣告|
|`pat`|模式（pattern）|
|`block`|區塊（如 `{ … }`）|
|`meta`|屬性內容（attribute 內的 meta）|
|`literal`|字面值（數字、字串、字元等）|
|`item`|項目（函式、struct、impl 等頂層項目）|
|`vis`|可見度修飾詞（如 `pub(crate)`）|

> **範例**：捕獲一個運算式與一個型別
> 
> ```rust
> macro_rules! foo {
>     ($e:expr; $t:ty) => {
>         {
>             let x: $t = $e;
>             x
>         }
>     };
> }
> // 使用：let v = foo!(1 + 2; i32);
> ```

---

## 3. 重複（Repetition）語法

重複語法可用來匹配重複出現的模式。

```rust
$( 子模式 ),*    // 0 或 多個，以逗號分隔
$( 子模式 ),+    // 1 或 多個，以逗號分隔
$( 子模式 ),?    // 0 或 1 次，以逗號分隔
```

- **`$(…)*`**：可重複 0 次以上
    
- **`$(…)+`**：至少要重複 1 次
    
- **`$(…)?`**：最多一次
    

> **範例：彈性參數**
> 
> ```rust
> macro_rules! my_vec {
>     ( $( $x:expr ),* $(,)? ) => {
>         {
>             let mut v = Vec::new();
>             $(
>                 v.push($x);
>             )*
>             v
>         }
>     };
> }
> // 可以接受 zero、one、多個參數，結尾逗號可有可無
> let v1 = my_vec![];
> let v2 = my_vec![1];
> let v3 = my_vec![1, 2, 3,];
> ```

重複可以巢狀，並且在展開式中對應相同的結構：

```rust
macro_rules! table {
    (
>       $(
>         $row:ident => [ $( $cell:expr ),* ]
>       ),*
>     ) => {
>       {
>         let mut tbl = std::collections::HashMap::new();
>         $(
>           let mut vec = Vec::new();
>           $(
>             vec.push($cell);
>           )*
>           tbl.insert(stringify!($row), vec);
>         )*
>         tbl
>       }
>     };
}
let t = table!(
    row1 => [1, 2],
    row2 => [3, 4, 5]
);
```

---

## 4. 分隔符（Separator）

在重複語法前，可以指定任何字元（逗號、分號、`=>`，甚至換行）作為分隔符：

```rust
$( 子模式 ; )*    // 以分號分隔
$( 子模式 => )*  // 以 => 分隔
```

---

## 5. 巨集衛生（Hygiene）與 `$crate`

Rust 宏預設「巨集衛生」，避免命名衝突。但如果你想引用本 crate 內部項目，通常用 `$crate`：

```rust
macro_rules! log_num {
    ($e:expr) => {
        $crate::logger::log($e);
    };
}
```

這樣不管在哪個模組或外部 crate 呼叫，都能正確地指向原本的 `logger`。

---

## 6. 匹配優先順序

1. 依照宣告順序依序嘗試每個分支模式。
    
2. 越具體的模式應放在前面，否則可能被「廣義模式」攔截。
    

```rust
macro_rules! example {
    // 具體：單引數
    ($x:expr) => { println!("single: {}", $x); };
    // 廣義：多引數
    ( $( $x:expr ),* ) => { println!("multiple: {:?}", vec![$($x),*]); };
}
```

若將多引數的放在前面，所有呼叫皆會走到第一分支。

---

## 7. 錯誤排除技巧

1. **手動展開**：將宏呼叫寫到簡易範例裡，用手動貼上展開後程式碼協助除錯。
    
2. **少量模式**：先寫最簡版的模式，再逐步增加複雜度。
    
3. **`cargo expand`**：安裝並使用，可直接看到展開後完整程式碼：
    
    ```bash
    cargo install cargo-expand
    cargo expand
    ```
    

---

## 8. 小結

- `macro_rules!` 透過 **模式匹配** + **重複語法**，實現在編譯期生成程式碼。
    
- 熟悉**metavariable 片段類型**與**重複控制符**，可寫出靈活的宣告式巨集。
    
- 遇到複雜邏輯需求時，可考慮轉向「過程式巨集（procedural macro）」。
    

掌握上述語法細節後，你就能在專案中運用 `macro_rules!` 有效地減少樣板、提升可讀性與維護性。祝你玩轉 Rust 巨集！