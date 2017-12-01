```java
public class Bingo {
    private int[] board; // using one dimensional array to represent the board
    private int[][] lines; // there are (2*size+2) lines
    private int size;
    public Bingo(int size) {
        this.size = size;
        genMap();
        genLine();
    }
    private void genMap() {
        int len = size*size;
        board = new int[len];
        for (int i = 0; i < len; i++) {
            board[i] = i+1;
        }
        // random swap each position 4 times
        for (int i = 0; i < len*4; i++) {
            // try to swap x and y
            int x = i % len;
            int y = (int)(Math.random()*len);
            int tmp = board[x];
            board[x] = board[y];
            board[y] = tmp;
        }
    }
    public void print() {
        int len = size * size;
        for (int i = 0; i < len; i++) {
        	  if (board[i] < 10) System.out.print(" ");
            System.out.print(" "+board[i]);
            if (i % size == size-1) {
                System.out.println();
            }
        }
        System.out.println();
    }
    public int choose() {
        int len = size*size;
        int[] weight = new int[len];
        for (int i = 0; i < 2*size+2; i++) {
            int degree = 0;
            for (int j = 0; j < size; j++) {
                if (board[lines[i][j]] <= 0) {
                    degree++;
                }
            }
            // already know the degree of line i
            int delta = degree*degree*degree;
            for (int j = 0; j < size; j++) {
                if (board[lines[i][j]] > 0) {
                    weight[lines[i][j]] += delta;
                }
            }
        }
        int start = (int)(Math.random()*len);
        int at;
        int maxWeight = 0;
        int pos = len/2;
        for (int i = 0; i < len; i++) {
            at = (start+i)%len;
            if (board[at] > 0 && maxWeight < weight[at]) {
                maxWeight = weight[at];
                pos = at;
            }
        }
        return board[pos];
    }
    private void genLines() {
        lines = new int[2*size+2][size];
        for (int i = 0; i < size; i++) { // horizontal lines
            for (int j = 0; j < size; j++) {
                lines[i][j] = i*size + j;
            }
        }
        for (int i = 0; i < size; i++) { // vertical lines
            for (int j = 0; j < size; j++) {
                lines[i+size][j] = i + size*j;
            }
        }
        for (int j = 0; j < size; j++) { // \
            lines[2*size][j] = (size+1) * j;
            lines[2*size+1][j] = size-1+j*(size-1);
        }
    }
    public boolean bingo() {
        int i, j;
        int count = 0;
        for (i = 0; i < 2*size+2; i++) { // check each line
            for (j = 0; j < size; j++) {
                if (board[lines[i][j]] > 0) {
                    break;
                }
            }
            if (j == size) { // found aline
                count++;
            }
        }
        if (count >= size) return true;
        return false;
    }
    public boolean setChecked(int num) {
        int pos;
        int len = size*size;
        for (pos = 0; pos < len; pos++) {
            if (board[pos] == num) {
                break;
            }
        }
        if (pos == len) { // error!!!
            print();
            System.out.println("error set +"+pos);
            System.exit(1);
        }
        board[pos] = -board[pos];
        return bingo();
    }
    public static void main(String[] argv) {
        int size = 5;
        if (argv.length > 0) {
            try {
                size = Integer.parseInt(argv[0]);
            } catch(NumberFormatException nfe) {
                System.out.println("Warning: Command line argument must be an integer");
            }
        }
        Bingo x = new Bingo(size);
        x.print();
        Bingo y = new Bingo(size);
        y.print();
        boolean exit = false;
        int count = 0;
        while (!exit) {
            int num = x.choose();
            System.out.println("x choose "+num);
            if (x.setChecked(num)) {
                System.out.println("X bingo");
                exit = true;
            }
            if (y.setChecked(num)) {
                System.out.println("Y bingo");
                exit = true;
            }
            count ++;
            if (!exit) {// y's turn
                num = y.choose();
                System.out.println("y choose "+num);
                if (x.setChecked(num)) {
                    System.out.println("X bingo");
                    exit = true;
                }
                if (y.setChecked(num)) {
                    System.out.println("Y bingo");
                    exit = true;
                }
                count ++;
            }
        }
        System.out.println(count+" numbers were choosed");
    }
}
```