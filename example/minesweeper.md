```java
/*
 * 踩地雷 
 * FileName: Mine2.java
 * Author:   Shiuh-Sheng Yu
 * Date:     5/19/1999
 * Last Update: 5/05/2002
 */
import java.awt.*;
import java.awt.event.*;
import java.util.Random;
public class Mine2 extends Component implements MouseListener, ActionListener {
    private byte[] map; // 以一維陣列來取代二維,以減少物件的使用
    private int x, y;
    private Frame f;
    private int[] directions;
    // 前四個bits用來記錄此格的資料, 後面四個bits用來記錄旁邊的地雷數
    private static final byte BOMB = 0x10; // 用以紀錄本格有地雷
    private static final byte VISIT = 0x20; // 用以紀錄已按左鍵
    private static final byte MARK = 0x40; //  用以紀錄已按右鍵
    private static final byte NEB = 0x0f; // 用以取出旁邊的地雷數
    void checkWin() {
        int width = x + 2;
        for (int i = 1; i <= y; i++) { // 由上而下
            int tmp = i * width;
            for (int j = 1; j <= x; j++) { // 由左而右
                int tt = tmp + j;
                if ((((map[tt] & MARK) > 0) && ((map[tt] & BOMB) == 0)) || (((map[tt] & MARK) == 0) && ((map[tt] & BOMB) > 0))) {
                    return;
                }
            }
        }
        showAll();
        showWin();
    }
    void showAll() {
        int width = x + 2;
        for (int i = 1; i <= y; i++) { // 由上而下
            int tmp = i*width;
            for (int j=1; j<=y; j++) { // 由左而右
                int tt = tmp + j;
                if ((map[tt] & BOMB) != 0) { // 畫上標示
                    map[tt] |= MARK;
                } else {
                    map[tt] = (byte)((map[tt] | VISIT) & ~MARK); // 設為按過, 再把標示拿掉
                }
            }
        }
        repaint();
    }
    void showWin() {
        Dialog d;
        (d = new Dialog(f, true)).addWindowListener(new CloseWindow(d));
        ((Button)d.add(new Button("恭喜過關"))).addActionListener(new CloseWindow(d));
        d.pack();
        d.show();
    }
    public Dimension getPreferredSize() {
        return new Dimension(22*x,22*y);
    }
    void reset() {
        map = new byte[(x+2)*(y+2)];
        int width = x + 2;
        for (int i = 0; i <= y + 1; i++) { // 由上而下
            int tmp = i * width;
            for (int j = 0; j <= x + 1; j++) { // 由左而右
               int tt = tmp + j;
               if (i==0 || i == (x + 1) || j == 0 || j == (y + 1)) {
                   map[tt] = VISIT; // boundary
               } else {
                   map[tt] = 0; // normal position
               }
            }
        }
        // 15%為地雷
        for (int i = 0, bombNumber = (int)(x*y*0.15); i < bombNumber;) { // setup bombs
            int row = (int)(Math.random()*y)+1;
            int col = (int)(Math.random()*x)+1;
            int tt = row * width + col;
            if ((map[tt] & BOMB) == 0) { // has no bomb yet
                i++;
                map[tt] |= BOMB;
            }
        }
        // 計算旁邊的地雷數
        for (int i = 1; i <= y; i++) { // 由上而下
            int tmp = i * width;
            for (int j = 1; j <= x; j++) { // 由左而右
                int tt = tmp + j;
                for (int k = 0; k < 8; k++) {
                    if ((map[tt + directions[k]] & BOMB) != 0) { // 旁有地雷
                        map[tt]++; // 後面四個bit用來記錄旁邊的地雷數
                    }
                }
            }
        }
        repaint();
    }
    // 畫出盤面
    public void paint(Graphics g) {
        g.setColor(Color.red);
        for (int i = 1; i <= x; i++) { // 畫縱線
            g.drawLine(i*20, 0, i*20, 200);
        }
        for (int j = 1; j <= y; j++) { // 畫橫線
            g.drawLine(0, j*20, 200, j*20);
        }
        int width = x + 2;
        // 畫出每一格目前的狀態
        for (int i = 1; i <= y; i++) { // 由上而下
            int tmp = i * width;
            for (int j = 1; j <= x; j++) { // 由左而右
                int tt = tmp + j;
                if ((map[tt] & VISIT) != 0) { // 已經翻過來了
                    g.setColor(Color.blue);
                    if ((map[tt] & NEB) != 0) { // 旁有地雷, show出地雷數
                        g.drawString(Integer.toString(map[tt] & NEB),  j * 20 - 15, i * 20 - 5);
                    } else { // 沒有地雷, 填空白
                        g.fillRect((j - 1) * 20, (i - 1) * 20, 20, 20);
                    }
                } else if ((map[tt] & MARK) != 0) { // 已用右鍵做記號
                    g.setColor(Color.green);
                    g.fillOval((j - 1) * 20, (i - 1) * 20, 20, 20);
                }
            }
        }
    }
    public void actionPerformed(ActionEvent e) {
        String command = e.getActionCommand();
        if (command.equals("Exit")) {
            System.exit(0);
        } else if (command.equals("New Game")) {
            reset();
        }  else {
            System.out.println("Unknown Command");
        }
    }
    public Mine2(int x, int y) {
        MenuBar mb;
        Menu m;
        this.x = x;
        this.y = y;
        int width = y + 2;
        directions = new int[8];
        directions[0] = 1;
        directions[1] = -1;
        directions[2] = width;
        directions[3] = -width;
        directions[4] = -width - 1;
        directions[5] = -width + 1;
        directions[6] = width - 1;
        directions[7] = width + 1;
        (f = new Frame("踩地雷")).add(this).addMouseListener(this);
        f.addWindowListener(new CloseWindow(f,true));
        f.setMenuBar(mb=new MenuBar());
        mb.add(m = new Menu("File"));
        m.add(new MenuItem("New Game")).addActionListener(this);
        m.add(new MenuItem("Exit")).addActionListener(this);
        reset();
        f.pack();
        f.show();
    }
    void setTouched(int tt) {
        map[tt] |= VISIT;
        if ((map[tt] & NEB) == 0) { // no bomb around, auto flip
            for (int i = 0; i < 8; i++) {
                if ((map[tt + directions[i]] & VISIT) == 0) { // 已翻開就不用再做了
                    setTouched(tt + directions[i]);
                }
            }
        }
    }
    public void mouseClicked(MouseEvent e) {}
    public void mouseEntered(MouseEvent e) {}
    public void mouseExited(MouseEvent e) {}
    public void mouseReleased(MouseEvent e) {}
    public void mousePressed(MouseEvent e) {
        int row = e.getY()/20 + 1;
        int col = e.getX()/20 + 1;
        int tt = row*(y + 2) + col;
        if (row > x || col > y || (map[tt] & VISIT) != 0) { // 超出邊界或已經翻過就不必處理
            return;
        }
        if (e.isMetaDown()) { // 按右鍵, 作個標記就好了
            map[tt] ^= MARK;
        } else if ((map[tt] & BOMB) != 0) { // 按到地雷了
            showAll();
            return;
        } else {
            setTouched(tt);
        }
        checkWin();
        repaint();
    }
    public static void main(String[] argv) {
        new Mine2(10,10);
    }
}
```
