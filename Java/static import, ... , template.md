
---

## 1. import static

### 功能與目的

- **什麼是 import static？**  
    Java 中的 `import static` 語法可以讓你直接引用某個類別的靜態（static）成員（包括靜態方法、靜態變數、靜態常數），而不用每次都用類別名稱作前綴。例如：
    
    ```java
    import static java.lang.Math.*;
    ```
    
    有了這行宣告之後，你可以直接使用 `sqrt()`、`pow()` 等方法，而無需寫成 `Math.sqrt()` 或 `Math.pow()`。
    

### 優點與注意事項

- **簡化程式碼：**  
    當你需要頻繁使用某些靜態方法或常數時，能夠使程式碼更簡潔易讀。
    
- **命名衝突風險：**  
    如果不同類別中有相同名字的靜態成員，直接引入可能會造成混淆，此時需要小心選擇或只針對部分成員使用靜態匯入（例如：`import static java.lang.Math.PI;`）。
    

### 範例

```java
import static java.lang.Math.PI;
import static java.lang.Math.sin;

public class StaticImportDemo {
    public static void main(String[] args) {
        double angle = PI / 2;
        System.out.println("sin(90°) = " + sin(angle));
    }
}
```

---

## 2. ... (省略號) 的用法：可變參數（Varargs）

### 功能與目的

- **可變參數：**  
    在方法參數中使用 `...` 表示該參數可以接受零個或多個同類型的參數。這是一種語法糖，讓呼叫方法時不用手動組成陣列，而是可以直接傳入一系列值。
    
- **語法說明：**  
    寫法形如：
    
    ```java
    public void methodName(Type... paramName) { ... }
    ```
    
    在方法內，`paramName` 會被視為 `Type[]` 的陣列。
    

### 優點與注意事項

- **優點：**  
    呼叫端可以直接傳入多個參數，語法更簡單；若沒有參數傳入，也不會發生錯誤。
    
- **注意事項：**  
    每個方法最多只能有一個可變參數，而且必須是方法參數列表中的最後一個參數。
    

### 範例

```java
public class VarargsDemo {
    public static void printNumbers(int... numbers) {
        for (int n : numbers) {
            System.out.print(n + " ");
        }
        System.out.println();
    }
    
    public static void main(String[] args) {
        printNumbers(1, 2, 3, 4, 5);
        printNumbers(); // 空參數也可以
    }
}
```

---

## 3. 模板的用法：Java 泛型（Generics）

在 Java 中，所謂的「模板」通常指的是泛型（Generics），它提供了一種在類別、介面和方法中使用參數化類型的機制，從而在編譯時檢查類型安全，並減少向下轉型（cast）的需求。

### 3.1 泛型類別

- **基本語法：**  
    泛型類別的宣告格式為：
    
    ```java
    public class Box<T> {
        private T item;
        
        public void setItem(T item) {
            this.item = item;
        }
        
        public T getItem() {
            return item;
        }
    }
    ```
    
    這裡 `T` 是一個型別參數，使用者可以在實例化時指定具體型別，如 `Box<String>` 或 `Box<Integer>`。
    

### 3.2 泛型方法

- **基本語法：**  
    泛型方法在方法宣告時使用型別參數：
    
    ```java
    public class GenericMethodDemo {
        public static <E> void printArray(E[] array) {
            for (E element : array) {
                System.out.print(element + " ");
            }
            System.out.println();
        }
        
        public static void main(String[] args) {
            Integer[] intArray = {1, 2, 3, 4, 5};
            String[] strArray = {"Hello", "World"};
            printArray(intArray);
            printArray(strArray);
        }
    }
    ```
    
    這裡 `<E>` 就是泛型方法中的型別參數。
    

### 3.3 泛型通配符

- **用途：**  
    泛型通配符（`?`）用於表示未知型別，例如在方法中希望接受任何型別的集合，但又不關心集合內元素的具體型別。
    
- **常見形式：**
    
    - `List<?>` 表示未知型別的 List。
        
    - `List<? extends Number>` 表示元素型別是 Number 或其子類別的 List。
        
    - `List<? super Integer>` 表示元素型別是 Integer 或其父類別的 List。
        

### 3.4 注意事項

- **型別擦除（Type Erasure）：**  
    Java 泛型在編譯後會進行型別擦除，這意味著運行時所有泛型資訊都會消失，僅保留原始型別。因此在運行時無法直接取得泛型型別資訊。
    
- **限制：**  
    例如不能直接建立泛型陣列、無法使用 instanceof 來檢查泛型參數的型別等。
    

### 範例：泛型類別與泛型方法

```java
// 泛型類別
public class Box<T> {
    private T item;
    
    public void setItem(T item) {
        this.item = item;
    }
    
    public T getItem() {
        return item;
    }
}

// 泛型方法
public class GenericUtil {
    public static <T> void display(T item) {
        System.out.println("內容: " + item);
    }
    
    public static void main(String[] args) {
        Box<String> stringBox = new Box<>();
        stringBox.setItem("Hello Generics");
        System.out.println("Box中的值: " + stringBox.getItem());
        
        // 呼叫泛型方法
        display(100);
        display("泛型方法示例");
    }
}
```

---

## 小結

1. **import static：**  
    用於匯入某個類別的靜態成員，從而可以直接使用而無需類名前綴。
    
2. **...（省略號）：**  
    用於方法的可變參數（varargs），讓你能夠傳入零個或多個同型別參數，方法內會將其視作陣列處理。
    
3. **模板（泛型）的用法：**  
    Java 泛型允許類別和方法參數化型別，從而在編譯時進行類型檢查，提高程式的安全性與重用性；並且還有通配符（?）可以用來處理不確定型別的情況。
    

希望這些解釋和範例能夠幫助你更好地理解並應用這些語法！