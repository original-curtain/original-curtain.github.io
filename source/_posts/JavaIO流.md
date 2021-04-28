---
title: JavaIO流
date: 2021-04-28 19:52:04
tags: Java
---
输入流就是从外部文件输入到内存，输出流主要是从内存输出到文件。

IO流中有很多类，IO流主要分为字符流和字节流。
- 在应用中，经常要完全是字符的一段文本输出去或读进来,由于这样的需求很广泛，所以专门提供了字符流的包装类。
- 字节流的操作不会经过缓冲区（内存）而是直接操作文本本身的，而字符流的操作会先经过缓冲区（内存）然后通过缓冲区再操作文件。
<!--more-->
# IO流模式
## 装饰者模式
就是动态地给一个对象添加一些额外的职责（对于原有功能的扩展）

## 适配器模式
将一个类的接口转换成客户期望的另一个接口，让原本不兼容的接口可以合作无间。

# NIO流
传统的IO流是阻塞式的，对于NIO，它是非阻塞式
## 与IO的区别
- 面向流与面向缓冲：IO是面向流的，NIO是面向缓冲区的
- 阻塞与非阻塞
- 选择器：Java NIO的选择器允许一个单独的线程来监视多个输入通道，你可以注册多个通道使用一个选择器，然后使用一个单独的线程来“选择”通道：这些通道里已经有可以处理的输入，或者选择已准备写入的通道。这种选择机制，使得一个单独的线程很容易来管理多个通道

# 常见IO流
- 字节流
    - 抽象基类：InputStream和OutputStream
    - 派生类：FileInputStream、FileOutputStream
    - 带缓冲：BufferedInputStream 、BufferedOutputStream
- 字符流
    - 抽象基类:Reader和Writer
    - 派生类：FileReader、FileWriter
    - 带缓冲：BufferedReader、BufferedWriter
- 操作：
```
// 输入字节流
public static void readTest4() throws IOException{          

    File file = new File("F:\\a.txt");        //找到目标文件

    FileInputStream fis = new FileInputStream(file);   //建立数据输入通道

    int length = 0;    //为什么要用length ？下面会提到

    byte[] buf = new byte[4];  //建立缓冲数组，一次读取4个长度，最后如不够4个长度，只读取剩下的长度内容

    while((length = fis.read(buf))!=-1){

        System.out.print(new String(buf,0,length));

        }

    fis.close();

    }

// 输入字符流
public static void readTest1() throws IOException{          

    File file = new File("E:\\a.txt");              

    FileReader fr = new FileReader（file);               

    int length=0;

    char[] buf = new char[1024];

    while((length=fr.read(buf))!=-1){

        System.out.print(new String(buf,0,length));

        }

//缓冲输入字节流
public static void readTest1() throws IOException{          

    File file = new File("F:\\a.txt");              //找到目标文件

    FileInputStream fis = new FileInputStream(file);        //建立数据输入通道

    BufferedInputStream bis = new BufferedInputStream（fis）; //建立缓冲输入字节流

    int content = 0;                    

    while((content=bis.read())!=-1){

        System.out.print((char)content);
    }

    bis.close();


    }

//缓冲输入字符流
public static void readTest1() throws IOException{          

    File file = new File("E:\\a.txt");              //找到目标文件

    FileReader fr = new FileReader（file);       //建立数据输入通道

    BufferedReader br = new BufferedReader(fr);

    String line = null;

    while((line=br.readLine())!=null){      //readline一次读取一行

        System.out.print(line);

        }

    br.close();

    }

//输出字节流
    public static void writeTest2() throws IOException{     

        File file = new File("F:\\a.txt");      //找到目标文件,如果不存在，自动创建，如果存在，先清空数据再写入

        FileOutputStream fos = new FileOutputStream(file,true);     //建立数据输出通道,加true表示不清空，在原文件后面添加

        String data="hello word";

        fos.write(data.getBytes()); 

        fos.close();


    }

//输出字符流
    public static void writerTest1() throws IOException{            

    File file = new File("E:\\a.txt");              

    FileWriter fw = new FileWriter（file);   

    String data="今天很好！";

    fw.write(data); 

    fw.close();

    }

//缓冲输出字节流
public static void writeTest1() throws IOException{         

    File file = new File("F:\\a.txt");              //找到目标文件

    FileOutputStream fos = new FileOutputStream(file);      //建立数据输入通道

    BufferedOutputStream bos = new BufferedOutputStream（fos）;   //建立缓冲输入字节流

    bos.write("hello world".getbytes（）);

    bos.close();                //只有调用close方法才会写到硬盘，否则只是写到内部的字节数组中


    }

//缓冲输出字符流
public static void writerTest1() throws IOException{            

    File file = new File("E:\\a.txt");              //找到目标文件

    FileWriter fw = new FileWriter（file);       //建立数据输出通道

    BufferedWriter bw = new BufferedWriter(fw);

    bw.write("大家好！");

    bw.close();

    }
```
为什么用length
```
public static void runio() throws IOException{

        File file=new File("E:\\a.txt");

        FileInputStream fis=new FileInputStream(file);

        byte[] bytes= new  byte[4];

        int length=-1;

        while ((length=fis.read(bytes,0,2))!=-1){
            //虽然数组长度为4,但是这里我们设置了2所以每次输出2
            System.out.println(length);
            //因为每次得到的是新的数组,所以每次都是新数组的"0-length"
            System.out.println(new String(bytes,0,length));

        }

    }
//上述只是读数据，当写数据时：

 public static void runio() throws IOException{

        File file=new File("E:\\a.txt");
        FileInputStream fis=new FileInputStream(file);
        FileOutputStream fos = new FileOutputStream("E:\\b.txt");

        byte[] bytes= new  byte[4];

        int length=-1;

        while ((length=fis.read(bytes,0,2))!=-1){
            //往流里边写入缓冲字节数组中的所有内容,，如果不使用length，不满整个数组长度的”空余内容”也会加入
            fos.write(bytes,0,length);

        }

    }
```