# Java Virtual Machine

C 語言的開發模式，是編寫 `.c` 的 Source Code，再經由 Compiler 編譯成 Object Code。所謂 Object Code 指的是和硬體相關的機器指令，也就是說當我們想要把 C 程式移植到不同的硬體時，必須要重新 Compile，以產生新的執行檔。除了需要重新編譯外，新系統是否具備應用程式所需的程式庫，include 的檔案是否相容，也是程式能否在新機器上順利編譯和執行的條件之一。

在實務上，為了讓 C 程式能在不同的 UNIX 版本上都能順利編譯，原作者往往必須使用前置處理器的 `#ifdef` 指令，判斷不同環境的適當寫法。如果想把在 UNIX 上開發的 C 程式移植到 Windows 上，則有用到專屬程式庫的部分 \(如 UNIX 的使用者介面可能用到 X Window 的 API，Windows 就沒有支援，必須一台一台灌程式庫才行，很可能還要花錢買\)，就必須重寫才行。

解決此類問題的方法之一，是定義一種 Virtual Machine\(虛擬機器\)，讓程式語言編譯時不要翻成實體機器的指令，而是翻成 Virtual Machine 的目的碼。Virtual Machine 一般是以軟體來模擬的，只要新的平台有 Virtual Machine，則原始程式不用 Compile，執行舊機器上已有的 Virtual Machine 目的碼，就可以了。當然要達到完全不用重新 Compile 就能執行的理想，還要配合標準的程式庫才行。

Java 語言基於上述理念，定義了 Java Virtual Machine，它所用的指令稱為 byte code。使用 Virtual Machine 的缺點之一，是執行的速度較慢，代價是開發的速度變快了。以現在的硬體來說，大部分應用程式的執行速度已經沒有那麼重要，反倒是軟體的開發速度和品質越來越值得重視。

此外 JVM 的技術不斷進步，諸如 Just In Time\(JIT\) Compiler，或 HotSpot 等技術都可以讓 Java 程式以非常接近原生碼 \(Native Code\) 的速度執行。因此不要因為某些偏頗的報告或直覺，就不使用 Java 了。

開發 Java 應用程式的工具中，最常見的是由 Java 的原創公司 Sun Micro 所出版的 JDK\(Java Development Kit\)。JDK 可以免費下載。以 Text Editor 寫好的 Hello.java 原始檔：

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

就可以利用 Java Virtual Machine\(此處是 java 這個執行檔\) 來執行了。

上述過程中幾個比較會發生的問題是：

* javac 找不到：請設定 path 這個環境變數。
* javac 抱怨 class Hello 找不到：請確定你的檔名是大寫 `Hello.java`，程式內的 `public class Hello` 有沒有大小寫的問題。
* java 抱怨找不到 main：請確定 `public static void main(String[] argv)` 毫無錯誤。

# Java 是物件導向 \(Object-Oriented\) 程式語言

Java 是由 C++ 簡化來的。由於 C++ 要和 C 完全相容，又很注重效能問題，因此 C++ 算是很複雜的程式語言。Java 在設計之初，考量的重點之一就是簡單，因此和 C++ 比起來，不僅更為物件導向，而且比 C++ 容易學習。

Java 許多運算符號和敘述語法都是來自 C 語言，假設各位已經對 C 語言有所了解，本章後面的部分只將 Java 和 C 在運算符號和敘述語法上的差異點出來，相同的部分請參見 C 語言的課程內容。

# 資料型別

Java 語言所定義的基本資料型別有

| 型別名稱 | 位元長度 | 範圍 |
| :--- | :--- | :--- |
| boolean | 1 | true 或 false |
| byte | 8 | -128 ~ 127 |
| short | 16 | -32768 ~ 32767 |
| char | 16 | Unicode characters |
| int | 32 | -2147483648 ~ 2147483647 |
| long | 64 | -9223372036854775808 ~ 9223372036854775807 |
| float | 32 | +-3.4028237\_10+38 ~ +-1.30239846\_10-45 |
| double | 64 | +-1.76769313486231570\_10+308 ~ 4.94065645841246544\_10-324 |

Java 的資料型態裡沒有 unsigned。

Java 對數值型態的轉換比 C 稍微嚴格一點，下列左邊的部分都可以指定 \(assignment\) 給右邊的型別:

`byte` --&gt; `short` --&gt; `int` --&gt; `long` --&gt; `float` --&gt; `double`

