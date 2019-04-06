## JavaIO---File输入输出流、打印流、Java.util.Scanner类、装饰设计模式及序列化

#### IO基于抽象类。

>File文件操作类（字节输入输出流、字符输入输出流）
>
>字符编码
>
>转换流
>
>内存流
>
>打印流
>
>System对IO类的支持
>
>两种输入流
>
>Java.util.Scanner类
>
>序列化

### File文件操作类

在Java.io包中，File类是唯一一个与文件本身操作（创建、删除、获取信息等）有关的程序类。

**File类不包括文件内容，它是文件本身，文件的读取与写入需要使用输入输出流。**

#### 1.1File类的使用--文件/文件夹

##### 文件：

```java
//File类源码
public class File implements Serializable, Comparable<File>{}
```

由源码可得File类是Java.io包中的一个普通类，我们可以直接对其实例化对象。实例化对象可用到以下构造方法：

```java
1.public File(String pathname){};//根据文件的绝对路径产生File对象
```

创建一个新文件：

```java
public boolean createNewFile() throws IOException {}//根据绝对路径创建新文件
```

删除已有文件：

```java
public boolean delete() {}//删除已存在文件
```

判断一个文件是否存在：

```java
public boolean exists() {}//判断指定路径文件是否存在
```

判断是否为文件：

```java
public boolean isFile() {}
```

路径分隔符：出现原因是在Windows下路径分隔符通常为“//”,linux操作系统下通常为"/".故使用路径分隔符可更好的达到跨平台性。

```java
public static final String separator = "" + separatorChar;
//使用时为File.separator(因separator在File类中为静态属性可通过类名直接调用)
```

若为文件即可获得文件大小：

```java
 public long length() {}//获得当前路径下的文件大小（字节）通常使用时/1024（KB）
```

获取文件最新修改时间：

```java
public long lastModified() {}//获得当前路径文件最新修改时间，但要结合Date类使用，否则直接调用得到的是从1970年1月1日至文件最新修改时间的差值（无意义）
```

对以上方法的简单实用：

```java
import java.io.*;
import java.text.SimpleDateFormat;
import java.util.Date;

public class Test{
    public static void main(String[] args) throws Exception {
        //在桌面查找名称为lala.jpg的文件
        File file=new File("C:"+File.separator+"Users"
                +File.separator+"14665"+File.separator+"Desktop"+File.separator+"lala.jpg");
        //判断当前路径下文件是否存在,不存在就创建
        if(!file.exists()){
            file.createNewFile();
        }
        //获取文件大小
        System.out.println(file.length()/1024);
        //获得文件最新修改时间(三种方法)
        //1
        System.out.println(file.lastModified());
        //2
        System.out.println(new Date(file.lastModified()));
        //3
        Date date=new Date(file.lastModified());
        SimpleDateFormat simpleDateFormat=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        System.out.println(simpleDateFormat.format(date));
    }
}
```

![1](C:\Users\14665\source\JAVA学习\学习总结File输入输出流、打印流、Java.util.Scanner类及装饰设计模式\1.png)

##### 文件夹（目录）：

创建目录（一次性创建多级不存在的父路径）：

```java
public boolean mkdirs() {}
```

取得父路径/父路径File对象：

```java
public String getParent(){};//获取父路径
public File getParentFile(){};//获得父路径File对象
```

判断是否为文件夹：

```java
public boolean isDirectory() {}
```

获得当前目录下所有文件：

```java
public File[] listFiles() {}//但直接调用该方法只能得到当前路径下的文件及目录，若想进一步得到该路径下所有目录的文件就需采用递归方法
```

创建目录方法简单使用：

```java
import java.io.*;

public class Test{
    public static void main(String[] args) throws Exception {
        //在桌面创建新目录
        File file=new File("C:"+File.separator+"Users"
                +File.separator+"14665"+File.separator+"Desktop"+File.separator
                + "www"+File.separator+"ss"+File.separator+"hello.txt");
        //判断该文件夹是否存在
        if(!file.getParentFile().exists()){
            file.getParentFile().mkdirs();
            //若该处为file.mkdirs()那么hello.txt也会被创建为文件夹
        }
        if(file.exists()){
            file.delete();
        }else {
            file.createNewFile();
        }
    }
}
```

![2](C:\Users\14665\source\JAVA学习\学习总结File输入输出流、打印流、Java.util.Scanner类及装饰设计模式\2.png)

查找桌面路径下所有文件：

```java
import java.io.*;

public class Test{
    public static void main(String[] args) throws Exception {
        File file=new File("C:"+File.separator+"Users"
                +File.separator+"14665"+File.separator+"Desktop");
        //测试所需时间
        long start=System.currentTimeMillis();
        listAllFiles(file);
        long end=System.currentTimeMillis();
        System.out.println("共耗时："+(end-start)+"毫秒");
    }
    public static void listAllFiles(File file){
        if(file.isFile()){//递归出口（若为文件直接输出）
            System.out.println(file);
        }else{
            if(file.exists()&&file.isDirectory()){//判断文件是否存在且是否为文件夹
                File[] files=file.listFiles();
                for(File file1:files){//对文件夹递归
                    listAllFiles(file1);
                }
            }
        }
    }
}
//924毫秒
```

