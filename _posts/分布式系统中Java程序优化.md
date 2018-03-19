---
title: 分布式系统中Java程序优化
date: 2018-03-19 10:18:36
tags: Java程序优化
categories: 分布式 
---

# 分布式系统中Java程序优化
选取图书《大型分布式网站架构设计与实践》

## 单例
对于I/O处理、数据库连接、配置文件解析加载等一些非常消耗资源的操作，我们必须对这些势力的创建进行限制，或者始终使用过一个公共的实例，以节约系统开销，这种情况下就需要使用单例模式。  

```
public class Singleton {
    private static Singleton instance;
    static{
        instance=new Singleton();
    }
    private Singleton(){}
    public static Singleton getInstance() {
        return instance;
    }
}

```
“饿汉”式的单例模式在类加载时进行Singleton对象的实例化，通过在构造函数中执行消耗资源的初始化操作，将构造函数设置为私有，使得外部不能够实例化对象，只能够通过getInstance方法来获得单例。
## Future模式
假设一个任务执行起来需要花费一些时间，为了省去不必要的等待时间，可以先获取一个“提货单”，即Future，然后继续处理别的任务，直到"货物"到达，即任务执行完得到结果，此时便可以用"提货单"进行提货，即通过Future对象的到返回值。  
如下代码：

```
public class TestFuture {
    static class Job<Object> implements Callable<Object> {

        @Override
        public Object call() throws Exception {
            return loadData();
        }

        private Object loadData() {
            //do something
            return null;
        }
    }
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<Object> futureTask=new FutureTask<Object>(new Job<>());
        new Thread(futureTask).start();
        //do something else
        Object result=(Object) futureTask.get();
    }
}
```

FutureTask类实现了Future和Runnalbe 借口，FutureTask开始后，loadData()执行可能较长，因此可以先处理其他事情，等其他实情处理好之后，再通过futureTask.get()来获取结果，如果loadData()还未执行完毕，则此时县城会阻塞等待。
## 线程池
有些时候程序执行慢是因为没有充分利用硬件资源，比如，明明是多核CPU，但是程序中却使用单现场串行操作，这种情况下可以将原来的串行操作改为多线程并发，以提高效率。  

但是在生产环节中，如果为每一个任务创建一个线程，将导致大量的线程呗创建，线程创建和销毁本事是有一定代价的，并且活跃的线程会消耗一定的系统资源，如内存。如果课运行线程的可比处理器数量要多，则会有线程闲置，而闲置的线程就会导致内存空间的浪费，并且如果并发树太高，占内存过多，还有可能OutOfMemory.  

这时可以用线程池，代码如下：    

```
public class TestExecutorService {
    static class Job implements Runnable{
        @Override
        public void run() {
            doWork();
        }
        private void doWork() {
            System.out.println("doing...");
        }
    }
    public static void main(String[] args) {
        ExecutorService exec= Executors.newFixedThreadPool(5);
        for (int i=0;i<10;i++)
            exec.execute(new Job());
    }
}
```
使用线程池将互不依赖的几个动作切分，通过线程对串行工作进行改进，将成倍地提高工作效率。  
## 选择就绪
JDK 自1.4起开始提供全新的I/O编程类库，简称NIO，其不但引入了全新的高效的Buffer和Channel,同时还引入了基于Selector的非阻塞I/O机制，将多个异步的I/O操作集中到一个或几个线程当中进行处理。使用NIO代理阻塞I/O能提高程序的并发吞吐能力，降低系统的开销。  

举例说明，假如客户端需要与服务器进行通信，给服务发送hello：   

```
Socket client = new Socket("127.0.0.1", 9999);
OutputStream outputStream=client.getOutputStream();
String hello="hello!";
outputStream.write(hello.getBytes());
outputStream.flush();
outputStream.close();
```

JDK1.4 以前是阻塞的I/O ，服务端一半的处理方式如下:

```
    public static void main(String[] args) throws IOException {
        ServerSocket server = new ServerSocket(9999);
        while (true) {
            Socket s = server.accept();
            new Thread(new ProcessJob(s)).start();
        }
    }
    static class ProcessJob implements Runnable {
        private Socket s;
        public ProcessJob(Socket s) {
            this.s = s;
        }
        @Override
        public void run() {
            int count = 0;
            byte[] b = new byte[1024];
            try {
                while ((count = s.getInputStream().read(b)) > 0) {
                    System.out.write(b, 0, count);
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```

对于每一个请求，单独开一个线程进行相应的逻辑处理，如果客户端的数据传递并不是一直进行，而是断断续续的，则相应的线程需要I/O等待，并进行上下文切换。

使用NIO引入Selector机制后，可以提升程序的并发效率。

