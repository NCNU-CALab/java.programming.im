過河問題是數學上頗具趣味的題目，有夫妻過河、商人過河、農夫過河、人狗雞米過河問題。  

夫妻過河問題是古老的阿拉伯智力題，敘述有三對夫妻要過河，小船只能載兩人，妻子不能在丈夫不在場時和別的異性渡河。  

這類的數學模型稱為狀態轉移模型。把 M 個丈夫、F 個妻子視為一種狀態，以 (M,F) 表示。其中 M、F 都是不大於三的正整數。  

![equation 1](https://github.com/NCNU-CALab/java.programming.im/raw/master/example/pic/17-1.gif)  

並非所有狀態都是被允許的，例如 (1,2) 就不被允許，因為妻子 2 在丈夫 2 不在場時與異性 1 相處。把允許的狀態整理如下：  

S = { (0,0), (0,1), (0,2), (0,3), (1,1), (2,2), (3,3), (3,2), (3,1), (3,0), (2,1) }  

有了允許狀態集合後，還要知道轉換的過程，如何從都在此岸的 (3,3) 轉換成都在彼岸的 (0,0)。  

每一次小船載人渡河都視為一次運算，第 n 次運算 d<sub>n</sub> 由第 n + 1 個狀態 S<sub>n</sub> + 1 減去第 n 個狀態 S<sub>n</sub> 決定。  

d<sub>n</sub> = S<sub>n</sub> + 1 - S<sub>n</sub>  

小船由此岸駛向彼岸時，此岸人數變少，當船駛回此岸，人數又變多，又因為小船不能載超過兩人，故 d<sub>n</sub> 必然滿足下式：  

![equation 2](https://github.com/NCNU-CALab/java.programming.im/raw/master/example/pic/17-2.gif)  

其中 p 和 q 分別表示小船離岸時船上的丈夫、妻子人數，當 S<sub>n</sub> + 1 和 S<sub>n</sub> 都是允許狀態時，S<sub>n</sub> + 1 = S<sub>n</sub> + d<sub>n</sub> 稱為狀態轉移方程。接下來是求解： (3,3) + d<sub>1</sub> + d<sub>2</sub> + … + d<sub>m</sub> = (0,0)  

以下是夫妻過河問題的一條路徑：  

![pic 1](https://github.com/NCNU-CALab/java.programming.im/raw/master/example/pic/17-3.gif)  

當討論的變量不很多時，也常用作圖的方法解決狀態轉移問題，在 M - F 平面上標出允許狀態 S 的點，將允許狀態運算看成沿方格移動一格或兩格。  

以實線標記小船由此岸到彼岸、虛線是由彼岸至此岸，下圖是夫妻過河的圖解法：  

![pic 2](https://github.com/NCNU-CALab/java.programming.im/raw/master/example/pic/17-4.gif)  

以下是用 java 寫的野蠻人渡河問題，仔細審題後，會發現題目的限制是相同的。

```java
import java.util.*;
public class MB {
    private byte boatAt; // 0表示在河左岸,1表示在河右岸
    private byte mat[], bat[]; // 兩岸的傳教士和野人數量,最多127人
    private short people; // 傳教士+野人總人數
    public MB(int Msize, int Bsize) {
        if (Msize>127 || Bsize>127) {
            System.out.println("Current implementation support size <= 127");
            System.exit(0);
        }
        mat = new byte[2];
        bat = new byte[2];
        mat[0] = (byte)Msize;
        bat[0] = (byte)Bsize;
        this.people = (byte)(Msize+Bsize); // 一開始所有的人都在左岸
    }
    public int getKey() {
        return (boatAt<<14)+(mat[0]<<7)+bat[0]; // 左岸傳教士和野人數目,以及船在哪一邊三個參數可以唯一決定盤面
    }
    public boolean isSolution() {
        return (mat[1]+bat[1])==people; // 所有人都到右岸就是解了
    }
    public boolean isValid() {
        // 人數必須>=0,且任何一邊傳教士人數不得少於野人
        return (mat[0]>=0 && mat[1]>=0 && bat[0]>=0 && bat[1]>=0 && (mat[0]==0 || mat[0]>=bat[0]) && (mat[1]==0 || mat[1]>=bat[1]));
    }
    public String getSide() { // 傳回此狀態船可以走的方向
        return (boatAt==0) ? "往左邊" : "往右邊";
    }
    public void move(int m, int b) { // 移動m個傳教士,b個野人
        byte otherSide = (boatAt==1) ? (byte)0 : (byte)1;
        mat[boatAt] -= m;
        mat[otherSide] += m;
        bat[boatAt] -= b;
        bat[otherSide] += b;
        boatAt = otherSide;
    }
    private static final byte[][] branches = {{1,0},{0,1},{2,0},{0,2},{1,1}};
    private static final String[] display = {"一個傳教士","一個野蠻人","兩個傳教士","兩個野蠻人","一個傳教士和一個野蠻人"};
    private static int numSol;
    private static boolean[] visited;
    private static Vector history; // steps runs to this situation
    private void expand() {
        if (isSolution()) { // 如果目前的盤面是一組解
            numSol++;
            return;
        }
        for (int i=0; i < branches.length; i++) { // 展開五種可能性
            move(branches[i][0], branches[i][1]); // 更改盤面成新的狀態
            // 如果此盤面符合規則,而且沒有走過
            if (isValid() && visited[getKey()]==false) {
                visited[getKey()] = true; // 紀錄來過此盤面
                history.add(display[i]+getSide()); // 紀錄此步驟
                expand(); // recursive call to expand all possibles
                history.remove(history.size()-1); // 移除此步驟
                visited[getKey()] = false; // 移除來過此盤面
            }
            move(branches[i][0], branches[i][1]); // 改為原來的狀態
        }
    }
    public static void findSolutions(int Msize, int Bsize) {
        // 設定需要的資料結構
        numSol = 0;
        visited = new boolean[32786]; // 走過的盤面
        MB init = new MB(Msize, Bsize); // 初始盤面
        visited[init.getKey()] = true; // 目前的盤面已經來過了
        history = new Vector();
        init.expand();
        System.out.println(numSol+" solutions found");
    }
    public static void main(String[] argv) throws Exception {
        int Msize = 3;
        int Bsize = 3;
        System.out.println("Usage java MB Msize Bsize.\nThe default size is 3");
        if (argv.length > 0) {
            Msize = Integer.parseInt(argv[0]);
        }
        if (argv.length > 1) {
            Bsize = Integer.parseInt(argv[1]);
        }
        findSolutions(Msize, Bsize);
    }
}
```