对此我们可以看到只是查找桌面就耗时较多，那么查找C盘呢？时间会更久。出现这种情况的原因是主线程堵塞，因所有程序运行都在主线程中，如果listAllFiles()方法没有完成，那么main后续的执行将无法完成。这种情况造成了主线程堵塞，**故对此我们可以新建立一个线程对其优化。**

```java
import java.io.*;

public class Test{
    public static void main(String[] args) throws Exception {
        File file=new File("C:"+File.separator+"Users"
                +File.separator+"14665"+File.separator+"Desktop");
        //将IO操作放在子线程中进行
        new Thread(()->{
            long start=System.currentTimeMillis();
            listAllFiles(file);
            long end=System.currentTimeMillis();
            System.out.println("共耗时："+(end-start)+"毫秒");
        }).start();
        System.out.println("主线程结束");
    }
    public static void listAllFiles(File file){
        if(file.isFile()){
            System.out.println(file);
        }else{
            if(file.exists()&&file.isDirectory()){
                File[] files=file.listFiles();
                for(File file1:files){
                    listAllFiles(file1);
                }
            }
        }
    }
}
```

**一般来说，将IO操作放置子线程中进行。原因是这样IO操作就不影响主线程，且安卓开发中IO操作必须放在子线程中运行否则错误，虽然java中IO操作放置主线程中也可正常运行，但为了更好的跨平台性我们将IO操作放置子线程中运行。**

#### 1.2字节流与字符流

我们知道File类只与文件本身有关，若想进行文件读取或写入必须使用输入输出流。在Java.io包中，流分为字节流与字符流：

```java
1.字节流：输入流：InputStream,输出流：OutputStream(均为抽象类)
public abstract class InputStream implements Closeable {}//实现资源关闭流接口
Closeable(资源关闭流接口):public interface Closeable extends AutoCloseable {}//资源关闭流接口继承自动关闭流接口
AutoCloseable（自动关闭流）：public interface AutoCloseable {}
//实现资源关闭流接口与强制刷新缓冲区接口
public abstract class OutputStream implements Closeable, Flushable {}
Flushable（强制刷新缓冲区）：public interface Flushable {}
2.字符流：输入流：Reader,输出流：Writer
public abstract class Reader implements Readable, Closeable {}
public interface Readable {}
//Attempts to read characters into the specified character buffer.
public abstract class Writer implements Appendable, Closeable, Flushable {}
public interface Appendable {}
//Appends the specified character sequence to this <tt>Appendable</tt>.
```

#### **字节输出流（从程序中向文件中写入数据）OutputStream：**

OutputStream类中其他常用方法：

```java
1.public void write(byte b[]) throws IOException {}//将给定字节数组内容全部输出（直接输出到终端（文件、网络、内存中，不存在缓冲区））
2.public void write(byte b[], int off, int len) throws IOException {}//将部分字节数组内容输出(将给定字节数组从off位置开始输出len长度后停止)
3.public abstract void write(int b) throws IOException;//输出单个字节
```

我们知道OutputStream类为抽象类不可直接实例化对象，故要想实例化必须使用子类。如果要进行文件操作，可以使用FileOutputStream类来处理：

```java
1.public FileOutputStream(String name) throws FileNotFoundException {}//覆写
2.public FileOutputStream(String name, boolean append)throws FileNotFoundException{}//叠加
```

方法简单操作：

```java
import java.io.*;

public class Test{
    public static void main(String[] args) throws Exception {
        //1.获取终端对象
        File file = new File("C:" + File.separator + "Users"
                + File.separator + "14665" + File.separator + "Desktop"+File.separator+"TestIO.txt");
        //必须保证父目录存在
        if(!file.getParentFile().exists()){
            file.getParentFile().mkdirs();
        }
        //2.获得输出流
       OutputStream out=new FileOutputStream(file);
        //3.数据写入
        String str="hello";
        out.write(str.getBytes());
        //4.关闭流
        out.close();
    }
}
```

![3](C:\Users\14665\source\JAVA学习\学习总结File输入输出流、打印流、Java.util.Scanner类及装饰设计模式\3.png)

若将输出流实例化处改为叠加，

```java
import java.io.*;

public class Test{
    public static void main(String[] args) throws Exception {
        //1.获取终端对象
        File file = new File("C:" + File.separator + "Users"
                + File.separator + "14665" + File.separator + "Desktop"+File.separator+"TestIO.txt");
        //必须保证父目录存在
        if(!file.getParentFile().exists()){
            file.getParentFile().mkdirs();
        }
        //2.获得输出流(叠加)
       OutputStream out=new FileOutputStream(file,true);
        //3.数据写入
        String str="hello";
        out.write(str.getBytes(),0,3);
        //4.关闭流
        out.close();
    }
}
```

![4](C:\Users\14665\source\JAVA学习\学习总结File输入输出流、打印流、Java.util.Scanner类及装饰设计模式\4.png)

**如上代码简单使用后，我们可以发现在使用字节输出流时指定路径文件不存在时无需手动调用createNewFile()方法创建文件，而是系统自动生成文件(但必须保证该文件父目录存在)，若父目录不存在生成即可。**

#### **字节输入流（在程序中读取文件中数据）InputStream：**

InputStream类中其他常用方法：

