### SMTP Client

```java
/* Program Name: SMTP.java
   Subject: 寄信程式 
   Editor: 俞旭昇 Shiuh-Sheng Yu
           National ChiNan University
           Department of Information Management
   Edit Date: 04/24/2000
   Last Update Date: 04/24/2000
   Modify Date: 2004/09/21, 加入mime content type
   Modify Date: 2005/09/16, 改用UTF-8來傳送資料
   Modify Date: 2005/09/22, 改回用big5 :(
   Modify Date: 2006/02/16, 失敗改retry, 利用queue來smooth需求量
   ToolKit: JDK1.1
*/
package ncnu.net;
import java.io.*;
import java.net.*;
import ncnu.rule.Environment;
import ncnu.gui.SpeedBar;
class Request {
    String sender;
    String[] receiver;
    String subject;
    String[] data;
}
class Queue {
    private Request[] data;
    int size;
    private int head;
    private int tail;
    public Queue() {
        size = head = tail = 0;
        data = new Request[3000];
    }
    public synchronized Request deQueue() {
        while (size==0) {
            try {
                wait();
            } catch(Exception ex) {};
        }
        Request tmp = data[head];
        data[head] = null;
        head = (head+1)%data.length;
        size--;
        if (size==data.length-1) {
            notifyAll();
        }
        return tmp;
    }
    public synchronized void enQueue(Request c) {
        while (size==data.length) {
            try {
                wait();
            } catch(Exception ex) {};
        }
        data[tail++] = c;
        tail %= data.length;
        size++;
        if (size==1) {
            notifyAll();
        }
    }
    public boolean isFull() {
        return size == data.length;
    }
}
public class SMTP extends Thread {
    private static Queue queue = new Queue();
    private static int nowHave;
    public static void showProgress() {
        SpeedBar sb = new SpeedBar("待送郵件數目");
        while (queue.size > 0) {
            sb.setSpeed(queue.size,"待送郵件數目:");
            try {
                Thread.currentThread().sleep(3000);
            } catch (Exception epp) {}
        }
        sb.dispose();
    }
    /**
     * Send email to a group of users. The method will be called by all mail fuctions
     * @param sender Mail sender's email account
     * @param resceiver Array of receivers' email account
     * @param subject email subject
     * @param data An array of String which constructs the content of the email
     */
    public static void send(String sender,String[] receiver,String subject,String[] data) {
        Request req = new Request();
        req.sender = sender;
        req.receiver = receiver;
        req.subject = subject;
        req.data = data;
        queue.enQueue(req);
        while (nowHave<5) {
            (new SMTP()).start();
            nowHave++;
        }
    }
    public void run() {
        for (;;) {
            Request tmp = queue.deQueue();
            String sender = tmp.sender;
            String[] receiver = tmp.receiver;
            String subject = tmp.subject;
            String[] data = tmp.data;
            for (;;) {
                try {
                    Socket smtps = new Socket(Environment.getMailServer(),25);
                    PrintStream smtpsOutput = new PrintStream(smtps.getOutputStream());
                    BufferedReader smtpsInput = new BufferedReader(new InputStreamReader(smtps.getInputStream()));
                    smtpsInput.readLine();
                    smtpsOutput.println("HELO "+Environment.getDomain());
                    smtpsOutput.flush();
                    smtpsInput.readLine();
                    smtpsOutput.println("MAIL FROM:"+sender);  // 寄 信 人 
                    smtpsOutput.flush();
                    smtpsInput.readLine();
                    for (int i=0; i");
                    smtpsOutput.flush();
                    smtpsOutput.print("To: ");
                    for (int i=0; i"); // 收 信 人
                    }
                    smtpsOutput.println("");
                    smtpsOutput.flush();
                    smtpsOutput.println("MIME-Version: 1.0");
                    smtpsOutput.flush();
                    smtpsOutput.println("Content-Type: text/plain;\n\tcharset=\"big5\"");
                    smtpsOutput.flush();
                    smtpsOutput.println("Content-Transfer-Encoding: 8bit");
                    smtpsOutput.flush();
                    smtpsOutput.print("Subject:");  // 標 題 
                    smtpsOutput.write(subject.getBytes("big5"));
                    smtpsOutput.println();
                    smtpsOutput.flush();
                    smtpsOutput.println();
                    smtpsOutput.flush();
                    for (int i=0; i0 && data[i].charAt(0)=='.') {
                            smtpsOutput.write('.');
                        }
                        smtpsOutput.write(data[i].getBytes("big5"));
                        smtpsOutput.println();
                        smtpsOutput.flush();
                    }
                    smtpsOutput.println(".");  // 資料結束 
                    smtpsOutput.flush();
                    smtpsInput.readLine();
                    smtpsOutput.println("QUIT");  // 斷線 
                    smtpsOutput.close();
                    smtpsInput.close();
                    smtps.close();
                    break;
                } catch(Exception sendEx) {
                    if (queue.isFull()) {
                        System.out.println("drop mail "+sender+" "+subject);
                        break;
                    }
                }
            }
        }
    }
}
```

### HTTP Server

