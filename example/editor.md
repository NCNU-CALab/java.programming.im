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