---
title: Java反编译工具jad
tags:
  - Java
  - 反编译
  - jad
originContent: >+
  ![image.png](/images/2019/09/06/c8ac68c0-d064-11e9-baaa-a18a2b353fbb.png)


  # Jad(JAva Decompiler)


  Jad(JAva
  Decompiler)是一个Java的反编译器，可以通过命令行把Java的class文件反编译成源代码。下载[点击](https://varaneckas.com/jad/)


  使用方法：


  [1] 反编译一个class文件：jad example.class，会生成example.jad，用文本编辑器打开就是java源代码


  [2] 指定生成源代码的后缀名：jad -sjava example.class，生成example.java


  [3] 改变生成的源代码的名称，可以先使用-p将反编译后的源代码输出到控制台窗口，然后使用重定向，输出到文件：jad -p example.class >
  myexample.java


  [4] 把源代码文件输出到指定的目录：jad -dnewdir -sjava example.class，在newdir目录下生成example.java


  [5] 把packages目录下的class文件全部反编译：jad -sjava packages/*.class


  [6] 把packages目录以及子目录下的文件全部反编译：jad -sjava
  packages/**/*.class，不过你仍然会发现所有的源代码文件被放到了同一个文件中，没有按照class文件的包路径建立起路径


  [7] 把packages目录以及子目录下的文件全部反编译并建立和java包一致的文件夹路径，可以使用-r命令：jad -r -sjava
  packages/**/*.class


  [8] 当重复使用命令反编译时，Jad会提示“whether you want to overwrite it or not”，使用-o可以强制覆盖旧文件


  [9] 还有其他的参数可以设置生成的源代码的格式，可以输入jad命令查看帮助，这里有个人做了简单的翻译：jad命令总结


  [10] 当然，你会发现有些源文件头部有些注释信息，不用找了，jad没有参数可以去掉它，用别的办法吧。


  # 测试


  Main.java

  ```java


  public class Main {

      static volatile int t = 0;
      public static void main(String[] args) {
          int n = 100;
          Thread[] threads = new Thread[n];
          for (int i = 0; i < n; i++) {
              threads[i] = new Thread(new Runnable() {

                  @Override
                  public void run() {
                      for (int i = 0; i < 10000; i++) {
                          add();
                      }
                  }
              });
              threads[i].start();
          }

          while (Thread.activeCount() > 1)
              Thread.yield();

          System.out.println(t);
      }

      static void add() {
          t++;
      }
  }

  ```


  Main.class


  ```class

  cafe babe 0000 0034 0037 0a00 0d00 1d07

  001e 0700 1f0a 0003 001d 0a00 0200 200a

  0002 0021 0a00 0200 220a 0002 0023 0900

  2400 2509 000c 0026 0a00 2700 2807 0029

  0700 2a01 000c 496e 6e65 7243 6c61 7373

  6573 0100 0174 0100 0149 0100 063c 696e

  6974 3e01 0003 2829 5601 0004 436f 6465

  0100 0f4c 696e 654e 756d 6265 7254 6162

  6c65 0100 046d 6169 6e01 0016 285b 4c6a

  6176 612f 6c61 6e67 2f53 7472 696e 673b

  2956 0100 0d53 7461 636b 4d61 7054 6162

  6c65 0700 2b01 0003 6164 6401 0008 3c63

  6c69 6e69 743e 0100 0a53 6f75 7263 6546

  696c 6501 0009 4d61 696e 2e6a 6176 610c

  0011 0012 0100 106a 6176 612f 6c61 6e67

  2f54 6872 6561 6401 0006 4d61 696e 2431

  ...

  ```



  ![image.png](/images/2019/09/06/fcb05680-d065-11e9-baaa-a18a2b353fbb.png)



  反编译后的结果


  ```java

  // Decompiled by Jad v1.5.8g. Copyright 2001 Pavel Kouznetsov.

  // Jad home page: http://www.kpdus.com/jad.html

  // Decompiler options: packimports(3) 

  // Source File Name:   Main.java


  import java.io.PrintStream;


  public class Main

  {

      public Main()
      {
      }

      public static void main(String args[])
      {
          byte byte0 = 100;
          Thread athread[] = new Thread[byte0];
          for(int i = 0; i < byte0; i++)
          {
              athread[i] = new Thread(new Runnable() {

                  public void run()
                  {
                      for(int j = 0; j < 10000; j++)
                          Main.add();

                  }

              }
  );
              athread[i].start();
          }

          for(; Thread.activeCount() > 1; Thread.yield());
          System.out.println(t);
      }

      static void add()
      {
          t++;
      }

      static volatile int t = 0;

  }

  ```


categories:
  - 文章
toc: false
date: 2019-09-06 13:21:38
---

![image.png](/images/2019/09/06/c8ac68c0-d064-11e9-baaa-a18a2b353fbb.png)

# Jad(JAva Decompiler)

Jad(JAva Decompiler)是一个Java的反编译器，可以通过命令行把Java的class文件反编译成源代码。下载[点击](https://varaneckas.com/jad/)

使用方法：

[1] 反编译一个class文件：jad example.class，会生成example.jad，用文本编辑器打开就是java源代码

[2] 指定生成源代码的后缀名：jad -sjava example.class，生成example.java

[3] 改变生成的源代码的名称，可以先使用-p将反编译后的源代码输出到控制台窗口，然后使用重定向，输出到文件：jad -p example.class > myexample.java

[4] 把源代码文件输出到指定的目录：jad -dnewdir -sjava example.class，在newdir目录下生成example.java

[5] 把packages目录下的class文件全部反编译：jad -sjava packages/*.class

[6] 把packages目录以及子目录下的文件全部反编译：jad -sjava packages/**/*.class，不过你仍然会发现所有的源代码文件被放到了同一个文件中，没有按照class文件的包路径建立起路径

[7] 把packages目录以及子目录下的文件全部反编译并建立和java包一致的文件夹路径，可以使用-r命令：jad -r -sjava packages/**/*.class

[8] 当重复使用命令反编译时，Jad会提示“whether you want to overwrite it or not”，使用-o可以强制覆盖旧文件

[9] 还有其他的参数可以设置生成的源代码的格式，可以输入jad命令查看帮助，这里有个人做了简单的翻译：jad命令总结

[10] 当然，你会发现有些源文件头部有些注释信息，不用找了，jad没有参数可以去掉它，用别的办法吧。

# 测试

Main.java
```java

public class Main {

    static volatile int t = 0;
    public static void main(String[] args) {
        int n = 100;
        Thread[] threads = new Thread[n];
        for (int i = 0; i < n; i++) {
            threads[i] = new Thread(new Runnable() {

                @Override
                public void run() {
                    for (int i = 0; i < 10000; i++) {
                        add();
                    }
                }
            });
            threads[i].start();
        }

        while (Thread.activeCount() > 1)
            Thread.yield();

        System.out.println(t);
    }

    static void add() {
        t++;
    }
}
```

Main.class

```class
cafe babe 0000 0034 0037 0a00 0d00 1d07
001e 0700 1f0a 0003 001d 0a00 0200 200a
0002 0021 0a00 0200 220a 0002 0023 0900
2400 2509 000c 0026 0a00 2700 2807 0029
0700 2a01 000c 496e 6e65 7243 6c61 7373
6573 0100 0174 0100 0149 0100 063c 696e
6974 3e01 0003 2829 5601 0004 436f 6465
0100 0f4c 696e 654e 756d 6265 7254 6162
6c65 0100 046d 6169 6e01 0016 285b 4c6a
6176 612f 6c61 6e67 2f53 7472 696e 673b
2956 0100 0d53 7461 636b 4d61 7054 6162
6c65 0700 2b01 0003 6164 6401 0008 3c63
6c69 6e69 743e 0100 0a53 6f75 7263 6546
696c 6501 0009 4d61 696e 2e6a 6176 610c
0011 0012 0100 106a 6176 612f 6c61 6e67
2f54 6872 6561 6401 0006 4d61 696e 2431
...
```


![image.png](/images/2019/09/06/fcb05680-d065-11e9-baaa-a18a2b353fbb.png)


反编译后的结果

```java
// Decompiled by Jad v1.5.8g. Copyright 2001 Pavel Kouznetsov.
// Jad home page: http://www.kpdus.com/jad.html
// Decompiler options: packimports(3) 
// Source File Name:   Main.java

import java.io.PrintStream;

public class Main
{

    public Main()
    {
    }

    public static void main(String args[])
    {
        byte byte0 = 100;
        Thread athread[] = new Thread[byte0];
        for(int i = 0; i < byte0; i++)
        {
            athread[i] = new Thread(new Runnable() {

                public void run()
                {
                    for(int j = 0; j < 10000; j++)
                        Main.add();

                }

            }
);
            athread[i].start();
        }

        for(; Thread.activeCount() > 1; Thread.yield());
        System.out.println(t);
    }

    static void add()
    {
        t++;
    }

    static volatile int t = 0;

}
```


