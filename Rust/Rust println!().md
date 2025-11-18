在 Rust 中，`println!` 宏用于格式化输出，支持多种格式化选项。以下是 `println!("{:.?}")`、`println!("{:?}")` 和 `println!()` 的详细说明和示例：

---

### 1. **`println!("{:?}")`**
- **用途**：用于调试输出（Debug 输出），通常用于打印结构体、元组、数组等复杂类型。
- **特点**：
  - 会调用类型实现的 `Debug` trait。
  - 输出格式通常为 `"{:?}"`，适用于调试目的。
  比如：
  ```rust
  struct Point {
      x: i32,
      y: i32,
  }

  impl std::fmt::Debug for Point {
      fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
          write!(f, "Point({},{})", self.x, self.y)
      }
  }

  fn main() {
      let p = Point { x: 1, y: 2 };
      println!("{:?}", p); // 输出: Point(1,2)
  }
  ```

---

### 2. **`println!("{:.?}")`**
- **用途**：与 `{:?}` 类似，但通常用于更详细的调试输出（例如，当需要显示结构体的字段名时）。
- **特点**：
  - 与 `{:?}` 的行为几乎相同，但在某些情况下（如自定义类型）可能显示更详细的字段信息。
  - 实际上，`{:.?}` 和 `{:?}` 在大多数情况下是等价的，但某些库或自定义实现可能对两者有不同的处理。
  比如：
  ```rust
  struct Point {
      x: i32,
      y: i32,
  }

  impl std::fmt::Debug for Point {
      fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
          write!(f, "Point(x={}, y={})", self.x, self.y)
      }
  }

  fn main() {
      let p = Point { x: 1, y: 2 };
      println!("{:.?}", p); // 输出: Point(x=1, y=2)
  }
  ```

---

### 3. **`println!()`**
- **用途**：直接输出字符串，不带格式化参数。
- **特点**：
  - 用于打印固定文本。
  - 无需格式化参数，直接输出内容。
  比如：
  ```rust
  fn main() {
      println!("Hello, world!"); // 输出: Hello, world!
  }
  ```

---

### 4. **其他常见格式化选项**
以下是一些常用的格式化选项及其用途：

| 格式化字符串 | 用途 |
|--------------|------|
| `{:}`        | 默认格式化（如字符串、数字等） |
| `{:?}`       | 调试格式化（Debug 输出） |
| `{:b}`       | 二进制格式（如 `1010`） |
| `{:o}`       | 八进制格式（如 `12`） |
| `{:x}`       | 十六进制格式（如 `a1`） |
| `{:p}`       | 指针地址（如 `0x7ffee4b0`） |
| `{:>5}`      | 右对齐，总宽度为 5（如 `  123`） |
| `{:0>5}`     | 右对齐，总宽度为 5，填充 0（如 `00123`） |
| `{:.<5}`     | 左对齐，总宽度为 5，填充 `.`（如 `123..`） |

---

### 5. **自定义类型的格式化**
若需要自定义类型输出格式，需实现 `std::fmt::Display` 或 `std::fmt::Debug` trait：

#### 示例：自定义 `Display` 格式
```rust
struct Point {
    x: i32,
    y: i32,
}

impl std::fmt::Display for Point {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}

fn main() {
    let p = Point { x: 1, y: 2 };
    println!("{}", p); // 输出: (1, 2)
}
```

#### 示例：自定义 `Debug` 格式
```rust
impl std::fmt::Debug for Point {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "Point(x={}, y={})", self.x, self.y)
    }
}

fn main() {
    let p = Point { x: 1, y: 2 };
    println!("{:?}", p); // 输出: Point(x=1, y=2)
}
```

---

### 6. **注意事项**
- **`{:.?}` vs `{:?}`**：在大多数情况下，两者行为相同，但某些库或自定义实现可能对两者有不同的处理。
- **格式化参数**：确保格式化字符串与参数类型匹配，否则会引发编译错误。
- **性能**：`{:?}` 和 `{:.?}` 用于调试，不适合生产环境的性能敏感场景。

---

### 总结
- **`{:?}`** 和 **`{:.?}`**：用于调试输出，显示结构体或复杂类型的详细信息。
- **`println!()`**：直接输出字符串，无需格式化参数。
- **自定义格式化**：通过实现 `Display` 或 `Debug` trait 控制输出格式。

通过合理使用这些格式化选项，可以更灵活地控制 Rust 程序的输出行为。
