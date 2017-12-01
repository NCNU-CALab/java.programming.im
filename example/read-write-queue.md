```java
public class ReaderWriter extends Thread {
    public static final int READER = 1;
    public static final int WRITER = 2;
    private Queue q;
    private int mode;
    private String id;
    public void run() {
        for (int i=0; i < 1000; i++) {
            if (mode==READER) {
                q.deQueue();
            } else if (mode==WRITER) {
                q.enQueue(new Integer(i));
            }
        }
    }
    public ReaderWriter(String id, Queue q, int mode) {
        this.id = id;
        this.q = q;
        this.mode = mode;
    }
    public static void main(String[] args) {
        Queue q = new Queue(5);
        ReaderWriter r1, r2, w1, w2;
        (w1 = new ReaderWriter("W1", q, WRITER)).start();
        (w2 = new ReaderWriter("W2", q, WRITER)).start();
        (r1 = new ReaderWriter("R1", q, READER)).start();
        (r2 = new ReaderWriter("R2", q, READER)).start();
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