title: CountDownLatch的使用
date: 2019-1-29
tags: [Java]
categories: Java API
description: 姑且当你是信号量了
---
　　在BInderPool中有用到countdownLatch.整理如下：
  ## 简介
  　　CountDownLatch是一个倒计数的同步锁存器。它和信号量有些差异。CountDownLatch是通过先调用await()阻塞当前进程，在其它线程中一直调用countdown（）减小锁存器中的值count。当其它线程任务完成。count减至0.await()释放锁。而信号量则是信号量大于０时，可正常并发。而小于等于０时阻塞.CountDownLatch不支持重置count。如需重置count，可采用CyclicBarrier。
	
## 样例

```java
public class CountDownLatchDemo {
    private static final int PLAYER_AMOUNT = 5;
    public CountDownLatchDemo() {
        // TODO Auto-generated constructor stub    
     }
    /**
     * @param args
     */
    public static void main(String[] args) {
        //对于每位运动员，CountDownLatch减1后即结束比赛
        CountDownLatch begin = new CountDownLatch(1);
        //对于整个比赛，所有运动员结束后才算结束
        CountDownLatch end = new CountDownLatch(PLAYER_AMOUNT);
        Player[] plays = new Player[PLAYER_AMOUNT];
        
        for(int i=0;i<PLAYER_AMOUNT;i++)
            plays[i] = new Player(i+1,begin,end);
        
        //设置特定的线程池，大小为5
        ExecutorService exe = Executors.newFixedThreadPool(PLAYER_AMOUNT);
        for(Player p:plays)
            exe.execute(p);            //分配线程
        System.out.println("Race begins!");
        begin.countDown();//比赛开始
        try{
            end.await();            //等待end状态变为0，即为比赛结束
        }catch (InterruptedException e) {
            // TODO: handle exception
            e.printStackTrace();
        }finally{
            System.out.println("Race ends!");
        }
        exe.shutdown();
    }
}
```
```java
public class Player implements Runnable {

    private int id;
    private CountDownLatch begin;
    private CountDownLatch end;
    public Player(int i, CountDownLatch begin, CountDownLatch end) {
        // TODO Auto-generated constructor stub
        super();
        this.id = i;
        this.begin = begin;
        this.end = end;
    }

    @Override
    public void run() {
        // TODO Auto-generated method stub
        try{
            begin.await();        //等待哨响，等待begin的状态为0
            Thread.sleep((long)(Math.random()*100));    //随机分配时间，即运动员完成时间
            System.out.println("Play"+id+" arrived.");
        }catch (InterruptedException e) {
            // TODO: handle exception
            e.printStackTrace();
        }finally{
            end.countDown();    //跑到终点使end状态减1，最终减至0
        }
    }
}
```

## 生产者消费者

```java
public class BlockingQueueTest {
    private static ArrayBlockingQueue<Integer> queue = new ArrayBlockingQueue<Integer>(5, true); //最大容量为5的数组堵塞队列
    //private static LinkedBlockingQueue<Integer> queue = new LinkedBlockingQueue<Integer>(5);

    private static CountDownLatch producerLatch; //倒计时计数器
    private static CountDownLatch consumerLatch;

    public static void main(String[] args) {
        BlockingQueueTest queueTest = new BlockingQueueTest();
        queueTest.test();
    }

    private void test(){
        producerLatch = new CountDownLatch(10); //state值为10
        consumerLatch = new CountDownLatch(10); //state值为10

        Thread t1 = new Thread(new ProducerTask());
        Thread t2 = new Thread(new ConsumerTask());

        //启动线程
        t1.start();
        t2.start();

        try {
            System.out.println("producer zero...");
            producerLatch.await(); //main线程进入producerLatch倒计时等待状态，直到state值为0，再继续往下执行
            System.out.println("producer end");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        try {
            System.out.println("consumer waiting...");
            consumerLatch.await(); //main线程进入consumerLatch倒计时等待状态，直到state值为0，再继续往下执行
            System.out.println("consumer end");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        //结束线程
        t1.interrupt();
        t2.interrupt();

        System.out.println("end");
    }

    //生产者
    class ProducerTask implements Runnable{
        private Random rnd = new Random();

        @Override
        public void run() {
            try {
                while(true){
                    queue.put(rnd.nextInt(100)); //如果queue容量已满，则当前线程会堵塞，直到有空间再继续

                    //offer方法为非堵塞的
                    //queue.offer(rnd.nextInt(100), 1, TimeUnit.SECONDS); //等待1秒后还不能加入队列则返回失败，放弃加入
                    //queue.offer(rnd.nextInt(100));

                    producerLatch.countDown(); //state值减1
                    //TimeUnit.SECONDS.sleep(2); //线程休眠2秒
                }
            } catch (InterruptedException e) {
                //e.printStackTrace();
            }  catch (Exception ex){
                ex.printStackTrace();
            }
        }
    }

    //消费者
    class ConsumerTask implements Runnable{
        @Override
        public void run() {
            try {
                while(true){
                    Integer value = queue.take(); //如果queue为空，则当前线程会堵塞，直到有新数据加入

                    //poll方法为非堵塞的
                    //Integer value = queue.poll(1, TimeUnit.SECONDS); //等待1秒后还没有数据可取则返回失败，放弃获取
                    //Integer value = queue.poll();

                    System.out.println("value = " + value);

                    consumerLatch.countDown(); //state值减1
                    TimeUnit.SECONDS.sleep(2); //线程休眠2秒
                }
            } catch (InterruptedException e) {
                //e.printStackTrace();
            } catch (Exception ex){
                ex.printStackTrace();
            }
        }
    }

}
```

ArrayBlockingQueue和LinkedBlockingQueue的区别：

1. 队列中锁的实现不同

    ArrayBlockingQueue实现的队列中的锁是没有分离的，即生产和消费用的是同一个锁；

    LinkedBlockingQueue实现的队列中的锁是分离的，即生产用的是putLock，消费是takeLock

2. 在生产或消费时操作不同

    ArrayBlockingQueue实现的队列中在生产和消费的时候，是直接将枚举对象插入或移除的；

    LinkedBlockingQueue实现的队列中在生产和消费的时候，需要把枚举对象转换为Node<E>进行插入或移除，会影响性能

3. 队列大小初始化方式不同

    ArrayBlockingQueue实现的队列中必须指定队列的大小；

    LinkedBlockingQueue实现的队列中可以不指定队列的大小，但是默认是Integer.MAX_VALUE