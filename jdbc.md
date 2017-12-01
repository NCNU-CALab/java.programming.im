JDBC (Java DataBase Connectivity) 是 Java 程式用來連結資料庫的程式庫 `java.sql.*`。  

其中主要的類別有
* DriverManager：用來產生資料庫連線 (Connection)
* Connection：用來產生下達命令給資料庫的敘述 (Statement, PreparedStatement 和 CallableStatement)
* Statement：用來下達查詢 (`executeQuery()` 傳回 ResultSet) 和更新 (`executeUpdate()` 傳回整數代表異動的資料數目) 命令
* ResultSet：利用其 `next()` 和 `getXXX()` 方法抓出查詢的結果
* PreparedStatement：先送出命令的模板，然後再傳遞所需要的參數
* CallableStatement：呼叫資料庫內的 Stored Procedure
* DatabaseMetaData：用來查詢資料庫綱要

下面的 `JDBCBridgeTest.java` 說明上面幾種 Class 的用法  

```java
import java.sql.*;
import ncnu.sql.*;
public class JDBCBridgeTest {
    public static void main(String[] argv) {
        try {
            Class.forName ("sun.jdbc.odbc.JdbcOdbcDriver");
            String pass = "password";
            String db = "jdbc:odbc:ncnu";
            if (argv.length>0) {
                pass = argv[0];
            }
            if (argv.length>1) {
                db = argv[1];
            }
            Connection con = DriverManager.getConnection (db, "sa", pass);
            con.setAutoCommit(true);
            PreparedStatement pstmt;
            Statement stmt;
            ResultSet rs;
            stmt = con.createStatement();
            int count = 0;
            long start = System.currentTimeMillis();
            rs = stmt.executeQuery("select courseid,year,class,studentid from selected");
            while (rs.next()) {
                count++;
                rs.getBytes(1);
                rs.getBytes(2);
                rs.getBytes(3);
                rs.getBytes(4);
            }
            rs.close();
            System.out.println("getBytes version for "+count+" rows: "+(System.currentTimeMillis()-start));
            start = System.currentTimeMillis();
            rs = stmt.executeQuery("select courseid,year,class,studentid from selected");
            while (rs.next()) {
                SQL.toString(rs.getBytes(1));
                SQL.toString(rs.getBytes(2));
                SQL.toString(rs.getBytes(3));
                SQL.toString(rs.getBytes(4));
            }
            rs.close();
            System.out.println("SQL.toString version for "+count+" rows: "+(System.currentTimeMillis()-start));
            start = System.currentTimeMillis();
            rs = stmt.executeQuery("select courseid,year,class,studentid from selected");
            while (rs.next()) {
                rs.getString(1);
                rs.getString(2);
                rs.getString(3);
                rs.getString(4);
            }
            rs.close();
            System.out.println("System getString version for "+count+" rows: "+(System.currentTimeMillis()-start));
            stmt.executeUpdate("delete printlog");
            con.commit();
            start = System.currentTimeMillis();
            for (int i=1; i<=10000; i++) {
                stmt.executeUpdate("insert printlog(printid,studentid,class,type,copy,year,email,success,ip,price) values("+i+",'90213001','B','中文歷年成績單',1,'901','ssyu@ncnu.edu.tw','Y','163.22.22.22',20)");
                con.commit();
            }
            System.out.println("Statement insert 10000 rows: "+(System.currentTimeMillis()-start));
            start = System.currentTimeMillis();
            for (int i=1; i<=10000; i++) {
                stmt.executeUpdate("delete printlog where printid="+i);
                con.commit();
            }
            System.out.println("Statement delete 10000 rows: "+(System.currentTimeMillis()-start));
            stmt.close();
            start = System.currentTimeMillis();
            pstmt = con.prepareStatement("insert printlog(printid,studentid,class,type,copy,year,email,success,ip,price) values(?,?,?,?,?,?,?,?,?,?)");
            for (int i=1;i<=10000; i++) {
                pstmt.setInt(1,i);
                pstmt.setString(2,"90213001");
                pstmt.setString(3,"B");
                pstmt.setString(4,"中文歷年成績單");
                pstmt.setInt(5,3);
                pstmt.setString(6,"901");
                pstmt.setString(7,"ssyu@ncnu.edu.tw");
                pstmt.setString(8,"Y");
                pstmt.setString(9,"163.22.22.22");
                pstmt.setInt(10,20);
                pstmt.executeUpdate();
                con.commit();
            }
            System.out.println("PreparedStatement version1 insert 10000 rows: "+(System.currentTimeMillis()-start));
            pstmt.close();
            start = System.currentTimeMillis();
            pstmt = con.prepareStatement("delete printlog where printid=?");
            for (int i=1; i<=10000; i++) {
                pstmt.setInt(1,i);
                pstmt.executeUpdate();
                con.commit();
            }
            pstmt.close();
            System.out.println("PrepareStatement delete 10000 rows: "+(System.currentTimeMillis()-start));
            start = System.currentTimeMillis();
            pstmt = con.prepareStatement("insert printlog(printid,studentid,class,type,copy,year,email,success,ip,price) values(?,?,?,?,?,?,?,?,?,?)");
            for (int i=1;i<=10000; i++) {
                pstmt.setInt(1,i);
                pstmt.setBytes(2,SQL.toAscii("90213001"));
                pstmt.setBytes(3,SQL.toAscii("B"));
                pstmt.setBytes(4,SQL.toAscii("中文歷年成績單"));
                pstmt.setInt(5,3);
                pstmt.setBytes(6,SQL.toAscii("901"));
                pstmt.setBytes(7,SQL.toAscii("ssyu@ncnu.edu.tw"));
                pstmt.setBytes(8,SQL.toAscii("N"));
                pstmt.setBytes(9,SQL.toAscii("163.22.22.22"));
                pstmt.setInt(10,20);
                pstmt.executeUpdate();
                con.commit();
            }
            System.out.println("PreparedStatement version2 insert 10000 rows: "+(System.currentTimeMillis()-start));
            pstmt.close();
            stmt = con.createStatement();
            stmt.executeUpdate("delete printlog");
            stmt.close();
            con.close();
       } catch (Exception ex) {
            ex.printStackTrace();
            System.exit(1);
        }
    }
}
```

