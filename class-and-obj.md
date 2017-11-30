# 類別與物件的基本概念

所謂物件，說得白話一點，可稱之為「東西」。這是個很抽象的名詞，我們若以它具體的特性來描述，會比較清楚：
- Object 有生命週期，會「產生」和「消滅」
- Object 具有其內部狀態， 同一類別的不同 Object ，其的內部狀態可能都不一樣
- Object 可以接收「訊息」，並依據訊息的參數以及該物件的內部狀態，做出反應，並可能因而改變該物件的內部狀態

屬於同一個 Class 的 Object ，會具有該 Class 所定義的以上三種特質。

除此之外， Class 之間可以定義繼承 (Inheritance) 關係，子類別 (Sub Class) 繼承父類別 (Super Class) 的所有特性，子類別還可以定義其專屬的特性。

以 Object-Oriented (物件導向) Language 寫作程式時，寫作的主體是 Class 。 Class 定義了所有屬於該 Class 的 Object 的特性，這些特性可分類如下:
- Object  產生時一定要呼叫的方法，稱為 Constructor (建構子)
- Object 消滅需要呼叫的方法，稱為 Destructor (解構子)
- 表達 Object 內部狀態的變數，稱為 Object Variable(物件變數成員)
- Object 可以接收的訊息，稱為 Object Method(物件方法成員)
- 上述兩個可總稱為 Object Member
- 屬於 Class 的變數，稱為 Class Variable (類別變數)
- 屬於 Class 的方法，成為 Class Method (類別方法)
- 上述兩個可總稱為 Class Member
- 和其他 Class 間的繼承關係

# 如何以Java撰寫類別

Java 規定公共類別 (public class) 必須寫在該公共類別名稱的 `.java` 檔案內，例如 `public class Example` 就必須寫在 `Example.java` 這個檔案內。`Example.java` 裡面也可以定義其他的類別，但是只有 `class Example` 能夠宣告為 `public`，其他 `Example.java` 裡的 `class` 都不能宣告為 `public`。當 Java Virtual Machine 啟動時，它會去找命令列上所指定的 `class` 裡的 `public static void main(String[] argv)` 方法，當做是程式的進入點。這有點像是 C 語言的 `main` ， 不同處在於每個 java class 都可以定義自己的 `public static void main(String[] argv)`。

`$ java Example`

啟動上述的 JVM 時，JVM 會去執行 `class Example` 裡的 `public static void main(String[] argv)` 。以下範例 `Example.java` 說明如何定義 Java 的 class。

```java
class Vehicle {
    private int speed; // Object Variable
    private String direction; // Object Variable, direction is a reference to String Object
    private static int numVehicle = 0; // Class Variable
    public Vehicle() { // Constructor, called when new a Object
        this(0,"north"); // call another constructor to do initialization
    }
    public Vehicle(int s, String dir) { // Another Constructor. Use overloading to define two constructors
        float speed; // define a local variable
        speed = s; // the speed here refers to the above local variable
        this.speed = s; // If we want to set object variable, use this.speed to refer object variable speed
        direction = dir; // dir is a reference to object, not the object itself
        numVehicle++;   // increase the Vehicle number
    }
    protected void finalize() { // Destructor, called when the object is garbage collected by JVM
        System.out.println("finalize has been called");
        numVehicle--;
    }
    void setSpeed(int newSpeed) { // Object Method
        this.speed = newSpeed;
    }
    void setDir(String dir) { // Object Method
        this.direction = dir;
    }
    int getSpeed() { // Object Method
        return speed;
    }
    String getDir() { // Object Method
        return direction;
    }
    public static int totalVehicle() { // Class Method
        return numVehicle;
    }
}
public class Example {
    public static void main(String[] argv) {
        Vehicle v1 = new Vehicle(50, "west"); // new 敘述用來產生物件. 物件產生時需要呼叫Constructor來初始化物件
        Vehicle v2;
        v1.setSpeed(30);
        v1.setDir("north");
        System.out.println("V1: speed is "+v1.getSpeed()+", direction is "+v1.getDir()+".\n");
        v2 = new Vehicle(40, "south");
        System.out.println("There are "+Vehicle.totalVehicle()+" Vehicles in the world.\n");
        v1 = v2; // let reference v1 point to where v2 is pointing
        System.out.println("V1: speed is "+v1.getSpeed()+", direction is "+v1.getDir()+".\n");
        System.gc(); // force system to do garbage collection, the object previously pointed by v1 shall be destroyed
        System.out.println("There are "+Vehicle.totalVehicle()+" Vehicles in the world.\n");
    }
}
```

上述例子裡所用到的關鍵字或類別名稱說明如下：
- `public`：可用在
	- class 前面表示此 class 可以供其他 package 裡的類別使用。同一個目錄下的 class 均可視為屬於同一個 package。
	- object or class member 前面，表示所有的 class 均可存取此 member。
