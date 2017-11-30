```java
/**
 * FileName: Matrix.java
 * Author:   Shiuh-Sheng Yu
 * Since:     1998/3/17
 * Modify Date: 2005/07/11, add more comments
 */
public class Matrix {
    private double[][] data;

    /* constructor */
    public Matrix(double[][] array) {
        data = array;
    }
    
    /* constructor */
    private Matrix(int x, int y) {
        data = new double[x][y];
    }

    /*
     *  複製物件
     */
    public Object clone() {
        double[][] x = new double[data.length][data[0].length];
        for (int i=0; i < x.length; i++) {
            for (int j=0; j < x[0].length; j++) {
                x[i][j] = data[i][j];
            }
        }
        return new Matrix(x);
    }

    /**
     * 矩陣加法
     * X is m by n, Y is m by n, Z = X + Y is m by n
     * Zij = Xij + Yij
     */
    public Matrix add(Matrix x) throws MalMatrixException {
        /* 判斷兩矩陣是否同樣大小 */
        if (data.length != x.data.length || data[0].length != x.data[0].length) {
            throw new MalMatrixException(); // 丟出例外
        }
        Matrix rel = (Matrix)this.clone(); // 先將this物件複製到rel
        for (int i=0; i < data.length; i++) {
            for (int j=0; j < data[0].length; j++) {
                rel.data[i][j] += x.data[i][j];
            }
        }
        return rel;
    }
    /**
     * 矩陣相乘
     * X is m by n, Y is n by p
     * result Z is m by p where for each 0<= i < m, 0 <= j < p
     * Zij = Sum Xik*Ykj
     *       k=0 to n-1
     */
    public Matrix multiply(Matrix x) throws MalMatrixException {
        if (data[0].length!=x.data.length) {
            throw new MalMatrixException();
        }
        Matrix rel = new Matrix(data.length,x.data[0].length);
        for (int i=0; i < data.length; i++) {
            for (int j=0; j < x.data[0].length; j++) {
                double tmp = 0;
                for (int k=0; k < data[0].length; k++) {
                    tmp += data[i][k] * x.data[k][j];
                }
                rel.data[i][j]= tmp;
            }
        }
        return rel;
    }

    /**
     * 將矩陣內容印到螢幕上
     */
    public void print() {
        for (int i=0; i < data.length; i++) {
            for (int j=0; j < data[0].length; j++) {
                System.out.print(data[i][j]+" ");
            }
            System.out.println("");
        }
    }

    /**
     * 高斯消去法求行列式值
     */
    public double determinant() throws NonSquareMatrixException {
        if (data.length != data[0].length) {
            throw new NonSquareMatrixException();
        }
        int n = data.length;
        int i, j, k, l;
        double sign = 1;
        double tmp, factor, result;
        double[][] right = new double[n][n];
        // 將資料copy到另一個陣列
        for (i=0; i < n; i++) {
            for (j=0; j < n; j++) {
                right[i][j] = data[i][j];
            }
        }
        for (i=0; i < n-1; i++) { // 由左而右
            for (k=i; k < n; k++) { // 找對角線以下非0之元素
                if (right[k][i]!=0) {
                    break;
                }
            }
            if (k==n) { // 找不到非0的元素
                return 0;
            }
            if (k != i) { // 對角線上是0, 需做列交換
                for (l=i; l < n; l++) {
                    tmp = right[i][l];
                    right[i][l] = right[k][l];
                    right[k][l] = tmp;
                }
                sign *= -1;  // 列交換需乘-1
            }
            for (k=i+1;k < n;k++) { // 由上而下
                if (right[k][i] != 0) { // 列運算. 若right[k][i]是0的話就不用再算了
                    factor = right[k][i]/right[i][i];
                    right[k][i]=0;
                    for (j=i+1; j < n; j++) {
                        right[k][j] -= right[i][j]*factor;
                    }
                }
            }
        }
        for (i=1, result=right[0][0]; i < n; i++) {
            result*=right[i][i];
        }
        return sign*result;
    }
}

MalMatrix.java如下

public class MalMatrixException extends Exception {
    MalMatrixException() {
        super("MalMatrixException:");
    }
}

NonSquareMatrixException.java如下

public class NonSquareMatrixException extends Exception {
    NonSquareMatrixException() {
        super("NonSquareMatrixException:");
    }
}
```