```java
1.public int read(byte b[]) throws IOException {};//读取数据到字节数组中并返回数据的读取个数
该返回值有三种情况：
a.返回b.length。此种情况为未读取数据>存放的缓冲区大小，返回字节数组的大小
b.返回一个大于0小于b.length的数。此种情况为未读取数据<存放的缓冲区大小，返回实际数据大小
c.返回-1。终止标记，表示此时数据已读取完毕。
2.public int read(byte b[], int off, int len) throws IOException {}；//读取部分数据到字节数组中，该返回值有三种情况：
a.返回len。若读取满了则返回长度len
b.返回实际数据个数。若没有读取满则返回读取的数据个数
c.返回-1。若读取到最后没有数据了返回-1
3.public abstract int read() throws IOException;//每次读取一个字节，知道没有数据返回-1
```

我们也知道InputStream为抽象类，进行文件操作时实例化该类需要借助FileInputStream类。

```java
import java.io.*;

public class Test{
    public static void main(String[] args) throws Exception {
        //1.获取终端对象
        File file = new File("C:" + File.separator + "Users"
                + File.separator + "14665" + File.separator + "Desktop"+File.separator+"TestIO.txt");
        //必须保证该文件存在才可读取数据
        if(file.exists()&&file.isFile()){
            //2.获得输入流
            InputStream in=new FileInputStream(file);
            //3.数据写入
            byte[] data=new byte[1024];
            int len=in.read(data);
            System.out.println(new String(data,0,len));
            //4.关闭流
            in.close();
        }
    }
}
//hellohel
```

我们注意到在进行文字读取与写入时最后都要关闭流，且以上代码均是需要程序员手动关闭流，那么自动关闭流的方法是什么呢？

AutoClassable:自动关闭流(了解即可)

JDK1.7之后增加AutoClassable接口，该接口主要作用是支持自动关闭流操作，**但必须结合try...catch使用（必须在try()中进行实例化对象），故一般建议使用手动调用close()方法关闭流。**

```java
public interface AutoCloseable {}
public interface Closeable extends AutoCloseable {}
public abstract class OutputStream implements Closeable, Flushable {}
//故我们可以得出OutputStream类是AutoCloseable接口的子类
```

```java
import java.io.*;
class A implements AutoCloseable{
    @Override
    public void close() throws Exception {
        System.out.println("这是自动关闭流方法");
    }
    public void fun(){
        System.out.println("这是普通方法");
    }
}
public class Test {
    public static void main(String[] args){
        //必须在try()中实例化对象，否则不会自动调用close()方法
       try (A a=new A()){
           a.fun();
       }catch (Exception e){
           e.printStackTrace();
       }
    }
}
//这是普通方法
//这是自动关闭流方法
```

#### 字符输出流(适用于处理中文文本):Writer

```
public abstract class Writer implements Appendable, Closeable, Flushable {}
```

Writer类中也有writer()方法：

![5](C:\Users\14665\source\JAVA学习\学习总结File输入输出流、打印流、Java.util.Scanner类及装饰设计模式\5.png)

**但要注意的是，Writer类提供了一个输出字符串的方法：**

```java
public void write(String str) throws IOException {}
```

```java
import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.io.Writer;

public class Test {
    public static void main(String[] args) throws IOException {
        File file = new File("C:" + File.separator + "Users"
                + File.separator + "14665" + File.separator + "Desktop"+File.separator+"TestIO.txt");
        if(!file.getParentFile().exists()){
            file.getParentFile().mkdirs();
        }
        Writer writer=new FileWriter(file);
        //可直接输出字符串，字节输出流需将字符串转为字节再输出
        writer.write("你好中国");
        writer.close();
    }
}
```

#### 字符输入流:Reader

```java
public abstract class Reader implements Readable, Closeable {}
```

我们了解到字符输出流中提供了直接输出字符串的方法，**但Reader类中没有直接读取字符串类型的方法，只能通过字符数组进行读取操作。**

```java
public int read(char cbuf[]) throws IOException {}
```

```java
import java.io.*;

public class Test {
    public static void main(String[] args) throws IOException {
        File file = new File("C:" + File.separator + "Users"
                + File.separator + "14665" + File.separator + "Desktop"+File.separator+"TestIO.txt");
        if(file.exists()){
            Reader reader=new FileReader(file);
            char[] data=new char[1024];
            int len=reader.read(data);
            System.out.println(new String(data,0,len));
            reader.close();
        }
    }
}
```

#### 字节流VS字符流

我们通过以上学习可以看出字节输入输出流与字符输入输出流相差不大，字符流在处理中文方面更加便利一点。但从实际开发角度来看，字节流是优先考虑的，只有在处理中文时才会考虑字符流。

##### 字节流与字符流最大差别：字节流无缓冲区，字符流有输出缓冲区。故使用字节输出流时若没关闭流不影响数据输出到终端，但字符流不关闭流或者没有强制刷新缓冲区就会导致数据保存在缓冲区中不会输出到终端。

```java
import java.io.*;

public class Test {
    public static void main(String[] args) throws IOException {
        File file = new File("C:" + File.separator + "Users"
                + File.separator + "14665" + File.separator + "Desktop"+File.separator+"TestIO.txt");
//        if(!file.getParentFile().exists()){
//            file.getParentFile().mkdirs();
//        }
        //字节输出流，不关闭流也可将数据输出到终端
//        OutputStream outputStream=new FileOutputStream(file);
//        String str="你好";
//        outputStream.write(str.getBytes());
        //字符输出流（必须关闭流或强制刷新缓冲区）
//        Writer writer=new FileWriter(file);
//        writer.write("hello");
//        writer.flush();
        if(file.exists()){
            //字节输入流
//            InputStream inputStream=new FileInputStream(file);
//            byte[] data=new byte[1024];
//            int len=inputStream.read(data);
//            System.out.println(new String(data,0,len));
            Reader reader=new FileReader(file);
            char[] data=new char[1024];
            int len=reader.read(data);
            System.out.println(new String(data,0,len));
        }
    }
}
```

