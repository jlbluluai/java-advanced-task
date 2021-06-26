## Week1-作业-2

### 题目

> 自定义一个 Classloader，加载一个 Hello.xlass 文件，执行 hello 方法，此文件内容是一个 Hello.class 文件所有字节（x=255-x）处理后的文件。


### 代码

```
package com.xyz.linzone.study.jike;

import java.io.*;
import java.lang.reflect.Method;

/**
 * @author Zhu WeiJie
 * @date 2021/6/23
 **/
public class MyClassLoader extends ClassLoader {

    public static void main(String[] args) throws Exception {
        Class<?> hello = new MyClassLoader().findClass("Hello");
        Object o = hello.newInstance();

        Method method = hello.getMethod("hello");
        method.invoke(o, null);
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        File file = new File("Hello.xlass");
        long length = file.length();
        // 作业写法 一次性读完
        byte[] buf = new byte[(int) length];
        try (InputStream input = new FileInputStream(file)) {
            input.read(buf);

            for (int i = 0; i < length; i++) {
                buf[i] = (byte) (255 - buf[i]);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }

        return defineClass(name, buf, 0, buf.length);
    }
}
```