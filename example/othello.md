```java
/**
 * Program Name: Othello.java
 * Purpose: Showing how to AWT to write Othello
 * Since: 2005/05/23
 * Modify Date: 2005/05/24
 */
import java.awt.*;
import java.awt.event.*;
import javax.swing.*;
import java.util.*;
public class Othello extends JFrame implements ActionListener {
    private OX oxBoard;
    private MenuItem black, white;
    private Othello() {
        super("Othello");
        Menu m;
        MenuBar mb;
        add(oxBoard = new OX(this));
        CloseWindow close = new CloseWindow(this, true);
        setMenuBar(mb = new MenuBar());
        mb.add(m = new Menu("遊戲")).add(new MenuItem("新遊戲")).addActionListener(this);
        m.add(black = new MenuItem("電腦下黑方")).addActionListener(this);
        m.add(white = new MenuItem("電腦下白方")).addActionListener(this);
        m.add(new MenuItem("結束")).addActionListener(close);
        mb.add(new Menu("說明")).add(new MenuItem("關於本遊戲")).addActionListener(this);
        addWindowListener(close);
        pack();
        setResizable(false);
        setVisible(true);
    }
    public static void main(String argv[]) {
        new Othello();
    }
    // implements the ActionListener interface
    public void actionPerformed(ActionEvent e) {
        String command = e.getActionCommand();
        if (command.equals("關於本遊戲")) {
            new ErrorDialog(this,"程式設計黑白棋(蘋果花)範例.\n作者:俞旭昇於暨南大學資管系");
        } else if (command.equals("新遊戲")) {
            oxBoard.newGame();
        } else if (command.equals("ˇ電腦下黑方")) {
            oxBoard.setBlackPlayer(0);
            black.setLabel("電腦下黑方");
        } else if (command.equals("電腦下黑方")) {
            oxBoard.setBlackPlayer(1);
            black.setLabel("ˇ電腦下黑方");
        } else if (command.equals("ˇ電腦下白方")) {
            oxBoard.setWhitePlayer(0);
            white.setLabel("電腦下白方");
        } else if (command.equals("電腦下白方")) {
            oxBoard.setWhitePlayer(1);
            white.setLabel("ˇ電腦下白方");
        }
    }
}
class OX extends Component implements MouseListener, MouseMotionListener, Runnable {
    private int[] board; // 盤面狀況,表達有邊框的10*10盤面
    private int turn, diskdiff; // 現在哪方可下, 與敵方的子數差異
    private OX parent; // 由哪一個盤面變化而來
    private double val = -1000000; // 估計此盤面的優勢狀況
    private int hashval; // for hashtable
    private int[] legals; // 儲存此盤面可以下的著手
    public static final int EMPTY = 0x00; // 空格
    public static final int BLACK = 0x01; // 黑子
    public static final int WHITE = 0x02; // 白子
    public static final int STONE = 0x03; // 上面兩個 or
    public static final int BOUND = 0x04; // 邊界
    public static final int ADEMP = 0x08; // 是否鄰接子的空點
    private static final Cursor normalCursor = new Cursor(Cursor.DEFAULT_CURSOR); // 箭頭游標
    private static final Cursor hintCursor = new Cursor(Cursor.HAND_CURSOR); // 手形游標
    private static final Cursor thinkCursor = new Cursor(Cursor.WAIT_CURSOR); // 漏斗游標
    private static Dimension mySize = new Dimension(600,400); // 固定畫面的大小為寬600,高400
    private static JFrame top; // 包含此元件的最上層Frame
    private static Thread thinking; // 計算中的Thread
    private static final byte[] directions = {1,-1,10,-10,9,-9,11,-11}; // 一維陣列下的8個方向
    private static final int HASHSIZE = 63999979; // 小於64M的最大質數
    public static int whoPlayBlack, whoPlayWhite;
    public static final int HUMAN = 0, COMPUTER = 1;
    private static int newboard[] = { // 遊戲開始的最初畫面
        BOUND,BOUND,BOUND,BOUND,BOUND,BOUND,BOUND,BOUND,BOUND,BOUND,
        BOUND,EMPTY,EMPTY,EMPTY,EMPTY,EMPTY,EMPTY,EMPTY,EMPTY,BOUND,
        BOUND,EMPTY,EMPTY,EMPTY,EMPTY,EMPTY,EMPTY,EMPTY,EMPTY,BOUND,
        BOUND,EMPTY,EMPTY,ADEMP,ADEMP,ADEMP,ADEMP,EMPTY,EMPTY,BOUND,
        BOUND,EMPTY,EMPTY,ADEMP,WHITE,BLACK,ADEMP,EMPTY,EMPTY,BOUND,
        BOUND,EMPTY,EMPTY,ADEMP,BLACK,WHITE,ADEMP,EMPTY,EMPTY,BOUND,
        BOUND,EMPTY,EMPTY,ADEMP,ADEMP,ADEMP,ADEMP,EMPTY,EMPTY,BOUND,
        BOUND,EMPTY,EMPTY,EMPTY,EMPTY,EMPTY,EMPTY,EMPTY,EMPTY,BOUND,
        BOUND,EMPTY,EMPTY,EMPTY,EMPTY,EMPTY,EMPTY,EMPTY,EMPTY,BOUND,
        BOUND,BOUND,BOUND,BOUND,BOUND,BOUND,BOUND,BOUND,BOUND,BOUND};
    public OX(JFrame p) {
        addMouseListener(this);
        addMouseMotionListener(this);
        top = p;
        board = new int[100];
        System.arraycopy(newboard, 0, board, 0, 100);
        turn = BLACK;
        legals = new int[] {34,43,56,65};
    }
    // 複製p的狀態
    public OX(OX p) {
        board = new int[100];
        System.arraycopy(p.board, 0, board, 0, 100);
        turn = p.turn;
        diskdiff = p.diskdiff;
        val = -1000000;
    }
    public void setBlackPlayer(int who) {
        if (whoPlayBlack == who) return;
        if (whoPlayBlack == 0 && thinking == null && (hasLegal(turn) || hasLegal(turn^STONE))) {
            (thinking = new Thread(this)).start();
        }
        whoPlayBlack = who;
    }
    public void setWhitePlayer(int who) {
        if (whoPlayWhite == who) return;
        if (whoPlayWhite == 0 && thinking == null && (hasLegal(turn) || hasLegal(turn^STONE))) {
            (thinking = new Thread(this)).start();
        }
        whoPlayWhite = who;
    }
    // 檢查pos是否合法
    boolean isLegal(int pos) {
        return isLegal(turn, pos);
    }
    // 檢查side這個顏色,能否下在pos
    boolean isLegal(int side, int pos) {
        int opp = side^STONE;
        for (int i = 0, scan; i < 8; i++) {
            scan = pos+directions[i];
            if (board[scan] == opp) {
                    for (scan+=directions[i]; board[scan] == opp; scan+=directions[i]);
                    if ((board[scan] & side) != 0) { // 可夾住對方
                        return true;
                }
            }
        }
        return false;
    }
    // 檢查side是否有合法的著手可下
    boolean hasLegal(int side) {
        for (int i=11; i < 89; i++) {
            if ((board[i]==ADEMP) && isLegal(side, i)) {
                return true;
            }
        }
        return false;
    }
    // 下在pos,並改變盤面結構. 若pos為0, 表示此著手為pass
    boolean addMove(int pos) {
        int opp = turn^STONE;
        if (pos != 0) { // 0 表示pass
            int legal = diskdiff;
            for (int i = 0, scan; i < 8; i++) {
                scan = pos+directions[i];
                if (board[scan] == opp) { // 此方向緊鄰著敵方的子
                    // 跳過連續的敵方子
                    for (scan += directions[i]; board[scan] == opp; scan+=directions[i]);
                    if (board[scan] == turn) { // 可夾住對方
                        // 將所有敵方子變成我方子
                        for (int c = pos+directions[i]; c!=scan ;board[c]=turn, c+=directions[i], diskdiff+=2);
                    }
                }
            }
            if (diskdiff==legal) { // 如果都沒有吃到
                return false;
            }
            diskdiff++;
            board[pos] = turn;
            for (int i = 0; i < 8; i++) { // 設定此點旁的空點為ADEMP
                if (board[pos+directions[i]] == EMPTY) {
                    board[pos+directions[i]] = ADEMP;
                }
            }
        }
        turn = opp; // 換對方下了
        diskdiff = -diskdiff;
        hashval=(hashval*64+(pos-11))%HASHSIZE;
        return true;
    }
    // Thread的進入點
    public void run() {
        setCursor(thinkCursor);
        for (;;) { // 當敵方需pass時,我方一直下
            if (turn==BLACK && whoPlayBlack == HUMAN) { // 先檢查是否改由人下
                break;
            }
            if (turn==WHITE && whoPlayWhite == HUMAN) { // 先檢查是否改由人下
                break;
            }
            addMove(best());
            repaint(); // ask winder manager to call paint() in another thread
            if (turn==BLACK && whoPlayBlack==HUMAN && hasLegal(turn)) { // 人可以下了
                break;
            }
            if (turn==WHITE && whoPlayWhite==HUMAN && hasLegal(turn)) { // 人可以下了
                break;
            }
            if (!hasLegal(turn) && !hasLegal(turn^STONE)) { // 對手和自己都不能下了
                new ErrorDialog(top, "Game Over");
                break;
            }
            if (!hasLegal(turn)) {
                addMove(0);
            }
        }
        setCursor(normalCursor);
        thinking = null;
    }
    // The following 2 methods implement the MouseMotionListener interface
    public void mouseDragged(MouseEvent e) {}
    public void mouseMoved(MouseEvent e) {
        if (thinking != null) return;
        int row = e.getY()/40;
        int col = e.getX()/40;
        if (row >= 8 || col >= 8) {
            setCursor(normalCursor);
            return; // 超過邊界
        }
        int pos = row*10 + col + 11;
        if (board[pos]==ADEMP && isLegal(turn, pos)) {
            setCursor(hintCursor);
        } else {
            setCursor(normalCursor);
        }
    }
    // The following 5 methods implement the MouseListener interface
    public void mouseClicked(MouseEvent e) {}
    public void mouseEntered(MouseEvent e) {}
    public void mouseExited(MouseEvent e) {}
    public void mouseReleased(MouseEvent e) {}
    public void mousePressed(MouseEvent e) {
        int row = e.getY()/40;
        int col = e.getX()/40;
        if (row >= 8 || col >= 8) return; // 超過邊界
        if (thinking != null) return; // 電腦思考中
        int pos = row*10+col+11;
        if (board[pos] == ADEMP && addMove(pos)) { // 此位置可以下
            repaint();
            if (hasLegal(turn)) {
                if ((turn==WHITE && whoPlayWhite==COMPUTER) || (turn==BLACK && whoPlayBlack==COMPUTER)) { // let computer play
                    (thinking = new Thread(this)).start();
                }
            } else {
                if (!hasLegal(turn^STONE)) { // 雙方都不能下
                    new ErrorDialog(top, "Game Over");
                    return;
                }
                addMove(0); // 對方不能下,force pass
            }
        }
    }
    // 棋力強弱關鍵的求值函數
    private void eval() {
        val = diskdiff;
    }
    private void alphaBeta(int level) {
        if (legals == null) {
            findLegals();
        }
        for (int i=0; i<legals.length; i++) {
            OX tmp = new OX(this);
            tmp.addMove(legals[i]);
            if (level<1) {
                tmp.eval();
            } else {
                tmp.alphaBeta(level-1);
            }
            // alphaBeta cut
            if (val < -tmp.val) {
                val = -tmp.val;
                for (OX p = parent; p != null;) {
                    if (val >= -p.val) { // 對手不會選擇這條路的
                        return;
                    }
                    // 往上跳兩層
                    p = p.parent;
                    if (p != null) p = p.parent;
                }
            }
        }
    }
    private void findLegals() {
        int count = 0;
        int[] tmp = new int[60];
        for (int i=11; i<89; i++) {
            if (board[i]==ADEMP && isLegal(turn, i)) {
                tmp[count++] = i;
            }
        }
        legals = new int[count];
        System.arraycopy(tmp, 0, legals, 0, count);
    }
    private int best() {
        int bestMove = 0;
        findLegals();
        val = -100000000;
        for (int i=0; i<legals.length; i++) {
            OX tmp = new OX(this);
            tmp.addMove(legals[i]);
            tmp.alphaBeta(3);
            if (-tmp.val > val) {
                bestMove = legals[i];
                val = -tmp.val;
            }
        }
        return bestMove;
    }
    // override paint() defined in Component
    public void paint(Graphics g) {
        int black, white;
        black = white = 0;
        for (int i = 0; i <= 8; i++) {  // draw grids
            g.drawLine(0, i*40, 320, i*40);
            g.drawLine(i*40, 0, i*40, 320);
        }
        for (int i = 0; i < 8; i++) {
            for (int j = 0; j < 8; j++) {
                int pos = i*10 + j + 11;
                if ((board[pos] & BLACK) != 0) {
                    g.fillOval(j*40,i*40,40,40);
                    black++;
                } else if ((board[pos] & WHITE) != 0) {
                    g.drawOval(j*40,i*40,40,40);
                    white++;
                }
            }
        }
        g.drawString("BLACK:"+black, 400, 100);
        g.drawString( "WHITE:"+white, 400, 150);
    }
    public void newGame() {
        System.arraycopy(newboard, 0, board, 0, 100);
        turn = BLACK;
        hashval = diskdiff = 0;
        if (thinking != null) {
            try {
                thinking.join();
            } catch(Exception epp) {}
        }
        if (whoPlayBlack == COMPUTER) {
            (thinking = new Thread(this)).start();
        }
        repaint();
    }
    // override getPreferredSize defined in java.lang.Component,
    // so that the Component has proper size on screen
    public Dimension getPreferredSize() {
        return mySize;
    }
    // override hashCode() in java.lang.Object
    public int hashCode() {
        return hashval;
    }
    public boolean equals(Object o) {
        if (!(o instanceof OX)) return false;
        OX t = (OX) o;
        for (int i=11; i<89; i++) {
            if (board[i] != t.board[i]) return false;
        }
        return true;
    }
}
// WindowAdapter implements the WindowLister interface
// We extends WindowAdapter to reduce the line numer of code
class CloseWindow extends WindowAdapter implements ActionListener {
    private Window target;
    private boolean exit;
    public CloseWindow(Window target, boolean exit) {
        this.target = target;
        this.exit = exit;
    }
    public CloseWindow(Window target) {
        this.target = target;
    }
    public void windowClosing(WindowEvent e) {
        target.dispose();
        if (exit) System.exit(0);
    }
    public void actionPerformed(ActionEvent e) {
        target.dispose();
        if (exit) System.exit(0);
    }
}
class AddConstraint {
    public static void addConstraint(Container container, Component component,
          int grid_x, int grid_y, int grid_width, int grid_height,
          int fill, int anchor, double weight_x, double weight_y,
          int top, int left, int bottom, int right) {
        GridBagConstraints c = new GridBagConstraints();
        c.gridx = grid_x; c.gridy = grid_y;
        c.gridwidth = grid_width; c.gridheight = grid_height;
        c.fill = fill; c.anchor = anchor;
        c.weightx = weight_x; c.weighty = weight_y;
        c.insets = new Insets(top,left,bottom,right);
        ((GridBagLayout)container.getLayout()).setConstraints(component,c);
        container.add(component);
    }
}
class ErrorDialog extends JDialog {
    public ErrorDialog(JFrame parent, String all[]) {
        this(parent, all, null);
    }
    public ErrorDialog(JFrame parent, String all[], String msg) {
        super(parent,"",true);
        StringBuffer sb = new StringBuffer();
        for (int i=0; i<all.length; i++) {
            sb.append(all[i]);
            sb.append('\n');
        }
        if (msg!=null) {
            sb.append(msg);
        }
        setup(parent, sb.toString());
    }
    public ErrorDialog(JFrame parent, String message) {
        super(parent,"",true);
        setup(parent, message);
    }
    private void setup(JFrame parent, String message) {
        this.getContentPane().setLayout(new GridBagLayout());
        int row=0, col=0, i, width=0;
        Font font = new Font("Serif", Font.PLAIN, 16);
        char c=' ';
        for (i=0; i<message.length(); i++) {
            c = message.charAt(i);
            if (c=='\n') {
               row++;
               if (width>col) {
                   col = width;
               }
               width=0;
            } else if (c=='\t') {
                width += 7-width%7;
            } else {
                if (c>0x00FF) {
                    width+=2;
                } else {
                    width++;
                }
            }
        }
        if (c!='\n') {
           row++;
           if (width>col) {
               col = width;
           }
        }
        col++;
        // 希望視窗出來不要太大或太小
        row = (row>24) ? 24 : row;
        if (row<5) {
            row=5;
        }
        if (col<20) {
            col = 20;
        }
        TextArea tx = new TextArea(message,row,col);
        tx.setEditable(false);
        tx.setFont(font);
        AddConstraint.addConstraint(this.getContentPane(), tx, 0, 0, 1, 1,
            GridBagConstraints.BOTH,
            GridBagConstraints.NORTHWEST,
            1,1,0,0,0,0);
        Button b = new Button("確定");
        b.setFont(font);
        AddConstraint.addConstraint(this.getContentPane(), b, 0, 1, 1, 1,
            GridBagConstraints.HORIZONTAL,
            GridBagConstraints.CENTER,
            1,0,0,0,0,0);
        CloseWindow cw = new CloseWindow(this);
        this.addWindowListener(cw);
        b.addActionListener(cw);
        pack();
        setVisible(true);
    }
}
```