**以后在IO处理时，字节流可处理图片、音乐、文字等，字符流只有在处理中文时才会使用。**

#### 2.字符编码--使用UTF-8即可

##### 常用字符编码

1)GBK、GB2312：表示的是国标编码，GBK包含简体中文和繁体中文，而GB2312只包含简体中文。也就是说这两种编码都是描述中文的编码。

2)UNICODE编码：java提供的16进制编码，可以描述世界上任意文字信息。但是编码进制数太高，编码体积较大。

3)ISO8859-1:国际通用编码（浏览器默认编码），但不支持中文。

4)UTF编码：结合UNICODE、ISO8859-1。也就是说使用16进制文字使用UNICODE，只是字母就是用ISO8859-1,常用的就是UTF-8编码。

乱码产生原因：

95%的编码不一致，5%的数据丢失。

```java
System.getProperties().list(System.out);//该方法可查看系统当前默认编码
```

乱码产生代码实例：

```java
import java.io.*;

public class Test{
    public static void main(String[] args) throws Exception {
        File file=new File("C:"+File.separator+"Users"+File.separator+
                "14665"+File.separator+"Desktop"+File.separator+"TestIO.txt");
        OutputStream outputStream=new FileOutputStream(file);
        String str="你好号";
        outputStream.write(str.getBytes("ISO8859-1"));
        outputStream.close();
    }
}
```

![8](C:\Users\14665\source\JAVA学习\学习总结File输入输出流、打印流、Java.util.Scanner类及装饰设计模式\8.png)



#### 3.转换流（只能字节流->字符流，不可逆转）

**转换流主要的是其存在意义：转换流作为中间层可将字节流转换为字符流。**

字符流的具体子类大都是通过转换流将字节流转换为字符流(FileWriter继承转换流)。

```java
//Writer对于文字的输出要比OutputStream方便
OutputStreamWriter:将字节输出流转变为字符输出流
public class OutputStreamWriter extends Writer {}
public OutputStreamWriter(OutputStream out) {}
//InputStream读取的是字节，不方便中文的处理
InputStreamReader:将字节输入流变为字符输入流
public class InputStreamReader extends Reader {}
public InputStreamReader(InputStream in) {}//构造方法
```

OutputStream与Writer转换关系：

![6](C:\Users\14665\source\JAVA学习\学习总结File输入输出流、打印流、Java.util.Scanner类及装饰设计模式\6.png)

InputStream与Reader转换关系：

![7](C:\Users\14665\source\JAVA学习\学习总结File输入输出流、打印流、Java.util.Scanner类及装饰设计模式\7.png)

```java
import java.io.*;

public class Test {
    public static void main(String[] args) throws IOException {
        File file = new File("C:" + File.separator + "Users"
                + File.separator + "14665" + File.separator + "Desktop" + File.separator + "TestIO.txt");
        //字节输出流->字符输出流
//        if(!file.getParentFile().exists()){
//            file.getParentFile().mkdirs();
//        }
//        Writer writer=new OutputStreamWriter(new FileOutputStream(file));
//        writer.write("hahaha");
//        writer.close();
        //字节输入流->字符输入流
        if(file.exists()){
            Reader reader=new InputStreamReader(new FileInputStream(file));
            char[] data=new char[1024];
            int len=reader.read(data);
            System.out.println(new String(data,0,len));
            reader.close();
        }
    }
}
```

通过以上学习，我们现在可以进行一下综合使用，实现文件拷贝。

文件拷贝思路：需要结合字节输入输出流，因首先需要将源文件利用字节输入流拷贝一份，然后再使用字节输出流将其输出到终端即可。

```java
import java.io.*;
class CopyFileUtil{
    private CopyFileUtil(){}
    public static void createParentFilePath(String path){
        File file=new File(path);
        if(!file.getParentFile().exists()){
            file.getParentFile().mkdirs();
        }
    }
}
public class Test {
    public static void main(String[] args) {
        //源文件绝对路径
        String sourcepath="C:"+File.separator+"Users"+File.separator+"14665"
                +File.separator+"Desktop"+File.separator+"泫雅.jpg";
        //目标文件绝对路径
        String deskpath="C:"+File.separator+"Users"+File.separator+"14665"
                +File.separator+"Desktop"+File.separator+"泫雅1.jpg";
        //判断父路径是否存在，若不存在则创建父路径
        CopyFileUtil.createParentFilePath(sourcepath);
        //将IO操作放置子线程中进行
        new Thread(()->{
            long start=System.currentTimeMillis();
            try {
                copy(sourcepath,deskpath);
            } catch (IOException e) {
                e.printStackTrace();
            }
            long end=System.currentTimeMillis();
            System.out.println("共耗时："+(end-start)+"毫秒");
            }).start();
    }
    public static void copy(String sourcepath,String deskpath) throws IOException {
        //获得终端对象及输出输入流
        File sourceFile=new File(sourcepath);
        File deskFile=new File(deskpath);
        InputStream inputStream=new FileInputStream(sourceFile);
        OutputStream outputStream=new FileOutputStream(deskFile);
        //拷贝（以缓冲区为单位进行读取速率会提高很多相比于一个一个字节的拷贝，但并不意味着缓冲区开的越大读取速度越快，只要有了缓冲区读取快慢就取决于磁盘读取速度）
        byte[] data=new byte[1024];
        int len=0;
        while ((len=inputStream.read(data))!=-1){
            outputStream.write(data,0,len);
        }
        //关闭流
        inputStream.close();
        outputStream.close();
    }
}
```



