## Program, Process, Thread

在介紹 Thread 之前，我們必須先把 Program 和 Process 這兩個觀念作一個釐清。

* Program：一群程式碼的集合，用以解決特定的問題。以物件導向的觀念來類比，相當於 Class。
* Process：由 Program 所產生的執行個體，一個 Program 可以同時執行多次，產生多個Process。以物件導向的觀念來類比，相當於 Object。  
  每一個 Process 又由以下兩個東西組成
  * 一個 Memory Space：相當於 Object 的 variable，不同 Process 的 Memory Space 也不同，彼此看不到對方的 Memory Space。
  * 一個或以上的 Thread：Thread 代表從某個起始點開始(例如 `main`)，到目前為止所有函數的呼叫路徑，以及這些呼叫路徑上所用到的區域變數。當然程式的執行狀態，除了紀錄在主記憶體外，CPU 內部的暫存器(如 `Program Counter`、`Stack Pointer`、`Program Status Word` 等)也需要一起紀錄。  
    所以 Thread 又由下面兩項組成
    * Stack：紀錄函數呼叫路徑，以及這些函數所用到的區域變數
    * 目前 CPU 的狀態

由上面的描述中，我們在歸納 Thread 的重點如下
* 一個 Process 可以有多個 Thread。
* 同一個 Process 內的 Thread 使用相同的 Memory Space，但這些 Thread 各自擁有其 Stack。換句話說，Thread 能透過 reference 存取到相同的 Object，但是 local variable 卻是各自獨立的。
* 作業系統會根據 Thread 的優先權以及已經用掉的 CPU 時間，在不同的 Thread 作切換，以讓各個 Thread 都有機會執行。

## 如何產生 Thread

Java 以 `java.lang.Thread` 這個類別來表示 Thread。

Class Thread 有兩個 Constructor：
1. `Thread()`
2. `Thread(Runnable)`

第一個 Constrctor 沒有參數，第二個需要一個 Runnable 物件當參數。

Runnable 是一個 interface，定義於 `java.lang` 內，其宣告為

```java
public interface Runnable {
    public void run();
}
```

使用 `Thread()` 產生的 Thread，其進入點為 Thread 裡的 `run()`；使用 `Thread(Runnable)` 產生的 Thread，其進入點為 Runnable 物件裡的 `run()`。

當 `run()` 結束時，這個 Thread 也就結束了；這和 `main()` 結束有相同的效果。其用法以下面範例說明：

```java
public class ThreadExample1 extends Thread {
    public void run() { // override Thread's run()
        System.out.println("Here is the starting point of Thread.");
        for (;;) { // infinite loop to print message
            System.out.println("User Created Thread");
        }
    }
    public static void main(String[] argv) {
        Thread t = new ThreadExample1(); // 產生Thread物件
        t.start(); // 開始執行t.run()
        for (;;) {
            System.out.println("Main Thread");
        }
    }
}
```

以上程式執行後，螢幕上會持續印出 `User Created Thread` 或 `Main Thread` 的字樣。

利用 Runnable 的寫法如下：

```java
public class ThreadExample2 implements Runnable {
    public void run() { // implements Runnable run()
        System.out.println("Here is the starting point of Thread.");
        for (;;) { // infinite loop to print message
            System.out.println("User Created Thread");
        }
    }
    public static void main(String[] argv) {
        Thread t = new Thread(new ThreadExample2()); // 產生Thread物件
        t.start(); // 開始執行Runnable.run();
        for (;;) {
            System.out.println("Main Thread");
        }
    }
}
```

## Thread 的優先權與影響資源的相關方法

`Thread.setPriority(int)` 可以設定 Thread 的優先權，數字越大優先權越高。

Thread 定義了 3 個相關的 static final variable：

```java
public static final int MAX_PRIORITY 10
public static final int MIN_PRIORITY 1
public static final int NORM_PRIORITY 5
```

要提醒讀者的是，優先權高的 Thread 其佔有 CPU 的機會比較高，但優先權低的也都會有機會執行到。其他有關 Thread 執行的方法有：
* `yield()`：先讓給別的 Thread 執行
* `sleep(int time)`：休息 time mini second (1/1000秒)
* `join()`：呼叫 `ThreadA.join()` 的執行緒會等到 ThreadA 結束後，才能繼續執行

你可以執行下面的程式，看看 `yield()` 的效果

