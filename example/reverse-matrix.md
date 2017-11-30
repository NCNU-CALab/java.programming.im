gnu.gleem.linalg  
### Class NonSquareMatrixException  
java.lang.Object  
&emsp;|  
&emsp;+-- java.lang.Throwable  
&emsp;&emsp;&emsp;|  
&emsp;&emsp;&emsp;+-- java.lang.Exception  
&emsp;&emsp;&emsp;&emsp;&emsp;|  
&emsp;&emsp;&emsp;&emsp;&emsp;+-- java.lang.RuntimeException  
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;|  
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;+-- <b>gnu.gleem.linalg.NonSquareMatrixException  </b>
                    
All Implemented Interfaces: 
java.io.Serializable 

Download package [gnu.gleem.linalg.*]()

```java
/**
 * FileName: Matrix.java
 * Author:   Shiuh-Sheng Yu
 * Date:     4/13/1998
 * Last Update: 4/13/2003
 */
public class Matrix {
    private double[][] data;
    public Matrix(double[][] array) {
        data = array;
    }
    /**
     * 高斯消去法求反矩陣
     */
    public Matrix inverse() throws NonSquareMatrixException {
        int n = data.length;
        if (n != data[0].length) {
            throw new NonSquareMatrixException();
        }
        double factor, tmp;
        // 建立兩陣列,左邊同原資料,右邊為單位矩陣
        double[][] rel = new double[n][n];
        double[][] left = new double[n][n];
        for (int row=0; row<n; row++) {
            for (int col=0; col<n; col++) {
                left[row][col] = data[row][col];
                if (row==col) {
                    rel[row][col]=1;
                } else {
                    rel[row][col]=0;
                }
            }
        }
        /* 消去左下角 */
        for (int col=0; col<n; col++) { // 由左而右
            int nonzero;
            for (nonzero=col; nonzero<n && left[nonzero][col]==0; nonzero++) ; // 找非0之元素
            if (nonzero==n) { // 反矩陣不存在
                return null;
            }
            if (nonzero != col) { // 需要列交換,記得要乘-1
                for (int each=0; each<n; each++) {
                    tmp = left[col][each];
                    left[col][each] = left[nonzero][each];
                    left[nonzero][each] = -tmp;
                    tmp = rel[col][each];
                    rel[col][each] = rel[nonzero][each];
                    rel[nonzero][each] = -tmp;
                }
            }
            for (int row=col+1;row<n;row++) { // 由上而下
                if (left[row][col] != 0) { // 需要做列運算
                    factor = left[row][col]/left[col][col];
                    for (int each=0; each<n; each++) { // 由左而右做列運算
                        left[row][each] -= left[col][each]*factor;
                        rel[row][each] -= rel[col][each]*factor;
                    }
                }
            }
        }
        /* 消去右上角, 此時對角線上不可能是0 */
        for (int col=n-1; col>0; col--) { // 由右而左
            for (int row=col-1; row>=0; row--) { // 由下而上
                if (left[row][col] != 0) {
                    factor = left[row][col]/left[col][col];
                    for (int each=0; each<n; each++) { // 由左而右做列運算
                        // 左邊陣列的右上角不必管了
                        rel[row][each] -= rel[col][each]*factor;
                    }
                }
            }
        }
        /* 將只剩對角線的左邊矩陣變成單位矩陣 */
        for (int row=0; row<n; row++) {
            for (int col=0; col<n; col++) {
                rel[row][col] /= left[row][row];
            }
        }
        return new Matrix(rel);
    }

    /**
     * 高斯消去法求值
     */
    public double value() throws NonSquareMatrixException {
        int n = data.length;
        if (n != data[0].length) {
            throw new NonSquareMatrixException();
        }
        double tmp, factor, result = 1.0;
        // 複製原矩陣
        double[][] right = new double[n][n];
        for (int row=0; row<n; row++) {
            for (int col=0; col<n; col++) {
                right[row][col] = data[row][col];
            }
        }
        for (int col=0; col<n-1; col++) { // 由左而右
            int nonzero;
            for (nonzero=col; nonzero<n && right[nonzero][col]==0; nonzero++) ; // 找非0之元素
            if (nonzero==n) { // 矩陣值為0
                return 0;
            }
            if (nonzero != col) { // 列交換
                result *= -1.0;
                for (int each=0; each<n; each++) {
                    tmp = right[col][each];
                    right[col][each] = right[nonzero][each];
                    right[nonzero][each] = tmp;
                }
            }
            for (int row=col+1; row<n; row++) { // 由上而下
                if (right[row][col] != 0) { // 需要列運算
                    factor = right[row][col]/right[col][col];
                    right[row][col]=0;
                    // col左邊的怎麼算都是0,不用管他
                    for (int each=col+1; each<n; each++) {
                        right[row][each] -= right[col][each]*factor;
                    }
                }
            }
        }
        // 對角線上的連乘績即為行列式值
        result *= right[0][0];
        for (int i=1; i<n; i++) {
            result*=right[i][i];
        }
        return result;
    }
}
```