`SQL.java` 的 Source Code 如下  

```java
package ncnu.sql;
import sun.io.*;
import ncnu.rule.Environment;
/**
 * The class is used to work around a bug in JDBC drivers. The driver truncate the leading
 * byte of Unicode character before sending to DBMS. This only works for ASCII character.
 * To handle strings to DBMS, you have to use SQL.toSQL() to wrap the output
 * string. JDBC drivers also treat the data from DBMS as ASCII code by adding 0 before every
 * bytes of the incoming data. This also produce errors for Non-ASCII characters. To work
 * around this bug, you have to wrap the incoming data with SQL.fromSQL()
 */
public class SQL {
    static ByteToCharConverter toChar;
    static CharToByteConverter toByte;
    static boolean trans = true;
    static {
        try {
            if (System.getProperty("java.version").startsWith("1.1")) {
                trans = true;
            } else {
                trans = false;
            }
            String encoding = "Big5";
            String domain = Environment.getDomain();
            if (domain.endsWith(".tw")) {
                encoding = "Big5";
            }
            SQL.toChar = ByteToCharConverter.getConverter(encoding);
            SQL.toByte = CharToByteConverter.getConverter(encoding);
        } catch(Exception ex) {
            ex.printStackTrace();
            System.exit(1);
        }
    }
    public static void setEncoding(String encoding) {
        try {
            SQL.toChar = ByteToCharConverter.getConverter(encoding);
            SQL.toByte = CharToByteConverter.getConverter(encoding);
        } catch(Exception ex) {}
    }
    /**
     * convert a string to ASCII byte array
     */
    public static byte[] toAscii(String s) {
        if (s==null) return new byte[0];
        try {
            synchronized(toByte) {
                return toByte.convertAll(s.toCharArray());
            }
        } catch(Exception ex) {
            System.out.println(ex);
            return new byte[0];
        }
    }
    public static char[] toChars(byte[] data) {
        if (data==null) {
            return new char[0];
        }
        try {
            synchronized(toChar) {
                return toChar.convertAll(data);
            }
        } catch(Exception ex) {
            System.out.println(ex);
            return new char[0];
        }
    }
    public static String toStringFast(byte[] data) {
        if (data==null) {
            return "";
        }
        char[] tmp = new char[data.length];
        for (int i=0; i<data.length; i++) {
            tmp[i] = (char)data[i];
        }
        return new String(tmp);
    }
    public static String quote(String s) {
        if (s.indexOf("'")<0) {
            return s;
        }
        StringBuffer sb = new StringBuffer();
        for (int i=0; i<s.length(); i++) {
            char c = s.charAt(i);
            sb.append(c);
            if (c=='\'') {
                sb.append(c);
            }
        }
        return sb.toString();
    }
    public static String toString(byte[] data) {
        if (data==null) {
            return "";
        }
        try {
            synchronized(toChar) {
                return new String(toChar.convertAll(data));
            }
        } catch(Exception ex) {
            System.out.println(ex);
            return "";
        }
    }
    /**
     * Convert a string for output to DBMS
     * @param s The string needs to be converted.
     * @return A string which will be truncated by JDBC to produce Big5 characters.
     */
    public static String toSQL(String s) {
        if (s==null) return "";
        if (!trans) return s;
        byte[] orig;
        try {
            synchronized(toChar) {
                orig = toByte.convertAll(s.toCharArray());
            }
            char[] dest = new char[orig.length];
            for (int i=0; i < orig.length; i++)
                dest[i] = (char)orig[i];
            return new String(dest);
        } catch (Exception e) {
            e.printStackTrace();
            return s;
        }
    }
    /**
     * Convert an incorrect string produeced by JDBC drivers to correct Unicode string.
     * @param s The string needs to be converted.
     * @return A correct Unicode string.
     */
    public static String fromSQL(String s) {
        int i, j;
        if (s==null) return "";
        if (!trans) return s;
        char[] orig = s.toCharArray();
        byte[] dest = new byte[orig.length];
        for (i=0; i < orig.length; i++)
            dest[i] = (byte) orig[i];
        try {
            synchronized(toChar) {
                char[] tmp = toChar.convertAll(dest);
                return new String(tmp);
            }
        } catch (Exception e) {
            e.printStackTrace();
            return s;
        }
    }
}
```

