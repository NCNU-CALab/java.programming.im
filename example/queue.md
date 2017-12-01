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
        while (size==0) {
            // Let current Thread wait this object(to sleeping mode)
            try {
                wait();
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
    }
    public synchronized void enQueue(Object c) {
        while (size==data.length) {
            // Let current thread wait this object(to sleeping mode)
            try {
                wait();
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
```