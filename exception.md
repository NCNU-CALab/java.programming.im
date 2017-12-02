## 簡介

所謂例外 (Exception)，指的是程式發生不正常的錯誤，而導致無法繼續執行的情形。

例外處理 (Exception Handling) 顧名思義，就是當例外發生時的處理機制。

C 語言裡並沒有例外處理的機制，使用函數庫時，可能會發生無法處理的狀況，此時必須由呼叫者小心檢查傳回值才行。如果不檢查，就會發生無法預期的結果

```c
#include <stdio.h>
main() {
    int data;
    FILE* f = fopen("sample.txt", "r"); // sample.txt may not exist, fopen returns NULL
    fscanf(f, "%d", &data); // may result segmentation fault;
}
```

上述例子裡，如果檔案 sample.txt 不存在，則無法開啟該檔案，因此 `fopen()` 傳回 NULL。如果不檢查傳回值，則會產生 segmentation fault。比較好的寫法是：

```c
#include <stdio.h>
main() {
    int data;
    FILE* f = fopen("sample.txt", "r"); // sample.txt may not exist, fopen returns NULL
    if (f == NULL) {
        printf("Can't open file sample.txt. Please check if it exists and you have privilege to access.\n");
        return;
    }
    fscanf(f, "%d", &data); // may result segmentation fault;
}
```

又例如 `sqrt()` 可以用來求平方根，但如果我們給他負數，`sqrt()` 該如何處理？

根據手冊，`sqrt()` 遇到負數參數，會傳回 `NaN` (Not a Number，是浮點數裡的一個特別數值)。使用 `sqrt()` 的函數必須特別檢查該值，否則計算出來的東西都變成了 `NaN`。

又例如除法運算時分母是 0 的情況，嚴格說起來也是一種錯誤，某些系統也會產生 `floating exception (core dumped)`。

沒有提供例外處理機制的語言，程式的正確性必須靠極端小心的設計者才行。為了減少程式錯誤的機會，讓軟體很強固 (robust)，Java 提供了例外處理的機制。

## 相關語法

在 Java 裡，Exception 是一個 Class，`Exception extends Throwable`、`Throwable extends Object`。

`Exception`、`Throwable` 這兩個類別均定義於 `java.lang` 這個 package 內。設計者也可以自訂自己的 Exception 類別。相關的 Exception 語法如下：

* 自訂 Exception 類別  
  ```java
  public class MyException extends Exception {
      //
  }
  ```
* 宣告某 method 會產生例外  
  ```java
  import java.io.*;
  public class ExceptionExample {
      public void someMethod() throws Exception { // 請注意throws最後面是s
          // some code may fail
          FileInputStream f;
          try {
              f = new FileInputStream("abc.txt"); // if abc.txt does not exist, FileNotFoundException will be caught
          } catch(FileNotFoundException fnf) {
              System.out.println("File not found. Generate an exception and throw it");
              throw fnf; // 注意throw後面沒有s
              // throw new Exception(); // or you can throw a new Exception object
          }
      }
      public static void main(String[] argv) {
          ExceptionExample s = new ExceptionExample();
          try {
              s.someMethod();
          } catch(Exception epp) {
              System.out.println("An Exception has been caught.");
          }
      }
  }
  ```
* 攔截 exception 的語法  
  ```java
  try {
      //
  } catch (TypeOneException e1) {
      //
  } catch (TypeTwoException e2) {
      //
  } catch (TypeThreeException e3) {
      //
  } finally {
      //
  }
  ```

`try {} catch()` 類似像 `if then else if` 的結構。

當 `try {}` 裡面某一行指令產生 Exception 時，`try {}` 區塊會立刻中斷執行，然後到第一個 `catch()` 判斷抓到的 Exception 是否 `instanceof TypeOneException`，如果是則執行該 `catch()` 區塊；如果不是，則進一步比較 `instanceof TypeTwoException`。

也就是說雖然可以寫很多個 `catch()` 區塊，但執行時最多只有一塊會執行到。

離開 `try {}` 或 `catch()` 區塊以前，如果有 `finally` 區塊，則 `finally` 區塊一定會被執行到。

一般來說 `finally` 區塊裡面的程式碼大多用來作資源回收，或清理資料結構的工作，以確保不論有無發生狀況，程式都能繼續正常執行。

