## Week1-作业-1

### 题目

> 自己写一个简单的 Hello.java，里面需要涉及基本类型，四则运行，if 和 for，然后自己分析一下对应的字节码。


### 源码

```
package com.xyz.linzone.study.jike;

/**
 * @author Zhu WeiJie
 * @date 2021/6/23
 **/
public class Hello {

    public static void main(String[] args) {
        int a = 1;
        int b = 1;

        int sum = 0;
        for (int i = 0; i < 100; i++) {
            if (sum > 200) {
                break;
            }
            sum += a * b;
            a++;
            b = b + 2;
        }
    }
}
```


### 字节码+分析

```
Classfile /Users/zhuweijie/personalProjects/linzone/target/classes/com/xyz/linzone/study/jike/Hello.class
  Last modified 2021-6-23; size 634 bytes
  MD5 checksum 467ccdf86514aec9a4ddb32ccaa9ba17
  Compiled from "Hello.java"
public class com.xyz.linzone.study.jike.Hello
  minor version: 0 // 小版本号
  major version: 52 // 大版本号
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:  // 静态常量池
   #1 = Methodref          #3.#25         // java/lang/Object."<init>":()V
   #2 = Class              #26            // com/xyz/linzone/study/jike/Hello
   #3 = Class              #27            // java/lang/Object
   #4 = Utf8               <init>
   #5 = Utf8               ()V
   #6 = Utf8               Code
   #7 = Utf8               LineNumberTable
   #8 = Utf8               LocalVariableTable
   #9 = Utf8               this
  #10 = Utf8               Lcom/xyz/linzone/study/jike/Hello;
  #11 = Utf8               main
  #12 = Utf8               ([Ljava/lang/String;)V
  #13 = Utf8               i
  #14 = Utf8               I
  #15 = Utf8               args
  #16 = Utf8               [Ljava/lang/String;
  #17 = Utf8               a
  #18 = Utf8               b
  #19 = Utf8               sum
  #20 = Utf8               StackMapTable
  #21 = Class              #16            // "[Ljava/lang/String;"
  #22 = Utf8               MethodParameters
  #23 = Utf8               SourceFile
  #24 = Utf8               Hello.java
  #25 = NameAndType        #4:#5          // "<init>":()V
  #26 = Utf8               com/xyz/linzone/study/jike/Hello
  #27 = Utf8               java/lang/Object
{
  public com.xyz.linzone.study.jike.Hello(); // 默认构造函数
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 7: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/xyz/linzone/study/jike/Hello;

  public static void main(java.lang.String[]);  // main函数
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=5, args_size=1   // 栈大小3，本地变量表大小5，参数1个
         0: iconst_1					// 将1压栈 对应变量a
         1: istore_1					// 将1出栈并并存储到本地变量表槽位1
         2: iconst_1					// 将1压栈 对应变量b
         3: istore_2					// 将1出栈并并存储到本地变量表槽位2
         4: iconst_0					// 将0压栈 对应变量sum
         5: istore_3					// 将0出栈并并存储到本地变量表槽位3
         6: iconst_0					// 将0压栈 对应for循环的变量i
         7: istore        4			// 将0出栈并并存储到本地变量表槽位4
         9: iload         4			// 加载本地变量表槽位4（即i值）入栈
        11: bipush        100			// 将100压栈 对应for循环的上限值
        13: if_icmpge     45			// 将前后入栈的i和100出栈比较 若i>=100 跳到45 就是return
        16: iload_3						// 加载本地变量表槽位3（即sum值）入栈
        17: sipush        200			// 将200压栈 对应 sum > 200 的200
        20: if_icmple     26			// 将前后入栈的sum和200出栈比较 若sum<=200 跳到26继续 否则到下一步直接goto 45终止
        23: goto          45			// 就是代码中的break 跳出循环
        26: iload_3						// 加载本地变量表槽位3（即sum值）入栈
        27: iload_1						// 加载本地变量表槽位1（即a值）入栈
        28: iload_2						// 加载本地变量表槽位2（即b值）入栈
        29: imul							// 将前后入栈的a和b出栈乘法运算 结果入栈
        30: iadd							// 将前后入栈的a*b结果和sum出栈加法运算 结果入栈
        31: istore_3					// 将栈顶结果出栈存储到本地变量表3 即更新sum值
        32: iinc          1, 1		// 本地变量表槽位1加1 即a++
        35: iload_2						// 加载本地变量表槽位2（即b值）入栈
        36: iconst_2					// 将2压栈
        37: iadd							// 将前后入栈的b和2出栈 b+2 结果入栈
        38: istore_2					// 将加的结果出栈并并存储到本地变量表槽位2 即更新b值
        39: iinc          4, 1		// 本地变量表槽位4加1 即i++
        42: goto          9			// for循环一次结束 回到循环开始处
        45: return						// 方法结束
      LineNumberTable:
        line 10: 0
        line 11: 2
        line 13: 4
        line 14: 6
        line 15: 16
        line 16: 23
        line 18: 26
        line 19: 32
        line 20: 35
        line 14: 39
        line 22: 45
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            9      36     4     i   I
            0      46     0  args   [Ljava/lang/String;
            2      44     1     a   I
            4      42     2     b   I
            6      40     3   sum   I
      StackMapTable: number_of_entries = 3
        frame_type = 255 /* full_frame */
          offset_delta = 9
          locals = [ class "[Ljava/lang/String;", int, int, int, int ]
          stack = []
        frame_type = 16 /* same */
        frame_type = 250 /* chop */
          offset_delta = 18
    MethodParameters:
      Name                           Flags
      args
}
SourceFile: "Hello.java"
```