- `private`：可用在 object or class member 前面，表示只有定義這些 member 的 class 才可存取。
- `static`：可用在 member 前面。如果 member 前面有 static ，表示該 member 屬於 class ，否則屬於 object 。不論系統創造了多少 object, class variable 只有一個；而每個 object 都有其 object variable。存取class method 的語法是 `ClassName.classMethod();` 存取 object method 時,則必須以 `reference.objectMethod()` 來呼叫。在 Object Method 裡，可用 `this` 表示目前的物件。但在 Class Method 裡就不能存取 object member 了。
- `this`：表示目前這個物件
- `String`：定義於 java.lang package 下面的類別，屬於 Java 語言定義的標準程式庫。
- `System`：定義於 java.lang package 下面的類別，屬於 Java 語言定義的標準程式庫。
- `System.out` 是 class System 裡面的一個 Class Variable，其型態為 reference to 定義於 java.io package 裡面的 PrintStream。我們可透過該變數所指到的物件，將訊息印到螢幕上。
- `System.gc` 是 class System 裡面的一個 Class Method，呼叫該方法可強迫 JVM 回收沒有被任何 reference 指到的物件。當物件被回收時，該物件的 finalize 方法會被呼叫。
- `new`：用來產生新的物件。後面必須跟著某個 constructor ，以便進行初始化的動作。

**Object Method 的名稱如果和 Class 的名稱相同, 則表示該 Method 為 Constructor。Constructor 不能宣告傳回值。**

要附帶說明的是，Java 以 `new` 指令來產生物件，但不像 C++ 有提供相對應的 `delete` 指令來消滅物件。Java 採用 Garbage Collection 的觀念，當系統於閒置期間自動呼叫或由使用者強制呼叫 `System.gc()` 時，沒有被任何 reference 指到的 Object 就會被回收。

Class 裡面一定要定義一個以上的 Constructor，但為了方便起見，如果 Compiler 發現某 Class 沒有定義 Constructor，則 Compiler 會幫我們產生一個不做任何事的 Constructor：

```java
public class A {
}
```

就相當於

```java
public class A {
    public A() {}
}
```

# Overloading

同一個 class 裡的 Method 名稱可以重複使用，只要可以由 Method 的參數個數和型態來區分就可以了。這種觀念稱為 overloading 。

不只一般的 method 可以 overloading, constructo r也可以 overloading。

```java
public class Overloading {
    int data;
    public Overloading() {
        this(0); // call constructor Overloading(int)
    }
    public Overloading(int data) {
        this.data = data;
    }
    public void print() {
        this.print(0); // call method print(int)
    }
    public void print(int msg) {
    }
    public void print(float msg) {
    }
    public void print(int msg, String others) {
    }
}
```

上面的例子裡說明 constructor 也可以 overloading 。要特別注意的是，傳回值並不能用來分辨要呼叫哪個 method，因此若再加上 `public int print()` 的宣告，就會造成編譯錯誤了。

# 初始化的執行順序

Class variable 是在該類別載入 JVM 時進行初始化的，因此寫作上經常在 class variable 的宣告後面加上初始化的動作。對 Object Variable 來說，是在產生 Object 時進行初始化的，但初始化的步驟可以寫在變數宣告後，也可以寫在 constructor 內，因此必須對其執行順序有所了解。步驟如下：

先將所有變數設為內定值。對數值型態來說，其值為 0；對 reference 來說，其值為 null；對 boolean 來說，其值為 false。
呼叫父類別的 constructor。
執行變數宣告的初始化動作。
執行自己的 constructor。
因此在如下的範例內

```java
public class InitSequence {
    int data = 2;
    public InitSequence(int data) {
        this.data = data;
    }
    public static void main(String[] argv) {
        InitSequence s = new InitSequence(3);
        System.out.println(s.data);
    }
}
```

`data` 的變化如下：
1. 設為內定值 0
2. 呼叫父類別的 Constructor。因為類別 `InitSequence` 沒有宣告繼承任何類別， Java 規定此情況會自動繼承 `java.lang.Object` 這個類別。 Object 的 Constructor 不做任何事。
3. 執行變數宣告的初始動作，成為 2
4. 執行自己的 constructor，成為 3
因此最後執行的結果會在螢幕上印出數字 3。

Java 語言還可以定義 static block：

```java
public class StaticBlock {
    static { // this is a static block
        data = (int)(Math.random()*100);
    }
    static int data;
    public static void main(String[] argv) {
        System.out.println(data);
    }
}
```

static block 內的程式碼，是在該 class 載入 JVM 之後，進行 class variable 初始化之前的時間內執行。一般比較會使用 static block 的場合，是該 class 用到一些非 Java 的程式庫，需要透過 `System.loadLibrary(String libName)` 方法把外界的程式碼載入時。這樣寫的好處是只有當該 class 第一次被使用到時，才會下載相關軟體，以節省記憶體空間，避免重複下載，並可以把實作的細節和外界隔離開來。對沒有這種機制的 C 語言來說，很可能就必須在主程式內寫上一堆很難懂的啟動程式碼。

```java
class ClassNeedToLoadLibrary {
    static {
        System.loadLibrary("mylib");
    }
}
public class Main {
    public static void main(String[] argv) {
    }
}
```

# final 關鍵字

final 關鍵字用在變數宣告時，表示該變數的值只能在宣告時給定，然後就不能再更改了。
```java
public class Main {
    public static final double PI = 3.14159;
    public final int x = 10;
    public static void main(String[] argv) {
        final int local = 10;
        Main m = new Main();
        PI = 100; // Compile Error, final variable can only be set at initialization
        m.x = 10; // Compile Error
        local = 100; // Compile Error
    }
}
```