```

    static class AcceptJob implements Runnable {

        private Selector clientSelector;
        private Selector serverSelector;

        public AcceptJob(Selector clientSelector, Selector serverSelector) {
            this.clientSelector = clientSelector;
            this.serverSelector = serverSelector;
        }

        @Override
        public void run() {
            try {

                while (true) {
                    int n = serverSelector.selectNow();
                    if (n <= 0) continue;
                    Iterator<SelectionKey> it = serverSelector.selectedKeys().iterator();
                    while (it.hasNext()) {
                        SelectionKey key = it.next();

                        if (key.isAcceptable()) {
                            ServerSocketChannel server = (ServerSocketChannel) key.channel();
                            SocketChannel channel = server.accept();
                            channel.configureBlocking(false);
                            channel.register(clientSelector, SelectionKey.OP_READ);
                        }
                        it.remove();
                    }
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    static class ProcessJob implements Runnable {
        private Selector clientSelector;

        public ProcessJob(Selector clientSelector) {
            this.clientSelector = clientSelector;
        }

        @Override
        public void run() {
            int count = 0;
            ByteBuffer bb = ByteBuffer.allocate(1024);
            try {
                while (true) {
                    int n = clientSelector.selectNow();
                    if (n <= 0) continue;
                    Iterator<SelectionKey> it = clientSelector.selectedKeys().iterator();
                    while (it.hasNext()) {
                        SelectionKey key = it.next();

                        if (key.isReadable()) {
                            SocketChannel client = (SocketChannel) key.channel();
                            while ((count = client.read(bb)) > 0) {
                                System.out.write(bb.array(), 0, count);
                                bb.clear();
                            }
                        }
                        it.remove();
                    }
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws IOException {
        Selector clientSelector = Selector.open();
        Selector serverSelector = Selector.open();

        ServerSocketChannel serverChannel = ServerSocketChannel.open();
        ServerSocket serverSocket = serverChannel.socket();
        serverChannel.bind(new InetSocketAddress(9999));
        serverChannel.configureBlocking(false);
        serverChannel.register(serverSelector, SelectionKey.OP_ACCEPT);

        AcceptJob acceptJob = new AcceptJob(clientSelector, serverSelector);
        ProcessJob processJob = new ProcessJob(clientSelector);
        Thread acceptThread = new Thread(acceptJob);
        Thread processThread = new Thread(processJob);
        acceptThread.start();
        processThread.start();
    }
```

AcceptJob 负责接收新的请求，而ProcessJob负责具体的请求处理，通过clientSelector.selectNow()能够判断目前是否有客户端就绪，而clientSelector.selectedKeys().iterator()能够对当前已经就绪的客户端进行迭代。只有当客户端就绪时，如SelectionKey.isReadable()为true时，ProcessJob才会进行相应的I/O处理。Selector机制使得线程不必等待客户端的I/O就绪，当客户端还没有就绪时，可以处理其他请求，提高服务器的并发吞吐能力，降低了资源消耗。
## 减少上下文切换
锁的竞争会使得更多的线程等待，本来并行的操作变为串行，并且会导致上下文频繁地切换。因此，减少锁的竞争能够提高程序的性能。  
降低锁竞争的一种有效的方式是尽可能地缩短锁持有的时间，比如可以将一部分的锁无关的代码移出同步代码块，特别是执行起来开销较大的操作，以及可能使当前线程被阻塞的操作。  
举例来说，假设一个计算器，对当前正在执行的Job进行计数，当Job开始执行时，计算器加1，当Job执行完毕后，计算器减1，一种实现方式如下：  

```

    static class RunningCount {
        private Integer runningCount = 0;

        public synchronized void run(Job job) {
            runningCount++;
            doSomething(job);
            runningCount--;
        }

        private void doSomething(Job job) {
        }
    }
```
runningCount表示当前正在运行的Job数量，给run()方法加上同步关键子，使得线程不仅需要等待runningCount++和runningCount--操作，还需要等待doSomething()方法的执行，而doSomething()方法又比较消耗时，这样单个线程锁的持有时间将会很长，通过改进，代码的实现如下：  

```
    static class RunningCount {
        private Integer runningCount = 0;

        public void run(Job job) {
            synchronized (runningCount){
                runningCount++;
            }
            doSomething(job);
            synchronized (runningCount) {
                runningCount--;
            }
        }

        private void doSomething(Job job) {
        }
    }
```

在run()方法中，只对runningCount对象加锁，当runningCount的++或者--操作执行完毕后，锁迅速释放，以减少其它线程的等待时间。  
另一种减小锁持有时间的方式是减小锁的粒度，将原先使用单独锁来保护的多个变量采用多个相互独立的锁分别进行保护，这样就能降低线程请求锁的几率，从而减少竞争发生的可能性。当然，使用的锁越多，发生死锁的风险就越高。  
举例来说，假设某站点需要进行投票，统计喜欢平的人数和喜欢梨的人数，并且在投票完成以后改变选择，取消之前的操作，一种实现方式如下：   

```
class LikeCount {

    private Integer likeApple = 0;
    private Integer likePear = 0;

    public synchronized void likeApple() {
        likeApple++;
    }

    public synchronized void likePear() {
        likePear++;
    }

    public synchronized void unlikeApple() {
        likeApple--;
    }

    public synchronized void unlikePear() {
        likePear--;
    }

}  
```  

两个变量likeApple和likePear分别代表喜欢苹果和喜欢梨，同步方法使得对likeApple的操作同时进行。由于LikeCount对象被锁，操作lifePear线程也必须等待，而实际上likeApple和likePear两个变量没有任何内在联系，因此可以将实现方式改为：  

```
class LikeCount {
    //AtomicInteger;
    private Integer likeApple = 0;
    private Integer likePear = 0;
    public void likeApple() {
        synchronized (likeApple) {
            likeApple++;
        }
    }
    public void likePear() {
        synchronized (likePear) {
            likePear++;
        }
    }
    public void unlikeApple() {
        synchronized (likeApple) {
            likeApple--;
        }
    }
    public void unlikePear() {
        synchronized (likePear) {
            likePear--;
        }
    }
}
```  

通过这种方式，将likeApple与likePear两个本身没有任何关系的变量解耦，在对likeApple进行操作的同时，也可以对likePear变量进行操作，提高了程序的吞吐。  
第三种降低锁竞争的方式就是放弃使用独占锁，而使用其它更友好的并发方式来保障数据的同步,比如前文所提到的原子变量，或者是使用读写锁。读/写锁实现了多个读取操作的单个写入操作情况下的数据同步，对于读操作不会修改公共资源，因此在执行读操作时不需要进行加锁，而写入操作必须获得独占锁，以保障队公共资源的修改不会出错。对于多读少写的情况，使用读写锁能够比使用独占锁提供更高的并发数量。