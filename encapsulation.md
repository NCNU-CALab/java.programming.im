# 封裝的意義

物件導向程式設計的原則之一，是讓實作和界面分開，以便讓同一界面但不同的實作的物件能以一致的面貌讓外界存取，為了達到此目標，java 允許設計人員規範類別成員以及類別本身的存取限制。

# 類別成員的存取

所謂封裝 (Encapsulation), 是指 class A 的設計者可以指定其他的 class 能否存取 A 的某個 member。Java 定義了四種存取範圍：

- private：只有 A 自己才可以存取，使用 keyword private
- package：只有和 A 同一個 package 的 class 才可以存取，沒有相對應的 keyword
- protected：只有同一個 package 或是 A 的子類別才可以存取，使用 keyword protected
- public：所有的 class 都可以存取，使用 keyword public

```java
public class EncapsulationExample {
    private int privateVariable;
    int packageVariable;
    protected int protectedVariable;
    public int publicVariable;
    private int privateObjectMethod() {}
    int packageObjectMethod() {}
    protected int protectedObjectMethod();
    public int publicObjectMethod();
    static private int privateClassMethod() {}
    static int packageClassMethod() {}
    static protected int protectedClassMethod();
    static public int publicClassMethod();
}
```

如果 member 的前面沒有 `private`, `protected`, `public` 其中一個修飾字，則該 member 的存取範圍就是 `package`。從以上的敘述，讀者可以推知這四種存取範圍的大小是 `public` > `protected` > `package` > `private`。

# Package 的定義

所謂 `package`, 可以想成是在設計或實作上相關的一群 `class`。要宣告某 `class` 屬於某 `package`, 其語法為

```java
package myPackage;
public class MyClass {
}

```
如果沒有宣告 `package` 的話，如下面這兩個 `class`，就會被歸類為「匿名」的 `package`:

```java
// in file ClassA.java
public class ClassA {
    public static void main(String[] argv) {
        ClassB x = new ClassB();
    }
}
// in file ClassB.java
public class ClassB {
}
```
為了讓 JVM 在執行期間能夠找到所需要的 `class`，同一個 `package` 的 `class` 會放在同一個目錄下。不過只知道目錄的名稱還不夠，還需要指定該目錄在檔案系統內的路徑。``classpath`` 這個環境變數是由多個以分號隔開的路徑所組成，JVM 透過 `classpath` 配合 `package` 的名稱就可以找到所需要的 `class`。如果我們只用到 Java 標準的程式庫，則不需要指定 `classpath`。指定 `classpath` 環境變數時，要特別注意的是，不要忘了把 `.` (代表目前的工作目錄) 放到最前面，否則就找不到「匿名」`package` 裡的 `class` （別忘了絕大部分簡單的範例都沒有宣告 `package`，所以都是匿名的）。

```
classpath=.;n:\計網中心系統組\project;
```

`classpath` 裡除了路徑外，也可以指定 zip 或 jar(java archive format) 格式的檔案。zip 和 jar 可以把目錄及其子目錄內的檔案都壓縮起來，因此可以透過這類檔案抓到所需的 `class` 檔。

```
classpath=.;c:\mylib.zip;c:\otherlib.jar;n:\計網中心系統組\project;
```

`package` 的宣告可以用 `.` 號構成複雜的 package tree。

```java
package mylib.package1
public class A {}
package mylib.package2
public class B {}
```

屬於 `mylib.package1` 的 `class` 會放在 `mylib` 目錄下的 `package1` 目錄內。Java 所提供的標準應用程式介面 (Application Programming Interface, API) 就是一個複雜的 package tree。

同一個 `.java` 檔裡面, 可以定義好幾個 `class`，但最多只能有一個宣告為 `public class`。此限制是因為 java 希望每一個編譯的單元 (`.java` 檔) 都有唯一的界面宣告。那麼 `public class` 和 `class` 的區別何在？`non public class` 只能給同一個 `package` 內的其他 `class` 引用，`public class` 則可以給任何 `class` 引用。

# Package 的引用

假如 `Class A` 用到 `package myPackage` 裡的 `Class B`，為了檢查 `A` 使用到 `B` 的部分是否符合 `B` 的原始定義，諸如方法存不存在，參數正不正確等問題，Compiler 必須引入 `class B` 的定義，以便進行編譯時期的檢查。引入的語法為

```java
import myPackage.B;
```

這裡要強調的是，`import` 指令告知 Compiler 在 Compiler time 所要檢查的類別定義在哪裡。但有時候我們編譯的環境和執行的環境可能不同，例如編譯時用 JDK 1.4，執行時卻用 JDK 1.2，若程式使用到 JDK 1.4 才有的 API，那麼會在執行期間產生錯誤。

有時候我們會引用相當多個同屬某 `package` 的類別，如果要一個一個 `import`，會很煩人，因此 Java 允許我們使用萬用字元 `*` 來代表某 `package` 裡的所有 `class`：

```java
import myPackage.*;
public class A {
    public static void main(String[] argv) {
        B x = new B();
    }
}
```

眼尖的讀者會發現我們並沒有 `import String` 的定義啊，怎麼都沒有問題？由於寫程式多多少少都會用到一點系統提供的程式庫，如果連很簡單的程式都要 `import` 一堆 `class`，也真煩人。因此 Java Compiler 會自動幫我們引入 `java.lang.*`

```java
public class Hello {
    public static void main(String[] argv) {
        System.out.println("Hello World.");
    }
}
```