#### 4.内存流：以内存为终端的输入输出流

概念：内存流是以内存为终端的输入输出流，可看做临时文件，全局唯一(因只有一个内存)。

**内存流应用场景：需要进行IO处理但是又不希望产生文件时使用内存流。**

#### 字节内存流

```java
1.ByteArrayOutputStream:字节内存输出流
public class ByteArrayOutputStream extends OutputStream {}
2.ByteArrayInputStream：字节内存输入流
public class ByteArrayInputStream extends InputStream {
```

字节内存流构造方法

```java
public ByteArrayOutputStream(){}//输出流不需要参数，因为内存只有一个，唯一（FileOutputStream()需要参数是因为文件终端有多个）
public ByteArrayInputStream(byte buf[]) {}//将指定的字节数组内容存放到内存中
```

#### 字符内存流

```java
1.CharArrayWriter:字符内存输出流
public class CharArrayWriter extends Writer {}
2.CharArrayReader：字符内存输入流
public class CharArrayReader extends Reader {}
```

输出流继承关系：

![9](C:\Users\14665\source\JAVA学习\学习总结File输入输出流、打印流、Java.util.Scanner类及装饰设计模式\9.png)

输入流继承关系：

![10](C:\Users\14665\source\JAVA学习\学习总结File输入输出流、打印流、Java.util.Scanner类及装饰设计模式\10.png)

#### 通过内存流实现大小写转换。

```java
import java.io.*;

public class Test{
    public static void main(String[] args) throws Exception {
        String str="hello";
        ByteArrayInputStream input=new ByteArrayInputStream(str.getBytes());
        ByteArrayOutputStream output=new ByteArrayOutputStream();
        int temp=0;
        while ((temp=input.read())!=-1){
            output.write(Character.toUpperCase(temp));
        }
        System.out.println(output);
        input.close();
        output.close();
    }
}
```



#### 5.打印流：输出流的强化版本---属于装饰设计模式

**打印流设计的主要目的是为了解决OutputStream的设计问题(因OutputStream所有的数据输出到终端时均需要转换为字节数组，那么输出数据为Int、double类型就会不方便)，**其本质不会脱离OutputStream。

自行设计简单打印流：

```java
import java.io.*;

class PrintUtil{
    private OutputStream out;

    public PrintUtil(OutputStream out) {
        this.out = out;
    }
    public void print(String str){
        try {
            this.out.write(str.getBytes());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    public void println(String str){
        this.print(str+"\r\n");
    }
    public void print(int data){
        this.print(String.valueOf(data));
    }
    public void println(int data){
        this.print(data+"\r\n");
    }
    public void print(char[] data){
        this.print(String.valueOf(data));
    }
    public void println(char[] data){
        this.print(data+"\r\n");
    }
    public void close(){
        try {
            this.out.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
public class Test{
    public static void main(String[] args) throws FileNotFoundException {
        //1.获取file对象
        File file=new File("C:"+File.separator+"Users"
                +File.separator+"14665"+File.separator+"Desktop"+File.separator+"TestIO.txt");
        if(!file.getParentFile().exists()){
            file.getParentFile().mkdirs();
        }
        //2.获取输出流
        PrintUtil printUtil=new PrintUtil(new FileOutputStream(file));
        //3.数据写入
        printUtil.print("haha");
        printUtil.println(900);
        char[] data=new char[]{'a','b','c','d'};
        printUtil.print(data);
        //4.关闭流
        printUtil.close();
    }
}
```

![11](C:\Users\14665\source\JAVA学习\学习总结File输入输出流、打印流、Java.util.Scanner类及装饰设计模式\11.png)

通过以上我们可以看到经过简单的处理后，使用OutputStream更加灵活方便，打印流其本质是对输出流进行包装而已。Java提供专门的**字节打印流(PrintStream)、字符打印流(PrintWriter)**，其内部实现与上述自定义打印流一致。**使用PrintWriter的几率相对较高。**

字节打印流继承关系：

![字节打印流PrintStream源码剖析](C:\Users\14665\source\JAVA学习\学习总结File输入输出流、打印流、Java.util.Scanner类及装饰设计模式\字节打印流PrintStream源码剖析.jpg)

字符打印流继承关系：

![字符打印流PrintWriter源码剖析](C:\Users\14665\source\JAVA学习\学习总结File输入输出流、打印流、Java.util.Scanner类及装饰设计模式\字符打印流PrintWriter源码剖析.jpg)

PrintStream、PrintWriter继承结构：

![12](C:\Users\14665\source\JAVA学习\学习总结File输入输出流、打印流、Java.util.Scanner类及装饰设计模式\12.png)

