Java 的 IO 可分為以下幾類。  

* 處理 `byte` 的類別有：  
  * [`InputStream`](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/io/InputStream.html) 可作為 input 使用，主要的子類別有：  
    * [`ByteArrayInputStream`](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/io/ByteArrayInputStream.html) 讀入來源為 byte array  
    * [`FileInputStream`](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/io/FileInputStream.html) 讀入來源為 File  
    * [`ObjectInputStream`](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/io/ObjectInputStream.html) 將其他 InputStream 加工後，可用來讀入 Object  
    * [`DataInputStream`](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/io/DataInputStream.html) 將其他 InputStream 加工後, 可用來讀入 `int` `float` `double` `boolean` `byte` `char` 等基本資料型態  
  * [`OutputStream`](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/io/OutputStream.html) 可作為 input 使用，主要的子類別有：  
    * [`ByteArrayOutputStream`](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/io/ByteArrayOutputStream.html) 輸出到 byte array  
    * [`FileOutputStream`](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/io/FileOutputStream.html) 輸出到 File  
    * [`ObjectOutputStream`](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/io/ObjectOutputStream.html) 將其他 OutputStream 加工後，可用來輸出 Object  
    * [`DataOutputStream`](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/io/DataOutputStream.html) 將其他 OutputStream 加工後，可用來輸出 `int` `float` `double` `boolean` `byte` `char` 等基本資料型態  
    * [`PrintStream`](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/io/PrintStream.html) 可將各種基本資料型態和字串以文字形式輸出，`System.out` 就是一個 PrintStream  
  
  
  
* 處理 `char` 的類別有：  
  * [`Reader`](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/io/Reader.html) 作為 `char` 輸入，具有將輸入編碼轉為 Unicode 編碼的能力，主要的子類別有：  
    * [`BufferedReader`](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/io/BufferedReader.html) 具有緩衝功能，可讓 IO 比較有效率  
    * [`CharArrayReader`](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/io/CharArrayReader.html) 由某個字元陣列作為輸入  
    * [`InputStreamReader`](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/io/InputStreamReader.html) 作為 InputStream 轉為 Reader 的介面  
    * [`FileReader`](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/io/FileReader.html) 由檔案讀入 `char`  
  * [`Writer`](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/io/Writer.html) 作為 `char` 輸出，具有將 Unicode 編碼轉為輸出編碼的能力，主要的子類別有：  
    * [`BufferedWriter`](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/io/BufferedWriter.html) 具有緩衝功能，可讓 IO 比較有效率  
    * [`CharArrayWriter`](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/io/CharArrayWriter.html) 輸出到某個字元陣列  
    * [`OutputStreamWriter`](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/io/OutputStreamWriter.html) 作為 Writer 轉到 OutputStream 的介面  
    * [`FileWriter`](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/io/FileWriter.html) 將 `char` 輸出到檔案  
  
  
  
* 處理檔案的類別有：
  * [`File`](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/io/File.html) 具有新增刪除檔案、查詢檔案長度、查詢檔案屬性、查詢路徑等功能  
  
### 範例

```java
/**
 * Programming Methodology Homework #2 Example.
 * Author: Shiuh-Sheng Yu
 * Date:   3/16/1998
 * Last updated date: 3/17/1998
 */
import java.io.*;
import java.util.*;
public class H2 {
    private static String nextToken(BufferedReader din) throws IOException {
        StringBuffer sb = new StringBuffer();
        int c;
        try {
            do {
                c = din.read();
            } while (c==' ' || c=='\n' || c=='\r' || c=='\t');
            while (c!=-1 && c!=' ' && c!='\n' && c!='\t' && c!='\r') {
                sb.append((char)c);
                c = din.read();
            }
        } catch (EOFException e) {
        }
        if (sb.length()==0) {
            return "";
        }
        return sb.toString();
    }

    private static Matrix readMatrix(BufferedReader din) throws MalMatrixException, IOException {
        String token = nextToken(din);
        int x, y;
        double[][] data;
        if (!token.equals("matrix")) {
            throw new MalMatrixException();
        }
        token = nextToken(din);
        try {
            x = Integer.parseInt(token);
        } catch (NumberFormatException eNum) {
            throw new MalMatrixException();
        }
        token = nextToken(din);
        try {
            y = Integer.parseInt(token);
        } catch (NumberFormatException eNum) {
            throw new MalMatrixException();
        }
        data = new double[x][y];
        for (int i=0; i<x; i++) {
            for (int j=0; j<y; j++) {
                try {
                    token = nextToken(din);
                    data[i][j] = Double.valueOf(token).doubleValue();
                } catch (NumberFormatException eNum) {
                    throw new MalMatrixException();
                }
            }
        }
        return new Matrix(data);
    }

    public static void main(String[] args) {
        BufferedReader din;

        if (args.length != 1) {
            System.out.println("Usage: java H2 fname");
            System.out.println("where fname is a file contains matrix data.");
            System.exit(0);
        }
        try {
            String inputbuf;
            din = new BufferedReader(new FileReader(args[0]));
            for (;!(inputbuf=nextToken(din)).equals("");) {
                if (inputbuf.equals("+")) {
                    Matrix a = readMatrix(din);
                    Matrix b = readMatrix(din);
                    a.print();
                    a.add(b);
                    System.out.println("+");
                    b.print();
                    System.out.println("=");
                    a.print();
                } else if (inputbuf.equals("*")) {
                    Matrix a = readMatrix(din);
                    Matrix b = readMatrix(din);
                    Matrix c = a.multiply(b);
                    a.print();
                    System.out.println("*");
                    b.print();
                    System.out.println("=");
                    c.print();
                } else if (inputbuf.equals("value")) {
                    Matrix a = readMatrix(din);
                    System.out.println("value of ");
                    a.print();
                    System.out.println("=");
                    System.out.println(a.determinant());
                } else {
                    break;
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
            System.exit(1);
        }
    }
}
```
