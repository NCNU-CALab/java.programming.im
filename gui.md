## GUI 簡介

GUI 是由 Component、LayoutManager 以及 Listener 所組成的。

Component 是畫面的主要組成成分，本身並沒有可見的實體，其子類別才有顯現的效果。

如果 Component 可以放入其他的 Component，則這種 Component 就稱為是 Container。

Container 裡面的 Component 的排放位置，可以用絕對座標，也可以透過 LayoutManager 加以調整。

使用者以 Mouse、Keyboard 等操作 GUI 時，會產生相對應的 Event，並由視窗系統將該 Event 傳遞給程式設計者所指定的 Listener。

例如使用者以 Mouse 的左鍵按下 Button (一種 Component) 時，視窗系統會將 ActionEvent 傳給該 Button 的 ActionListener (設計者實作的物件)，由 ActionListener 決定採取的步驟。

這種由使用者觸發的執行動作，和程序式語言由主程式開始呼叫其他程序的模式不同，我們稱之為事件驅動 (Event Driven) 模式。

AWT (Abstract Window Toolkit `java.awt.*`) 是 JDK 1.0 所提出來有關 GUI 的函式庫，在 JDK 1.1 時又修改了 Event Handling 的機制，成為現在的樣子。

當初為了快速開發出 AWT，以符合 Java 出版的時程，因此在設計上，是利用視窗系統已有的 GUI 元件為主。

由於 Java 原始的設計理念具有跨平台的性質，因此 AWT 只好取各家視窗系統 (Windows、Open Look、Motif、Machintosh) 上元件的交集，因此 AWT 頗為陽春。

為了彌補 AWT 的不足，JDK 自 1.2 以後提供新的 JFC(Java Foundation Class)/Swing 的程式庫 `javax.swing`。

Swing 是以 Java 語言所寫出來的，因此提供了更多樣化而複雜的元件，如 `Image JButton`、`Image JLabel`、`JTree`、`JTable`、`JTabbedPane`、`JPopupMenu` 等等。

而且由於 Swing 並不依靠底層的視窗系統畫出元件的外觀，因此 Swing 可以讓使用者自行選定其所喜愛的 look and feel。

為了讓原先使用 AWT 的應用程式能改用 Swing 的程式庫，AWT 內所有的元件都有相對應以 J 開頭的 Swing 元件，如 `Button` 對應 `JButton`、`TextField` 對應 `JTextField`。

原先 AWT 的程式只要把 Source Code 裡面的 AWT 元件前面加上 J 即可，其餘 Event Handling 和 LayoutManager 均可沿用(除了 `JFrame`、`JDialog`、`JApplet`、`JWindow` 等最上層的 Container 不能直接加入元件，必須透過 `getContentPane()` 取得放置的地方)。

在以下的範例中，我們主要採用 AWT，並在最後面給一個 Swing 的範例，以說明兩者間的相似性。

## AWT