从以上继承结构，我们可以看到字节打印流是对OutputStream类进行包装、字符打印流可对Writer、OutputStream类进行包装实现。通过分析我们可以看出这种设计模式有点类似代理设计模式，**但代理设计模式有两个特点如下：**

**1）代理是以接口为使用原则的设计模式；**

**2）最终用户可以调用的方法一定是接口定义的方法。**

#### 打印流的设计属于装饰设计模式，基于抽象类。其核心特点是某个类的功能（如OutputStream抽象类的write()方法），但为了更好的使用效果，让其支持的功能更多一些，对其进行包装。

**装饰设计模式优点：很容易更换装饰类来达到不同的操作；缺点：造成类结构复杂。**

字符打印流的简单使用：

```java
import java.io.File;
import java.io.FileNotFoundException;
import java.io.PrintWriter;

public class Test{
    public static void main(String[] args) throws FileNotFoundException {
        File file=new File("C:"+File.separator+"Users"
         +File.separator+"14665"+File.separator+"Desktop"+File.separator+"TestIO.txt");
        if(!file.getParentFile().exists()){
            file.getParentFile().mkdirs();
        }
        //使用字符打印流
        PrintWriter printWriter=new PrintWriter(file);
        //数据写入
        printWriter.print(200);
        printWriter.print("haha");
        //关闭流
        printWriter.close();
    }
}
```

![13](C:\Users\14665\source\JAVA学习\学习总结File输入输出流、打印流、Java.util.Scanner类及装饰设计模式\13.png)

我们在以后向文件中写入数据时就可直接使用打印流，方便多个类型直接写入。

**但需要注意的是打印流写入文件调用print()/println()方法，所有数据类型均可直接写入。**

我们知道在c语言中有printf()函数，在输出时可使用一些占位符，如%s(字符串)、%d(整数)等。**从JDK1.5以后，PrintStream类中也追加了此种操作(格式化操作)。**

```java
//printf方法源码实现(内部调用format()方法实现）
public PrintWriter printf(String format, Object ... args) {
        return format(format, args);
    }
public PrintWriter format(String format, Object ... args) {}
```

```java
//printf()方法简单应用
import java.io.File;
import java.io.FileNotFoundException;
import java.io.PrintWriter;

public class Test{
    public static void main(String[] args) throws FileNotFoundException {
        String name="lala";
        Integer age=12;
        double salary=20.0000000004;
        File file=new File("C:"+File.separator+"Users"
                +File.separator+"14665"+File.separator+"Desktop"+File.separator+"TestIO.txt");
        if(!file.getParentFile().exists()){
            file.getParentFile().mkdirs();
        }
        //使用字符打印流
        PrintWriter printWriter=new PrintWriter(file);
        //数据写入
        printWriter.printf("姓名：%s,年龄：%d,工资：%1.2f",name,age,salary);
        //关闭流
        printWriter.close();
    }
}

//String类中也存在一个格式化字符串的方法(format)
public class Test{
    public static void main(String[] args){
        String name="lala";
        Integer age=12;
        double salary=20.0000000004;
        System.out.println(String.format("姓名：%s,年龄：%d,薪水：%1.2f",name,age,salary));
    }
}

//姓名：lala,年龄：12,薪水：20.00
```

#### 6.System类对IO的支持

通过自定义打印流简单模拟系统给定的打印流方法的内部实现我们可以发现，该类中定义的print()、println()方法都很熟悉。**实际上我们一直使用的系统输出就是利用IO流模式实现。**在System类中定义了三个操作常量：

```java
1)标准输出(显示器)：public static final PrintStream out;(System.out)
2)错误输出：public static final PrintStream err;
3)标准输入(键盘)：public static final PrintStream in;(System.in)
```

##### 6.1系统输出

系统输出总共有两个常量out、err。

```java
public class Test{
    public static void main(String[] args){
        String name="lala";
        System.out.println("out:"+name);
        System.err.println("err:"+name);
    }
}
```

![14](C:\Users\14665\source\JAVA学习\学习总结File输入输出流、打印流、Java.util.Scanner类及装饰设计模式\14.png)

我们可以看到out、err两个唯一的区别是out打印数据颜色为黑色，err打印数据颜色为红色，除此之外均一样。现在err只是一个保留的属性，几乎用不到。

由于System.out是PrintStream的实例化对象，PrintStream是OutputStream的子类，**故使用System.out作为OutputStream的实例化对象可使输出终端变为显示屏。**

```java
public class Test{
    public static void main(String[] args) throws IOException {
        OutputStream outputStream=System.out;
        outputStream.write("hahahah".getBytes());
    }
}
```

**6.2系统输入**

系统输入指的是用户通过键盘进行输入。java本身没有直接的用户输入处理，要想实现此功能，必须通过java.io模式实现。

**System.in是InputStream的直接对象。**

```java
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.io.PrintStream;

public class Test{
    public static void main(String[] args) throws IOException {
        InputStream in=System.in;
        byte[] data=new byte[1024];
        System.out.println("请输入内容：");
        int len=in.read(data);
        System.out.println("输出内容为："+new String(data,0,len));
    }
}
```

但此种模式会进入阻塞模式，也就是必须等到用户输入完成(敲下回车)后才继续向下执行。针对此问题，**我们可以引入内存流，将数据先保存在数据流中之后一次取出。**

