

##### 案例

三个线程同时执行不同时间之后，同时处理返回结果。

```java
public class CyclicBarrierDemo {
    private static final Random random = new Random();
    private static final int NUMBER_OF_SOURCES = 3;
    private static String[] dataSources = {"用户信息", "包裹信息", "仓库信息"};

    private static CyclicBarrier cyclicBarrier;
    private static final ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(NUMBER_OF_SOURCES);
        try {
            for (String data : dataSources) {
                executor.submit(new DataLoadRunnable(data, random.nextInt(3000) + 2000));
            }
        } finally {
            executor.shutdown();
        }

        cyclicBarrier = new CyclicBarrier(NUMBER_OF_SOURCES, () -> {
            System.out.println("所有数据加载完毕: ");
            int sum = map.values().stream().mapToInt(Integer::intValue).sum();
            System.out.println("获取最终结果: " + sum);
        });
    }

    static class DataLoadRunnable implements Runnable {
        private String dataName;
        private long delayTime;

        DataLoadRunnable(String dataName, long delayTime) {
            this.dataName = dataName;
            this.delayTime = delayTime;
        }

        @Override
        public void run() {
            System.out.println("开始获取 " + dataName );
            try {
                Thread.sleep(delayTime);
            } catch (InterruptedException e) {
                e.printStackTrace();
                System.out.println(dataName + " 加载失败!");
            }
            int res = random.nextInt(100);
            map.put(dataName, res);

            System.out.println(dataName + " 数据加载成功 = " + res);
            try {
                cyclicBarrier.await();
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
            System.out.println(dataName + " 数据返回~");
        }
    }
}
```

当然可以基于 notify 机制进行实现：启动四个线程，三个线程启动完毕之后，唤醒最终核算线程。

```java 
public class NotifyDemo {
    private static final Random random = new Random();
    private static final int NUMBER_OF_SOURCES = 3;
    private static String[] dataSources = {"用户信息", "包裹信息", "仓库信息"};

    private static final ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
    private static final Object lock = new Object();

    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(NUMBER_OF_SOURCES + 1);
        try {
            for (String data : dataSources) {
                executor.submit(new DataLoadRunnable(data, random.nextInt(3000) + 2000));
            }
            executor.submit(new ResultCalculator());
        } finally {
            executor.shutdown();
        }
    }

    static class DataLoadRunnable implements Runnable {
        private String dataName;
        private long delayTime;

        DataLoadRunnable(String dataName, long delayTime) {
            this.dataName = dataName;
            this.delayTime = delayTime;
        }

        @Override
        public void run() {
            System.out.println("开始获取 " + dataName);
            try {
                Thread.sleep(delayTime);
            } catch (InterruptedException e) {
                e.printStackTrace();
                System.out.println(dataName + " 加载失败!");
            }
            int res = random.nextInt(100);
            map.put(dataName, res);

            System.out.println(dataName + " 数据加载成功 = " + res);
            synchronized (lock) {
                lock.notify();
            }
        }
    }

    static class ResultCalculator implements Runnable {
        @Override
        public void run() {
            synchronized (lock) {
                while (map.size() < NUMBER_OF_SOURCES) {
                    try {
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
            int sum = map.values().stream().mapToInt(Integer::intValue).sum();
            System.out.println("所有数据加载完毕");
            System.out.println("获取最终结果: " + sum);
        }
    }
}
```



