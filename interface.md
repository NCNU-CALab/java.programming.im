## 為什麼 Java 提供 Interface

雖然程式語言提供了基本資料型別，但由於各種應用都有其特定的資料結構需求，因此程式語言都提供使用者自訂型別的能力。型別自訂後，其使用的方法和基本資料型態類似。

Class 就是一種使用者自定的型別。Java 提供了 `instanceof` 的保留字，用以判斷某 reference 所指到的物件，其型態和某 Class 是否相容：

```java
Object ref;
ref = new Bird();
if (ref instanceof Animal) { // correct
    System.out.println("ref is currently pointing to an Animal Object.");
}
```

在物件導向的觀念裡，物件可以具有多個型別，例如：「附有橡皮擦的鉛筆」，可以當成是「書寫工具」，也可以當成是「擦拭工具」。

物件可有多種型別的觀念，不僅在日常生活中常見，在軟體開發上也有實際的需求，要使物件具有多種型別，可透過繼承來達成。

例如 Bird 物件就同時具有 Bird 和 Animal 兩種型別。

由於 Java 的 Class 只能有單一繼承，因此像「附有橡皮擦的鉛筆」同時具有「書寫工具」和「擦拭工具」兩種互不相關的型別，就無法透過 Class 的單一繼承來達成了。

許多語言提供 Class 的多重繼承，但 Java 考量諸如下面的多重繼承問題，選擇不引進 Class 多重繼承：

假設 B 繼承 A ，C 繼承 A，D 又多重繼承 B、C，該語言又使用 virtual function 則

>如果 B、C 都有 overwrite A 的 methodM 方法，而 A ref 指到 D 類別的物件，請問透過 ref 傳遞 methodM 訊息時，應該使用 B 還是 C 的 methodM？

在不引進 Class 多重繼承的前提下，為了讓物件具有多種型態，Java 提供了 Interface (界面) 的觀念。

Interface 可視為沒有實作的自訂型別，和 Class 用來作為 Object 的模板有所不同。

Class 可以宣告實作多個 Interface，而 Interface 之間可以有多重繼承。

## Java 有關 Interface 的語法

* 宣告 Interface  
  ```java
  public interface Listener {
      double PI = 3.14149; // 同public static final
      void listen(); // 同public abstract
  }
  public interface Runnalbe {
      int PERIOD = 10;
      void run();
  }
  public interface AnotherRun {
      int PERIOD = 20;
      void run();
      int run(int);
  }
  ```  
  注意上述函數宣告沒有 `{}`，也就是說沒有實作的意思。
* Interface 的繼承  
  ```java
  public interface ActionListener extends Listener {
      //
  }
  public interface MultiInterface extends Listener, Runnalbe {
      //
  }
  ```
* Class 實作 Interface 的宣告  
  ```java
  public class A implements Listener {
      public void listen() {
          //
      }
  }
  public class B implements Listener, Runnable {
      public void listen() {
          //
      }
      public void run() {
          //
      }
  }
  public class C implements MultiInterface {
      public void listen() {
          //
      }
      public void run() {
          //
      }
  }
  public class D extends A implements Runnable, AnotherRun {
      public void run() {
          //
      }
      public int run(int period) {
          //
      }
  }
  ```
  
Interface 如同 Class 一樣可以作為一種型態的宣告，因此如下的判斷都是正確的
```java
D ref = new D();
ref instanceof D; // true
ref instanceof Runnable; // true
ref instanceof AnotherRun; // true
ref instanceof A; // true
ref instanceof Listener; // true
```

Interface 中宣告的變數具有以下特質
* public：所謂 Interface (界面) 指的是外界觀看某物件時，所能看到的表象以及溝通的管道，因此 Interface 內的成員一定是 public。也就是說即便宣告時沒寫 public 關鍵字，Compiler 也會幫我們加上去。
* static：既然 Interface 沒有實作，就不可能透過 Interface 產生物件。換言之，Interface 內的變數一定是屬於 Class，而不屬於 Object。
* final：Interface 可視為一種約定或契約，我們自然不希望裡面的 variable 可以隨便更改。

Interface 中宣告的 method 具有以下特質
* public：同變數說明。
* abstract：Interface 沒有實作，裡面定義的 method 只是宣告而已。沒有實作的 method，在 Java 裡用 abstract 這個關鍵字來表達。有關 abstract 的詳細說明，請見 [下一節](#abstract-class-and-method)

當 Interface 繼承多個 Interface，或 Class 實作多個 Interface 時，如果有多個同名的函數或變數時，應該如何處理？例如 Runnable 和 AnotherRun 這兩個界面都定義了變數 PERIOD 和方法 run。
* 相同變數名稱：由於 interface 內的變數具有 static 的性質，因此使用這些變數時，必須加上 Interface 的名稱才行，如 `Runnable.PERIOD`、`AnotherRun.PERIOD`，因此不會造成任何混淆。
* 相同函數名稱：如果 signature (參數個數、型態以及傳回值型態) 完全相同，則 Class 只要實作一次即可，例如 Runnable 和 AnotherRun 均定義 `void run()`，因此 Class D 只要實作一次就好了。如果同名函數符合 Overloading，把它們分別當成不同的 method 即可。如果參數完全相同，但傳回值不同，則違反了 Overloading 的原則，會產生 Compile Error。

## Abstract Class and Method

只有參數宣告，沒有實作的方法，稱為 `abstract method`。

某些情況下，雖然有實作，但我們希望強迫子類別必須 override 該方法時，也可以宣告為 `abstract method`。

Interface 裡的方法一定沒有實作，因此必然為 `abstract method`。

如果 Class 裡有一個以上的 `abstract method`，則該 class 必須宣告為 abstract。

有時候即使沒有 `abstract method`，也可以宣告該 class 為 abstract。我們不可以直接 new 該 class 的物件，只能 new 其子類別物件。

```java
public abstract class AbstractExample {
    int x;
    public void abstract abstractMethod() {
    }
    public AbstractExample() {
        x = 5;
    }
}
public class SubClass extends AbstractExample {
    public void abstractMethod() { // must override this method, or SubClass be declared as abstract class
        x = 10;
    }
}
public class Main {
    public static void main(String[] argv) {
        AbstractExample a = new SubClass(); // correct
        a.abstractMethod(); // virtual function, call SubClass's abstractMethod
        a = new AbstractExample(); // Compile error, you can't new abstract class
    }
}
```

綜合以上所述，可列出以下幾點特徵
* 具有 `abstract method` 的 class 必須宣告為 `abstract class`。
* 繼承 `abstract class` 的子類別必須 override 所有父類別的 `abstract method`，否則子類別也必須宣告為 `abstract class`。
* 實作 Interface A 的 Class 必須實作 A 裡的所有 method，否則必須宣告自己為 `abstract class`。
* 不能直接 new abstract class，只能 new 其非 abstract class 的子類別。