```java
public class ThreadExample1 extends Thread {
    public void run() { // overwrite Thread's run()
        System.out.println("Here is the starting point of Thread.");
        for (;;) { // infinite loop to print message
            System.out.println("User Created Thread");
            yield();
        }
    }
    public static void main(String[] argv) {
        Thread t = new ThreadExample1(); // 產生Thread物件
        t.start(); // 開始執行t.run()
        for (;;) {
            System.out.println("Main Thread");
            yield();
        }
    }
}
```

觀看 `join()` 的效果

```java
public class JoinExample extends Thread {
    String myId;
    public JoinExample(String id) {
        myId = id;
    }
    public void run() { // overwrite Thread's run()
	for (int i=0; i < 500; i++) {
            System.out.println(myId+" Thread");
        }
    }
    public static void main(String[] argv) {
        Thread t1 = new JoinExample("T1"); // 產生Thread物件
        Thread t2 = new JoinExample("T2"); // 產生Thread物件
        t1.start(); // 開始執行t1.run()
        t2.start();
        try {
            t1.join(); // 等待t1結束
            t2.join(); // 等待t2結束
        } catch (InterruptedException e) {}
        for (int i=0;i < 5; i++) {
            System.out.println("Main Thread");
        }
    }
}
```

觀看 `sleep()` 的效果

```java
public class SleepExample extends Thread {
    String myId;
    public SleepExample(String id) {
        myId = id;
    }
    public void run() { // overwrite Thread's run()
        for (int i=0; i < 500; i++) {
            System.out.println(myId+" Thread");
            try {
                sleep(100);
            } catch (InterruptedException e) {}
        }
    }
    public static void main(String[] argv) {
        Thread t1 = new SleepExample("T1"); // 產生Thread物件
        Thread t2 = new SleepExample("T2"); // 產生Thread物件
        t1.start(); // 開始執行t1.run()
        t2.start();
    }
}
```

## Critical Section (關鍵時刻) 的保護措施

如果設計者沒有提供保護機制的話，Thread 取得和失去 CPU 控制權的時機是由作業系統來決定。

也就是說 Thread 可能在執行任何一個機器指令時，被作業系統取走 CPU 控制權，並交給另一個 Thread。

由於某些真實世界的動作是不可分割的，例如跨行轉帳 X 元由 A 帳戶到 B 帳戶，轉帳前後這兩個帳戶的總金額必須相同，但以程式來實作時，卻無法用一個指令就完成，如轉帳可能要寫成下面的這一段程式碼

```
if (A >= X) {
    A = A - X; // 翻譯成3個機器指令LOAD A, SUB X, STORE A
    B = B + X;
}
```

如果兩個 Thread 同時要存取 A、B 兩帳戶進行轉帳，假設當 Thread 1 執行到 SUBX 後被中斷，Thread 2 接手執行完成另一個轉帳要求，然後 Thread 1 繼續執行未完成的動作，請問這兩個轉帳動作正確嗎？

我們以 A = 1000、B = 0 分別轉帳 100、200 元來說明此結果

```
LOAD A // Thread 1 現在 A 還是 1000
SUB 100 // Thread 1
LOAD A // 假設此時 Thread 1 被中斷，Thread 2 接手，因為 Thread 1 還沒有執行 STORE A，所以變數 A 還是 1000
SUB 200 // Thread 2
STORE A // Thread 2，A = 800
LOAD B // Thread 2，B 現在是 0
ADD 200 // Thread 2
STORE B // B = 200
STORE A // Thread 1 拿回控制權，A = 900
LOAD B // Thread 1，B = 200
ADD 100 // Thread 1
STORE B // B = 300
```

你會發現執行完成後 A = 900、B = 300，也就是說銀行平白損失了 200 元，當然另外的執行順序可能造成其他不正確的結果。

我們把這問題再整理一下：
1. 寫程式時假設指令會循序執行
2. 某些不可分割的動作，需要以多個機器指令來完成
3. Thread 執行時可能在某個機器指令被中斷
4. 兩個 Thread 可能執行同一段程式碼，存取同一個資料結構
5. 這樣就破壞了第 1 點的假設

因此在撰寫多執行緒的程式時，必須特別考慮這種狀況(又稱為 race condition)。

Java 的解決辦法是，JVM 會在每個物件上擺一把鎖(lock)，然後程式設計者可以宣告執行某一段程式(通常是用來存取共同資料結構的程式碼，又稱為 Critical Section)時，必須拿到某物件的鎖才行，這個鎖同時間最多只有一個執行緒可以擁有它。

