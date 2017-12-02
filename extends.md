# 基本觀念

Encapsulation, Message Passing 以及 Inheritance 是構成 Object-Oriented 的三大要素，如果某程式語言只具備前面兩項特性，一般成為 Object-Based。所謂 Inheritance(繼承)，是指 Sub Class(子類別) 繼承 Super Class(父類別) 後，就會自動取得父類別特性。如果子類別繼承了一個以上的父類別，則稱為 Multiple Inheritance(多重繼承)。Java 為了避開多重繼承的複雜性，class 只允許單一繼承。

Java 使用關鍵字 `extends` 來表達繼承觀念：

```java
public class Animal {
    public String moveMethod() {
        return "Unspecified";
    }
}
public class Bird extends Animal {
    public String moveMethod() {
        return "Fly";
    }
}
public class Dog extends Animal {
    public String moveMethod() {
        return "run";
    }
}
public class Fish extends Animal {
    public String moveMethod() {
        return "swim";
    }
}
```

若 class 宣告時沒有指定 `extends`, 則 Java 會自動 `extends java.lang.Object`。

```java
public class A {
}
```

和下面的寫法相同

```java
public class A extends java.lang.Object {
}
```

# UpCasting(向上轉型) 和 DownCasting(向下轉型)

所謂 casting 是指型態轉換，UpCasting 是將子類別型態的 reference 轉型為父類別型態，DownCasting 則是將父類別型態的 reference 轉型成子類別型態。由於子類別可以視為和父類別相容，如 `Fish`, `Dog`, `Bird` 都是一種 `Animal`，因此 UpCasting 一定沒有問題：

```java
Animal a;
Bird b;
a = b; // upcasting, Bird is a kind of Animal
```

父類別的 reference 可以指到子類別的 Object，這種觀念稱為 **Polymorphism(多型)**。

但在 downcasting 的情況下，父類別的 reference 和子類別並不相容，如 `Animal` 不見得是一個 `Bird`，因此必須使用 (SubClass) 的 casting 語法來做強迫轉換。

```java
Animal a = new Bird(); // upcasting
Bird b;
b = (Bird)a; // downcasting, compile correct
if (a instanceof Bird) { // true
}
```

downcasting 除了必須由設計者下達外, JVM 在 runtime 也會檢查實際的物件能否和 reference 的型態相容

```java
Animal a = new Dog(); // upcasting
Bird b;
b = (Bird) a; // downcasting, compile correct, but runtime error
```

比較完整的範例如下

```java
public class InheritanceExample {
    public static void main(String[] argv) {
        Animal a1, a2, a3, a4;
        Bird b;
        Dog d;
        Fish f;
        a2 = a1 = new Animal();
        b = new Bird();
        d = new Dog();
        f = new Fish();
        System.out.println(a1.moveMethod());
        System.out.println(b.moveMethod());
        System.out.println(d.moveMethod());
        System.out.println(f.moveMethod());
        a1 = b; // Correct, we call this upcasting
        b = a1; // Compile Error, type not compatible
        b = (Bird)a1; // downcasting, Compile Correct
        a2 = b; // Correct,we call this upcasting
        d = a2; // Compile Error, type not compatible
        d = (Dog)a2; // Compile Correct, but runtime error
    }
}
```

# Override(覆寫)

子類別重新定義它所能看到的父類別中的 method(如 `public`, `protected`，如果子類別和父類別在同一個 `package` 裡，則沒有修飾字的 method 也可以)，稱為 override。

```java
public class Animal {
    public String moveMethod() {
        return "Unspecified";
    }
}
public class Bird extends Animal {
    // override Animal's moveMethod
    public String moveMethod() {
        return "Fly";
    }
}
```

要特別強調的是

- 如果子類別看不到父類別的方法 (如父類別的 `private` 方法，或子父類別不在同一個 `package` 而子類別定義了父類別內的 package method)，則就算定義了同樣的 method，也不是 override
- 重複定義 static method 也不算 override
- 子類別不可縮小父類別方法的存取範圍