```java
import java.io.*;

public class Test{
    public static void main(String[] args) throws IOException {
        InputStream in=System.in;
        ByteArrayOutputStream byteArrayOutputStream=new ByteArrayOutputStream();
        byte[] data=new byte[10];
        System.out.println("请输入内容：");
        int len=0;
        while ((len=in.read(data))!=-1){
            byteArrayOutputStream.write(data,0,len);
            if(len<data.length){
                break;
            }
        }
        in.close();
        byteArrayOutputStream.close();
        System.out.println("输出内容为："+new String(byteArrayOutputStream.toByteArray()));
    }
}
```

但此种实现太过于复杂，一般不用。

#### 7.两种输入流

7.1**字符输入流(BufferReader)、字节输入流(BufferedInputStream)**

**BufferReader类属于一个缓冲的输入流，而且是字符流的操作对象。**

```java
public BufferedReader(Reader in){};
public class BufferedReader extends Reader {}
//该方法可每次读取一行数据（默认以回车作为换行符）
public String readLine() throws IOException {}
```

BufferReader类继承结构图：



利用BufferReader类实现键盘输入：

```java
import java.io.*;

public class Test{
    public static void main(String[] args) throws IOException {
       BufferedReader bufferedReader=new BufferedReader(new InputStreamReader(System.in));
        System.out.println("请输入信息");
        String str=bufferedReader.readLine();//默认使用回车换行
        System.out.println(str);
}
}
```

使用该类实现键盘输入最大缺点为默认使用回车换行。该类已经被Scanner类替代。

#### 7.2Java.util.Scanner类（唯一一个与IO有关但却不在IO包中的类）

```java
public final class Scanner implements Iterator<String>,Closeable{}
```

Scanner类解决了BufferRead类的缺陷。

**Scanner类是一个专门进行输入流处理的程序类，利用该类可处理各种数据，包括正则表达式。**

```java
1.public boolean hasNextXxx();//判断是否有指定类型数据输入
2.public 数据类型 nextXxx();//取得指定类型的参数
3.public Scanner useDelimiter(Pattern pattern);//定义分隔符
4.public Scanner(InputStream source);//构造方法
```

使用Scanner类实现数据输入：

```

```

```java
import java.io.*;
import java.util.Scanner;

public class Test{
    public static void main(String[] args) throws IOException {
        Scanner scanner=new Scanner(System.in);
        System.out.println("请输入数据：");
        //有输入数据，默认空格为分隔符(故不判断空字符串)
        if(scanner.hasNext()){
            System.out.println("输入内容为："+scanner.next());
        }
        scanner.close();
}
}
```

![15](C:\Users\14665\source\JAVA学习\学习总结File输入输出流、打印流、Java.util.Scanner类及装饰设计模式\15.png)

通过该代码运行结果我们可以看到默认空格为分隔符，但这种就很有可能会丢失数据。针对此问题，我们可自定义分隔符解决；

```java
import java.io.*;
import java.util.Scanner;

public class Test{
    public static void main(String[] args) throws IOException {
        Scanner scanner=new Scanner(System.in);
        System.out.println("请输入数据：");
        //可使用useDelimiter方法自定义分隔符(换行)
        scanner.useDelimiter("\n");
        if(scanner.hasNext()){
            System.out.println("输入内容为："+scanner.next());
        }
        scanner.close();
}
}

//请输入数据：
//nihao ha ha
//输入内容为：nihao ha ha
```

Scanner类可以对接收的数据类型使用正则表达式判断：

```java
import java.io.*;
import java.util.Scanner;

public class Test{
    public static void main(String[] args) throws IOException {
        Scanner scanner=new Scanner(System.in);
        System.out.println("请输入生日：");
        if(scanner.hasNext("\\d{4}-\\d{2}-\\d{2}")){//d表示数字，{4}表示此处必须为4个数字
            String birthday=scanner.next();
            System.out.println("输入生日为："+birthday);
        }else{
            System.out.println("输入的格式非法");
        }
        scanner.close();
}
}

//请输入生日：
//2000-09-80
//输入生日为：2000-09-80

//请输入生日：
//2000-9-8
//输入的格式非法
```

但是现在开发中不可能数据输入是从键盘上直接输入，一般都是从文件中直接读取即可。

使用Scanner本身接收InputStream对象，那么也就意味着可以接收任意输入流，例如文件输入流。Scanner完美替代了BufferReader，而且更好实现了InputStream的操作。

Scanner类操作文件：

```java
import java.io.*;
import java.util.Scanner;

public class Test{
    public static void main(String[] args) throws IOException {
        File file=new File("C:"+File.separator+"Users"
                +File.separator+"14665"+File.separator+"Desktop"+File.separator+"TestIO.txt");
        //1.获取Scanner类对象
        Scanner scanner=new Scanner(new FileInputStream(file));
        //2.文件读取
        scanner.useDelimiter("\n");
        while (scanner.hasNext()){
            System.out.println(scanner.next());
        }
        //3.关闭流
        scanner.close();
}
}
```

#### 总结：以后除了处理二进制文件拷贝之外，只要是针对程序的信息输出都是使用打印流(PrintStream、PrintWriter)，信息输入使用Scanner.

#### 8.序列化与反序列化

序列化概念：将类的内存中类的实例化对象转成二进制数据流(object->byte[])，传输到网络、数据库或保存到文件中。

