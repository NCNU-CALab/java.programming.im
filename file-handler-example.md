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
