# Table of Contents

* [Week4-作业-2](#week4-作业-2)
  * [题目](#题目)
  * [解题](#解题)
    * [方案1](#方案1)
    * [方案2](#方案2)
    * [方案3](#方案3)
    * [方案4](#方案4)
    * [方案5](#方案5)
    * [方案6](#方案6)
    * [方案7](#方案7)
    * [方案8](#方案8)
    * [方案9](#方案9)


# Week4-作业-2

## 题目

> 思考有多少种方式，在 main 函数启动一个新线程，运行一个方法，拿到这 个方法的返回值后，退出主线程?

## 解题

### 方案1

通过`List`接收子线程的值，通过`join`让主线程等待子线程结束。

```
public class Week4Task {

    private static String sayHi() {
        return "Hi";
    }

    public static void main(String[] args) throws InterruptedException {
        List<String> data = new ArrayList<>(1);
        Thread thread = new Thread(() -> {
            String s = sayHi();
            data.add(s);
        });
        thread.start();

        thread.join();
        System.out.println(data.get(0));
    }

}
```

### 方案2

通过`List`接收子线程的值，通过`CountDownLatch`让主线程等待子线程结束。

```
public class Week4Task {

    private static String sayHi() {
        return "Hi";
    }

    public static void main(String[] args) {
        CountDownLatch countDownLatch = new CountDownLatch(1);

        List<String> list = new ArrayList<>(1);
        new Thread(new Runnable() {
            @Override
            public void run() {
                list.add(sayHi());
                countDownLatch.countDown();
            }
        }).start();

        try {
            countDownLatch.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println(list.get(0));
    }

}
```

### 方案3

通过`List`接收子线程的值，通过`CyclicBarrier`让主线程和子线程达到都到达屏障同步点才输出。

```
public class Week4Task {

    private static String sayHi() {
        return "Hi";
    }

    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(2);

        List<String> list = new ArrayList<>(1);
        new Thread(new Runnable() {
            @Override
            public void run() {
                list.add(sayHi());
                try {
                    cyclicBarrier.await();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();

        try {
            cyclicBarrier.await();
        } catch (Exception e) {
            e.printStackTrace();
        }

        System.out.println(list.get(0));
    }

}
```

### 方案4

通过`线程池`配合`Future`让主线程等待子线程结果。

```
public class Week4Task {

    private static String sayHi() {
        return "Hi";
    }

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newSingleThreadExecutor();

        Future<String> future = executorService.submit(Week4Task::sayHi);

        String s = null;
        try {
            s = future.get();
        } catch (Exception e) {
            e.printStackTrace();
        }

        System.out.println(s);
    }

}
```

### 方案5

通过`CompletableFuture`让主线程等待子线程结果。

```
public class Week4Task {

    private static String sayHi() {
        return "Hi";
    }

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newSingleThreadExecutor();

        CompletableFuture<String> future = new CompletableFuture<>();
        executorService.execute(() -> {
            String s = sayHi();
            future.complete(s);
        });

        String s = null;
        try {
            s = future.get();
        } catch (Exception e) {
            e.printStackTrace();
        }
        System.out.println(s);
    }

}
```

### 方案6

通过`FutureTask`让主线程等待子线程结果。

```
public class Week4Task {

    private static String sayHi() {
        return "Hi";
    }

    public static void main(String[] args) {
        FutureTask<String> futureTask = new FutureTask<>(() -> sayHi());
        new Thread(futureTask).start();

        try {
            String s = futureTask.get();
            System.out.println(s);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

### 方案7

通过`LockSupport`暂停主线程，等待子线程结束恢复主线程。

```
public class Week4Task {

    private static String sayHi() {
        return "Hi";
    }

    public static void main(String[] args) throws InterruptedException {
        Thread main = Thread.currentThread();

        List<String> data = new ArrayList<>(1);
        new Thread(() -> {
            String s = sayHi();
            data.add(s);
            LockSupport.unpark(main);
        }).start();

        LockSupport.park(main);
        System.out.println(data.get(0));
    }

}
```

### 方案8

通过`AtomicInteger`让子线程结束执行`incr`，主线程循环等待`AtomicInteger`结果不为0时。

```
public class Week4Task {

    private static String sayHi() {
        return "Hi";
    }

    public static void main(String[] args) throws InterruptedException {
        AtomicInteger atomicInteger = new AtomicInteger();

        List<String> data = new ArrayList<>(1);
        new Thread(() -> {
            String s = sayHi();
            data.add(s);
            atomicInteger.incrementAndGet();
        }).start();

        while (atomicInteger.get() == 0) {
            Thread.sleep(100);
        }
        System.out.println(data.get(0));
    }

}
```

### 方案9

主线程和子线程加一把锁，主线程`wait`，子线程`notifyAll`

```
public class Week4Task {

    private static String sayHi() {
        return "Hi";
    }

    public static void main(String[] args) throws InterruptedException {
        List<String> data = new ArrayList<>(1);
        Thread thread = new Thread(() -> {
            synchronized (data) {
                String s = sayHi();
                data.add(s);
                data.notifyAll();
            }
        });
        thread.start();

        synchronized (data){
            data.wait();
            System.out.println(data.get(0));
        }
    }

}
```