除上述外，其他型別間的轉換都必須下達型別轉換 \(Type Casting\) 命令來處理，其形式為圓括弧裡寫上型別名稱，如 \(double\)

由於 Java 在 char 的型態部分採用 Unicode，因此字元常數的表示法，除因循 C 的規則外，也可以直接指定 16 bits Unicode 編碼給 char 型別的變數。例如由 Windows 「字元對應表」 程式中可查到象棋中的紅車的 unicode 編碼為 `4FE5`，Java 可用 `\u4fe5` 來表達。Java 的變數也可以用 Unicode 來命名，換句話說，你可以用中文取變數名稱。

除了這些基本資料型別外，Java 還有一個稱為 Reference\(參考\) 的型別。Reference 用來存取 Object\(物件\)，其功能和 C 語言的 pointer 用來存取記憶體有點像，但沒有 pointer 的 `&+-` 等運算符號，而且 Reference 只能存取型態相符合的類別。宣告 `Reference` 的語法是 `ClassName varName`，例如

`String s`;  
宣告 s 是一個型態為 reference 的變數，這表示我們可透過 s 來存取屬於 String 類別的物件 \(s is a reference to String object\)。

要特別強調的是，**s 並不是物件，而是用來指向 String 物件的 reference**。打個比方，

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

# 運算符號 \(Operator\)

Java 語言在運算式的部分，和 C 語言極為類似，除了沒有 `sizeof`, `pointer` 和 `struct` 相關的運算符號外，另外新增了 `>>>` 向右無號 shift，以及用來判斷物件型態的 `instanceof`。Java 的常數的表示法也和 C 相同，而 Java 裡的新資料型態 `boolean` 的合法值為 `true` 和 `false` 兩個常數。

## 算術 \(Arithmetic\) 運算符號

| 運算符號 | 功能敘述 |
| :--- | :--- |
| + | 加 |
| \* | 乘 |
| - | 減 |
| / | 除 |
| % | 餘數 |
| ++ | 加一 |
| -- | 減一 |

## 邏輯 \(logic\) 運算符號

| 運算符號 | 功能敘述 |
| :--- | :--- |
| &gt; | 大於 |
| &lt; | 小於 |
| &gt;= | 大於等於 |
| &lt;= | 小於等於 |
| == | 等於 |
| != | 不等於 |
| && | logic AND |
| \|\| | logic OR |
| ! | logic NOT |
| instanceof | reference instanceof ClassName 判斷 reference 所指到的物件其型態是否和 ClassName 相容 |

Java 語言和 C 語言有關邏輯運算最大的不同, 在於 Java 以 boolean 資料型態 \(只有 true 和 false 兩種值\) 判斷條件是否成立, 而 C 語言只能使用 0 或非 0。

## 位元 \(Bit\) 運算符號

| 運算符號 | 功能敘述 |
| :--- | :--- |
| & | bit AND |
| &lt;&lt; | left bit shift |
| \| | bit OR |
| &gt;&gt; | right bit shift with sign |
| ^ | bit XOR |
| ~ | 1 補數 |
| &gt;&gt;&gt; | 同 &gt;&gt; 但左邊一律補零 |

## 其他運算符號

| 運算符號 | 功能敘述 |
| :--- | :--- |
| = | 將右邊的值複製到左邊的變數 |
| \(type\) | 將右邊的數值或 reference 轉換成 type 型別 |
| += | 將右邊的數值加上左邊的數值然後指定給左邊的變數 |
| ?: | 若 `?` 左邊成立則做 `:` 左邊否則做 `:` 右邊 |
| , | 合併兩個運算視為一個敘述 |
| \(運算式\) | 表示 \(\) 內優先運算 |
| . | Reference.ObjectMember 或 ClassName.ClassName 存取物件或類別成員 |
| new | 產生物件 |

## 優先權

| 種類 | 運算符號 | 結合順序 |
| :--- | :--- | :--- |
| group | \(op\) | left to right |
| postfix | \[\] . \(params\) op++ op-- | right to left |
| prefix | ++op --op +op -op ~ ! | right to left |
| creation or casting | new \(type\)op | right to left |
| multiplicative | \* / % | left to right |
| additive | + - | left to right |
| shift | &lt;&lt;&gt;&gt; &gt;&gt;&gt; | left to right |
| relational | &lt;&gt; &lt;=&gt;= instanceof == | left to right |
| equality | == != | left to right |
| bitwise and | & | left to right |
| bitwise exclusive or | ^ | left to right |
| bitwise inclusive or | \| | left to right |
| logical and | && | left to right |
| logical or | \|\| | left to right |
| conditional | ? : | right to left |
| assignment | = += -= \*= /= %= &= ^= \|= &lt;&lt;=&gt;&gt;= &gt;&gt;&gt;= | right to left |
| seperator | , | left to right |

