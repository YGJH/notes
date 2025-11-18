
- Subclasses must declare the “missing pieces” to become **“concrete” classes** 子類別必須要實現abstract superclass的抽象方法(abstract method)

## dynamic binding v.s. static binding

### static binding
可以在編譯過程中就知道哪些是 final,private,static
- 因為這些方法不能被override，所以編譯器可以輕鬆知道要調用誰
- *static binding* == *early binding*
```java
public class StaticBindingVSdynamicBinding {

	public static void main(String[] args) {
		Human obj1 = new Human();
		Human obj2 = new Boy();
		obj1.walk();
		obj2.walk();
	}
}
class Human{
	public static void walk() {
		System.out.println("Human walks.");
	}
}
class Boy extends Human{
	public static void walk() {
		System.out.println("Boy walks.");
	}
}
```


output:
Human walks.                                                                                                
Human walks.

### 分析

由於walk()方法是靜態的，編譯器知道他在紫類中不會被覆蓋，因此在編譯時就確定了調用哪個方法。如果把walk()方法前面的static去掉，就是稍後要說得動態
## dynamic binding

因為編譯器不知道要調用哪個方法，所以依賴隱式參數的實際類型，虛擬機運行時，會根據每個類的查找表而決定該使用哪個method

```java
public class StaticBindingVSdynamicBinding {

	public static void main(String[] args) {
		Human obj1 = new Human();
		Human obj2 = new Boy();
		obj1.walk();
		obj2.walk();
	}
}
class Human{
	public void walk() {
		System.out.println("Human walks.");
	}
}
class Boy extends Human{
	public void walk() {
		System.out.println("Boy walks.");
	}
}
```


output:
Human walks.                                                                                                
Boy walks.


### 分析
由於obj1是human對象 (依賴隱式參數的實際類型)，因此調用的是human中的walk方法，而obj2 是 boy 的對象，因此調用的boy的walk

## 總結

- private, static, final (方法和變量) 使用靜態綁定，對於其他方法就是用動態綁定
- 靜態綁定編譯時就已經確定了，而動態綁地要載運息時才去查找的
- 靜態綁定使用類型訊息(type information) 而動態綁定採用對性信息去解析綁定
- 重載overloading運用在靜態綁定中，而覆蓋override則運用在動態綁定中
- 