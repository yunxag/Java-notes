# Thread多线程

- **Thread多线程**
  - [线程与进程区分](#线程与进程区分)
  - [代码形式理解线程](#代码形式理解线程)
  - [Thread源码分析](#Thread源码分析)





















## <a id="线程与进程区分">线程与进程区分</a>

### 进程

​	当运行记事本程序（Nodepad）时，你就创建了一个Notepad.exe的进程

当你再运行Window资源管理器（explorer）时，你又创建一个explorer.exe的进程

<img src="..\images\thread_1.png" alt="thread_1" style="zoom: 67%;" />

从这里可以看出进程是用来区分各个软件应用的

### 线程

​	 打开Window资源管理器（explorer）复制文件的时候是这样子的，这个时候想再复制一个文件就得等 这个文件复制结束后，才能复制下一个

<img src="..\images\thread_2.png" alt="thread_2" style="zoom:50%;" />

这个时候一个进程同时想做两件事，就有了线程。

从这里开始一个进程的运行，其实在运行进程里的线程

<img src="..\images\thread_3.png" alt="thread_3" style="zoom:50%;" />

可以看出线程是让一个软件同时做两件事的





## <a id='代码形式理解线程'>代码形式理解线程</a>

​	当Java运行时，Java进程会默认把main方法作为一个线程的开始方法，这样的线程就叫做主线程。

​	当我要创建一个线程时，同样要把一个方法做为这个线程开始方法（这是因为Java代码体只能写在方法中）。一般的做法都是创建一个新方法，然后把方法作为这个线程的开始方法。

![thread_4](..\images\thread_4.png)

自从有了线程之后，一个进程的运行就是运行里面的线程



## <a id='Thread源码分析'>Thread源码分析</a>

​	综合上面，假如我有一个能创建开启线程的方法，

```java
native void start0();
```

​	并且能把一个方法作为线程的开始方法（java方法需要类中，我把这个类取名为Thread）

```java
public class Thread {
	//线程开始方法
    public void run() {
        //......
    }
    
    //开启一个线程，并且把this.run()方法作为这个线程的开始方法
    native void start0();
}
```

​	把this.run()方法作为一个线程的开始方法，这样一来其他类用不着这个方法了，故而私有化

```java
public class Thread {
	//线程开始方法
    public void run() {
        //......
    }
    // 加 synchronized 关键字是为了让这个线程在同一时间保持唯一
    // 保证了本类数据只有一个在操作
    public synchronized void start() {
        start0();
    }
    //开启一个线程，并且把this.run()方法作为这个线程的开始方法
    private native void start0();
}
```

​	上面代码根据java特性，每次都需要创建对象重写run方法。索性可以让它传一个方法进来。

```java
public class Thread {
    // 传一个参数进来当作运行方法
    private Runnable target;//Runnable之所以为接口，是为了不耽误别人继承（根据java特性一个类只能继承一个类）

    //Runnable传一个方法进来
    public Thread(Runnable target) {
        this.target = target;
    }
    public void run() {
        // 如果有参数传进来，那就在这里运行传进来的参数
        if (target != null) {
            target.run();
        }
        //如果没有的话，根据java特性这个方法肯定被重写了
    }
    //
    public synchronized void start() {
        start0();
    }
    //开启一个线程，并且把this.run()方法作为这个线程的开始方法
    private native void start0();
}
```



### 线程返回值

​	线程主要是把一个方法作为一个新线程的开始方法，是没有返回值的

​	如果想要返回值只需要运行一个有返回值的方法就行了

```java
class FutureTask<V> implements Runnable{//综合前面，必须要实现Runnable
    V outcome;
    @Override
    public void run() {
        outcome = call(); //通过运行call得到参数用 FutureTask.outcome获取
    }
    V call(){
        return null;
    }
}
```

上面代码根据java特性，每次都需要创建对象重写call方法。索性可以让它传一个方法进来

```java
class FutureTask<V> implements Runnable{//综合前面，必须要实现Runnable
    V outcome;
    // 传一个参数进来当作运行方法
    Callable<V> callable;
    public FutureTask(Callable callable){
       this.callable = callable;
    }
    @Override
    public void run() {
        outcome = callable.call();//实际运行的是call
    }
}
```

运行上面代码当获取outcome时，程序还没执行完就会之间返回outcome默认值null