# 流程控制敘述

Java 的流程控制敘述和 C 語言極為類似, 不同處在於 `break` 和 `continue` 兩個指令。Java 的 `break` 和 `continue` 指令後面可以加上標籤, 以指示要跳出或繼續的範圍。

```java
public class BreakContinueExample {
    public static void main(String[] argv) {
        int i, j;
        outerLoop:
        for (i = 0; i < 100; i++) {
            innerLoop:
            for (j = 0; j < 100; j++) {
                if (j == 50 && i == 50) {
                    break outerLoop;
                }
            }
        }
        System.out.println("Loop have been terminated.");
    }
}
```

在上面的例子中, 當 `j == 50` 且 `i == 50` 時，`break` 指令會跳出最外面的迴圈，直接印出迴圈終止訊息。如果 `break` 後面沒有 `outerLoop` 的話，只會跳出裡面的迴圈，然後 `i` 從 `51` 繼續做下去。

# 字串

C 語言定義以 0 結尾的字元陣列就是字串。但對 Java 來說，字串是由 String 類別來表達，也就是說 String 是物件而不是陣列。由於我們經常使用字串，為了寫作程式方便起見，Java Compiler 碰到 `+` 符號某一邊的型態是 String 時，就會把 `+` 翻譯成 StringBuffer 類別裡相對應的 append Method。例如：

```java
public class StringTest {
    public static void main(String[] argv) {
        int x = 5;
        float y = 1.5;
        System.out.println("x = " + x + ", y = " + y);
    }
}
```

會翻譯成：

```java
public class StringTest {
    public static void main(String[] argv) {
        int x = 5;
        float y = 1.5;
        System.out.println((new StringBuffer("x = ")).append(x).append(", y = ").append(y).toString());
    }
}
```

如果你會 C++，看到 Java 字串 `+` 符號的語法，千萬不要以為 Java 支援 operator overloading。Java 只是透過 Compiler 來做特別的轉換，稱這種技術為 Compiler Sugar 比較適合。

# Java 語言的寫作風格

寫作 Java 程式時，請注意下列幾種風格

* Class Name 請首字大寫
* Variable Name 和 Method Name 請首字小寫
* 如果名稱由數個英文字組成，第二個英文字以後首字大寫
* 內縮四個空格
* 註解部分如要變成說明文件，請遵照 javadoc 這個工具的寫作規則

```java
/**
 * 第一行的兩個**用來告訴javadoc此部份註解要變成HTML文件的一部份
 * 這段註解裡的所有文字都會變成此類別一開頭的說明
 */
public class Hello { // Class Name首字大寫
    /**
     * 此段註解會變成描述main方法的一部分
     * @param argv 使用@param註記會產生參數(parameter)argv的相關說明
     * @return 傳回值的意義說明
     */
     public static void main(String[] argv) { // Method Name首字小寫
        // argv: array of references to String object
        int myVariable; // 變數宣告
        int i, sum;
        for (i = 1, sum = 0; i <= 100; i++) {
            sum += i;
        }
        System.out.println("summation from 1 to 100 is "+sum);
    }
}
```

# 運算符號範例

## 攝氏溫度轉華氏溫度

```java
public class Example {
    public static void main(String[] argv) {
        float degree = 100.0;
        System.out.println("100C=" + (degree * 9.0 / 5.0 + 32.0));
    }
}
```

## 華氏溫度轉攝氏溫度

怎麼寫呢?

## $$ 1 + 2 + ... + n $$ 的總合

```java
public class Example {
    public static void main(String[] argv) {
        int n = 100;
        System.out.println("1+2+...+"+n+"  = " + ( n * (n + 1) / 2));
    }
}
```