#### 查詢資料庫綱要

```java
package ncnu.sql;
/* Program Name: Schema.java
   Subject: 資料綱要讀取程式 
   Author: 俞旭昇 Shiuh-Sheng Yu
           National ChiNan University
           Department of Information Management 
   Edit Date: 01/10/2000
   Last Update Date: 01/10/2000
   ToolKit: JDK1.1
*/

import java.sql.*;
import java.util.*;
import java.io.*;

public class Schema implements Serializable {
    Vector tables=null;
    String catalogSeparator=null, catalog, schema;
    Schema (Connection con, String catalog, String schema) {
        this.catalog = catalog;
        this.schema = schema;
        tables = new Vector();
        ForeignKey fk;
        try {
            ResultSet rs;
            DatabaseMetaData md = con.getMetaData();
            catalogSeparator = md.getCatalogSeparator();
            String[] types = new String[1];
            types[0] = "TABLE";
            // get all user defined tables
            rs = md.getTables(catalog,schema,null,types);
            while (rs.next()) {
                Table table = new Table();
                tables.addElement(table);
                table.catalogName = rs.getString(1);
                table.schemaName = rs.getString(2);
                table.tableName = rs.getString(3);
                table.totalName = getTotalName(table.catalogName,table.schemaName,table.tableName);
                table.tableType = rs.getString(4);
                table.remarks = rs.getString(5);
            }
            rs.close();
            // get all columns
            for (int i=0; i < tables.size(); i++) {
                Table check = (Table)tables.elementAt(i);
                rs = md.getColumns(check.catalogName, check.schemaName,check.tableName,null);
                while (rs.next()) {
                    Column col = new Column();
                    col.tableName = check.totalName;
                    col.columnName = rs.getString(4).trim();
                    col.dataType = rs.getInt(5);
                    col.typeName = rs.getString(6).trim();
                    col.columnSize = rs.getInt(7);
                    col.decimalDigits = rs.getInt(9);
                    col.radix = rs.getInt(10);
                    col.remarks = SQL.fromSQL(rs.getString(12));
                    col.columnDefault = SQL.fromSQL(rs.getString(13)).trim();
                    while (col.columnDefault.startsWith("'") || col.columnDefault.startsWith("(")) {
                        col.columnDefault = col.columnDefault.substring(1,col.columnDefault.length());
                    }
                    while (col.columnDefault.endsWith("'") || col.columnDefault.endsWith(")")) {
                        col.columnDefault = col.columnDefault.substring(0,col.columnDefault.length()-1);
                    }
                    col.isNullable = SQL.toString(rs.getBytes(18)).trim();
                    if (col.remarks==null || col.remarks.equals("")) {
                        col.remarks=col.columnName;
                    }
                    check.addColumn(col);
                }
                rs.close();
            }
            // get Primary key
            for (int i=0; i < tables.size(); i++) {
                Table check = (Table)tables.elementAt(i);
                rs = md.getPrimaryKeys(check.catalogName, check.schemaName,check.tableName);
                while (rs.next()) {
                    check.primaryKey.addElement(findColumn(check,rs.getString(4)));
                }
                rs.close();
            }
            // get Foreign Keys
            for (int i=0; i < tables.size(); i++) {
                Table check = (Table)tables.elementAt(i);
                rs = md.getImportedKeys(check.catalogName, check.schemaName,check.tableName);
                fk = null;
                while (rs.next()) {
                    String tmp1 = rs.getString(1); // remote primary catalog
                    String tmp2 = rs.getString(2); // remote primary schema
                    String tmp3 = rs.getString(3); // remote primary table
                    String tmp4 = rs.getString(4); // remote primary column
                    String tmp8 = rs.getString(8); // local foreign column
                    int tmp9 = rs.getInt(9);
                    if (tmp9 == 1) {
                        if (fk != null) {
                            check.addForeignKey(fk);
                        }
                        fk = new ForeignKey();
                    }
                    fk.primaryTable = findTable(tmp1,tmp2,tmp3);
                    fk.foreignTable = check;
                    fk.primary.addElement(findColumn(tmp1,tmp2,tmp3,tmp4));
                    fk.foreign.addElement(findColumn(check,tmp8));
                }
                rs.close();
                if (fk != null) {
                    check.addForeignKey(fk);
                }
            }
            // get Reference By
            for (int i=0; i < tables.size(); i++) {
                Table check = (Table)tables.elementAt(i);
                rs = md.getExportedKeys(check.catalogName, check.schemaName,check.tableName);
                fk = null;
                while (rs.next()) {
                    String tmp4 = rs.getString(4); // local primary column
                    String tmp5 = rs.getString(5); // remote foreign catalog
                    String tmp6 = rs.getString(6); // remote foreign schema
                    String tmp7 = rs.getString(7); // remote foreign table
                    String tmp8 = rs.getString(8); // remote foreign column
                    int tmp9 = rs.getInt(9);
                    if (tmp9 == 1) {
                        if (fk != null) {
                            check.addReferenceBy(fk);
                        }
                        fk = new ForeignKey();
                    }
                    fk.primaryTable = check;
                    fk.foreignTable = findTable(tmp5,tmp6,tmp7);
                    fk.primary.addElement(findColumn(check,tmp4));
                    fk.foreign.addElement(findColumn(tmp5,tmp6,tmp7,tmp8));
                }
                rs.close();
                if (fk != null) {
                    check.addReferenceBy(fk);
                }
            }
        } catch (SQLException ex) {
            ShowSQLException.show(ex);
        } finally {
            try {
               con.commit();
            } catch (Exception e2) {
            }
        }
    }

    public Table findTable(String catalog, String schema, String table) {
        return findTable(getTotalName(catalog, schema, table));
    }

    public Table findTable(String totalName) {
        int i, j;
        for (i=j=0; totalName.indexOf(catalogSeparator,i) != -1; j++) {
            i = totalName.indexOf(catalogSeparator,i) + catalogSeparator.length();
        }
        // How many full name still need to fill? eg student.name ==> ncnu.dbo.student.name
        if (j==0) { // we miss all
            totalName = catalog+catalogSeparator+schema+catalogSeparator+totalName;
        } else if (j==1) { // we miss catalogName
            totalName = catalog+catalogSeparator+totalName;
        } else { // we have all
        }
        for (i=0; i < tables.size(); i++) {
            if (((Table)tables.elementAt(i)).totalName.equals(totalName)) {
                return (Table)tables.elementAt(i);
            }
        }
        return null;
    }

    public Column findColumn(String totalName, String columnName) {
        Table t = findTable(totalName);
        if (t == null) {
            return null;
        }
        return findColumn(t, columnName);
    }

    public Column findColumn(String catalog, String schema, String table, String column) {
        Table t = findTable(catalog, schema, table);
        if (t == null) {
            return null;
        }
        return findColumn(t, column);
    }

    public Column findColumn(Table t, String column) {
        for (int i=0; i < t.columns.size(); i++) {
            if (((Column)t.columns.elementAt(i)).columnName.equals(column)) {
                return (Column)t.columns.elementAt(i);
            }
        }
        return null;
    }

    public String getTotalName(String catalog, String schema, String table) {
        String totalName = "";
        if (catalog!=null && catalog.trim().length()>0) {
            totalName = catalog.trim()+catalogSeparator;
        }
        if (schema!=null && schema.trim().length()>0) {
            totalName += schema.trim()+catalogSeparator;
        }
        if (table!=null && table.trim().length()>0) {
            totalName += table.trim();
        }
        return totalName;
    }

    public String getCatalogSeparator() {
        return catalogSeparator;
    }
    public Vector getTables() {
        return tables;
    }
}
```

