## argc (arguments count)
- 記錄傳入參數的變數
## argv (arguments vector)
- 記錄各個傳入參數的陣列
```cpp
int main(int argc, char argv[])
```
### example
```
./program -g -v
```
- 此時 argc = 3, argv\[0\] = "./program" , argv\[1\]  = "-g" , argv\[2\] = "-v'

在 C++ 中，`...`（省略号）並不是傳統意義上的「關鍵字」，而是一個語法符號，主要用於處理可變參數（variadic）的情境。下面將從幾個主要方面詳細介紹它的用法：

---

## C 風格的可變參數函式 ...

在 C 和 C++ 中，都可以定義接受不定數量參數的函式。這種函式的參數列表在最後使用 `...` 表示，常見於標準函式庫中的 `printf`。

### 使用方法

- **聲明與定義：** 在函式參數列表的最後加上 `...`，例如：
    
    ```cpp
    void myPrint(const char* format, ...);
    ```
    
- **存取參數：** 在函式內部，通常使用 `<cstdarg>` 提供的 `va_list`、`va_start`、`va_arg` 和 `va_end` 來訪問這些參數：
    
    ```cpp
    #include <cstdarg>
    #include <cstdio>
    
    void myPrint(const char* format, ...) {
        va_list args;
        va_start(args, format);
        while (*format != '\0') {
            if (*format == 'd') {  // 當前假設遇到整數
                int i = va_arg(args, int);
                std::printf("%d\n", i);
            }
            ++format;
        }
        va_end(args);
    }
    ```
    

---

## 2. 可變參數模板（Variadic Templates）

C++11 引入了可變參數模板，使得函式或類模板可以接受任意數量的模板參數。這裡的 `...` 用於「展開」模板參數包。

### 範例

- **基本範例：**
    
    ```cpp
    template<typename... Args>
    void print(Args... args) {
        // 這裡可以展開參數包，傳統方法是使用遞歸
        // 或利用 C++17 的折疊表達式（fold expression）
    }
    ```
    
- **使用折疊表達式（C++17 起）：**
    
    ```cpp
    #include <iostream>
    
    template<typename... Args>
    void print(Args... args) {
        (std::cout << ... << args) << '\n';  // 將所有參數依次輸出
    }
    ```
    
- **利用遞歸展開：**
    
    ```cpp
    #include <iostream>
    
    void print() {
        std::cout << "\n";
    }
    
    template<typename T, typename... Args>
    void print(T first, Args... args) {
        std::cout << first << " ";
        print(args...);
    }
    ```
    

### 工作原理

- **模板參數包：** `Args...` 表示零個或多個類型。
    
- **函式參數包：** `args...` 表示對應的參數列表。
    
- **展開（Expansion）：** 利用 `...` 可以將參數包展開成獨立的參數，常見形式有 `(func(args), ...)` 這樣的語法。
    

---

## 3. 可變參數巨集（Variadic Macros）

在 C++ 中（其實從 C99 起就支持），宏也可以使用 `...` 來定義可變參數。這使得定義接受多個參數的宏變得非常靈活。

### 範例

```cpp
#include <cstdio>

#define LOG(format, ...) fprintf(stderr, format, __VA_ARGS__)

int main() {
    LOG("Error: %d occurred at %s\n", 404, "main.cpp");
    return 0;
}
```

在這個例子中，`__VA_ARGS__` 代表了宏調用時提供的額外參數。

---

## 小結

- **語法符號而非保留字：** `...` 是一個語法符號，根據上下文可以表示不同的意義。
    
- **C 風格可變參數函式：** 用於聲明接受不定數量參數的函式，搭配 `<cstdarg>` 使用。
    
- **可變參數模板：** 在 C++11 及以後，用於定義模板函式或類模板，使得可以接受任意數量和類型的參數，並通過展開操作實現操作。
    
- **可變參數宏：** 利用宏定義時使用 `...` 與 `__VA_ARGS__`，使得宏可以接受不定數量的參數。
    

這些用法使得 C++ 在處理不定數量參數時既靈活又強大，無論是在傳統 C 語言風格的程式設計中，還是在現代模板元編程中，都能夠發揮重要作用。