反序列化概念：将二进制流文件转变为类的实例化对象(byte[]->object).

#### 注意：Java中要对一个类进行序列化，那么该类必须实现Serializable接口。

```java
package java.io;
public interface Serializable {//该接口中无任何方法，类实现该接口只是作为序列化的一个标志而已
}
```

**要实现序列化与反序列化操作，需要使用java.io包中的ObjectOutputStream(序列化)、ObjectInputStream(反序列化)类。**

**ObjectOutputStream：**

```java
1.public class ObjectOutputStream
    extends OutputStream implements ObjectOutput, ObjectStreamConstants
{}
2.public ObjectOutputStream(OutputStream out) throws IOException {}//构造方法接收OutputStream子类实例化对象
3.public final void writeObject(Object obj) throws IOException {}//该方法可接受任意对象，且可将对象转化为二进制
```

**ObjectInputStream：**

```java
1.public class ObjectInputStream
    extends InputStream implements ObjectInput, ObjectStreamConstants
{}
2.public ObjectInputStream(InputStream in) throws IOException {}//构造方法接收InputStream子类实例化对象
```

**ObjectOutputStream：继承结构**:

![17](C:\Users\14665\source\JAVA学习\学习总结File输入输出流、打印流、Java.util.Scanner类及装饰设计模式\17.png)

**ObjectInputStream：继承结构**:

![18](C:\Users\14665\source\JAVA学习\学习总结File输入输出流、打印流、Java.util.Scanner类及装饰设计模式\18.png)

序列化与反序列化简单实现：

```java
import javax.tools.JavaCompiler;
import java.io.*;

class Person implements Serializable{
    private String name;
    private Integer age;
    private String skill;
    public Person(String name, Integer age, String skill) {
        this.name = name;
        this.age = age;
        this.skill = skill;
    }
    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", skill='" + skill + '\'' +
                '}';
    }
}
public class Test{
    public static void main(String[] args) throws IOException {
        File file=new File("C:"+File.separator+"Users"
                +File.separator+"14665"+File.separator+"Desktop"+File.separator+"TestIO.txt");
        Person person=new Person("拉拉",20,"Java");
        if(file.getParentFile().exists()){
            try {
                ser(file,person);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        if(file.exists()){
            try {
                dser(file,person);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
    //序列化
    public static void ser(File file,Object object)throws Exception{
        //1.获得类反序列ObjectOutputStream对象
        ObjectOutputStream objectOutputStream=new ObjectOutputStream(new FileOutputStream(file));
        //2.数据输入
        objectOutputStream.writeObject(object);//writeObject(Object obj)可接收任意对象
        //3.关闭流
        objectOutputStream.close();
    }
    //反序列化
    public static void dser(File file,Object object)throws Exception{
        //1.获得类反序列化对象
        ObjectInputStream objectInputStream=new ObjectInputStream(new FileInputStream(file));
        //2.数据读取
        System.out.println(objectInputStream.readObject());;
        //3.关闭流
        objectInputStream.close();
    }
}

//Person{name='拉拉', age=20, skill='Java'}
```

#### transient关键字（重要）--修饰类中属性时，该类不参与序列化

某个类在进行序列化时，其中某些属性若不想被序列化(不想被保存)就可用transient关键字修饰。

```java
import javax.tools.JavaCompiler;
import java.io.*;

class Person implements Serializable{
    private String name;
    //age属性被transient修饰
    private transient Integer age;
    private String skill;
    public Person(String name, Integer age, String skill) {
        this.name = name;
        this.age = age;
        this.skill = skill;
        System.out.println("原Person打印结果:"+this.toString());
    }
    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", skill='" + skill + '\'' +
                '}';
    }
}
public class Test{
    public static void main(String[] args) throws IOException {
        File file=new File("C:"+File.separator+"Users"
                +File.separator+"14665"+File.separator+"Desktop"+File.separator+"TestIO.txt");
        Person person=new Person("拉拉",20,"Java");
        if(file.getParentFile().exists()){
            try {
                ser(file,person);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        if(file.exists()){
            try {
                dser(file,person);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
    //序列化
    public static void ser(File file,Object object)throws Exception{
        //1.获得类反序列ObjectOutputStream对象
        ObjectOutputStream objectOutputStream=new ObjectOutputStream(new FileOutputStream(file));
        //2.数据输入
        objectOutputStream.writeObject(object);//writeObject(Object obj)可接收任意对象
        //3.关闭流
        objectOutputStream.close();
    }
    //反序列化
    public static void dser(File file,Object object)throws Exception{
        //1.获得类反序列化对象
        ObjectInputStream objectInputStream=new ObjectInputStream(new FileInputStream(file));
        //2.数据读取
        System.out.println("后Person打印结果："+objectInputStream.readObject());;
        //3.关闭流
        objectInputStream.close();
    }
}

//原Person打印结果:Person{name='拉拉', age=20, skill='Java'}
//后Person打印结果：Person{name='拉拉', age=null, skill='Java'}
```

类序列化后文件中存储：

![16](C:\Users\14665\source\JAVA学习\学习总结File输入输出流、打印流、Java.util.Scanner类及装饰设计模式\16.png)

应用场景：类中敏感信息(如密码)属性可用transient关键字修饰。反序列化时不会直接打印出来，存储在文件中的二进制流人类也不能直接看懂，达到了很好的保密效果。