`Table.java` 如下  

```java
package ncnu.sql;
/* Program Name: Table.java
   Subject: 定義資料表格的屬性 
   CopyRight: 俞旭昇 Shiuh-Sheng Yu
           National Chi-Nan University
           Institute of Management Information 
   Edit Date: 01/03/1998
   Last Update Date: 01/04/1997
   ToolKit: JDK1.1.5
*/
import java.util.*;
import java.io.Serializable;
public class Table implements Serializable {
    String catalogName;
    String schemaName;
    String tableName;
    String totalName;
    String tableType;
    String remarks;
    Vector columns;
    Vector primaryKey;
    Vector foreignKeys;
    Vector referenceBy;
    Table() {
        columns = new Vector();
        foreignKeys = new Vector();
        primaryKey = new Vector();
        referenceBy = new Vector();
    }
    void addColumn(Column c) {
        columns.addElement(c);
    }
    Column getColumn(String cname) {
        for (int i=0; i < columns.size(); i++) {
            Column c = (Column)columns.elementAt(i);
            if (c.columnName.equalsIgnoreCase(cname)) {
                return c;
            }
        }
        return null;
    }
    void addForeignKey(ForeignKey fk) {
        foreignKeys.addElement(fk);
    }
    void removeForeignKey(Table t) {
        for (int i=0; i < foreignKeys.size(); i++) {
            ForeignKey fk = (ForeignKey)foreignKeys.elementAt(i);
            if (fk.primaryTable==t) {
                foreignKeys.removeElementAt(i);
                i--;
            }
        }
    }
    void addReferenceBy(ForeignKey fk) {
        referenceBy.addElement(fk);
    }
    void print() {
        System.out.println("Table: "+catalogName+"."+schemaName+"."+tableName);
        for (int i=0; i < columns.size(); i++) {
            ((Column)columns.elementAt(i)).print();
        }
        if (primaryKey != null) {
            System.out.println("PrimaryKey:");
            for (int i=0; i < primaryKey.size(); i++) {
                ((Column)primaryKey.elementAt(i)).print();
            }
        }
        if (foreignKeys != null) {
            for (int i=0; i < foreignKeys.size(); i++) {
                System.out.println("ForeignKey:");
                ((ForeignKey)foreignKeys.elementAt(i)).print();
            }
        }
        if (referenceBy != null) {
            for (int i=0; i < referenceBy.size(); i++) {
                System.out.println("Reference By:");
                ((ForeignKey)referenceBy.elementAt(i)).print();
            }
        }
    }
}
```

