# 訊息傳遞 (Message Passing) 和函數呼叫 (Function Call) 的不同點

Procedure Language 以函數作為寫作和執行的主體。函數由一群程式碼所組成，函數的開頭地址在 Compile time 或 Link time 就已經決定好了，而在 Runtime 呼叫函數時給予適當的參數。函數和記憶體之間沒有關聯性，而且函數 `A` 可以被任何其他函數呼叫，程式語言並沒有特別的規範。

Object-Oriented Language 以 Class 作為寫作的主體，執行時主要由 Object 來紀錄程式狀態。由於物件導向程式語言將 Object Variable 和 Object Method 一起定義在 Class 內，再透過 Encapsulation 的機制規範存取 Object Member 的範圍，因此 Object 的 Variable 和 Method 就組成了一個完整的個體。

雖然 Object Method 寫起來就像 Function，但執行 Method 內程式碼的機制和 Function 不同：

- Object Method 定義物件接受到訊息時的反應，也就是說執行 Method 時有一個隱形的參數，意即這個物件 (this)
- Function 的實際地址在 Compile 或 Link time 就已經決定好了，但對 OO 來說，Object 必須在 Runtime 接受到訊息後，才能決定實際要執行的 Method。

# Message Passing 的語法

由於 Java 只能透過 reference 來存取物件，因此 Message Passing 的語法是 `reference.method`

```java
/**
 * 假設People有variable age,其值限定於0到130之間
 */
public class People {
    private int age;
    public People(int d) {
        this.age = d;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int d) {
        if (d >= 0 && d <= 130) { // 檢查年齡是否合法
            age = d;
        }
    }
    public void increaseAge() {
        if (age < 130) { // 檢查年齡是否合法
            this.age++;
        }
    }
    public static void main(String[] argv) {
        People e1, e2; // e1,e2 are references to Object People, not the Object themselves
        e1 = new People(3);
        e2 = new People(5);
        e1.setAge(30);
        e2.setAge(50);
        e1.increaseAge();
        e2.increaseAge();
    }
}
```

`this` 這個 keyword 表示接收到此訊息的物件。由於設計時是定義 Class，而執行時則是 Object 接受訊息，Class 只有一個，但 Object 可以有很多個，因此必須使用 `this` 表達現在接收到此訊息的物件。

由於 Object Method 具有隱形的物件參數，因此 Class Method 不能去存取 Object Member

```java
public class ErrorCall {
    int data;  // object variable
    public void objectMethod() {
        data = 10;
    }
    public static void classMethod() {
        objectMethod(); // Compile Error
        data = 10; // Compile Error
    }
}
```

Message Passing 在執行期間才會決定實際 Method 的機制，我們將在 Inheritance(繼承) 裡敘述。