```java
public class Transfer extends Thread {
    public static Object lock = new Object();
    public static int A = 1000;
    public static int B = 0;
    private int amount;
    public Transfer(int x) {
        amount = x;
    }
    public void run() {
        synchronized(lock) { // 取得lock,如果別的thread A已取得,則目前這個thread會等到thread A釋放該lock
            if (A >= amount) {
                A = A - amount;
                B = B + amount;
            }
        } // 離開synchronized區塊後,此thread會自動釋放lock
    }
    public static void main(String[] argv) {
        Thread t1 = new Transfer(100);
        Thread t2 = new Transfer(200);
        t1.start();
        t2.start();
    }
}
```

除了 synchronized(ref) 的語法可以鎖定 ref 指到的物件外，synchronized 也可以用在 object method 前面，表示要鎖定 this 物件才能執行該方法。

以下是 Queue 結構的範例：

```java
public class Queue {
    private Object[] data;
    private int size;
    private int head;
    private int tail;
    public Queue(int maxLen) {
        data = new Object[maxLen];
    }
    public synchronized Object deQueue() {
        Object tmp = data[head];
        data[head] = null;
        head = (head+1)%data.length;
        size--;
        return tmp;
    }
    public synchronized void enQueue(Object c) {
        data[tail++] = c;
        tail %= data.length;
        size++;
    }
}
```

雖然上面的程式正確無誤，但並未考慮資源不足時該如何處理。

例如 Queue 已經沒有資料了，卻還想拿出來；或是 Queue 裡已經塞滿了資料，使用者卻還要放進去？

我們當然可以使用 Exception Handling 的機制：

```java
public class Queue {
    private Object[] data;
    private int size;
    private int head;
    private int tail;
    public Queue(int maxLen) {
        data = new Object[maxLen];
    }
    public synchronized Object deQueue() throws Exception {
        if (size == 0) {
            throw new Exception();
        }
        Object tmp = data[head];
        data[head] = null;
        head = (head+1)%data.length;
        size--;
        return tmp;
    }
    public synchronized void enQueue(Object c) throws Exception {
        if (size >= maxLen) {
            throw new Exception();
        }
        data[tail++] = c;
        tail %= data.length;
        size++;
    }
}
```

但假設我們的執行環境是，某些 Thread 專門負責讀取使用者的需求，並把工作放到 Queue 裡面，某些 Thread 則專門由 Queue 裡抓取工作需求做進一步處理。

這種架構的好處是，可以把慢速或不定速的輸入(如透過網路讀資料，連線速度可能差很多)和快速的處理分開，可使系統的反應速度更快，更節省資源。

那麼以 Exceptoin 來處理 Queue 空掉或爆掉的情況並不合適，因為使用 Queue 的人必須處理例外狀況，並不斷的消耗CPU資源：

```java
public class Getter extends Thread {
    Queue q;
    public Getter(Queue q) {
        this.q = q;
    }
    public void run() {
        for (;;) {
            try {
                Object data = q.deQueue();
                // processing
            } catch(Exception e) {
                // if we try to sleep here, user may feel slow response
                // if we do not sleep, CPU will be wasted
            }
        }
    }
}
public class Putter extends Thread {
    Queue q;
    public Putter(Queue q) {
        this.q = q;
    }
    public void run() {
        for (;;) {
            try {
                Object data = null;
                // get user request
                 q.enQueue(data);
            } catch(Exception e) {
                // if we try to sleep here, user may feel slow response
                // if we do not sleep, CPU will be wasted
            }
        }
    }
}
public class Main {
    public static void main(String[] argv) {
        Queue q = new Queue(10);
        Getter r1 = new Getter(q);
        Getter r2 = new Getter(q);
        Putter w1 = new Putter(q);
        Putter w2 = new Putter(q);
        r1.start();
        r2.start();
        w1.start();
        w2.start();
    }
}
```

為了解決這類資源分配的問題，Java Object 提供了下面三個 method：
* `wait()`：使呼叫此方法的 Thread 進入 Blocking Mode，並設為等待該 Object，呼叫 `wait()` 時，該 Thread 必須擁有該物件的 lock。Blocking Mode 下的 Thread 必須釋放所有手中的 lock，並且無法使用CPU。
* `notifyAll()`：讓等待該 Object 的所有 Thread 進入 Runnable Mode。
* `notify()`：讓等待該 Object 的某一個 Thread 進入 Runnable Mode。

所謂 Runnable Mode 是指該 Thread 隨時可由作業系統分配CPU資源。

Blocking Mode 表示該 Thread 正在等待某個事件發生，作業系統不會讓這種 Thread 取得 CPU 資源。

前一個 Queue 的範例就可以寫成：

