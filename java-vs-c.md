# Java Virtual Machine

C 語言的開發模式，是編寫 `.c` 的 Source Code，再經由 Compiler 編譯成 Object Code。所謂 Object Code 指的是和硬體相關的機器指令，也就是說當我們想要把 C 程式移植到不同的硬體時，必須要重新 Compile，以產生新的執行檔。除了需要重新編譯外，新系統是否具備應用程式所需的程式庫，include 的檔案是否相容，也是程式能否在新機器上順利編譯和執行的條件之一。

在實務上，為了讓 C 程式能在不同的 UNIX 版本上都能順利編譯，原作者往往必須使用前置處理器的 `#ifdef` 指令，判斷不同環境的適當寫法。如果想把在 UNIX 上開發的 C 程式移植到 Windows 上，則有用到專屬程式庫的部分 (如 UNIX 的使用者介面可能用到 X Window 的 API，Windows 就沒有支援，必須一台一台灌程式庫才行，很可能還要花錢買)，就必須重寫才行。

解決此類問題的方法之一，是定義一種 Virtual Machine(虛擬機器)，讓程式語言編譯時不要翻成實體機器的指令，而是翻成 Virtual Machine 的目的碼。Virtual Machine 一般是以軟體來模擬的，只要新的平台有 Virtual Machine，則原始程式不用 Compile，執行舊機器上已有的 Virtual Machine 目的碼，就可以了。當然要達到完全不用重新 Compile 就能執行的理想，還要配合標準的程式庫才行。

Java 語言基於上述理念，定義了 Java Virtual Machine，它所用的指令稱為 byte code。使用 Virtual Machine 的缺點之一，是執行的速度較慢，代價是開發的速度變快了。以現在的硬體來說，大部分應用程式的執行速度已經沒有那麼重要，反倒是軟體的開發速度和品質越來越值得重視。

此外 JVM 的技術不斷進步，諸如 Just In Time(JIT) Compiler，或 HotSpot 等技術都可以讓 Java 程式以非常接近原生碼 (Native Code) 的速度執行。因此不要因為某些偏頗的報告或直覺，就不使用 Java 了。

開發 Java 應用程式的工具中，最常見的是由 Java 的原創公司 Sun Micro 所出版的 JDK(Java Development Kit)。JDK 可以免費下載。以 Text Editor 寫好的 Hello.java 原始檔：

```java
public class Hello {
    public static int gvar;
    public static void say(String s) {
        int x = 10;
        System.out.print(s+x);
    }
    public static void main(String[] argv) {
        float y = 0;
        say("Hello, world\n");
    }
}
```

這程式的C版本如下

```c
#include <stdio.h>
int gvar;
void say(char[] s) {
    int x = 10;
    printf("%s%d", s, x);
}
int main(int argc, char** argv) {
    float y = 0;
    say("Hello, world\n");
}
```

經過：

`$ javac Hello.java`

編譯完成後會產生 byte code 格式的 `Hello.class`， 然後

`$ java Hello`

就可以利用 Java Virtual Machine(此處是 java 這個執行檔) 來執行了。

上述過程中幾個比較會發生的問題是：

- javac 找不到：請設定 path 這個環境變數。
- javac 抱怨 class Hello 找不到：請確定你的檔名是大寫 `Hello.java`，程式內的 `public class Hello` 有沒有大小寫的問題。
- java 抱怨找不到 main：請確定 `public static void main(String[] argv)` 毫無錯誤。

# Java 是物件導向 (Object-Oriented) 程式語言

Java 是由 C++ 簡化來的。由於 C++ 要和 C 完全相容，又很注重效能問題，因此 C++ 算是很複雜的程式語言。Java 在設計之初，考量的重點之一就是簡單，因此和 C++ 比起來，不僅更為物件導向，而且比 C++ 容易學習。

Java 許多運算符號和敘述語法都是來自 C 語言，假設各位已經對 C 語言有所了解，本章後面的部分只將 Java 和 C 在運算符號和敘述語法上的差異點出來，相同的部分請參見 C 語言的課程內容。

# 資料型別

Java 語言所定義的基本資料型別有

| 型別名稱 | 位元長度 | 範圍 |
|:--- |:--- |:--- |
| boolean | 1 | true 或 false |
| byte | 8 | -128 ~ 127 |
| short | 16 | -32768 ~ 32767 |
| char | 16 | Unicode characters |
| int | 32 | -2147483648 ~ 2147483647 |
| long | 64 | -9223372036854775808 ~ 9223372036854775807 |
| float | 32 | +-3.4028237*10+38 ~ +-1.30239846*10-45 |
| double | 64 | +-1.76769313486231570*10+308 ~ 4.94065645841246544*10-324 |

Java 的資料型態裡沒有 unsigned。

Java 對數值型態的轉換比 C 稍微嚴格一點，下列左邊的部分都可以指定 (assignment) 給右邊的型別:

`byte` --> `short` --> `int` --> `long` --> `float` --> `double`

除上述外，其他型別間的轉換都必須下達型別轉換 (Type Casting) 命令來處理，其形式為圓括弧裡寫上型別名稱，如 (double)

由於 Java 在 char 的型態部分採用 Unicode，因此字元常數的表示法，除因循 C 的規則外，也可以直接指定 16 bits Unicode 編碼給 char 型別的變數。例如由 Windows 「字元對應表」 程式中可查到象棋中的紅車的 unicode 編碼為 `4FE5`，Java 可用 `\u4fe5` 來表達。Java 的變數也可以用 Unicode 來命名，換句話說，你可以用中文取變數名稱。

除了這些基本資料型別外，Java 還有一個稱為 Reference(參考) 的型別。Reference 用來存取 Object(物件)，其功能和 C 語言的 pointer 用來存取記憶體有點像，但沒有 pointer 的 `&+-` 等運算符號，而且 Reference 只能存取型態相符合的類別。宣告 `Reference` 的語法是 `ClassName varName`，例如

`String s`;
宣告 s 是一個型態為 reference 的變數，這表示我們可透過 s 來存取屬於 String 類別的物件 (s is a reference to String object)。

要特別強調的是，==s 並不是物件，而是用來指向 String 物件的 reference==。打個比方，

```java
public class 動物 {
    動物 手指頭; // java 因字元編碼使用unicode, 所以可用中文當變數名稱
    public static void main(String[] arg) {
        動物 手指頭2;
        手指頭2 = new 動物();
    }
}
```

變數 `手指頭` 宣告為 reference，可指向屬於 class `動物` 的物件，`手指頭` 不是 `動物`，而是用 `手指頭` 指向某隻`動物`。

```java
java.lang.Float f;
java.lang.Double d;
java.lang.Integer i;
```

以上變數的型態都是 reference
