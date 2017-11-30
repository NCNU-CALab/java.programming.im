```java
/*
 * Program Name: Tax.java
 * Purpose: 計算所得稅(程設二作業)
 * Author: Shiuh-Sheng Yu
 *         Department of Information Management
 * Date: 03/24/2000
 * Last Update: 03/29/2000
 */
import java.io.*;
public class Tax {
    public static long calTax(long income) {
        long[] range = {300000, 300000, 400000, 600000, 800000, 1000000, Long.MAX_VALUE};
        double[] rate = {0, 0.06, 0.13, 0.21, 0.30, 0.45, 0.6};
        double tax = 0;
        int i;
        for (i = 0; income > range[i]; income -= range[i++]) {
            tax += range[i] * rate[i];
        }
        tax += income * rate[i];
        return (long)(tax+0.5);
    }
    public static long readIncome(InputStreamReader in) {
        StringBuffer sb = new StringBuffer();
        long income = 0;
        char c;
        try {
            while ((c=(char)in.read())!='\n' && c!='\r') {
                sb.append(c);
            }
            income = Long.valueOf(sb.toString()).longValue();
        } catch(IOException ioe) {
            System.out.println("讀入錯誤");
        } catch(NumberFormatException nfe) {
            System.out.println("格式錯誤:資料不為整數");
        } catch(Exception e) {
        }
        return income;
    }
    public static void main(String[] argv) {
        InputStreamReader in = new InputStreamReader(System.in);
        System.out.print("請輸入您今年的所得:");
        long income = readIncome(in);
        System.out.println("您今年需繳的所得稅為:"+calTax(income)+"元");
    }
}
```