```java
try {
    System.out.println("Opening FileInputStream");
    FileInputStream f = new FileInputStream("abc.txt"); // assume this operation generate FileNotFoundException
    System.out.println("File Opened");
} catch (FileNotFoundException e1) {
    System.out.println("FileNotFoundException caught");
} catch (Exception e2) {
    System.out.println("Exception caught");
} finally {
    System.out.println("Execute finally block");
}
```

上述的範例會印出：

```
Opening FileInputStream
FileNotFoundException caught
Execute finally block
```

如果把上面例子稍微改一下：

```java
try {
    System.out.println("Opening FileInputStream");
    FileInputStream f = new FileInputStream("abc.txt"); // assume this operation generate FileNotFoundException
    System.out.println("File Opened");
} catch (Exception e2) {
    System.out.println("Exception caught");
} catch (FileNotFoundException e1) {
    System.out.println("FileNotFoundException caught");
} finally {
    System.out.println("Execute finally block");
}
```

則聰明一點的 Compiler 會抱怨 `FileNotFoundException` 的區塊 unreachable (執行不到)。

這是因為 `FileNotFoundException` 是 Exception 的子類別，因此如果產生的例外是 `instanceof Exception`，則 `FileNotFoundException` 就不會執行了；若產生的例外不是 `instanceof Exception`，那就更不會執行到 `FileNotFoundException` 區塊了。

## 是否所有的 Exception 都要處理？

原則上是的。

只要用到的 method 有宣告 throws SomeException，則呼叫該 method 的地方，就要使用 `try {} catch(SomeException)` 的語法。

當然像是 `try {} catch(SuperClassOfSomeException)` 的用法也行。唯一的例外是 `java.lang.RuntimeException` 及其子類別可以不必處理。

哪些屬於 `RuntimeException` 呢？像是 `ArrayIndexOutOfBoundException` 就是其中之一，它發生在陣列索引不合法的情況下：

```java
public class Test {
    public static void main(String[] argv) {
        int[] x = new int[10];
        x[100] = 0; // will generate ArrayIndexOutOfBoundException
    }
}
```

exception 產生時，JVM 會由堆疊追蹤此錯誤點的呼叫資訊，並一一向外檢查，直到有 `try {} catch()` 區塊攔截此 exception 為止。

在上述的例子中，沒有任何 `try {} catch()` 的宣告，則 JVM 會終止該[執行緒](thread.md)。

## 類別 Exception 的相關方法

抓到例外後，可透過該例外物件，得到有趣的資訊
* `toString()`：以簡單的字串描述該例外
* `getMessage()`：列出細節訊息
* `printStackTrace()`：將堆疊資訊印在螢幕上，可幫助設計者快速找到錯誤點

## Error

前面提到 Exception 是 Throwable 的子類別。

另一個 Throwable 的子類別是 `java.lang.Error`。

所謂 Error 指的是嚴重的錯誤情況。當 Error 產生時，其行為和 Exception 類似，但是 `try {} catch()` 區塊沒有辦法攔下它們，最後會由 JVM 來處理 Error，並中斷執行緒的執行。

像 `OutOfMemoryError`、`StackOverflowError`都是 Error 的子類別。

## 範例

#### 用 Link List 實作 Stack

```java
public class Stack {
    private Node head;
    private int size;
    class Node {
        Object data;
        Node next;
    }
    public void push(Object s) {
        Node tmp = new Node();
        tmp.next = head;
        tmp.data = s;
        size++;
        head = tmp;
    }
    public Object pop() throws Exception {
        if (head == null) {
            throw new Exception();
        }
        Object tmp = head.data;
        head = head.next;
        size--;
        return tmp;
    }
}

public class Example {
    public static void main(String[] argv) {
        Stack s1 = new Stack();
        Stack s2 = new Stack();
        s1.push("abc");
        s1.push("def");
        s2.push("123");
        s2.push("456");
        try {
            s1.pop();
        } catch(Exception e) {}
    }
}

public class Example2 {
    public static void main(String[] argv) throws Exception {
        Stack s1 = new Stack();
        Stack s2 = new Stack();
        s1.push("abc");
        s1.push("def");
        s2.push("123");
        s2.push("456");
        s1.pop();
    }
}
```

#### 用 Link List 實作 Queue

```java
public class Queue {
    private Node head, tail;
    private int size;
    class Node {
        Object data;
        Node next;
    }
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
    public Object get() throws Exception {
        if (head == null) {
            throw new Exception();
        }
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