`ForeignKey.java` 如下  

```java
package ncnu.sql;
/* Program Name: ForeignKey.java
   Subject: Foreign Key, used bye View.java 
   CopyRight: 俞旭昇 Shiuh-Sheng Yu
           National ChiNan University
           Department of Information Management 
   Edit Date: 01/03/1998
   Last Update Date: 08/21/1998
   ToolKit: JDK1.1.6
*/
import java.util.Vector;
import java.io.Serializable;
public class ForeignKey implements Serializable {
    Table primaryTable;
    Table foreignTable;
    Vector primary; // vector of Columns
    Vector foreign; // vecotr of Columns
    ForeignKey() {
        primary = new Vector();
        foreign = new Vector();
    }
    String getJoinCondition(String sep) {
        String cond = "";
        for (int i=0; i < primary.size(); i++) {
            Column x = (Column)primary.elementAt(i);
            Column y = (Column)foreign.elementAt(i);
            cond += x.tableName+sep+x.columnName+"="+y.tableName+sep+y.columnName+" and ";
        }
        return cond;
    }
    void print() {
        System.out.println("ForeignKey:");
        for (int i=0; i<primary.size(); i++) {
            ((Column)primary.elementAt(i)).print();
        }
        System.out.println("references");
        for (int i=0; i<foreign.size(); i++) {
            ((Column)foreign.elementAt(i)).print();
        }
    }
}
```