```java
/*
 * File:    HTTPServer.java
 * Author:  Shiuh-Sheng Yu
 *          Department of Information Management
 *          National ChiNan University
 * Purpose: A Simple HTTP Server Example
 * Update Date: 01/11/2000
 * Modified Date: 0206/2002
 * Modified Date: 2005/03/20,加入Cleaner Thread, 以避免Client要求準備檔案後,卻不擷取
 */
import java.net.*;
import java.io.*;
import java.util.*;
public class HTTPServer extends Thread {
    Hashtable ht = new Hashtable();
    class Queue {
        private Socket[] data;
        private int size;
        private int head;
        private int tail;
        public Queue() {
            size = head = tail = 0;
            data = new Socket[100];
        }
        public synchronized Socket deQueue() {
            while (size==0) {
                try {
                    wait();
                } catch(Exception ex) {};
            }
            Socket tmp = data[head];
            data[head] = null;
            head = (head+1)%data.length;
            size--;
            if (size==data.length-1) {
                notifyAll();
            }
            return tmp;
        }
        public synchronized void enQueue(Socket c) {
            while (size==data.length) {
                try {
                    wait();
                } catch(Exception ex) {};
            }
            data[tail++] = c;
            tail %= data.length;
            size++;
            if (size==1) {
                notifyAll();
            }
        }
    }
    class Cleaner extends Thread {
        public void run() {
            for (;;) {
                try {
                    sleep(60000);
                    long currentTime = System.currentTimeMillis();
                    for (Enumeration e = HTTPServer.this.ht.elements(); e.hasMoreElements(); ) {
                        CacheData tmp = (CacheData)e.nextElement();
                        if (tmp.time < currentTime-30000) {
                            HTTPServer.this.ht.remove(tmp.fileName);
                        }
                    }
                } catch(InterruptedException epp) {}
            }
        }
    }
    class HTTPCon extends Thread {
        Queue queue;
        public HTTPCon(Queue q) {
            queue = q;
        }
        public void run() {
            BufferedOutputStream out = null;
            BufferedReader in = null;
            for(;;) { // loop forever to serve HTTP clients
                try {
                    Socket s = queue.deQueue();
                    out = new BufferedOutputStream(s.getOutputStream());
                    in = new BufferedReader(new InputStreamReader(s.getInputStream()));
                    String line;
                    String whichFile="/";
                    while ((line=in.readLine())!=null && !line.equals("")) {
                        //System.out.println(line); // dump for debugging
                        if (line.startsWith("GET")) {
                            // StringTokenizer將字串以空白分開
                            StringTokenizer st = new StringTokenizer(line);
                            st.nextToken();
                            whichFile = st.nextToken();
                            break;
                        }
                    }
                    if (whichFile.equals("/")) {
                        whichFile = "/Welcome.html";
                    }
                    // 過濾掉第一個 '/'
                    whichFile = whichFile.substring(1,whichFile.length());
                    String fileType = "text/plain";
                    if (whichFile.endsWith(".html")) {  // HTML文字檔
                        fileType = "text/html";
                    } else if (whichFile.endsWith(".txt")) {  // 文字檔
                        fileType = "text/plain";
                    } else if (whichFile.endsWith(".jpg")) { // JPG圖檔
                        fileType = "image/jpg";
                    } else if (whichFile.endsWith(".gif")) { // GIF圖檔
                        fileType = "image/gif";
                    }
                    // read from cache instead of file
                    CacheData tmp;
                    byte[] data;
                    if ((tmp=((CacheData)HTTPServer.this.ht.get(whichFile)))!=null) {
                        data = tmp.data;
                        HTTPServer.this.ht.remove(whichFile); // clear cache. Assume read once only
                        out.write("HTTP/1.0 200 OK\n".getBytes());
                        out.write("Connection: close\n".getBytes());
                        out.write(("Content-Type: "+fileType+"\n").getBytes());
                        out.write(("Content-Length: "+data.length+"\n\n").getBytes());
                        out.write(data);
                    } else {
                        out.write("HTTP/1.0 404 找不到物件\n".getBytes());
                        out.write("Content-Type: text/html\n\n".getBytes());
                        out.write("<html><body><h1>HTTP/1.0 404 找不到物件 </h1></body></html>".getBytes());
                    }
                    out.flush();
                    in.close();
                    out.close();
                    s.close();
                } catch(Exception ex) { // fail to listen port
                    //System.out.println(ex);
                }
            }
        }
    }
    class CacheData {
        String fileName;
        byte[] data;
        long time;
        public CacheData(String fileName, byte[] data, long time) {
            this.fileName = fileName;
            this.data = data;
            this.time = time;
        }
    }
    public String put(String extension, byte[] data) {
        String fileName;
        do {
            fileName = Integer.toString((int)(Math.random()*1000000))+extension;
        } while (ht.get(fileName)!=null);
        ht.put(fileName, new CacheData(fileName,data,System.currentTimeMillis()));
        return fileName;
    }
    public void run() {
        try {
            ServerSocket ss = new ServerSocket(1999);
            Queue q = new Queue();
            for (int i=0; i<3; i++) {
                (new HTTPCon(q)).start();
            }
            (new Cleaner()).start();
            for(;;) { // loop forever to serve HTTP clients
                try {
                    // 接收新的網路連線
                    Socket s = ss.accept();
                    s.setSoTimeout(5000); // 5秒內要讀到資料
                    q.enQueue(s);
                } catch(Exception ex2) { // fail to process Socket
                    ex2.printStackTrace();
                }
            }
        } catch(Exception ex) { // fail to listen port
            ex.printStackTrace();
        }
    }
}
```
