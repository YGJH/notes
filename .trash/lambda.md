**Explaining Java Lambdas**

The user wants to know how to write a lambda in Java, so I'll explain it in detail. Java lambdas are used to provide clear and concise syntax for writing code that can be executed on demand, often to implement a functional interface or to be used with streams. Lambdas follow the syntax `(parameters) -> expression`. I’ll also include examples to show how lambdas are incorporated with functional interfaces and streams. Since the question is in English, I’ll stick with that.

已思考 4 秒鐘

Below is a structured guide to writing and using lambda expressions in Java (introduced in Java 8).

---

### 1. What Is a Lambda Expression?

A _lambda expression_ is essentially an anonymous function — a block of code you can pass around as if it were an object. In Java, it lets you implement a single‑method (functional) interface more concisely than with an anonymous inner class.

---

### 2. Basic Syntax

```java
(parameters) -> expression
(parameters) -> { statements; }
```

- **Parameters**: comma‑separated list; types can be omitted if unambiguous
    
- **Arrow token**: `->` separates parameters from body
    
- **Body**: either a single expression (no braces needed) or a block in `{}`
    

**Examples:**

```java
// No parameters, returns a string
() -> "Hello, world!"

// One parameter, single expression (type inferred)
x -> x * x

// Two parameters, block body
(a, b) -> {
    int sum = a + b;
    return sum;
}
```

---

### 3. Targeting a Functional Interface

Lambdas can only be assigned to _functional interfaces_ (interfaces with exactly one abstract method). Java provides many in `java.util.function`:

|Interface|Abstract Method|Example Use|
|---|---|---|
|`Runnable`|`void run()`|Thread start|
|`Comparator<T>`|`int compare(T,T)`|Sorting|
|`Consumer<T>`|`void accept(T)`|Processing each stream element|
|`Function<T,R>`|`R apply(T)`|Mapping one type to another|
|`Predicate<T>`|`boolean test(T)`|Filtering|

**Assignment example:**

```java
Comparator<String> cmp = (s1, s2) -> s1.length() - s2.length();
```

---

### 4. Common Use Cases

1. **Sorting a List**
    
    ```java
    List<String> names = Arrays.asList("Bob","Alice","Carol");
    Collections.sort(names, (a, b) -> a.compareToIgnoreCase(b));
    ```
    
2. **Stream operations**
    
    ```java
    List<Integer> nums = List.of(1, 2, 3, 4);
    List<Integer> squares = nums.stream()
                                .map(x -> x * x)
                                .collect(Collectors.toList());
    ```
    
3. **Event listeners in GUI**
    
    ```java
    button.addActionListener(e -> System.out.println("Clicked!"));
    ```
    

---

### 5. Type Inference & Target Typing

The compiler infers parameter types from the context:

```java
// Both are valid if the target expects (String)->int
Function<String,Integer> f1 = s -> s.length();
Function<String,Integer> f2 = (String s) -> s.length();
```

If ambiguity remains, you may need explicit types.

---

### 6. Capturing Variables

Lambdas may capture _effectively final_ variables from the enclosing scope:

```java
int factor = 2;
Function<Integer,Integer> multiplier = x -> x * factor;
// ‘factor’ cannot be reassigned after this point
```

---

### 7. Differences from Anonymous Classes

- **Less boilerplate**: no class name, `new` keyword, or return type declaration
    
- **`this`**: refers to enclosing class, not the lambda itself
    
- **Serialization**: lambdas are not by default `Serializable`
    

---

### 8. Best Practices & Pitfalls

- **Use method references** when possible for clarity:
    
    ```java
    // Instead of x -> System.out.println(x)
    Consumer<String> printer = System.out::println;
    ```
    
- **Keep lambdas short**. If logic grows beyond a few lines, consider a named class or method.
    
- **Watch for side effects**. Pure functions are easier to reason about when used in streams.
    

---

### 9. Putting It All Together

```java
import java.util.*;
import java.util.stream.*;

public class LambdaDemo {
    public static void main(String[] args) {
        List<String> words = List.of("apple", "banana", "cherry");
        
        // Filter words longer than 5 chars and print them
        words.stream()
             .filter(w -> w.length() > 5)
             .forEach(System.out::println);
        
        // Sort by last character
        words.sort((a, b) -> a.charAt(a.length()-1) - b.charAt(b.length()-1));
        System.out.println(words);
    }
}
```

With lambdas you turn patterns like “one-method anonymous class” into concise, readable snippets that integrate seamlessly with the Java 8+ APIs.