`Column.java` 如下  

```java
package ncnu.sql;
/* Program Name: Column.java
   Subject: 定義資料欄位的屬性 
   CopyRight: 俞旭昇 Shiuh-Sheng Yu
           National Chi-Nan University
           Institute of Management Information 
   Edit Date: 01/03/1998
   Last Update Date: 01/04/1997
   ToolKit: JDK1.1.5
*/

import java.io.Serializable;
import java.sql.Types;
public class Column implements Serializable {
    String tableName;
    String columnName;
    String typeName;
    int dataType;
    int columnSize;
    int decimalDigits;
    int radix;
    String isNullable;
    String remarks;
    String columnDefault;
    public static boolean isNumeric(int type) {
        switch (type) {
            case Types.BIGINT:
            case Types.TINYINT:
            case Types.SMALLINT:
            case Types.NUMERIC:
            case Types.DECIMAL:
            case Types.FLOAT:
            case Types.INTEGER:
            case Types.REAL:
            case Types.DOUBLE:
            case Types.BIT:
            return true;
        }
        return false;
    }
    void print() {
        System.out.print(tableName+"."+columnName+" "+typeName+" "+columnSize+" ");
        if (typeName.equals("numeric")) {
            System.out.print("decimal:"+decimalDigits+" radix:"+radix+" ");
        }
        System.out.println(isNullable+" "+remarks+" default:"+columnDefault);
    }
}
```