特別注意上述的運算式裡 `/2` 要放到最後面，如果寫成 `n/2*(n+1)`，從數學式子的角度看好像沒問題，但別忘了，binary operator 的兩邊必須是同樣型別的資料，而且計算的結果也是同樣的型別。因此 `n/2*(n+1)` 會先計算 `n/2`，如果 `n` 不能被 `2` 整除的話，那麼為了符合計算結果必須是整數的限制，則小數點的部份就會無條件捨去，使得計算的結果錯誤。下面的範例一樣要注意相同的問題。

# $$ 1^2 + 2^2 + ... + n^2 $$ 的總和

怎麼寫?

## 把浮點數四捨五入為整數

Java 語言規定浮點數轉整數時，小數點部分無條件捨去。如果要達到浮點數四捨五入為整數的效果，可以使用下面的小技巧

```java
public class Example {
    public static void main(String[] argv) {
        double x = 20.6;
        System.out.println(x + " 四捨五入成為 " + (int)(x+0.5));
        System.out.println(x + " 四捨五入成為 " + round(x));
    }
    static int round(double y) {
        return (int)(y + 0.5);
    }
}
```

# 迴圈範例

## 寫一程式輸入 5 個整數數字，計算其總合和平均。解析：

1. 需要 1 個變數紀錄到目前為止所有 `inputNum` 的總和，稱此變數為 `sum`，其初始值為 `0`
2. 以迴圈執行 5 次，每次輸入數字加總到 `sum`，迴圈執行的次數以變數 `i` 來代表
3. 平均數為 `sum/5`
4. 如何讀入資料?

```java
import java.util.Scanner;
public class Example {
    public static void main(String[] argv) {
        int sum = 0, i = 0;
        Scanner in = new Scanner(System.in);
        while (i < 5 && in.hasNextInt()) {
            sum = sum + in.nextInt();
            i++;
        }
        System.out.println("sum is "+sum", average is "+(sum/5.0));
    }
}
```

## 寫一函數輸入參數 `int n`, 傳回 $$ 1 + 2 + 3 ... + n $$ 的總合。解析：

1. 要想辦法拜訪 `1, 2, 3, ..., n` 的每一個數字一次
2. 可用 `for (i = 1; i <= n; i++)` 的形式達成上述目標
3. 拜訪到這些數字時，就把它們加起來

```java
public class Example {
    public static int sum(int n) {
        int total = 0; // 紀錄到目前為止的總和
        for (int i = 1; i <= n; i++) {
            total += i;
        }
        return total;
    }
    public static void main(String[] argv) {
        System.out.println(sum(100));
    }
}
```

## 寫一函數輸入參數 `int n`, 傳回 $$ 1 + 3 + 5 ... + n $$ 的總合。解析：

1. 要想辦法拜訪 `1, 3, 5, ..., n` 的每一個數字一次，也就是從 1 開始每次加 2
2. 可用 `for (i = 1; i <= n; i += 2)` 的形式達成上述目標
3. 拜訪到這些數字時，就把它們加起來

怎麼寫?

## 寫一函數於螢幕上畫出九九乘法表。解析：

1. 總共有 `i = 1..9` 列，`j = 1..9` 行，對第 `i` 列第 `j` 行元素來說，其數值為 `i*j`

```java
public class Example {
    public static void main(String[] argv) {
        for (int i = 1; i <= 9; i++) {
            for (int j = 1; j <= 9; j++) {
                System.out.print(" " + (i * j));
            }
            System.out.println();
        }
    }
}
```

## 輸入參數 `int size`，並在螢幕上印出正方形，`size = 3` 的樣子如下

```
***
***
***
```

解析

1. 螢幕上的游標只能由上而下，由左而右，無法回頭。
2. 此圖形共有 `1 .. size` 列，每列有 `size` 個 `*`，因此可用兩層迴圈來做。
3. 要讓一個敘述執行 `size` 次，可用 `for ( i = 1; i <= size; i++ )` 的形式來達成

```java
public class Example {
    public static void print(int size) {
        int i, j; // 第i列,第j行
        for (i = 1; i <= size; i++) { // 印出第i列
            for (j = 1; j <= size; j++) { // 第i列有size個*
                System.out.print("*");
            }
            System.out.println();
        }
    }
    public static void main(String[] argv) throws Exception {
        BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
        print(Integer.parseInt(in.readLine()));
    }
}
```

## 輸入參數 `int size`，並在螢幕上印出直角三角形，`size = 3` 的樣子如下

```
*
**
***
```

解析

1. 螢幕上的游標只能由上而下，由左而右，無法回頭。
2. 此圖形共有 `1 .. size` 列，第 `i` 列有個 `*`，因此可用兩層迴圈來做。