就等同

```java
import java.lang.*;
public class Hello {
    public static void main(String[] argv) {
        System.out.println("Hello World.");
    }
}
```

由於 `class` 是放在類似樹狀結構的 package tree 裡面，因此引用的 `class` 應該加上完整的 `package` 路徑才是全名，例如

```java
public class Hello {
    public static void main(java.lang.String[] argv) {
        java.lang.System.out.println("Hello World.");
    }
}
```

只要不會造成混淆, 一般我們都使用省略 `package` 路徑的 `class` 簡稱。但是如果我們 `import P1` 和 `P2` 兩個 `package`, 而這兩個 `package` 碰巧都定義了同名的 `class A`, 則用到 `A` 的地方就比需以 `P1.A` 和 `P2.A `來區別了。

# 下面的程式碼哪裡有錯?

```java
package p1;
public class Access {
    private static void f1() {}
    static void f2() {}
    protected static void f3() {}
    public static void f4() {
        Access.f1();
    }
}
package p1;
public class Example1 {
    public static void main(String[] argv) {
        Access.f1();
        Access.f2();
        Access.f3();
        Access.f4();
    }
}
package p2;
import p1.*;
public class Example2 {
    public static void main(String[] argv) {
        Access.f1();
        Access.f2();
        Access.f3();
        Access.f4();
    }
}
package p1;
public class Example3 extends Access {
    public static void main(String[] argv) {
        Access.f1();
        Access.f2();
        Access.f3();
        Access.f4();
    }
}
package p2;
import p1.*;
public class Example4 extends Access {
    public static void main(String[] argv) {
        Access.f1();
        Access.f2();
        Access.f3();
        Access.f4();
    }
}
```

# Java 檔和 Class 檔的相依性

傳統程式開發的流程是 Compile 個別 Source Code，然後 Link 所有的 Object Code 成為執行檔。對大型的應用程式來說，常見的問題之一是如何確定 Link 時所需的 Object Code 是由最新的 Source 編譯而來? 尤其模組間存在相依性，如模組 `A` 可能用到模組 `B` 裡的函數，如果 `B` 的函數有修改參數，則 `A` 模組也要重新編譯。換句話說單看 source code 和 object code 的產生時間是不行的。最簡單的方法就是在 Link 前將所有的 Source Code 重新編譯一次，但這樣做有以下幾個問題:

1. 重新編譯大型專案全部的程式碼可能會浪費不少時間
2. 要對每個 Source File 下達編譯指令，不但費時，而且容易遺漏。即使寫個批次程式，也要隨時記得納入新的原始檔

由於這些問題的存在，某些原始碼管理系統便因應而生，例如 UNIX 上的 SCCS(Source Code Control System)。Java Compiler 具有下面兩個功能，可以在沒有原始碼管理系統的情況下，也能解決上述問題：

1. 可使用 `javac *.java` 來編譯目前目錄下的所有 java 檔案
2. 編譯 `A.java` 時，會自動檢查 `A` 所用到的其他 `class B`，比較 `B.java` 和 `B.class` 的產生時間，如果 `B.java` 比較新則 `B.java` 就會被重新編譯

如果應用軟體只有單一的進入點，例如 `class A` 的 `public static void main(String[] argv)`，則只要編譯 `A.java` 就會自動編譯其他需要重新編譯的 `.java` 檔。如果應用軟體有兩個以上的進入點，如網路程式的 client 端和 server 端的進入點會不一樣，只要寫個批次檔編譯相關進入點的 `.java` 檔即可。

## 用 Link List 實作 Stack

![](https://raw.githubusercontent.com/NCNU-CALab/java.programming.im/master/images/stack1.jpg)

```java
class Node {
    Object data;
    Node next;
}
public class Stack {
    private Node head;
    private int size;
    public void push(Object s) {
        Node tmp = new Node();
        tmp.next = head;
        tmp.data = s;
        size++;
        head = tmp;
    }
    public Object pop() {
        Object tmp = head.data;
        head = head.next;
        size--;
        return tmp;
    }
}
```

---

### Link List Stack Push Step 1
![](https://raw.githubusercontent.com/NCNU-CALab/java.programming.im/master/images/stack2.jpg)

### Link List Stack Push Step 2
![](https://raw.githubusercontent.com/NCNU-CALab/java.programming.im/master/images/stack3.jpg)

### Link List Stack Push Step 3
![](https://raw.githubusercontent.com/NCNU-CALab/java.programming.im/master/images/stack4.jpg)

### Link List Stack Pop
![](https://raw.githubusercontent.com/NCNU-CALab/java.programming.im/master/images/stack5.jpg)

---

```java
public class Example {
    public static void main(String[] argv) {
        Stack s1 = new Stack();
        Stack s2 = new Stack();
        s1.push("abc");
        s1.push("def");
        s2.push("123");
        s2.push("456");
    }
}
```

## 用 Link List 實作 Queue

```java
class Node {
    Object data;
    Node next;
}
public class Queue {
    private Node head, tail;
    private int size;
    public void put(Object s) {
        Node tmp = new Node();
        tmp.data = s;
        if (tail != null) {
            tail.next = tmp;
        } else {
            head = tmp;
        }
        tail = tmp;
        size++;
    }
    public Object get() {
        Object tmp = head.data;
        head = head.next;
        if (head == null) {
            tail = null;
        }
        size--;
        return tmp;
    }
}
```