```java
public class Queue {
    private Object[] data;
    private int size;
    private int head;
    private int tail;
    public Queue(int maxLen) {
        data = new Object[maxLen];
    }
    public synchronized Object deQueue() {
        while (size==0) { // When executing here, Thread must have got lock and be in running mode
            // Let current Thread wait this object(to sleeping mode)
            try {
                wait(); // to sleeping mode, and release all lock
            } catch(Exception ex) {};
        }
        Object tmp = data[head];
        data[head] = null;
        head = (head+1)%data.length;
        if (size==data.length) {
            // wake up all Threads waiting this object
            notifyAll();
        }
        size--;
        return tmp;
    } // release lock
    public synchronized void enQueue(Object c) {
        while (size==data.length) {  // When executing here, Thread must have got lock and be in running mode
            // Let current thread wait this object(to sleeping mode)
            try {
                wait(); // to sleeping mode, and release all lock
            } catch(Exception ex) {};
        }
        data[tail++] = c;
        tail %= data.length;
        size++;
        if (size==1) {
            // wake up all Threads waiting this object
            notifyAll();
        }
    }
}


public class ReaderWriter extends Thread {
    public static final int READER = 1;
    public static final int WRITER = 2;
    private Queue q;
    private int mode;
    public void run() {
        for (int i=0; i < 1000; i++) {
            if (mode==READER) {
                q.deQueue();
            } else if (mode==WRITER) {
                q.enQueue(new Integer(i));
            }
        }
    }
    public ReaderWriter(Queue q, int mode) {
        this.q = q;
        this.mode = mode;
    }
    public static void main(String[] args) {
        Queue q = new Queue(5);
        ReaderWriter r1, r2, w1, w2;
        (w1 = new ReaderWriter(q, WRITER)).start();
        (w2 = new ReaderWriter(q, WRITER)).start();
        (r1 = new ReaderWriter(q, READER)).start();
        (r2 = new ReaderWriter(q, READER)).start();
        try {
            w1.join(); // wait until w1 complete
            w2.join(); // wait until w2 complete
            r1.join(); // wait until r1 complete
            r2.join(); // wait until r2 complete
        } catch(InterruptedException epp) {
        }
    }
}
```

## Multiple Reader-Writer Monitors

上一節的 Queue 資料結構，不論是 `enQueue()` 或 `deQueue()` 都會更動到 Queue 的內容。而在許多應用裡，資料結構可以允許同時多個讀一個寫。

本節舉出幾個不同的例子，說明多個 Reader-Writer 時的可能排程法。

#### Single Reader-Writer：只同時允許一個執行緒存取

```java
public class SingleReaderWriter {
    int n; // number of reader and write, 0 or 1
    public synchronized void startReading() throws InterruptedException {
        while (n != 0) {
            wait();
        }
        n = 1;
    }
    public synchronized void stopReading() {
        n = 0;
        notify();
    }
    public synchronized void startWriting() throws InterruptedException {
        while (n != 0) {
            wait();
        }
        n = 1;
    }
    public synchronized void stopWriting() {
        n = 0;
        notify();
    }
}
// 這是一個使用範例, 程式能否正確執行要靠呼叫正確的start和stop
public class WriterThread extends Thread {
    SingleReaderWriter srw;
    public WriterThread(SingleReaderWriter srw) {
        this.srw = srw;
    }
    public void run() {
        startWring();
        // insert real job here
        stopWriting();
    }
}
public class ReaderThread extends Thread {
    SingleReaderWriter srw;
    public ReaderThread(SingleReaderWriter srw) {
        this.srw = srw;
    }
    public void run() {
        startReading();
        // insert real job here
        stopReading();
    }
}
public class Test {
    public static void main(String[] argv) {
        SingleReaderWriter srw = new SingleReaderWriter;
        // create four threads
        (new WriterThread(srw)).start();
        (new WriterThread(srw)).start();
        (new ReaderThread(srw)).start();
        (new ReaderThread(srw)).start();
    }
}
```

#### Reader 優先

