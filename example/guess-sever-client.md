以下是 [GuessClient.java](#client) 與 [GuessServer.java](#server)

<p id="client"></p>  

```java
/**
 * Program: GuessClient.java
 * Author: Shiuh-Sheng Yu
 *         Department of Information Management
 *         National ChiNan University
 * Last Update Date: 04/30/2000
 */
import java.io.*;
import java.net.*;
public class GuessClient {
    byte num[][];
    boolean checked[];
    byte[] studentID = {8,7,3,2,1,0,0,6};
    void init () {
        num = new byte[5040][4];
        checked = new boolean[5040];
        int m=0;
        for (byte i=0; i<=9; i++) {
            for (byte j=0; j<=9; j++) {
                if (j==i) continue;
                for (byte k=0; k<=9; k++) {
                    if (k==i || k==j)  continue;
                    for (byte l=0; l<=9; l++) {
                        if (l==i || l==j || l==k) continue;
                        num[m][0] = i;
                        num[m][1] = j;
                        num[m][2] = k;
                        num[m++][3] = l;
                    }
                }
            }
        }
    }
    void reset() {
        for (int i=0; i<5040; i++) checked[i] = false;
    }
    public GuessClient() {
        init();
        byte[] ans = new byte[2];
        try {
            Socket s = new Socket("163.22.20.114",1234);
            InputStream in = s.getInputStream();
            OutputStream out = s.getOutputStream();
            out.write(studentID);
            for (int i=0; i<100; i++) {
                reset();
                while (true) {
                    byte[] guess = guess();
                    out.write(guess);
                    in.read(ans);
                    if (ans[0]==4 && ans[1]==0) break;
                    eliminate(guess, ans);
                }
            }
            in.close();
            out.close();
            s.close();
        } catch(Exception e) {
            System.out.println(e);
        }
    }
    public byte[] guess() {
        for (int i=0; ; i++) {
            if (!checked[i]) {
                checked[i] = true;
                return num[i];
            }
        }
    }
    private boolean match(byte[] pre, byte[] x, byte[] answer) {
        int a=0, b=0;
        for (int i=0; i<4; i++)
            for (int j=0; j<4; j++)
                if (pre[j] == x[i]) {
                    if (i==j) a++;
                    else      b++;
                    break;
                }
        return(answer[0]==a && answer[1]==b);
    }
    public void eliminate(byte[] guess, byte[] answer) {
        for (int i=0; i<checked.length; i++)
            if (!checked[i] && !match(guess, num[i], answer)) checked[i] = true;
    }
    public static void main(String argv[]) {
        new GuessClient();
    }
}
```

<p id="server"></p>  

```java
/**
 * Program: GuessServer.java
 * Author: Shiuh-Sheng Yu
 *         Department of Information Management
 *         National ChiNan University
 * Last Update Date: 04/30/2000
 */
import java.net.*;
import java.io.*;
public class GuessServer {
    PrintWriter outFile;
    byte[][] num;
    public GuessServer() {
        try {
            ServerSocket listenSocket = new ServerSocket(1234);
            outFile = new PrintWriter(new FileOutputStream("guess.log",true));
            init();
            while (true) {
                new GuessConnection(listenSocket.accept(), this);
            }
        } catch (IOException e) {
            System.out.println(e);
            System.exit(0);
        }
    }
    private void init() {
        num = new byte[5040][4];
        int m=0;
        for (byte i=0; i<=9; i++) {
            for (byte j=0; j<=9; j++) {
                if (j==i) {
                    continue;
                }
                for (byte k=0; k<=9; k++) {
                    if (k==i || k==j) {
                        continue;
                    }
                    for (byte l=0; l<=9; l++) {
                        if (l==i || l==j || l==k) {
                            continue;
                        }
                        num[m][0] = i;
                        num[m][1] = j;
                        num[m][2] = k;
                        num[m][3] = l;
                        m++;
                    }
                }
            }
        }
    }
    public static void main(String[] argv) {
        new GuessServer();
    }
}

class GuessConnection extends Thread {
    Socket client;
    GuessServer master;
    OutputStream out;
    InputStream in;
    String hostName;
    byte[] studentID;
    public GuessConnection(Socket clientSocket, GuessServer parent) {
        client = clientSocket;
        master = parent;
        try {
            out = client.getOutputStream();
            in = client.getInputStream();
            hostName = client.getInetAddress().getHostAddress();
            studentID = new byte[8];
        } catch (Exception e) {
            System.out.println("Read from "+hostName+" failed");
            try {
                client.close();
            } catch (Exception ex) {}
            return;
        }
        this.setPriority(Thread.MIN_PRIORITY);
        this.start();
    }
    public void run() {
        int guessNum=0;
        byte[] inputNum = new byte[4];
        try {
            in.read(studentID);
            for (int i=0; i<100; i++) { // run 100 times to evaluate performance
                int answer = (int)(Math.random()*5040);
                while(true) {
                    in.read(inputNum);
                    guessNum++;
                    byte[] rel = nanb(inputNum, master.num[answer]);
                    out.write(rel);
                    if (rel[0]==4 && rel[1]==0) {
                        break;
                    }
                }
            }
            master.outFile.println((char)(studentID[0]+'0')+""+
                                   (char)(studentID[1]+'0')+""+
                                   (char)(studentID[2]+'0')+""+
                                   (char)(studentID[3]+'0')+""+
                                   (char)(studentID[4]+'0')+""+
                                   (char)(studentID[5]+'0')+""+
                                   (char)(studentID[6]+'0')+""+
                                   (char)(studentID[7]+'0')+" "+guessNum+" "+hostName);
            master.outFile.flush();
        } catch(Exception e) {
        } finally {
            try {
                in.close();
                out.close();
                client.close();
            } catch(Exception ex) {
            }
        }
    }
    static private byte[] nanb(byte[] x, byte[] y) {
        byte[] rel = new byte[2];
        rel[0] = rel[1] = 0;
        for (int i=0; i<4; i++) {
            for (int j=0; j<4; j++) {
                if (y[j]==x[i]) {
                    if (i==j) {
                        rel[0]++;
                    } else {
                        rel[1]++;
                    }
                    break;
                }
            }
        }
        return rel;
    }
}
```
