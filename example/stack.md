```java
public class Stack {
    private Object[] data;
    private int top;
    public Stack(int maxSize) {
        data = new Object[maxSize];
    }
    public void push(Object o) {
        data[top++] = o;
    }
    public Object pop() {
        return data[--top];
    }
    public Object peek() {
        return data[top-1];
    }
}
```