怎麼寫?

## 撰寫一函數輸入 `int size`，並在螢幕上印出等腰的三角形，`size = 3` 的樣子如下

```
  *
 ***
*****
```

解析

1. 總共有 `1 .. size` 列，對第 `i` 列而言，有 `size - i` 個空格，以及 `(2 * i- 1)` 個 `*`

怎麼寫?

## 寫一函數求兩個整數的最大公因數，解析：

1. 此函數需要兩個參數 `x`, `y`
2. 當 `y` 不能整除 `x` 時，將 `x` 設成為 `y`，`y` 設為 `x % y`，重複此步驟直到 `x % y` 為 `0`
3. 此時 `y` 就是這兩個數的最大公因數

```java
public class Example {
    public static void main(String[] argv) {
        System.out.println(gcd(12,18));
    }
    public static int gcd(int x, int y) {
        int tmp;
        // 如果x < y 則下面的迴圈執行第一次時就會交換x,y了
        while (x % y != 0) {
            tmp = y;
            y = x % y;
            x = tmp;
        }
        return y;
    }
}
```

## 寫一函數求兩個整數的最小公倍數

怎麼寫?

寫一函數求費氏數，解析：

1. `F(n) = n`, `if n <= 1;`
2. `F(n) = F(n-1) + F(n-2)`, otherwise
3. 可定義兩變數 `fn_1`, `fn_2` 表示最近兩個找出的費氏數
4. 下一個費氏數依定義為 `fn_1 + fn_2`
5. 找到最新的費氏數後, 最近的兩個費氏數就變成了 `fn_1 + fn_2` 以及 `fn_1`
6. 以變數 `i` 紀錄目前要求的是哪一個費氏數
7. 以變數 `tmp` 作為更新最新兩個費氏數所需的記憶體空間

```java
public class Example {
    public static void main(String[] argv) {
        System.out.println(fab(5));
    }
    public static int fab(int n) {
        int fn_1 = 1, fn_2 = 0; // 紀錄最近找到的兩個費氏數
        int i, tmp; // i表示目前要找F(i)
        if (n <= 1) return n;
        for (i = 2; i <= n; i++) {
            tmp = fn_1;   // 先把fn_1紀錄在tmp
            fn_1 += fn_2; // 最新的費氏數是前面兩個相加
            fn_2 = tmp;   // 第二新的就是原先的fn_1
        }
        return fn_1;
    }
}
```

# 遞迴 \(recursion\) 範例

求 $$ 1 + 2 + 3 + ... + n $$

解析

* 邊際條件是 `n = 1` 時，總合為 `1`
* 該函數可定成 `int sum(int n)`
* `sum(n) = n + sum(n - 1)`

```java
public class Example {
    public static void main(String[] argv) {
        System.out.println(sum(100));
    }
    public static int sum(int n) {
        if (n == 1) {
            return 1;
        }
        return n + sum(n - 1);
    }
}
```

## 以遞迴計算 `1 * 2 + 2 * 3 + 3 * 4 + ... + (n-1) * n` 之和

怎麼寫?

## 利用遞迴求得 `A` 的 `B` 次方

```java
public class Example {
    public static void main(String[] argv) {
        System.out.println(power(2, 6));
    }
    public static int power(int a, int b) {
        switch(b) {
        case 0: return 1;
        case 1: return a;
        default: return (a * power(a, b - 1));
    }
}
```

## 以遞迴求兩個整數 `m`, `n` 的最大公因數

解析

* 如果 `n == 0`，則最大公因數為 `m`
* 如果 `n` 不等於 `0`，則最大公因數為 `gcd(m, n) == gcd(n, m%n)`

怎麼寫?

## 費式數列

解析

* 費氏數列的定義為 `F(n) = n`, `if n <= 1`
* `F(n) = F(n-1) + F(n-2), if n > 1`。

```java
public class Example {
    public static void main(String[] argv) {
        System.out.println(fab(5));
    }
    public static int fab(int num) {
        if (num <= 1) {
            return num;
        }
        return fab(num - 1) + fab(num - 2);
    }
}
```

## Ackerman 函數

`A(m, n)` 定義為

1. `n + 1, if m = 0`
2. `A(m - 1, 1), if n = 0`
3. `A(m - 1, A(m, n - 1))`, otherwise

怎麼寫?