```java
public class C2 {
    public void a() {}
}
public class C1 extends C2 {
    protected void a() { // Compile Error,不得縮小存取範圍
    }
}
```

# Virtual Function(虛擬函數)

在訊息傳遞的章節裡，我們有提到過 Object 接收到訊息後，是在 Runtime 才決定實際所要呼叫的 Method。由於父類別的 reference 可以指到子類別物件 (Polymorphism)，而子類別和父類別可能都定義了相同的 Method(Override)，當使用父類別 reference 傳遞訊息給子類別物件時，應該要呼叫父類別的方法還是子類別的方法？如果

- 呼叫子類別的方法, 則稱為 Virtual Function
- 呼叫父類別的方法, 則稱為 Non-Virtual Function

有些程式語言，如 C++，以上兩種機制都提供，可由設計者自行決定。但是 Java 語言為了遵循物件導向的精神，並避免設計者因語言設計複雜而犯錯，因此只提供了 Virtual Function。

```java
public class InheritanceExample {
    public static void main(String[] argv) {
        Animal a1;
        a1 = new Animal();
        System.out.println(a1.moveMethod()); // print out "Unspecified"
        a1 = new Bird(); // polymorphism
        System.out.println(a1.moveMethod()); // print out "Fly"
    }
}
```

請注意上一小節所提到 Override 的注意事項

```java
class Animal {
    public static String moveMethod() {
        return "Unspecified";
    }
    public static void main(String[] argv) {
        Animal a1;
        a1 = new Bird();
        System.out.println(a1.moveMethod()); // print out "Unspecified"
    }
}
class Bird extends Animal {
    // we can't override static method
    public static String moveMethod() {
        return "Fly";
    }
}
```

上面的 `moveMethod()` 由於宣告為 `static`，因此是依照 reference 的 type 來決定執行的 method。

```java
class Animal {
    private String moveMethod() {
        return "Unspecified";
    }
    public static void main(String[] argv) {
        Animal a1;
        a1 = new Bird();
        System.out.println(a1.moveMethod()); // print out "Unspecified"
    }
}
class Bird extends Animal {
    // this is not override because Bird can't see Animal's moveMethod
    public String moveMethod() {
        return "Fly";
    }
}
```

由於上面 `Animal` 內的 `moveMethod` 宣告為 `private`，因此執行時印出 `"Unspecified"`。

採用 Virtual Function 的優點

- Runtime 自動尋找最特定的方法 (儘量用子類別的方法)，可用父類別 reference 呼叫到子類別的方法，因此增加新的子類別時，不需要修改程式

缺點

- 執行起來比較慢

# 本章觀念整理範例

```java
public class Shape2D { // define super class
    public double area() { // all Shape2D have their own area
        return 0;
    }
}
public class Rectangle extends Shape2D {
    private double length, width;
    public Rectangle(double l, double w) { // define constructor
        length = l;
        width = w;
    }
    public double area() { // Override
        return length * width;
    }
}
public class Circle extends Shape2D {
    private double radius;
    public Circle(double r) {
        radius = r;
    }
    public double area() { // Override
        return 3.141592654 * radius * radius;
    }
}
public class Parallelogram extends Shape2D {
    private double top, bottom, height;
    public Parallelogram(double t, double b, double h) {
        top = t;
        bottom = b;
        height = h;
    }
    public double area() { // Override
        return (top + bottom) * height / 2.0;
    }
}
publicclass Main {
    public static double sum(Shape2D[] shapes) {
        double total = 0;
        for (int i = 0; i < shapes.length; i++) {
            total += shapes[i].area(); // use Virtual Function to calculate area of Shape2D
                                       // Without Virtual Function, value of Shape2D.area() will be 0
        }
        return total;
    }
    public static void main(String[] argv) {
        Shape2D[] data; // array of reference to Shape2D
        data = new Shape2D[5]; // create array object
        data[0] = new Rectangle(2.4, 3.8); // Polymorphism
        data[1] = new Circle(3.9);
        data[2] = new Parallelogram(3.5, 6.7, 10.2);
        data[3] = new Rectangle(5.3, 7.2);
        data[4] = new Circle(4.6);
        System.out.println("Sum of all Shape2D is "+sum(data));
     }
}
```