```java
public class ReadersPreferredMonitor {
    int nr; // The number of threads currently reading, nr > = 0
    int nw; // The number of threads currently writing, 0 or 1
    int nrtotal; // The number of threads either reading or waiting to read, nrtotal > = nr
    int nwtotal; // The number of threads either writing or waiting to write
    public synchronized void startReading() throws InterruptedException {
        nrtotal++; // 想要read的thread又多了一個
        while (nw != 0) { // 還有write thread正在write
            wait();
        }
        nr++; // 正在讀的thread多了一個
    }
    public synchronized void startWriting() throws InterruptedException {
        nwtotal++; // 想要寫的thread又多了一個
        while (nrtotal+nw != 0) { // 只要有thread想要讀,或是有thread正在寫,禮讓
            wait();
        }
        nw = 1;
    }
    public synchronized void stopReading() {
        nr--; // 正在讀的少一個
        nrtotal--; // 想要讀的少一個
        if (nrtotal == 0) { // 如果沒有要讀的,叫醒想寫的
            notify();
        }
    }
    public synchronized void stopWriting() {
        nw = 0; // 沒有thread正在寫
        nwtotal--; // 想寫的少一個
        notifyAll(); // 叫醒所有想讀和想寫的
    }
}
```

#### Writer 優先

```java
public class WritersPreferredMonitor {
    int nr; // The number of threads currently reading, nr > = 0
    int nw; // The number of threads currently writing, 0 or 1
    int nrtotal; // The number of threads either reading or waiting to read, nrtotal > = nr
    int nwtotal; // The number of threads either writing or waiting to write
    public synchronized void startReading() throws InterruptedException {
        nrtotal++; // 想要read的thread又多了一個
        while (nwtotal != 0) { // 還有thread想要write
            wait();
        }
        nr++; // 正在讀的thread多了一個
    }
    public synchronized void startWriting() throws InterruptedException {
        nwtotal++; // 想要寫的thread又多了一個
        while (nr+nw != 0) { // 有thread正在讀,或是有thread正在寫
            wait();
        }
        nw = 1;
    }
    public synchronized void stopReading() {
        nr--; // 正在讀的少一個
        nrtotal--; // 想要讀的少一個
        if (nr == 0) { // 如果沒有正在讀的,叫醒所有的(包括想寫的)
            notifyAll();
        }
    }
    public synchronized void stopWriting() {
        nw = 0; // 沒有thread正在寫
        nwtotal--; // 想寫的少一個
        notifyAll(); // 叫醒所有想讀和想寫的
    }
}
```

#### Reader 和 Writer 交互執行

```java
public class AlternatingReadersWritersMonitor {
    int[] nr = new int[2]; // The number of threads currently reading
    int thisBatch; // Index in nr of the batch of readers currently reading(0 or 1)
    int nextBatch = 1; // Index in nr of the batch of readers waitin to read(always 1-thisBatch)
    int nw; // The number of threads currently writing(0 or 1)
    int nwtotal; // The number of threads either writing or waiting to write
    public synchronized void startReading() throws InterruptedException {
        if (nwtotal == 0) { // 沒有thread要write, 將reader都放到目前要處理的這一批
            nr[thisBatch]++;
        } else {
            nr[nextBatch]++;
            int myBatch = nextBatch;
            while (thisBatch != myBatch) {
                wait();
            }
        }
    }
    public synchronized void stopReading() {
        nr[thisBatch]--;
        if (nr[thisBatch] == 0) { // 目前這批的reader都讀完了,找下一個writer
            notifyAll();
        }
    }
    public synchronized void startWriting() throws InterruptedException {
        nwtotal++;
        while (nr[thisBatch]+nw != 0) { // 目前這批還沒完,或有thread正在寫
            wait();
        }
        nw = 1;
    }
    public synchronized void stopWriting() {
        nw = 0;
        nwtotal--;
        int tmp = thisBatch; // 交換下一批要讀的
        thisBatch = nextBatch;
        nextBatch = tmp;
        notifyAll();
    }
}
```

#### 給號依序執行

```java
public class TakeANumberMonitor {
    int nr; // The number of threads currently reading
    int nextNumber; // The number to be taken by the next thread to arrive
    int nowServing; // The number of the thread to be served next
    public synchronized void startReading() throws InterruptedException {
        int myNumber = nextNumber++;
        while (nowServing != myNumber) { // 還沒輪到我
            wait();
        }
        nr++; // 多了一個Reader
        nowServing++; // 準備檢查下一個
        notifyAll();
    }
    public synchronized void startWriting() throws InterruptedException {
        int myNumber = nextNumber++;
        while (nowServing != myNumber) { // 還沒輪到我
            wait();
        }
        while (nr >  0) { // 要等所有的Reader結束
            wait();
        }
    }
    public synchronized void stopReading() {
        nr--; // 少了一個Reader
        if (nr == 0) {
            notifyAll();
        }
    }
    public synchronized void stopWriting() {
        nowServing++; // 準備檢查下一個
        notifyAll();
    }
}
```
