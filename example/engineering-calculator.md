```java
/**
 * Author: Shiuh-Sheng Yu
 * Last Update: 4/21/1998
 * Modified Date: 03/31/2003
 */
import java.awt.*;
import java.awt.event.*;
public class Calc implements ActionListener {
    private Frame f;
    private TextField display;
    public Calc() {
        Font font = new Font("Serif",Font.PLAIN,16);
        f = new Frame("小算盤");
        MenuBar mb = new MenuBar();
        mb.setFont(font);
        f.setMenuBar(mb);
        Menu m = new Menu("說明");
        mb.add(m);
        MenuItem mi = new MenuItem("about");
        m.add(mi);
        f.addWindowListener(new CloseWindow(f,true));
        mi.addActionListener(this);
        f.setLayout(new GridBagLayout());
        f.setFont(font);
        display = new TextField(40);
        AddConstraint.addConstraint(f,display,0,0,1,1,
            GridBagConstraints.HORIZONTAL, GridBagConstraints.NORTHWEST,
            1,0,0,0,0,0);
        Panel funButtons = new Panel();
        AddConstraint.addConstraint(f,funButtons,0,1,1,1,
            GridBagConstraints.BOTH, GridBagConstraints.NORTHWEST,
            1,1,0,0,0,0);
        funButtons.setLayout(new GridLayout(1,1));
        ((Button)funButtons.add(new Button("clear"))).addActionListener(this);
        Panel keyButtons = new Panel();
        keyButtons.setLayout(new GridLayout(6,5));
        AddConstraint.addConstraint(f,keyButtons,0,2,1,1,
            GridBagConstraints.BOTH, GridBagConstraints.NORTHWEST,
            1,1,0,0,0,0);
        String[] buttons = {"7","8","9","/","^","4","5","6","*","%","1","2","3","-",".","0","+","=","sqrt","abs","acos","asin","atan","cos","sin","tan","exp","log","round"};
        for (int i=0; i<buttons.length; i++) {
            ((Button)keyButtons.add(new Button(buttons[i]))).addActionListener(this);
        }
        f.pack();
        f.show();
    }
    public static void main(String argv[]) {
        new Calc();
    }
    public void actionPerformed(ActionEvent e) {
        String see = display.getText();
        String rel;
        int i = display.getCaretPosition();
        String command = e.getActionCommand();
        if (command.equals("=")) {
            rel = eval(display.getText());
            display.setText(rel);
            display.setCaretPosition(rel.length());
        } else if (command.length()==1) { // 數字按鍵
            display.setText(see.substring(0,i)+command+see.substring(i));
            display.setCaretPosition((see.substring(0,i)+command).length());
        } else if (command.equals("clear")) {
            display.setText("");
        } else if (command.equals("about")) {
            new About(f,"俞旭昇寫好玩的");
        } else { // 其他函數
            display.setText(see.substring(0,i)+command+"()"+see.substring(i));
            display.setCaretPosition((see.substring(0,i)+command).length()+1);
        }
        display.requestFocus();
    }
    public static final int SQRT = 0;
    public static final int ABS = 1;
    public static final int ACOS = 2;
    public static final int ASIN = 3;
    public static final int ATAN = 4;
    public static final int COS = 5;
    public static final int SIN = 6;
    public static final int TAN = 7;
    public static final int EXP = 8;
    public static final int LOG = 9;
    public static final int ROUND = 10;
    public static final int PLUS     = 0;
    public static final int MINUS    = 1;
    public static final int MULTIPLY = 2;
    public static final int DIVIDE   = 3;
    public static final int MODE     = 4;
    public static final int POWER    = 5;
    public static final int FUN      = 6;
    public static final int LEFT_ROUND_BRACKET = 7;
    public static final int RIGHT_ROUND_BRACKET = 8;
    public static final int END = 9;
    public static final int LITERAL = 10;
    public static final int ERROR = 11;
    public static final int PUSH = 0;
    public static final int POP = 1;
    public static final int ELIMINATE = 2;
    public static final int DEAD = 3;
    public static final int TERMINATE = 4;
    public static final int table[][] = { // add more elements
    //   +     -    *    /    %    ^   FUN  (         )    end
        {POP, POP, POP, POP, POP, POP, POP, PUSH,     DEAD,PUSH},    // +
        {POP, POP, POP, POP, POP, POP, POP, PUSH,     DEAD,PUSH},    // -
        {PUSH,PUSH,POP, POP, POP, POP, POP, PUSH,     DEAD,PUSH},    // *
        {PUSH,PUSH,POP, POP, POP, POP, POP, PUSH,     DEAD,PUSH},    // /
        {PUSH,PUSH,POP, POP, POP, POP, POP, PUSH,     DEAD,PUSH},    // %
        {PUSH,PUSH,PUSH,PUSH,PUSH,PUSH,POP, PUSH,     DEAD,PUSH},    // ^
        {PUSH,PUSH,PUSH,PUSH,PUSH,PUSH,DEAD,PUSH,     DEAD,PUSH},    // FUN
        {PUSH,PUSH,PUSH,PUSH,PUSH,PUSH,PUSH,PUSH,     DEAD,PUSH},    // (
        {POP, POP, POP, POP, POP, POP, POP, ELIMINATE,DEAD,DEAD},    // )
        {POP, POP, POP, POP, POP, POP, POP, DEAD,     DEAD,TERMINATE}// end
    };
    String eval(String expression) {
        Expression exp = new Expression(expression);
        IntStack operator_stack = new IntStack();
        DoubleStack operand_stack = new DoubleStack();
        Token token;
        int act, op;
        double op1,op2,tmp;
        operator_stack.push(END); // put sentinel
        for (;;) {
            token = exp.next_token();
            if (token.type == ERROR) {
                return "算式不合法";
            }
            if (token.type == LITERAL) {
                operand_stack.push(Double.valueOf(token.data).doubleValue());
            } else {
                act = table[token.type][operator_stack.peek()];
                while (act == POP) { // pop out all higher priority operator
                    op2 = operand_stack.pop();
                    op1 = operand_stack.pop();
                    tmp = 0;
                    op = operator_stack.pop();
                    if (op == PLUS) {
                        tmp = op1 + op2;
                    } else if (op == MINUS) {
                        tmp = op1 - op2;
                    } else if (op == MULTIPLY) {
                        tmp = op1 * op2;
                    } else if (op == MODE) {
                        if ((int)op2 != 0) {
                            tmp = (int)op1 % (int)op2;
                        } else {
                            return "zero divisor";
                        }
                    } else if (op == DIVIDE) {
                        if (op2 != 0) {
                            tmp = op1 / op2;
                        } else {
                            return "zero divisor";
                        }
                    } else if (op == POWER) {
                        tmp = Math.pow(op1,op2);
                    } else if (op == FUN) {
                        if (op1 == SQRT) {
                            tmp = Math.sqrt(op2);
                        } else if (op1 == ABS) {
                            tmp = Math.abs(op2);
                        } else if (op1 == ACOS) {
                            tmp = Math.acos(op2);
                        } else if (op1 == ASIN) {
                            tmp = Math.asin(op2);
                        } else if (op1 == ATAN) {
                            tmp = Math.atan(op2);
                        } else if (op1 == COS) {
                            tmp = Math.cos(op2);
                        } else if (op1 == SIN) {
                            tmp = Math.sin(op2);
                        } else if (op1 == TAN) {
                            tmp = Math.tan(op2);
                        } else if (op1 == EXP) {
                            tmp = Math.exp(op2);
                        } else if (op1 == LOG) {
                            tmp = Math.log(op2);
                        } else if (op1 == ROUND) {
                            tmp = Math.round(op2);
                        } else {
                            return("內部錯誤");
                        }
                    }
                    operand_stack.push(tmp);
                    act = table[token.type][operator_stack.peek()];
                }
                if (act == PUSH) {
                    operator_stack.push(token.type);
                    if (token.type == FUN) {
                        if (token.data.equals("sqrt")) {
                            operand_stack.push((double)SQRT);
                        } else if (token.data.equals("abs")) {
                            operand_stack.push((double)ABS);
                        } else if (token.data.equals("acos")) {
                            operand_stack.push((double)ACOS);
                        } else if (token.data.equals("asin")) {
                            operand_stack.push((double)ASIN);
                        } else if (token.data.equals("atan")) {
                            operand_stack.push((double)ATAN);
                        } else if (token.data.equals("cos")) {
                            operand_stack.push((double)COS);
                        } else if (token.data.equals("sin")) {
                            operand_stack.push((double)SIN);
                        } else if (token.data.equals("tan")) {
                            operand_stack.push((double)TAN);
                        } else if (token.data.equals("exp")) {
                            operand_stack.push((double)EXP);
                        } else if (token.data.equals("log")) {
                            operand_stack.push((double)LOG);
                        } else if (token.data.equals("round")) {
                            operand_stack.push((double)ROUND);
                        } else {
                            return("算式不合法");
                        }
                    }
                } else if (act == ELIMINATE) {
                    if (operator_stack.peek() == LEFT_ROUND_BRACKET) {
                        operator_stack.pop();
                    } else {
                        return"括弧不對稱";
                    }
                } else if (act == TERMINATE) {
                    return Double.toString(operand_stack.pop());
                } else { // DEAD
                    return("算式不合法");
                }
            }
        }
    }

}
class DoubleStack {
    private int top;
    private double data[];
    public DoubleStack() {
        top = 0;
        data = new double[100];
    }
    public void push(double val) {
        data[top++] = val;
    }
    public double pop() {
        return data[--top];
    }
    public double peek() {
        return data[top-1];
    }
}

class IntStack {
    private int top;
    private int data[];
    public IntStack() {
        top = 0;
        data = new int[100];
    }
    public void push(int val) {
        data[top++] = val;
    }
    public int pop() {
        return data[--top];
    }
    public int peek() {
        return data[top-1];
    }
}

class Token {
    int type;
    String data;
    Token(int t, String d) {
        type = t;
        data = d;
    }
}
class Expression {
    private int ptr;
    private String data;
    private boolean wantOperand = true;
    public Expression(String s) {
        ptr = 0;
        data = s;
    }
    public Token next_token() {
        int i;
        for (i = ptr; i < data.length() && data.charAt(i)==' '; i++) {
        }
        ptr = i;
        if (i == data.length()) {
            return new Token(Calc.END,"");
        }
        switch (data.charAt(i)) {
        case '+':
            ptr++;
            wantOperand = true;
            return new Token(Calc.PLUS,"+");
        case '*':
            ptr++;
            wantOperand = true;
            return new Token(Calc.MULTIPLY,"*");
        case '/':
            ptr++;
            wantOperand = true;
            return new Token(Calc.DIVIDE,"/");
        case '%':
            ptr++;
            wantOperand = true;
            return new Token(Calc.MODE,"%");
        case '^':
            ptr++;
            wantOperand = true;
            return new Token(Calc.POWER,"^");
        case '(':
            ptr++;
            wantOperand = true;
            return new Token(Calc.LEFT_ROUND_BRACKET,"(");
        case ')':
            ptr++;
            wantOperand = false;
            return new Token(Calc.RIGHT_ROUND_BRACKET,")");
        default:
            if (data.charAt(i) == '-' && !wantOperand) {
                ptr++;
                wantOperand = true;
                return new Token(Calc.MINUS,"-");
            }
            StringBuffer buf = new StringBuffer();
            if (data.charAt(i)=='-'
                ||(data.charAt(i)>='0' && data.charAt(i)<='9')
                || data.charAt(i)=='.') {
                wantOperand = false;
                if (data.charAt(i) == '-') {
                    buf.append(data.charAt(i++));
                }
                while (i<data.length()
                       && data.charAt(i)>='0'
                       && data.charAt(i)<='9') {
                    buf.append(data.charAt(i++));
                }
                if (i<data.length() && data.charAt(i) == '.') {
                    buf.append(data.charAt(i++));
                    while (i<data.length()
                        && data.charAt(i)>='0'
                        && data.charAt(i)<='9') {
                        buf.append(data.charAt(i++));
                    }
                }
                if (i<data.length() && data.charAt(i) == 'E') {
                    buf.append(data.charAt(i++));
                    if (i<data.length() && data.charAt(i) == '-') {
                        buf.append(data.charAt(i++));
                    }
                    while (i<data.length()
                        && data.charAt(i)>='0'
                        && data.charAt(i)<='9') {
                        buf.append(data.charAt(i++));
                    }
                }
                ptr = i;
                if (buf.length()==0) {
                    return new Token(Calc.ERROR,"");
                } else {
                    return new Token(Calc.LITERAL,buf.toString());
                }
            } else {
                while(i<data.length()
                    && data.charAt(i)>='a'
                    && data.charAt(i)<='z') {
                    buf.append(data.charAt(i++));
                }
                ptr = i;
                if (buf.length()==0) {
                    return new Token(Calc.ERROR,"");
                } else if (i>=data.length() || data.charAt(i)!='(') {
                    return new Token(Calc.ERROR,"");
                } else {
                    return new Token(Calc.FUN,buf.toString());
                }
            }
        }
    }
}

class About {
    public About(Frame f, String message) {
        Dialog d = new Dialog(f,true);
        d.addWindowListener(new CloseWindow(d,false));
        Label l = new Label(message);
        d.setLayout(new GridBagLayout());
        AddConstraint.addConstraint(d,l,0,0,1,1,
            GridBagConstraints.BOTH, GridBagConstraints.NORTHWEST,
            1,1,0,0,0,0);
        Button b = new Button("OK");
        AddConstraint.addConstraint(d,b,0,1,1,1,
            GridBagConstraints.BOTH, GridBagConstraints.NORTHWEST,
            1,1,0,0,0,0);
        b.addActionListener(new CloseWindow(d,false));
        d.pack();
        d.show();
    }
}
class CloseWindow extends WindowAdapter implements ActionListener {
    private Window target;
    private boolean exit;
    public CloseWindow(Window target, boolean exit) {
        super();
        this.target = target;
        this.exit = exit;
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