如果程式語言不支援 virtual function 的話，則上面的範例就得寫成下面的形式才行

```java
public class Main { // example for non-virtual function implementation
    public double sum(Shape2D[] shapes) {
        double total = 0;
        for (int i = 0;  i < shapes.length; i++) {
            if (shapes[i] instanceof Rectangle) {
                total += ((Rectangle)shapes[i]).area();
            } else if (shapes[i] instanceof Circle) {
                total += ((Circle)shapes[i]).area();
            } else if (shapes[i] instanceof Parallelogram) {
                total += ((Parallelogram)shapes[i]).area();
            } // modify source code here for new sub classes
        }
        return total;
    }
    public static void main(String[] argv) {
        Shape2D[] data; // array of reference to Shape2D
        data = new Shape2D[5]; // create array object
        data[0] = new Rectangle(2.4, 3.8); // Polymorphism
        data[1] = new Circle(3.9);
        data[2] = new Parallelogram(3.5, 6.7, 10.2);
        data[3] = new Rectangle(5.3, 7.2);
        data[4] = new Circle(4.6);
        System.out.println("Sum of all Shape2D is "+sum(data));
     }
}
```

# final 修飾字

`final` 除可用來修飾變數外，也可放在 class 和 object method 前面：

```java
public final class FinalClass {
    public final void finalMethod() {
    }
}
```

放在 `class` 前面表示 class 不可被繼承，放在 object method 表示不可被 Override。

# 繼承關係下的 Constructor 執行順序

- 先將所有變數設為內定值。對數值型態來說，其值為 0；對 reference 來說，其值為 `null`；對 boolean 來說，其值為 `false`。
- 呼叫父類別的 constructor。如果子類別 Constructor 裡沒有指定父類別的 Constructor，則使用父類別沒有參數的 Constructor。
- 執行變數宣告的初始化動作。
- 執行自己的 constructor。

如果要指定父類別其他的 constructor，則必須在子類別的 constructor 的第一行使用關鍵字 `super` 來處理。

```java
class Animal {
    int aMask = 0x00FF;
    public Animal() {
    }
    public Animal(int mask) {
        aMask = mask;
    }
}
public class Bird extends Animal {
    int bMask = 0xFF00;
    int fullMask;
    public Bird() {
        // Compiler add super() here
        fullMask = bMask | aMask;
    }
    public Bird(int mask) {
        /* 若有super,則必須放在第一行,連變數宣告也不能擺在super前面 */
        super(mask);
        fullMask = bMask | aMask;
    }
    public static void main(String[] argv) {
        Bird b = new Bird();
        System.out.println(b.fullMask);
        b = new Bird(0x0011);
        System.out.println(b.fullMask);
    }
}
```

當執行 `new Bird()` 時，此物件內各個變數的變化如下

| 步驟 | aMask | bMask | fullMask |
|:--- |:--- |:--- |:--- |
| default | 0 | 0 | 0 |
| call Bird() | 0 | 0 | 0 |
| call Animal() | 0 | 0 | 0 |
| Animal initialize | 0x00FF | 0 | 0 |
| execute Animal() | 0x00FF | 0 | 0 |
| Bird initialize | 0x00FF | 0xFF00 | 0 |
| execute Bird() | 0x00FF | 0xFF00 | 0xFFFF |


當執行 `new Bird(0x0011)` 時，此物件內各個變數的變化如下

| 步驟 | aMask | bMask | fullMask |
|:--- |:--- |:--- |:--- |
| default | 0 | 0 | 0 |
| call Bird(0x0011) | 0 | 0 | 0 |
| call Animal(0x0011) | 0 | 0 | 0 |
| Animal initialize | 0x00FF | 0 | 0 |
| execute Animal(0x0011) | 0x0011 | 0 | 0 |
| Bird initialize | 0x0011 | 0xFF00 | 0 |
| execute Bird(0x0011) | 0x0011 | 0xFF00 | 0xFF11 |
