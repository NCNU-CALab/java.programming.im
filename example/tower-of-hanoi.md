[Hanoi.html](#Hanoi.html)  
[Hanoi.java](#Hanoi.java)：An applet shows how to solve Hanoi tower problem  
[Pile.java](#Pile.java)：Maintain and render a pile of disks  
[AddConstraint.java](#AddConstraint.java)：GridBagLayout輔助類別，簡化GridBagLayout的使用  

<p id="Hanoi.html"></p>

```html
<html>
    <applet code="Hanoi.class" width=300 height=300>
    </applet>
</html>
```

***

<p id="Hanoi.java"></p>

```java
/**
 * Program Name: Hanoi.java
 * Subject: An applet shows how to solve Hanoi tower problem
 * Author: 俞旭昇 Shiuh-Sheng Yu
 *         Department of Information Management
 * Edit Date: 3/20/1999
 * Last Update Date: 3/21/1999
 * Toolkit: JDK1.1.7B
 */
import java.applet.Applet;
import java.awt.*;
import java.awt.event.*;
public class Hanoi extends Applet implements ActionListener, Runnable {
    TextField input;
    Pile a=null, b=null, c=null;
    Panel display;
    int size;
    // Setup GUI
    public void init() {
        this.setLayout(new GridBagLayout());
        AddConstraint.addConstraint(this, new Label("Enter integer and press Enter:"), 0, 1, 1, 1,
            GridBagConstraints.NONE,
            GridBagConstraints.WEST,
            0,0,0,0,0,0);
        AddConstraint.addConstraint(this, input = new TextField(10), 1, 1, 1, 1,
            GridBagConstraints.HORIZONTAL,
            GridBagConstraints.WEST,
            1,0,0,0,0,0);
        input.addActionListener(this);
        AddConstraint.addConstraint(this, display = new Panel(), 0, 0, 2, 1,
            GridBagConstraints.BOTH,
            GridBagConstraints.CENTER,
            1,1,0,0,0,0);
        display.setLayout(new GridLayout(1,3));
        input.requestFocus();
    }

    // process user's action in TextField input
    public void actionPerformed(ActionEvent e) {
        try {
            size = Integer.parseInt(e.getActionCommand());
            if (size > 0) {
                display.removeAll(); // 清掉前一次的元件
                display.add(a = new Pile(size));
                display.add(b = new Pile(0));
                display.add(c = new Pile(0));
                display.validate(); // Ensures that this component has a valid layout.
                // Creates a thread to move our disks.
                // So that we can return control back immediately to window system.
                (new Thread(this)).start();
            }
        } catch(NumberFormatException ex) { // not a number in TextField
            input.setText("");
        }
    }

    // starts to move disks
    public void run() {
        move(size, a, b, c);
    }

    // recursively solve the Hanoi problem
    private void move(int n, Pile from, Pile to, Pile free) {
        if (n > 0) {
            move(n-1, from, free, to);
            to.push(from.pop());
            try { // sleep 1 second to let users see what happened
                Thread.currentThread().sleep(1000);
            } catch(InterruptedException ex) {
            }
            move(n-1, free, to, from);
        }
    }
}
```

***

<p id="Pile.java"></p>

```java
/* Program Name: Pile.java
   Subject: Maintain and render a pile of disks 
   Author: 俞旭昇 Shiuh-Sheng Yu
           National ChiNan University
           Department of Information Management
   Edit Date: 03/20/1999
   Last Update Date: 03/21/1999
   ToolKit: JDK1.1.7B
*/

import java.awt.*;
public class Pile extends Canvas {
    private int top;
    private int[] data;
    /**
     * @param initDisks 一開始的碟子數
     */
    public Pile(int initDisks) {
        super();
        top = 0;
        data = new int[100];
        for (int i = initDisks; i > 0; i--) {
            data[top++] = i;
        }
    }

    /**
     * Push a disk of size n to this pile. This method enfoces screen update ASAP.
     * @param size the size of the disk
     */
    public void push(int size) {
        data[top++] = size;
        repaint();
    }

    /**
     * Pop out a disk. This method enfoces screen update ASAP.
     * @return size of the disk on top of the pile
     */
    public int pop() {
        top--;
        repaint();
        return(data[top]);
    }

    /**
     * Draw disks of the pile on Graphics g
     */
    public void paint(Graphics g) {
        // getSize is define in Component
        int x = getWidth()/2;
        int y = getHeight();
        // 設為藍色
        g.setColor(Color.blue);
        // 畫出填滿的長方形, 當作柱子
        g.fillRect(x-2,0,4,y-2);
        g.setColor(Color.red);
        // 畫出碟子
        for (int i=0; i<top; i++) {
            g.fillRect(x - 5 * data[i], y - 5 * (i + 1), 10 * data[i], 3);
        }
    }
}
```

***

<p id="AddConstraint.java"></p>

```java
/* Program Name: AddConstraint.java
   Subject: GridBagLayout輔助類別 
   Author: 俞旭昇 Shiuh-Sheng Yu
           National ChiNan University
           Department of Information Management
   Edit Date: 06/14/1997
   Last Update Date: 10/28/1997
   ToolKit: JDK1.1.7
*/
import java.awt.*;
/**
 * 此類別是用來簡化GridBagLayout的使用, 在Java Tutorial這本書內可以找到相關資料 
 */
public class AddConstraint {
    /**
     * Set component position in a container whose layout manager is GridBagLayout.
     * @param container The container to be put in
     * @param component The component put in container
     * @param grid_x X coordinate
     * @param grid_y Y coordinate
     * @param grid_width Width
     * @param grid_height Height
     * @param fill Fill police when container is resized
     * @param anchor Ancho position when container is resized
     * @param weight_x Width expansion ratio when container is resized. 1 for 100%
     * @param weight_y Height expansion ratio when container is resized. 1 for 100%
     * @param top Gap with above components in points
     * @param left Gap with left components in points
     * @param bottom Gap with below components in points
     * @param right Gap with right components in points
     * @return void
     */
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