AWT 中直接繼承 Component 的子類別有：
* Button  
  ![Buttons](https://github.com/NCNU-CALab/java.programming.im/raw/master/images/Button-1.gif)
* [Canvas](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/Canvas.html)：空白的方形區域
* [Checkbox](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/Checkbox.html)：勾選
* [Choice](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/Choice.html)：拉下式表單
* [Container](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/Container.html)：裡面還可以放入其他 Component，其子類別有
  * [Window](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/Window.html)：最上層的視窗，又有 [`Frame`](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/Frame.html) (可以設定 [MenuBar](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/MenuBar.html) 的獨立視窗)、[`Dialog`](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/Dialog.html) (必須依附在 Frame 下的視窗)、[`FileDialog`](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/FileDialog.html) 等子類別
  * [Panel](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/Panel.html)：沒有顯示的容器
  * [ScrollPane](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/ScrollPane.html)：具有 ScrollBar 的容器
* [Label](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/Label.html)：可以顯示文字
* [List](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/List.html)：捲軸表單
* [Scrollbar](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/Scrollbar.html)：捲軸
* [TextComponent](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/TextComponent.html)：可用以編輯文字，其子類別有
  * [TextField](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/TextField.html) (單行)
  * [TextArea](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/TextArea.html) (多行)

AWT 提供的 [LayoutManager](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/LayoutManager.html) 有：
* [BorderLayout](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/BorderLayout.html)  
  ![BorderLayout](https://github.com/NCNU-CALab/java.programming.im/raw/master/images/BorderLayout-1.gif)
* [CardLayout](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/CardLayout.html)
* [FlowLayout](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/FlowLayout.html)  
  ![FlowLayout](https://github.com/NCNU-CALab/java.programming.im/raw/master/images/FlowLayout-1.gif)
* [GridBagLayout](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/GridBagLayout.html)  
  ![GridBagLayout](https://github.com/NCNU-CALab/java.programming.im/raw/master/images/GridBagLayout-1.gif)
* [GridLayout](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/GridLayout.html)  
  ![GridLayout](https://github.com/NCNU-CALab/java.programming.im/raw/master/images/GridLayout-1.gif)

AWT 定義的 Event 主要是 [`java.awt.AWTEvent`](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/AWTEvent.html)，其下又有許多子類別 `java.awt.event.*`
* [ActionEvent](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/event/ActionEvent.html)：component-defined action occured
* [AdjustmentEvent](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/event/AdjustmentEvent.html)：ScrollBar 被拉動
* [ComponentEvent](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/event/ComponentEvent.html)：一般不直接使用，其子類別有
  * [ContainerEvent](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/event/ContainerEvent.html)：Container 加入或移除 Component，也很少用
  * [FocusEvent](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/event/FocusEvent.html)：Gain or Lose Focus
  * [InputEvent](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/event/InputEvent.html)：有兩個子類別
    * [KeyEvent](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/event/KeyEvent.html)
    * [MouseEvent](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/event/MouseEvent.html)
  * [PaintEvent](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/event/PaintEvent.html)：供 AWT 內部使用，不用管他
  * [WindowEvent](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/event/WindowEvent.html)：視窗狀態改變
* [HierarchyEvent](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/event/HierarchyEvent.html)：AWT 自動處理，不用管他
* [InputMethodEvent](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/event/InputMethodEvent.html)
* [InvocationEvent](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/event/InvocationEvent.html)：由 AWT event dispatcher thread 呼叫 Runnable 物件裡的 `run()`
* [ItemEvent](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/event/ItemEvent.html)：an item was selected or deselected
* [TextEvent](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/event/TextEvent.html)：an object's text changed

以上的 Event 都有相對應的 Listener Interface 來處理，如 ActionEvent 相對應的是 [ActionListener](https://rawgit.com/NCNU-CALab/java.programming.im/master/docs/api/java/awt/event/ActionListener.html)

## 井字遊戲

```java
import java.awt.*;
import java.awt.event.*;
public class OXMain extends Frame implements ActionListener {
    private OX oxBoard;
    private OXMain() {
        super("井字遊戲");
        Menu m;
        MenuBar mb;
        oxBoard = new OX(this);
        this.add(oxBoard);
        CloseWindow close = new CloseWindow(this, true);
        this.setMenuBar(mb = new MenuBar());
        mb.add(m = new Menu("遊戲")).add(new MenuItem("新遊戲")).addActionListener(this);
        m.add(new MenuItem("結束")).addActionListener(close);
        mb.add(new Menu("說明")).add(new MenuItem("關於本遊戲")).addActionListener(this);
        this.addWindowListener(close);
        pack();
        setVisible(true);
    }
    public static void main(String argv[]) {
        new OXMain();
    }
    // implements the ActionListener interface
    public void actionPerformed(ActionEvent e) {
        String command = e.getActionCommand();
        if (command.equals("關於本遊戲")) {
             new ErrorDialog(this,"俞旭昇寫好玩的");
        } else if (command.equals("新遊戲")) {
            oxBoard.newGame();
        }
    }
}
class OX extends Component implements MouseListener {
    public static final byte EMPTY = 0;
    public static final byte CIRCLE = 1;
    public static final byte CROSS = 2;
    private byte[] board = new byte[9];
    private byte playing = CIRCLE;
    private Dimension mySize = new Dimension(300,300);
    private Frame parent;
    private byte[][] directions = {{0,1,2},{3,4,5},{6,7,8},{0,3,6},{1,4,7},{2,5,8},{0,4,8},{2,4,6}};
    public OX(Frame p) {
        this.addMouseListener(this);
        parent = p;
    }
    // The following 5 functions implement the MouseListener interface
    public void mouseClicked(MouseEvent e) {}
    public void mouseEntered(MouseEvent e) {}
    public void mouseExited(MouseEvent e) {}
    public void mouseReleased(MouseEvent e) {}
    public void mousePressed(MouseEvent e) {
        int row = e.getY()/100;
        int col = e.getX()/100;
        if (row >= 3 || col >= 3) return; // 超過邊界
        if (board[row*3+col] == EMPTY) { // 此位置可以下
            board[row*3+col] = playing;
            repaint(); // notify Window Manager
            // Anyone Win?
            for (int i=0; i < directions.length; i++) {
                int j;
                for (j = 0; j < 3 && board[directions[i][j]]==playing; j++) ;
                if (j==3) {
                    if (playing == CIRCLE) {
                        new ErrorDialog(parent,"圓圈贏了");
                    } else if (playing == CROSS) {
                        new ErrorDialog(parent,"叉叉贏了");
                    }
                    this.newGame();
                    return;
                }
            }
            playing ^= 0x03;
        }
    }
    // override paint() defined in Component
    public void paint(Graphics g) {
        g.drawLine(0,100,300,100);  g.drawLine(0,200,300,200);
        g.drawLine(100,0,100,300);  g.drawLine(200,0,200,300);
        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 3; j++) {
                if (board[i*3+j] == CIRCLE) {
                    g.drawOval(50+j*100-25,50+i*100-25,50,50);
                } else if (board[i*3+j] == CROSS) {
                    g.drawLine(25+j*100,25+i*100,75+j*100,75+i*100);
                    g.drawLine(75+j*100,25+i*100,25+j*100,75+i*100);
                }
            }
        }
    }
    public void newGame() {
        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 3; j++) {
                board[i*3+j] = EMPTY;
            }
        }
        playing = CIRCLE;
        repaint();
    }
    // override getPreferredSize defined in Component,
    // so that the Component has proper size on screen
    public Dimension getPreferredSize() {
        return mySize;
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
class ErrorDialog extends Dialog {
    public ErrorDialog(Frame parent, String all[]) {
        this(parent, all, null);
    }
    public ErrorDialog(Frame parent, String all[], String msg) {
        super(parent,"",true);
        StringBuffer sb = new StringBuffer();
        for (int i=0; i < all.length; i++) {
            sb.append(all[i]);
            sb.append('\n');
        }
        if (msg!=null) {
            sb.append(msg);
        }
        setup(parent, sb.toString());
    }
    public ErrorDialog(Frame parent, String message) {
        super(parent,"",true);
        setup(parent, message);
    }
    private void setup(Frame parent, String message) {
        this.setLayout(new GridBagLayout());
        int row=0, col=0, i, width=0;
        Font font = new Font("Serif", Font.PLAIN, 16);
        char c=' ';
        for (i = 0; i < message.length(); i++) {
            c = message.charAt(i);
            if (c=='\n') {
               row++;
               if (width > col) {
                   col = width;
               }
               width=0;
            } else if (c == '\t') {
                width += 7-width%7;
            } else {
                if (c > 0x00FF) {
                    width+=2;
                } else {
                    width++;
                }
            }
        }
        if (c!='\n') {
           row++;
           if (width > col) {
               col = width;
           }
        }
        col++;
        // 希望視窗出來不要太大或太小
        row = (row > 24) ? 24 : row;
        if (row < 5) {
            row=5;
        }
        if (col < 20) {
            col = 20;
        }
        TextArea tx = new TextArea(message,row,col);
        tx.setEditable(false);
        tx.setFont(font);
        AddConstraint.addConstraint(this, tx, 0, 0, 1, 1,
            GridBagConstraints.BOTH,
            GridBagConstraints.NORTHWEST,
            1,1,0,0,0,0);
        Button b = new Button("確定");
        b.setFont(font);
        AddConstraint.addConstraint(this, b, 0, 1, 1, 1,
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

## 井字遊戲的 Swing 版本

```java
/**
 * Program Name: SwingOXMain.java
 * Purpose: Showing how to Swing to write 井字遊戲
 * Modify Date: 2005/03/29
 */
import java.awt.*;
import java.awt.event.*;
import javax.swing.*;
public class SwingOXMain extends JFrame implements ActionListener {
    private OX oxBoard;
    private SwingOXMain() {
        super("井字遊戲");
        JMenu m;
        JMenuBar mb;
        oxBoard = new OX(this);
        this.getContentPane().add(oxBoard);
        CloseWindow close = new CloseWindow(this, true);
        this.setJMenuBar(mb = new JMenuBar());
        mb.add(m = new JMenu("遊戲")).add(new JMenuItem("新遊戲")).addActionListener(this);
        m.add(new JMenuItem("結束")).addActionListener(close);
        mb.add(new JMenu("說明")).add(new JMenuItem("關於本遊戲")).addActionListener(this);
        this.addWindowListener(close);
        pack();
        setVisible(true);
    }
    public static void main(String argv[]) {
        new SwingOXMain();
    }
    // implements the ActionListener interface
    public void actionPerformed(ActionEvent e) {
        String command = e.getActionCommand();
        if (command.equals("關於本遊戲")) {
            new ErrorDialog(this,"俞旭昇寫好玩的");
        } else if (command.equals("新遊戲")) {
            oxBoard.newGame();
        }
    }
}
class OX extends Component implements MouseListener {
    public static final byte EMPTY = 0;
    public static final byte CIRCLE = 1;
    public static final byte CROSS = 2;
    private byte[] board = new byte[9];
    private byte playing = CIRCLE;
    private Dimension mySize = new Dimension(300,300);
    private JFrame parent;
    private byte[][] directions = {{0,1,2},{3,4,5},{6,7,8},{0,3,6},{1,4,7},{2,5,8},{0,4,8},{2,4,6}};
    public OX(JFrame p) {
        this.addMouseListener(this);
        parent = p;
    }
    // The following 5 functions implement the MouseListener interface
    public void mouseClicked(MouseEvent e) {}
    public void mouseEntered(MouseEvent e) {}
    public void mouseExited(MouseEvent e) {}
    public void mouseReleased(MouseEvent e) {}
    public void mousePressed(MouseEvent e) {
        int row = e.getY()/100;
        int col = e.getX()/100;
        if (row >= 3 || col >= 3) return; // 超過邊界
        if (board[row*3+col] == EMPTY) { // 此位置可以下
            board[row*3+col] = playing;
            repaint(); // notify Window Manager
            // Anyone Win?
            for (int i=0; i < directions.length; i++) {
                int j;
                for (j=0; j &tl; 3 && board[directions[i][j]]==playing; j++) ;
                if (j==3) {
                    if (playing == CIRCLE) {
                        new ErrorDialog(parent,"圓圈贏了");
                    } else if(playing == CROSS) {
                        new ErrorDialog(parent,"叉叉贏了");
                    }
                    this.newGame();
                    return;
                }
            }
            playing ^= 0x03;
        }
    }
    // override paint() defined in Component
    public void paint(Graphics g) {
        g.drawLine(0,100,300,100);  g.drawLine(0,200,300,200);
        g.drawLine(100,0,100,300);  g.drawLine(200,0,200,300);
        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 3; j++) {
                if (board[i*3+j] == CIRCLE) {
                    g.drawOval(50+j*100-25,50+i*100-25,50,50);
                } else if (board[i*3+j] == CROSS) {
                    g.drawLine(25+j*100,25+i*100,75+j*100,75+i*100);
                    g.drawLine(75+j*100,25+i*100,25+j*100,75+i*100);
                }
            }
        }
    }
    public void newGame() {
        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 3; j++) {
                board[i*3+j] = EMPTY;
            }
        }
        playing = CIRCLE;
        repaint();
    }
    // override getPreferredSize defined in Component,
    // so that the Component has proper size on screen
    public Dimension getPreferredSize() {
        return mySize;
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
```

## 鍵盤事件處理範例

```java
/* Program Name: Field.java
   Subject: 單一欄位輸入 
   Author: 俞旭昇 Shiuh-Sheng Yu
           National ChiNan University
           Department of Information Management 
   Since 1997/08/17
   Last Update Date: 08/23/1998
   2003/12/17, fix delete, tab
   2004/04/26, 支援jdk1.2以後版本使用Ctrl-C,Ctrl-V複製貼上功能
   ToolKit: JDK1.1.6
*/
package ncnu.gui;
import java.awt.*;
import java.awt.event.*;
import java.sql.Types;
import java.awt.datatransfer.*;
import java.io.IOException;
public class Field implements ActionListener, KeyListener, Transferable, ClipboardOwner {
    static {
        try {
            if (System.getProperty("java.version").startsWith("1.1")) {
                checkCopy = false;
            } else {
                checkCopy = true;
            }
        } catch(Exception ex) {
        }
    }
    public DataFlavor[] getTransferDataFlavors() {
        DataFlavor[] myFlavors = new DataFlavor[1];
        myFlavors[0] = DataFlavor.stringFlavor;
        return myFlavors;
    }
    public boolean isDataFlavorSupported(DataFlavor flavor) {
        return flavor.equals(DataFlavor.stringFlavor);
    }
    public Object getTransferData(DataFlavor flavor) throws UnsupportedFlavorException, IOException {
        return myData;
    }
    public void lostOwnership(Clipboard clipboard, Transferable contents) {
    }
    static String myData;
    static boolean checkCopy;
    public static final int OVR=0;
    public static final int INS=1;
    TextField t;
    int limit;
    int type;
    int decimalDigits;
    int mode=0;
    KeyJump parent;
    public Field(KeyJump parent, TextField t, int limit, int type, int decimalDigits) {
        this.t = t;
        this.limit = limit;
        this.type = type;
        this.decimalDigits = decimalDigits;
        this.parent = parent;
        this.mode = OVR;
        t.addActionListener((ActionListener)this);
        t.addKeyListener((KeyListener)this);
    }
    public void keyPressed(KeyEvent e) {
        String s;
        int pos,sstart,send;
        StringBuffer sb;
        switch(e.getKeyCode()) {
        case KeyEvent.VK_PASTE:
        case KeyEvent.VK_COPY:
        case KeyEvent.VK_UNDO:
        case KeyEvent.VK_CUT:
        case KeyEvent.VK_FIND:
            return;
        case KeyEvent.VK_INSERT:
            if (mode==OVR) {
                mode = INS;
            } else {
                mode = OVR;
            }
            e.consume();
            break;
        case KeyEvent.VK_TAB:
        case KeyEvent.VK_ENTER:
        case KeyEvent.VK_DOWN:
            parent.next(t);
            e.consume();
            break;
        case KeyEvent.VK_UP:
            parent.previous(t);
            e.consume();
            break;
        case KeyEvent.VK_LEFT:
        case KeyEvent.VK_RIGHT:
        case KeyEvent.VK_HOME:
        case KeyEvent.VK_END:
            return;
        case KeyEvent.VK_PAGE_UP:
            parent.pageUp(t);
            e.consume();
            break;
        case KeyEvent.VK_PAGE_DOWN:
            parent.pageDown(t);
            e.consume();
            break;
        case KeyEvent.VK_DELETE:
            e.consume();
            if (!t.isEditable()) {
                return;
            }
            s = t.getText();
            pos = t.getCaretPosition();
            sstart=t.getSelectionStart();
            send=t.getSelectionEnd();
            if (send>sstart) {
                sb = new StringBuffer(s.substring(0,sstart));
                if (send!=s.length()) {
                    sb.append(s.substring(send));
                }
                if ((type==Types.NUMERIC||type==Types.DECIMAL) && !legalNumber(sb.toString())) {
                    return;
                }
                t.setText(sb.toString());
                t.setCaretPosition(sstart);
            } else {
                if (pos==s.length()) {
                    return;
                }
                sb = new StringBuffer(s.substring(0,pos));
                sb.append(s.substring(pos+1));
                if ((type==Types.NUMERIC || type==Types.DECIMAL) && !legalNumber(sb.toString())) {
                    return;
                }
                t.setText(sb.toString());
                t.setCaretPosition(pos);
            }
            return;
        case KeyEvent.VK_BACK_SPACE:
            e.consume();
            if (!t.isEditable()) {
                return;
            }
            s = t.getText();
            pos = t.getCaretPosition();
            sstart=t.getSelectionStart();
            send=t.getSelectionEnd();
            if (send>sstart) {
                sb = new StringBuffer(s.substring(0,sstart));
                if (send!=s.length()) {
                    sb.append(s.substring(send));
                }
                if ((type==Types.NUMERIC||type==Types.DECIMAL) && !legalNumber(sb.toString())) {
                    return;
                }
                t.setText(sb.toString());
                t.setCaretPosition(sstart);
            } else {
                if (pos==0) {
                    return;
                }
                if (pos<=1) {
                    sb = new StringBuffer();
                } else {
                    sb = new StringBuffer(s.substring(0,pos-1));
                }
                sb.append(s.substring(pos));
                if ((type==Types.NUMERIC||type==Types.DECIMAL) && !legalNumber(sb.toString())) {
                    return;
                }
                t.setText(sb.toString());
                t.setCaretPosition(pos-1);
            }
            return;
        }
    }
    public void keyTyped(KeyEvent e) {
        String s;
        int pos,sstart,send;
        StringBuffer sb;
        if (!t.isEditable()) {
            e.consume();
            return;
        }
        char ch = e.getKeyChar();
        // 3copy, 22 paste,127 delete,8 backspace
        if (checkCopy) {
            if (ch==3) {
                e.consume();
                s = t.getText();
                pos = t.getCaretPosition();
                sstart=t.getSelectionStart();
                send=t.getSelectionEnd();
                if (send>sstart) {
                    myData = s.substring(sstart,send);
                } else {
                    myData = "";
                }
                Toolkit.getDefaultToolkit().getSystemClipboard().setContents(this,this);
            }
            if (ch==22) {
                e.consume();
                Transferable tt = Toolkit.getDefaultToolkit().getSystemClipboard().getContents(this);
                if (tt != null) {
                    String data;
                    try {
                        data = (String)tt.getTransferData(DataFlavor.stringFlavor);
                    } catch(Exception eppp) {
                        System.out.println(eppp);
                        return;
                    }
                    s = t.getText();
                    pos = t.getCaretPosition();
                    sstart=t.getSelectionStart();
                    send=t.getSelectionEnd();
                    if (send>sstart) { // 有選起來
                        sb = new StringBuffer(s.substring(0,sstart));
                        sb.append(data);
                        if (send!=s.length()) {
                            sb.append(s.substring(send));
                        }
                        if ((type==Types.NUMERIC||type==Types.DECIMAL) && !legalNumber(sb.toString())) {
                            return;
                        }
                        t.setText(sb.toString());
                        t.setCaretPosition(sstart);
                    } else {
                        sb = new StringBuffer(s.substring(0,pos));
                        sb.append(data);
                        if (pos < s.length()) {
                            sb.append(s.substring(pos));
                        }
                        if ((type==Types.NUMERIC || type==Types.DECIMAL) && !legalNumber(sb.toString())) {
                            return;
                        }
                        t.setText(sb.toString());
                        t.setCaretPosition(pos);
                    }
                }
            }
        }
        if (ch==3 || ch==22 || ch==8 || ch==127 || ch=='\t' || ch=='\n') {
            e.consume();
            return;
        }
        if (ch!=KeyEvent.CHAR_UNDEFINED) {
            s = t.getText();
            sstart=t.getSelectionStart();
            send=t.getSelectionEnd();
            if (send > sstart) {
                sb = new StringBuffer(s.substring(0,sstart));
                sb.append(ch);
                if (send!=s.length()) {
                    sb.append(s.substring(send));
                }
            } else {
                pos = t.getCaretPosition();
                sb = new StringBuffer(s.substring(0,pos));
                sb.append(ch);
                if (pos!=s.length()) {
                    if (mode==INS) {
                        sb.append(s.substring(pos));
                    } else {
                        sb.append(s.substring(pos+1));
                    }
                }
            }
            if ((type==Types.NUMERIC||type==Types.DECIMAL) && !legalNumber(sb.toString())) {
                return;
            }
            pos = t.getCaretPosition();
            t.setText(inputTruncate(sb.toString()));
            t.setCaretPosition(pos+1);
        }
        e.consume();
    }
    public void keyReleased(KeyEvent e) {
        return;
    }
    public void actionPerformed(ActionEvent e) {
        int currentIndex, i;
        TextComponent next;
        switch(e.getID()) {
            case ActionEvent.ACTION_PERFORMED:
                parent.next(t);
                break;
            default:
                break;
        }
    }
    protected String inputTruncate(String s) {
        int i,j=limit;
        if (type==Types.NUMERIC||type==Types.DECIMAL) {
            int k=0, leading;
            if (decimalDigits==0) {
                leading = limit;
            } else {
                leading = limit-decimalDigits;
            }
            if (s.charAt(0)=='-') {
                k++;
            }
            for (; k < leading && k < s.length(); k++) {
                if (!(s.charAt(k) >='0' && s.charAt(k) <='9')) {
                    break;
                }
            }
            if (decimalDigits!=0) {
                k += decimalDigits+1;
            }
            j = k;
        }
        if (s.getBytes().length>j) {
            for (i=s.length()-1; i > 0 ;i--) {
                if (s.substring(0,i).getBytes().length<=j) {
                    break;
                }
            }
            return (s.substring(0,i));
        }
        return s;
    }
    protected boolean legalNumber(String s) {
        int leading;
        int i=0,j=0;
        if (s.length()==0) {
            return true;
        }
        if (decimalDigits == 0) {
           leading = limit;
        } else {
           leading = limit - decimalDigits;
        }
        if (s.charAt(i)=='-') {
            j++;
        }
        for (;(j < s.length()) && (j < leading); j++) {
            if (!(s.charAt(j) >='0' && s.charAt(j) <='9')) {
                break;
            }
        }
        if (j==s.length()) {
            return true;
        }
        if (decimalDigits==0) {
            return false;
        } else {
            if (s.charAt(j)=='.') {
                j++;
                for (i=j; i!=s.length() && i-j < decimalDigits; i++) {
                    if (!(s.charAt(i) >='0' && s.charAt(i) <='9')) {
                        break;
                    }
                }
                if (i==s.length()) {
                    return true;
                }
            }
            return false;
        }
    }
}
```

## 文字編輯器

```java
import java.awt.*;
import java.awt.event.*;
import javax.swing.*;
import javax.swing.event.*;
import java.io.*;
public class Editor extends WindowAdapter implements ActionListener {
    JFrame topWindow;
    JTextArea jta;
    public void actionPerformed(ActionEvent e) {
        String command = e.getActionCommand();
        if (command.equals("Open")) {
            JFileChooser fd = new JFileChooser(".");
            fd.setDialogType(JFileChooser.OPEN_DIALOG);
            int returnVal = fd.showOpenDialog(topWindow);
            if (returnVal == JFileChooser.APPROVE_OPTION) {
                try {
                    File file = fd.getSelectedFile();
                    BufferedReader fin = new BufferedReader(new FileReader(file));
                    StringBuffer sb = new StringBuffer();
                    String x;
                    while ((x=fin.readLine()) != null) {
                        sb.append(x);
                        sb.append("\n");
                    }
                    jta.setText(sb.toString());
                } catch(Exception epp) {}
                jta.setCaretPosition(0);
            }
        } else if (command.equals("Save")) {
            JFileChooser fd = new JFileChooser(".");
            fd.setDialogType(JFileChooser.SAVE_DIALOG);
            int returnVal = fd.showSaveDialog(topWindow);
            if (returnVal == JFileChooser.APPROVE_OPTION) {
                try {
                    File file = fd.getSelectedFile();
                    FileWriter fout = new FileWriter(file);
                    fout.write(jta.getText());
                    fout.close();
                } catch(Exception epp) {}
            }
        } else if (command.equals("Find")) {
            String x = jta.getText();
            int at = x.indexOf("int", jta.getCaretPosition());
            if (at >= 0) {
                jta.setCaretPosition(at);
                jta.setSelectionStart(at);
                jta.setSelectionEnd(at+3);
            }
        } else {
            System.out.println("Unknown "+command);
        }
    }
    public void windowClosing(WindowEvent e) {
        topWindow.dispose();
        System.exit(0);
    }
    public Editor() {
        Font myFont = new Font("細明體", Font.PLAIN, 20);
        topWindow = new JFrame("My IDE");
        // get the root container of JFrame
        Container c = topWindow.getContentPane();
        c.setLayout(new GridLayout(1,1));
        JMenuBar jmb = new JMenuBar();
        JMenu m = new JMenu("File");
        m.setFont(myFont);
        JMenuItem mi = new JMenuItem("Open");
        mi.setFont(myFont);
        mi.addActionListener(this);
        m.add(mi);
        mi = new JMenuItem("Save");
        mi.setFont(myFont);
        mi.addActionListener(this);
        m.add(mi);
        mi = new JMenuItem("Find");
        mi.setFont(myFont);
        mi.addActionListener(this);
        m.add(mi);
        jmb.add(m);
        topWindow.setJMenuBar(jmb);
        jta = new JTextArea(24,80);
        jta.setFont(myFont);
        jta.setCursor(new Cursor(Cursor.HAND_CURSOR));
        JScrollPane jsp = new JScrollPane(jta);
        c.add(jsp);
        topWindow.addWindowListener(this);
        topWindow.pack(); // 壓縮到最適當的大小
        topWindow.show(); // 顯示JFrame
    }
    public static void main(String[] argv) {
        new Editor();
    }
}
```
