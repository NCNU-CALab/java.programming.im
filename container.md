對絕大部分應用來說，資料量的大小，必須等到執行期間才知道。

由於 Java 的陣列一但產生後，就沒有辦法變更其大小，因此 JDK 在 `java.util` 裡面提供了相關的容器類別，比較常用的類別有：

* [Vector](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/util/Vector.html)：可視為長度沒有限制的陣列
* [Hashtable](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/util/Hashtable.html)：利用 get 和 put 函數把 (key, val) 放進該容器內
* [Enumeration](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/util/Enumeration.html)：可利用此類別將 Hashtable 內的資料逐一取出
* [Stack](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/util/Stack.html) with `push()` `pop()` methods

### 範例程式

```java
import java.util.*;
public class VectorExample {
    public static void main(String[] argv) {
        Vector v = new Vector();
        for (int i = 0; i < 100; i++) {
            v.add(new Integer(i)); // v.add(index, element)
        }
        for (int i = 0; i < v.size(); i++) {
            Integer x = v.get(i);
        }
        Integer[] y = (Integer[])v.toArray();
        for (int i = v.size()-1; i >= 0; i--) {
            v.remove(i);
        }
    }
    public static void fun() {
        Vector v = new Vector();
        for (int i = 0; i < 100; i++) {
            v.add(new Integer(i));
        }
        for (int i = 0; i < v.size(); i++) {
            Integer x = (Integer)v.get(i);
        }
        Integer[] y = (Integer[])v.toArray();
        for (int i = v.size()-1; i >= 0; i--) {
            v.remove(i);
        }
        for (int i = 0; i < 100; i++) {
            v.add(new Float(i));
        }
        for (int i = 0; i < v.size(); i++) {
            Float x = (Float)v.get(i);
        }
        Float[] z = (Float[])v.toArray();
        for (int i = v.size()-1; i >= 0; i--) {
            v.remove(i);
        }
    }
}

import java.util.*;
public class Example3 {
    public static void main(String[] argv) {
        Vector v = new Vector();
        for (int i = 0; i < 10; i++) {
            v.add(Integer.toString(i));
        }
        for (int i = 0; i < v.size(); i++) {
            String x = (String)v.get(i);
            System.out.println(x);
        }
        String[] tmp = new String[v.size()];
        v.copyInto(tmp);
        for (int i = 0; i < tmp.length; i++) {
            System.out.println(tmp[i]);
        }
        Hashtable ht = new Hashtable();
        for (int i = 0; i < 10; i++) {
            ht.put(new Integer(i), Integer.toString(i));
        }
        for (int i = 0; i < 10; i++) {
            String x = (String)ht.get(new Integer(i));
            System.out.println(x);
        }
        Enumeration enu = ht.elements();
        while (enu.hasMoreElements()) {
            String x = (String)enu.nextElement();
            System.out.println(x);
        }
    }
}
```
