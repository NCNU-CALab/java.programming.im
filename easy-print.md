JDK 對印表的支援不多，主要是以「所視即所得」的觀念，將 AWT 的原先利用 Graphics 繪圖的部分改為利用 PrintGraphics，即可達到印表的需求。  

以下是一個提供預覽以及背景列印的程式庫，該程式庫採用 [Template Design Pattern](http://www.javacamp.org/designPattern/)，透過 abstract methods 要求子類別必須實作特定的方法。  

本程式另將原始碼公開於 [GitHub](https://github.com/NCNU-CALab/EasyPrint) 歡迎指教  

### Print.java

```java
package ncnu.print;
/* Program Name: Print.java
   Subject: 印表與瀏覽抽象類別 
   Editor: 俞旭昇 Shiuh-Sheng Yu
           National ChiNan University
           Department of Information Management
   Edit Date: 01/05/1998
   Modify Date: 05/06/2001
   Modify Date: 2004/10/01,加入背景列印
   ToolKit: JDK1.1
*/
import ncnu.gui.*;
import java.awt.*;
import java.awt.event.*;
import java.sql.*;
import java.util.*;

public abstract class Print implements ActionListener {
    ScrollPane sp;
    protected Connection con;
    protected SimpleTable fg;
    protected TableGUI gui;
    protected boolean completed = true;
    protected String printer;
    protected int copy = 1;
    protected boolean background;
    public Print(Connection c, String printer, int copy) {
        this.con = c;
        this.printer = printer;
        this.copy = copy;
        this.background = true;
    }
    public Print(Connection c, Vector data) {
        this.con = c;
        if (data != null) {
            PrintCanvas canvas = createCanvas(data);
            canvas.setResolution(144);
            new Preview(canvas,"預覽報表");
        } else {
            fg = new SimpleTable("", this, 16);
            addQueryCondition();
            fg.addButton("確定");
            fg.addButton("取消");
            if (completed) {
                fg.addButton("預覽");
                fg.addButton("放大預覽");
            }
            fg.read();
        }
    }
    public Print(Connection c) {
        this(c, null);
    }
    protected abstract Vector getData();
    protected abstract PrintCanvas createCanvas(Vector data);
    protected abstract void addQueryCondition();

    protected void setPrinter(String pname) {
        printer = pname;
    }
    public boolean print() {
        Vector data = getData();
        if (data==null || data.size()==0) {
            return false;
        }
        PrintCanvas canvas = createCanvas(data);
        canvas.setResolution(72);
        PrintJob pjob = getPrintJob(canvas);
        if (pjob != null) {
            do {
                Graphics pg = pjob.getGraphics();
                canvas.update(pg);
                pg.dispose(); // flush page
            } while (canvas.nextPage());
            pjob.end();
            return true;
        } else {
            System.out.println("Unable to create print job "+printer);
            return false;
        }
    }
    protected PrintJob getPrintJob(PrintCanvas canvas) {
        if (System.getProperty("java.version").compareTo("1.3") < 0) {
            return gui.getToolkit().getPrintJob(fg, "列印報表", null);
        } else {
            PageAttributes pa = new PageAttributes();
            String media = canvas.getMedia();
            if (media.equalsIgnoreCase("letter")) {
                pa.setMedia(PageAttributes.MediaType.LETTER);
            } else if (media.equalsIgnoreCase("B4")) {
                pa.setMedia(PageAttributes.MediaType.JIS_B4);
            } else if (media.equalsIgnoreCase("A4")) {
                pa.setMedia(PageAttributes.MediaType.A4);
            } else if (media.equalsIgnoreCase("A3")) {
                pa.setMedia(PageAttributes.MediaType.A3);
            }
            String ori = canvas.getOrientation();
            if (ori.equalsIgnoreCase("portraint")) {
                pa.setOrientationRequested(PageAttributes.OrientationRequestedType.PORTRAIT);
            } else if (ori.equalsIgnoreCase("landscape")) {
                pa.setOrientationRequested(PageAttributes.OrientationRequestedType.LANDSCAPE);
            }
            JobAttributes jAttr = null;
            if (printer!=null) {
                jAttr = new JobAttributes();
                jAttr.setDialog(JobAttributes.DialogType.NONE);
                jAttr.setPrinter(printer);
                jAttr.setCopies(copy);
            }
            if (background) {
                return Toolkit.getDefaultToolkit().getPrintJob(null, "列印報表", jAttr, pa);
            } else {
                return gui.getToolkit().getPrintJob(fg, "列印報表", jAttr, pa);
            }
        }
    }
    public void actionPerformed(ActionEvent e) {
        PrintCanvas canvas;
        String returnString, query;
        String cmd = e.getActionCommand();
        if (cmd.equals("取消")) {
            fg.dispose();
            return;
        }
        Vector data = getData();
        if (data==null || data.size()==0) {
            fg.dispose();
            return;
        }
        if (!completed) {
            addQueryCondition();
            if (completed) {
                fg.addButton("預覽");
                fg.addButton("放大預覽");
            }
            fg.read();
            return;
        }
        if (cmd.equals("確定")) {
            canvas = createCanvas(data);
            canvas.setResolution(72);
            PrintJob pjob = getPrintJob(canvas);
            if (pjob != null) {
                do {
                    Graphics pg = pjob.getGraphics();
                    canvas.update(pg);
                    pg.dispose(); // flush page
                } while (canvas.nextPage());
                pjob.end();
            } else {
                System.out.println("User canceled job");
            }
            fg.dispose();
        } else {
            canvas = createCanvas(data);
            if (cmd.equals("預覽")) {
                canvas.setResolution(72);
            } else if (cmd.equals("放大預覽")) {
                canvas.setResolution(144);
            }
            new Preview(canvas,"預覽報表");
        }
    }
    class Preview extends WindowAdapter implements ActionListener {
        Frame f;
        Button previous, next, printOne, printAll, leave;
        PrintJob pjob = null;
        String title;
        PrintCanvas pc;
        ScrollPane sp = new ScrollPane() {
            public Dimension getPreferredSize() {
                Dimension x = Toolkit.getDefaultToolkit().getScreenSize();
                return new Dimension(x.width-20,x.height-50);
            }
        };

        public void actionPerformed(ActionEvent e) {
            String cmd = e.getActionCommand();
            if (cmd.equals("上一頁")) {
                doPrevious();
            } else if (cmd.equals("下一頁")) {
                doNext();
            } else if (cmd.equals("列印本頁")) {
                doPrintOne();
            } else if (cmd.equals("列印全部")) {
                doPrintAll();
            } else {
                System.out.println("Unknown command "+cmd);
            }
        }        
        Preview(PrintCanvas pc, String title) {
            this.pc = pc;
            this.title = title;
            f = new Frame(title);
            f.setLayout(new GridBagLayout());
            f.addWindowListener(new CloseWindow(f));
            sp.add(pc);
            AddConstraint.addConstraint(f,sp,0,0,1,1,
                GridBagConstraints.BOTH, GridBagConstraints.CENTER,
                1,1,0,0,0,0);
            Panel buttons = new Panel();
            buttons.setLayout(new GridLayout(1,5));
            AddConstraint.addConstraint(f,buttons,0,1,1,1,
                GridBagConstraints.HORIZONTAL, GridBagConstraints.CENTER,
                1,0,0,0,0,0);
            ((Button)buttons.add(previous=new Button("上一頁"))).addActionListener(this);
            ((Button))buttons.add(next=new Button("下一頁"))).addActionListener(this);
            ((Button))buttons.add(printOne=new Button("列印本頁"))).addActionListener(this);
            ((Button))buttons.add(printAll=new Button("列印全部"))).addActionListener(this);
            ((Button))buttons.add(leave=new Button("離開"))).addActionListener(new CloseWindow(f));
            previous.setEnabled(false);
            if (pc.lastPage()) {
                next.setEnabled(false);
            }
            f.pack();
            f.show();
        }
        void doPrevious() {
            pc.previousPage();
            if (pc.firstPage()) {
                previous.setEnabled(false);
            }
            if (pc.lastPage()) {
                next.setEnabled(false);
            } else {
                next.setEnabled(true);
            }
            pc.update(pc.getGraphics());
        }
        void doNext() {
            pc.nextPage();
            if (pc.lastPage()) {
                next.setEnabled(false);
            }
            if (pc.firstPage()) {
                previous.setEnabled(false);
            } else {
                previous.setEnabled(true);
            }
            pc.update(pc.getGraphics());
        }
        void doPrintOne() {
            if (pjob==null) {
                pjob = Print.this.getPrintJob(pc);
            }
            if (pjob != null) {
                Graphics pg = pjob.getGraphics();
                pc.buffer = null;
                int rel = pc.getResolution();
                pc.setResolution(72);
                pc.update(pg);
                pg.dispose(); // flush page
                pjob.end();
                pjob=null;
                pc.setResolution(rel);
            }
        }
        void doPrintAll() {
            if (pjob==null) {
                pjob = Print.this.getPrintJob(pc);
            }
            if (pjob != null) {
                pc.currentPage = -1;
                while (pc.nextPage()) {
                    Graphics pg = pjob.getGraphics();
                    pc.buffer = null;
                    pc.setResolution(72);
                    pc.update(pg);
                    pg.dispose();
                }
                pjob.end();
                pjob=null;
            }
        }
    }
}
```

### PrintCanvas.java

```java
package ncnu.print;
/* Program Name: PrintCanvas.java
   Subject: 印表畫布抽象類別 
   Editor: 俞旭昇 Shiuh-Sheng Yu
           National Chi-Nan University
           Institute of Management Information 
   Edit Date: 01/05/1998
   Last Update Date: 05/06/2001
   ToolKit: JDK1.1.4
*/

import java.awt.*;
import java.util.*;
public abstract class PrintCanvas extends Canvas {
    Image buffer;
    private int total;
    protected int dotPerInch=72;
    protected Vector data;
    protected> int currentPage;
    protected Pair pagePositions[]; // cached for previous page

    class Pair {
        int data, subpage;
        Pair(int data, int subpage) {
            this.data = data;
            this.subpage = subpage;
        }
    }
    public void setResolution(int rel) {
        dotPerInch=rel;
    }
    public int getResolution() {
        return dotPerInch;
    }
    public int getDataIndex() {
        return pagePositions[currentPage].data;
    }
    public int getSubPageIndex() {
        return pagePositions[currentPage].subpage;
    }
    public int totalPages() {
        int total = 0;
        for (int i=0; i < data.size();) {
            int sub = pagesPerData(data.elementAt(i));
            if (sub>1) { // multi page per data
                total += sub;
                i++;
            } else {
                i += dataPerPage(total);
                total++;
            }
        }
        return total;
    }
    protected abstract int dataPerPage(int pageNum);
    /**
     * How many pages are needed to print out data
     * @param data Data record to be printed
     * @return Number of pages for the data, 0 if more data are need to fill this page
     */
    protected abstract int pagesPerData(Object data); // return 0 if more than a page per record
    /**
     * print a page for a data record
     * @param pg Print graphics
     * @param data Data record to be printed
     * @param 
     */
    public PrintCanvas(Vector data) {
        this.data = data;
        currentPage=0;
        total = totalPages();
        pagePositions = new Pair[total];
        int currentData = 0;
        for (int i=0; i < total;) {
            int sub = pagesPerData(data.elementAt(currentData));
            if (sub>1) { // multi page per data
                for (int j=0; j < sub; j++) {
                    pagePositions[i+j] = new Pair(currentData,j);
                }
                i+=sub;
                currentData++;
            } else { // multi data per page
                pagePositions[i] = new Pair(currentData,0);
                currentData += dataPerPage(i);
                i++;
            }
        }
    }

    public Dimension getPreferredSize() {
        if (getMedia().equals("A4")) {
            if (getOrientation().equals("portrait")) {
                return new Dimension((int)(8.27*dotPerInch),(int)(11.69*dotPerInch)); // A4 直印
            } else {
                return new Dimension((int)(11.69*dotPerInch),(int)(8.27*dotPerInch)); // A4 橫印
            }
        } else if (getMedia().equals("B4")) {
            if (getOrientation().equals("portrait")) {
                return new Dimension((int)(9.84*dotPerInch),(int)(13.9*dotPerInch)); // B4 直印
            } else {
                return new Dimension((int)(13.9*dotPerInch),(int)(9.84*dotPerInch)); // B4 橫印
            }
        } else if (getMedia().equals("A3")) {
            if (getOrientation().equals("portrait")) {
                return new Dimension((int)(11.69*dotPerInch),(int)(16.54*dotPerInch)); // A3 直印
            } else {
                return new Dimension((int)(16.54*dotPerInch),(int)(11.69*dotPerInch)); // A3 橫印
            }
        } else if (getMedia().equals("LETTER")) {
            if (getOrientation().equals("portrait")) {
                return new Dimension((int)(8.5*dotPerInch), (int)(11*dotPerInch)); // letter 直印
            } else {
                return new Dimension((int)(11*dotPerInch), (int)(8.5*dotPerInch)); // letter 橫印
            }
        }
        return new Dimension((int)(8*dotPerInch),(int)(11.2*dotPerInch)); // A4 直印 size
    }

    public String getMedia() {
        return "A4";
    }
    public String getOrientation() {
        return "portrait";
    }
    public boolean firstPage() {
        return (currentPage<=0);
    }

    public boolean lastPage() {
        return ((currentPage+1)>=total);
    }

    public boolean nextPage() {
        if (currentPage+1>=total) {
            return false;
        }
        buffer=null;
        currentPage++;
        return true;
    }

    public boolean previousPage() {
        if (currentPage == 0) {
            return false;
        }
        buffer=null;
        currentPage--;
        return true;
    }

    public void update(Graphics pg) {
        Graphics g = pg;
        if (buffer!=null) {
            pg.drawImage(buffer,0,0,this);
            return;
        } else if (!(pg instanceof PrintGraphics)) {
            Dimension d = getPreferredSize();
            buffer = createImage(d.width, d.height);
            if (buffer!=null) {
                g = buffer.getGraphics();
            }
        }
        paint(g);
        if (buffer!=null) {
            pg.drawImage(buffer,0,0,this);
        }
    }
}
```

### 圖形與字型公用程式

```java
package ncnu.print;
/* Program Name: CharArea.java
   Subject: 印表公用程式 
   Editor: 俞旭昇 Shiuh-Sheng Yu
           National Chi-Nan University
           Institute of Management Information 
   Edit Date: 01/05/1998
   Last Update Date: 05/06/2001
   ToolKit: JDK1.1
*/
import java.awt.*;
/**
 * The class defines a rectangle area so that users can draw String vertically or
 * horizontally, and don't have to worry wrapping problem.
 */
public class CharArea {
    private Graphics g;
    private FontMetrics f;
    private int x, y, width, height, row, col;
    private int curx, cury;
    private boolean fromRight;
    public int fontHeight;

    /**
     * @param g Graphics to draw String
     * @param x x coordinate of the area in points
     * @param y y coordinate of the area in points
     * @param width width of the area in points
     * @param height height of the area in points
     * @param fromRight draw Lines from right or not
     */
    public CharArea(Graphics g, int x, int y, int width, int height, boolean fromRight) {
        this.g = g;
        this.x = x;
        this.y = y;
        this.width = width;
        this.height = height;
        this.fromRight = fromRight;
        col = row = 1;
        f = g.getFontMetrics();
        // expect font size
        if (System.getProperty("java.version").compareTo("1.4") >=0) {
            fontHeight = (int)(f.getAscent()*1);
        } else {
            fontHeight = (int)(f.getAscent()*1.2857); // 9/7 for strange differece between 1.3 and 1.4 fontMetrics
        }
        if (!fromRight) {
            curx = x;
        } else {
            curx = x + width - fontHeight;
        }
        cury = y + fontHeight;
    }

    /**
     * @param g Graphics to draw String
     * @param x x coordinate of the area in points
     * @param y y coordinate of the area in points
     * @param width width of the area in points
     * @param height height of the area in points
     */
    public CharArea(Graphics g, int x, int y, int width, int height) {
        this(g, x, y, width, height, false);
    }
    /*
        col and row starts from 1
    */
    public int getRow() {
        return row;
    }
    public void setRow(int row) {
        this.row = row;
        cury = y + row*fontHeight;
    }
    public int getCol() {
        return col;
    }
    public void setCol(int col) {
        this.col = col;
        curx = x + (col-1)*f.getHeight(); // assume 方塊字
    }

    /**
     * Draw a string vertically, and wrap to right if necessary. Reset row to 1 after drawing.
     */
    public void drawLineV(String s) {
        if (s==null || s.length()==0) {
            return;
        }
        drawV(s);
        col = col + 1;
        row = 1;
        curx += f.getHeight(); // assume 方塊字
        cury = y + fontHeight;
    }

    /**
     * Draw a string vertically, wrap to right if necessary.
     */
    public void drawV(String s) {
        int i, strlen;
        char[] chars;
        if (s==null || s.length()==0) {
            return;
        }
        strlen = s.length();
        chars = new char[strlen];
        s.getChars(0,strlen,chars,0);
        for (i=0; i < strlen; i++) {
            if ((cury-y) > height) {
                cury = y + fontHeight;
                col++;
                row = 1;
                if (fromRight) {
                    curx -= f.getHeight();
                } else {
                    curx += f.getHeight();
                }
            }
            g.drawChars(chars,i,1,curx,cury);
            cury +=fontHeight;
        }
    }

    /**
     * Draw a string horizontally, wrap down if necessary. Rest col to 1 after drawing.
     */
    public void drawLineH(String s) {
        if (s==null || s.length()==0) {
            return;
        }
        drawH(s);
        col = 1;
        row = row + 1;
        curx = x;
        cury += fontHeight;
    }

    /**
     * Draw a String horizontally, wrap down if necessary.
     */
    public void drawH(String s) {
        int i, strlen;
        char[] chars;
        if (s==null || s.length()==0) {
            return;
        }
        strlen = s.length();
        chars = new char[strlen];
        s.getChars(0,strlen,chars,0);
        f = g.getFontMetrics();
        for (i=0; i < strlen; i++) {
            if (chars[i]=='\n') {
               curx = x;
               col = 1;
               row = row + 1;
               cury += fontHeight;
               continue;
            }
            if (((curx-x)+f.charWidth(chars[i])) > width) {
                curx = x;
                col = 1;
                row = row + 1;
                cury += fontHeight;
            }
            g.drawChars(chars,i,1,curx,cury);
            curx += f.charWidth(chars[i]);
        }
    }

    /**
     * Fill a string vertically in the specified area, with proper space between characters.
     */
    public static void fillV(Graphics g, String s, int x, int y, int width, int height) {
        int vlen, hlen, i, strlen;
        FontMetrics f;
        double scale, currentPos;
        char[] chars;
        if (s==null || s.length()==0) {
            return;
        }
        f = g.getFontMetrics();
        strlen = s.length();
        vlen = f.getHeight();
        chars = new char[strlen];
        s.getChars(0,strlen,chars,0);
        if (strlen == 1) {
            g.drawString(s,(int)(x+(width-f.charWidth(chars[0])+1)/2),(int)(y+(height-vlen)/2+f.getHeight()));
        } else {
            scale = (double)(height-vlen*strlen)/(strlen);
            for (i=0, currentPos=y+f.getHeight()+scale/2; i < strlen; i++) {
                g.drawChars(chars,i,1,(int)(x+(width-f.charWidth(chars[i])+1)/2),(int)currentPos);
                currentPos += scale + vlen;
            }
        }
    }

    public static void drawCenter(Graphics g, String s, int x, int y) {
        int len, i, strlen;
        FontMetrics f;
        if (s==null || s.length()==0) {
            return;
        }
        f = g.getFontMetrics();
        len = f.stringWidth(s);
        g.drawString(s,x-(len+1)/2,y);
    }
    /**
     * Fill a string horizontally in the specified area, with proper space between characters.
     */
    public static void fillH(Graphics g, String s, int x, int y, int width, int height) {
        int len, i, strlen;
        FontMetrics f;
        if (s==null || s.length()==0) {
            return;
        }
        f = g.getFontMetrics();
        len = f.stringWidth(s);
        strlen = s.length();
        if (strlen==1) {
            len=f.stringWidth(s);
            g.drawString(s,(int)(x+(width-len+1)/2),y+(height+f.getHeight())/2);
        } else {
            char[] chars = new char[strlen];
            s.getChars(0,strlen,chars,0);
            for (i=0, len=0; i < strlen; i++) {
                len += f.charWidth(chars[i]);
            }
            double scale = ((double)(width-len))/(strlen);
            double currentPoint;
            y += (height+f.getHeight())/2;
            for (i=0, currentPoint=x+(scale+1)/2; i < strlen; i++) {
                g.drawChars(chars,i,1,(int)currentPoint,y);
                currentPoint += scale + f.charWidth(chars[i]);
            }
        }
    }
}
```

### 印表範例

使用以上類別的印表程式 `PrintYLNoticeA4.java`  

```java
/* Program Name: PrintYLNoticeA4.java
 *    Subject: 列印學期成績單通知單(A4紙版本)
 *    Author: Shiuh-Sheng Yu
 *    Edit Date: 01/13/2001
 *    Last Update Date: 02/06/2002
 *    Last Update Date: 06/12/2003 add 停修
 *    Modify Date: 2004/07/07 加入網路輸入成績但尚未致送判別
 *    ToolKit: JDK1.1
 */
import ncnu.print.*;
import ncnu.sql.*;
import ncnu.rule.*;
import ncnu.gui.*;
import java.util.*;
import java.sql.*;
import java.awt.*;
public class PrintYLNoticeA4 extends Print {
    static String mynote="註A 全班成績未到\n註B 個人成績未到(請向老師查明)\n註C 請假\n註D 缺考\n註E 停修";
    static String explain="說明：學生學期學業成績以各科任課教師送交本處之原始成績冊為憑；本通知單僅為提供參考，若需正式證明請至本處註冊組申請成績單。\n";
    static String explain2="\n※學生學期學業成績有下列情形者，應予退學：\n一、一般生：學期學業成績累計兩學期不及格科目之學分數，均達        各該學期修習學分總數二分之一者。\n二、僑生、外國生：學期學業成績累計三學期不及格科目之學分數        ，均達各該學期修習學分總數二分之一者。\n三、視、聽、語障礙生：學期學業成績累計三學期不及格科目之學        分數，均達各該學期修習學分總數三分之二者。\n(詳細規定見本校學則第三十二條)";
    String year;
    String period;
    public PrintYLNoticeA4(Connection con) {
        super(con);
    }
    public void addQueryCondition() {
        fg.setTitle("列印學期成績通知單");
        int[] col = {12,10};
        gui = fg.newTable(col);
        String acaYear = NCNUmisc.getAcaYear();
        String year = acaYear.substring(0,acaYear.length()-1);
        String period = acaYear.substring(acaYear.length()-1, acaYear.length());
        gui.addLabel("學年");
        gui.addTextField("year",year,3, 3,Types.CHAR);
        gui.addLabel("學期");
        gui.addTextField("period",period,1, 1,Types.CHAR);
        gui.addLabel("系所");
        gui.addTextField("deptid", "",3,3,Types.CHAR);
        gui.addLabel("班(部)別");
        gui.addTextField("class", "", 1, 1, Types.CHAR);
        gui.addLabel("年級");
        gui.addTextField("grade","",1,1,Types.CHAR);
        gui.addLabel("開始學號");
        gui.addTextField("startid","",9, 9,Types.CHAR);
        gui.addLabel("結束學號");
        gui.addTextField("endid","",9, 9,Types.CHAR);
        gui.addLabel("備註");
        gui.addTextArea("note",mynote, true);
    }

    protected Vector getData() {
        String returnString, query;
        Statement stmt;
        ResultSet rs;

        year = gui.getTextField("year").trim();
        period = gui.getTextField("period").trim();
        String startid = gui.getTextField("startid").trim();
        String endid = gui.getTextField("endid").trim();
        String classid = gui.getTextField("class").trim();
        String deptid = gui.getTextField("deptid").trim();
        String grade = gui.getTextField("grade").trim();
        mynote = gui.getTextField("note");
        try {
            stmt  = con.createStatement();
            StringBuffer sb = new StringBuffer("select S.studentid ,D.cname ,S.name, S.pname, S.grade,S.addr2, S.zipcode2,S.addr1, S.zipcode1, S.deptid,S.class from student S,department D where S.deptid=D.deptid and S.status='0'");
            if (!startid.equals("")) {
                sb.append(" and S.studentid>='"+startid+"'");
            }
            if (!endid.equals("")) {
                sb.append(" and S.studentid<='"+endid+"'");
            }
            if (!classid.equals("")) {
                sb.append(" and S.class='"+classid+"'");
            }
            if (!deptid.equals("")) {
                sb.append(" and S.deptid='"+deptid+"'");
            }
            if (!grade.equals("")) {
                sb.append(" and S.grade='"+grade+"'");
            }
            sb.append(" order by S.deptid,S.class,S.grade");
            rs = stmt.executeQuery(sb.toString());
            Vector ids = new Vector();
            String[] stuinfo;
            String tmp;
            while (rs.next()) {
                stuinfo= new String[11];
                stuinfo[7] = year;
                stuinfo[8] = period;
                stuinfo[6] = rs.getString(1); // 學號
                stuinfo[3] = SQL.fromSQL(rs.getString(2)); // 系所
                stuinfo[5] = SQL.fromSQL (rs.getString(3)); // student name
                tmp = SQL.fromSQL( rs.getString(4));
                tmp = tmp == null ? "" : tmp.trim ();
                //若家長欄為空，用學生名
                stuinfo[2] = tmp.equals ("") ? stuinfo[5] : tmp; //收件者姓名
                stuinfo[4] = rs.getString(5); // 年級
                String currentAddr = SQL.fromSQL(rs.getString(6)).trim(); // 通訊地址
                String currentZip = SQL.fromSQL(rs.getString(7)).trim(); // 通訊地址zipcode
                String perAddr = SQL.fromSQL(rs.getString(8)).trim(); // 戶籍地址
                String perZip = SQL.fromSQL(rs.getString(9)).trim(); // 戶籍地址zipcode
                if (!currentAddr.equals("")){
                    stuinfo[0] = currentAddr;
                    stuinfo[1] = currentZip;
                } else {
                    stuinfo[0] = perAddr;
                    stuinfo[1] = perZip;
                }
                stuinfo[9] = SQL.fromSQL(rs.getString(10)).trim(); // deptid
                stuinfo[10] = SQL.fromSQL(rs.getString(11)); // division
                ids.addElement (stuinfo);
            }
            rs.close();
            Vector data = new Vector();
            for (int i=0; i < ids.size(); i++) {
                String[] info = (String[])ids.elementAt(i);
                YLNoticeStudent stu = new YLNoticeStudent();
                stu.info = info;
                query="select E.courseid,E.year,E.score,C.cname,C.credit,C.division, opened.senddate,E.netscore" +
                    " from selected E, opened, course C " +
                    " where E.courseid=opened.courseid and E.year=opened.year and E.class=opened.class and E.courseid=C.courseid" +
                    " and E.studentid=\'"+info[6]+"\'"+" and E.year=\'"+year+period+"\' order by C.cname asc";
                rs = stmt.executeQuery(SQL.toSQL(query));
                while (rs.next()) {
                    String _courseid = SQL.fromSQL(rs.getString(1)).trim();
                    String _year=SQL.fromSQL(rs.getString(2)).trim();
                    double score = rs.getDouble(3);
                    String cname = SQL.fromSQL(rs.getString(4)).trim();
                    double credit = rs.getDouble(5);
                    String division = SQL.fromSQL(rs.getString(6));
                    String type = "選修";
                    String senddate = SQL.toString(rs.getBytes(7));
                    String netscore = SQL.toString(rs.getBytes(8));
                    // 網路輸入成績,尚未致送教務處
                    if (senddate.equals("") && netscore.equals("Y")) {
                        score = -1;
                    }
                    stu.instLearning (type, cname, credit, score, _courseid, _year,division);
                }
                rs.close();
                for(int j = 0; j < stu.learning.size (); j++){
                    String[] learning = (String[])stu.learning.elementAt (j);
                    int entrydate=Integer.parseInt(NCNUmisc.getEnterYear(info[6]).trim());
                    //查詢必修
                    query="select * from require where courseid=\'"+learning[4]+"\'"+" and deptid=\'"+info[9]+"\'"+" and start<="+entrydate+" and stop>="+entrydate;
                    rs = stmt.executeQuery(query);
                    if (rs.next()) learning[0] = "必修";
                    rs.close();
                    //判斷此科目是否為雙主修的必修
                    query="select deptid from  whodouble where studentid=\'"+info[6]+"\'";
                    rs = stmt.executeQuery(query);
                    if (rs.next()){
                        String _deptid2= SQL.fromSQL(rs.getString(1)).trim();
                        rs.close();
                        query="select * from require where courseid=\'"+learning[4]+"\'"+" and deptid=\'"+_deptid2+"\'"+" and start<="+entrydate+" and stop>="+entrydate;
                        rs = stmt.executeQuery(query);
                        if (rs.next()) learning[0] = "必修";
                    }
                    rs.close();
                    //判斷此科目是否為輔系的必修
                    query="select deptid from  whoassist where studentid=\'"+info[6]+"\'";
                    rs = stmt.executeQuery(query);
                    if (rs.next()){
                        String _deptid3= SQL.fromSQL(rs.getString(1)).trim();
                        rs.close();
                        query="select * from require where courseid=\'"+learning[4]+"\'"+" and deptid=\'"+_deptid3+"\'"+" and start<="+entrydate+" and stop>="+entrydate;
                        rs = stmt.executeQuery(query);
                        if (rs.next()) learning[0] = "必修";
                    }
                    rs.close();
                    if (learning[6].equals("B") && (stu.info[10].equals("G")||stu.info[10].equals("P"))) {
                        learning[0] = "下修";
                    }
                }
                query="select score from conduct where studentid=\'"+info[6]+"\'"+" and year=\'"+year+period+"\'";
                rs = stmt.executeQuery(query);
                while (rs.next()){
                    stu.bscore = rs.getDouble(1);
                }
                rs.close();
                data.addElement(stu);
            }
            stmt.close();
            con.commit();
            return data;
        } catch (Exception ex) {
            new ErrorDialog(fg,SQL.fromSQL (ex.toString()) );
            ex.printStackTrace ();
            return null;
        }
    }
    protected PrintCanvas createCanvas(Vector data) {
        return new PrintYLNoticeCanvas(data);
    }
    class YLNoticeStudent{
        //   0,  1,    2,   3,    4,    5,        6,   7,     8,     9
        // add,zip,pname,dept,grade,sname,studentid,year,period,deptid
        public String[] info;
        public Vector learning;
        public YLNoticeStudent(){
            learning = new Vector();
            tgcredit = 0;
            tscore = 0;
            course = 0;
            knownCredit = 0;
            bscore = -1;
        }
        private String doubleString(double x) {
            String tmp = Double.toString(x);
            if (tmp.equals(".0")) {
                return "0";
            }
            if (tmp.endsWith(".0")) {
                return tmp.substring(0,tmp.length()-2);
            }
            return tmp;
        }
        public void instLearning(String type, String name
            , double credit, double score, String courseid, String year, String division) {
            course ++;
            String[] s = new String[7];
            s[0] = type;
            s[1] = name;
            s[2] = doubleString(credit);
            s[4] = courseid;
            s[5] = year;
            s[6] = division;
            tcredit += credit;
            if (score == -1){
                s[3] = "註A";
            } else if (score == -2) {
                s[3] = "註B";
            } else if (score == -3) {
                s[3] = "註C";
            } else if (score == -4) {
                s[3] = "註D";
            } else if (score == -5) {
                s[3] = "註E";
            } else {
                s[3] = doubleString (score);
                if (division.equals("B") && (info[10].equals("G")||info[10].equals("P"))) {
                    // 下修
                } else {
                    knownCredit += credit;
                    tscore += score*credit;
                    if (score >= 70 || (score>=60 && !(info[10].equals("G") || info[10].equals("P")))) {
                        tgcredit += credit; //得到學分
                    }
                }
            }
            learning.addElement (s);
        }
    
        public double getAvgScore() {
            return knownCredit == 0 ? 0 : ((int)(tscore/knownCredit*100+0.5))/100.0;
        }
        public double tcredit;   //修習總學分數
        public double tgcredit;  //實得學分數
        public double tscore; //總學分數
        public int course; //學習課堂數
        public double knownCredit;
        public double bscore;//操行
    }
    class PrintYLNoticeCanvas extends PrintCanvas {
        public String getMedia() {
            return "A4";
        }
        public String getOrientation() {
            return "portrait";
        }
        protected int dataPerPage(int pageNum) {
            return 1;
        }
        protected int pagesPerData(Object data) {
            return 1;
        }
        public PrintYLNoticeCanvas(Vector data) {
            super(data);
        }
        public void paint(Graphics g) {
            CharArea ca;
            String title="";
            try {
                title = Environment.getTitle()+NCNUmisc.chineseDigit(Integer.parseInt(year))+"學年度第"+NCNUmisc.chineseDigit2(Integer.parseInt(period))+"學期成績通知單";
            } catch(Exception ex) {}
            YLNoticeStudent stu = (YLNoticeStudent)data.elementAt(getDataIndex());
            String tmp ;
            //  寄件地址 郵遞區號 收件者姓名 系所 年級 學生名 學號 學年 學期
            // 成績框
            g.drawRect((int)(0.3*dotPerInch),(int)((1.2)*dotPerInch),(int)(3.6*dotPerInch),(int)(9.1*dotPerInch)+1);
            g.drawLine((int)(0.57*dotPerInch),(int)(1.2*dotPerInch),(int)(0.57*dotPerInch),(int)(8.6*dotPerInch));
            g.drawLine((int)(3.3*dotPerInch),(int)(1.2*dotPerInch),(int)(3.3*dotPerInch),(int)(9.4*dotPerInch));
            g.drawLine((int)(3.6*dotPerInch),(int)(1.2*dotPerInch),(int)(3.6*dotPerInch),(int)(8.6*dotPerInch));
            g.drawLine((int)(0.3*dotPerInch),(int)(1.4*dotPerInch),(int)(3.9*dotPerInch),(int)(1.4*dotPerInch));
            g.drawLine((int)(0.3*dotPerInch),(int)(8.6*dotPerInch),(int)(3.9*dotPerInch),(int)(8.6*dotPerInch));
            g.drawLine((int)(0.3*dotPerInch),(int)(8.8*dotPerInch),(int)(3.9*dotPerInch),(int)(8.8*dotPerInch));
            g.drawLine((int)(0.3*dotPerInch),(int)(9*dotPerInch),(int)(3.9*dotPerInch),(int)(9*dotPerInch));
            g.drawLine((int)(0.3*dotPerInch),(int)(9.2*dotPerInch),(int)(3.9*dotPerInch),(int)(9.2*dotPerInch));
            g.drawLine((int)(0.3*dotPerInch),(int)(9.4*dotPerInch),(int)(3.9*dotPerInch),(int)(9.4*dotPerInch));
            g.setFont(new Font("Serif",Font.PLAIN,(int)(0.14*dotPerInch)));
            g.drawString(title,(int)(0.3*dotPerInch),(int)((0.6)*dotPerInch));
            g.setFont(new Font("Serif",Font.PLAIN,(int)(0.12*dotPerInch)));
            g.drawString("系所別:"+stu.info[3],(int)(0.3*dotPerInch),(int)(0.8*dotPerInch));
            g.drawString("學號:"+stu.info[6],(int)(0.3*dotPerInch),(int)(1.0*dotPerInch));
            g.drawString("年級:"+stu.info[4],(int)(2.3*dotPerInch),(int)(0.8*dotPerInch));
            g.drawString("姓名:"+stu.info[5],(int)(2.3*dotPerInch),(int)(1.0*dotPerInch));
            g.drawString("修別",(int)(0.32*dotPerInch),(int)(1.35*dotPerInch));
            g.drawString("課程名稱",(int)(0.9*dotPerInch),(int)(1.35*dotPerInch));
            g.drawString("學分",(int)(3.33*dotPerInch),(int)(1.35*dotPerInch));
            g.drawString("成績",(int)(3.63*dotPerInch),(int)(1.35*dotPerInch));
            CharArea.fillH(g,"學業成績總平均",(int)(0.3*dotPerInch),(int)(8.55*dotPerInch),(int)(3.1*dotPerInch),(int)(0.2*dotPerInch));
            CharArea.fillH(g,"修習總學分數",(int)(0.3*dotPerInch),(int)(8.75*dotPerInch),(int)(3.1*dotPerInch),(int)(0.2*dotPerInch));
            CharArea.fillH(g,"實得學分數",(int)(0.3*dotPerInch),(int)(8.95*dotPerInch),(int)(3.1*dotPerInch),(int)(0.2*dotPerInch));
            CharArea.fillH(g,"操行成績",(int)(0.3*dotPerInch),(int)(9.15*dotPerInch),(int)(3.1*dotPerInch),(int)(0.2*dotPerInch));
            ca = new CharArea(g,(int)(0.4*dotPerInch),(int)(9.4*dotPerInch),(int)(3.4*dotPerInch),(int)(1.2*dotPerInch));
            ca.drawH("註A 全班成績未到\n註B 個人成績未到(請向老師查明)\n註C 請假\n註D 缺考\n註E 停修");
            Calendar d = Calendar.getInstance ();
            g.drawString("填表日期: " + NCNUmisc.chineseDigit(d.get (Calendar.YEAR ) -1911)+" 年 " + NCNUmisc.chineseDigit(d.get (Calendar.MONTH )+1) + " 月 " + NCNUmisc.chineseDigit(d.get (Calendar.DATE )) + " 日" ,(int)(0.3*dotPerInch),(int)(10.5*dotPerInch));
            for(int i = 0; i < stu.learning.size (); i++){
                String[] data = (String[])stu.learning.elementAt(i);
                g.drawString (data[0], (int)((0.32)*dotPerInch), (int)((1.5 + i*0.14)*dotPerInch));
                g.drawString (data[1], (int)((0.6)*dotPerInch), (int)((1.5 + i*0.14)*dotPerInch));
                g.drawString (data[2], (int)((3.4)*dotPerInch), (int)((1.5 + i*0.14)*dotPerInch));
                g.drawString (data[3], (int)((3.65)*dotPerInch), (int)((1.5 + i*0.14)*dotPerInch));
            }
            g.drawString ("" + NCNUmisc.doubleString(stu.getAvgScore()), (int)((3.5)*dotPerInch), (int)((8.75)*dotPerInch));
            g.drawString ("" + NCNUmisc.doubleString(stu.tcredit), (int)((3.5)*dotPerInch), (int)((8.95)*dotPerInch));
            g.drawString ("" + NCNUmisc.doubleString(stu.tgcredit), (int)((3.5)*dotPerInch), (int)((9.15)*dotPerInch));
            g.drawString (stu.bscore < 0 ? "未到" : ""+NCNUmisc.doubleString(stu.bscore), (int)((3.5)*dotPerInch), (int)((9.35)*dotPerInch));
            g.drawRect((int)(4.4*dotPerInch),(int)(2.4*dotPerInch),(int)(0.4*dotPerInch),(int)(0.7*dotPerInch));
            ca = new CharArea(g,(int)(4.3*dotPerInch),(int)(6.0*dotPerInch),(int)(3.5*dotPerInch),(int)(4.*dotPerInch));
            ca.drawH(explain);
            if (Environment.getTitle().equals("國立暨南國際大學")) {
                ca.drawH(explain2);
            }
            g.drawRect((int)(4.3*dotPerInch),(int)((0.5)*dotPerInch),(int)(3.5*dotPerInch),(int)(1.8*dotPerInch));
            ca = new CharArea(g,(int)(4.4*dotPerInch),(int)(0.8*dotPerInch),(int)(3.4*dotPerInch),(int)(1.8*dotPerInch));
            g.setFont(new Font("Serif",Font.PLAIN,(int)(0.14*dotPerInch)));
            CharArea.fillV(g,"印刷品",(int)(4.4*dotPerInch),(int)(2.4*dotPerInch),(int)(0.4*dotPerInch),(int)(0.7*dotPerInch));
            g.drawString(Environment.getTitle()+"教務處  寄",(int)(5.0*dotPerInch),(int)(2.55*dotPerInch));
            g.drawString("校址:"+Environment.addr,(int)(5.0*dotPerInch),(int)(2.8*dotPerInch));
            ca.drawH(stu.info[1]+" "+stu.info[0]+"\n\n");
            g.setFont(new Font("Serif",Font.BOLD,(int)(0.20*dotPerInch)));
            ca.drawH("    "+stu.info[2]+"  君啟");
        }
    }
}
```

#### 執行後的結果

![執行後的結果](https://github.com/NCNU-CALab/java.programming.im/raw/master/images/print1.jpg)