---
title: Java基础---io类
date: 2018-04-07 10:43:48
tags: 
	- Java基础
---

# 介绍

`Java` `io`是为了实现“**文件、控制台、网络设备**”等输入输出设备之间的通信。

# IO框架分类

## 字节为单位的IO流

<center>![](http://p2g00vblr.bkt.clouddn.com/%E5%AD%97%E8%8A%82%E5%8D%95%E4%BD%8D%E8%BE%93%E5%85%A5%E8%BE%93%E5%87%BA%E6%B5%81.jpg)</center>

1. OutputStream/InputStream:以字节为单位的IO流的超类
2. ByteArrayOutputStream/ByteArrayInputStream:字节数组IO流
3. PipedOutputStream/PipedInputStream:管道IO流，实现多线程间的管道通信
4. FilterOutputStream/FilterInputStream:过滤IO流
5. DataOutputStream/DataInputStream:数据IO流
6. BufferedOutputStream/BufferedInputStream:缓冲IO流
7. PrintStream:打印输出流
8. FileOutputStream/FileInputStream:文件IO流
9. ObjectOutputStream/ObjectInputStream:对象IO流

## 字符为单位的IO流

<center>![](http://p2g00vblr.bkt.clouddn.com/%E5%AD%97%E7%AC%A6%E4%B8%BA%E5%8D%95%E4%BD%8D%E8%BE%93%E5%85%A5%E8%BE%93%E5%87%BA%E6%B5%81.jpg)</center>

1. Writer/Reader:以字符为单位的IO流的超类
2. CharArrayWriter/CharArrayReader:字符数组的IO流
3. PipedWriter/PopedReader:字符类型的管道IO流
4. FliterWriter/FliterReader:字符类型的过滤IO流
5. BufferedWriter/BufferedReader:字符缓冲IO流
6. OutputStreamWriter/InputStreamReader:字节转字符的IO流
7. FileWriter/FileReader:字符类型的IO流
8. PrintWriter:字符类型的打印输出流