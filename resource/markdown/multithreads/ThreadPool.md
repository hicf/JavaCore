<h3 style="padding-bottom:6px; padding-left:20px; color:#ffffff; background-color:#E74C3C;">线程池</h3>

#### interfaces
* Callable
* CompletableFuture.AsynchronousCompletionTask
* Future
* RunnableFuture
* Executor
* **ExecutorService**
* ThreadFactory

#### Classes
* AbstractExecutorService
* Executors
* **ThreadPoolExecutor**
* ThreadPoolExecutor.AbortPolicy
* ThreadPoolExecutor.CallerRunsPolicy
* ThreadPoolExecutor.DiscardOldestPolicy
* ThreadPoolExecutor.DiscardPolicy

#### Enums
* TimeUnit

---
* **ThreadPoolExecutor + ThreadFactory实现线程池**
~~~java
/** 使用 ThreadPoolExecutor 自定义池大小，以免无线占有资源，进而导致OOM问题 */
ExecutorService executorService = new ThreadPoolExecutor(500, 500, 0L, TimeUnit.MILLISECONDS,
                new LinkedBlockingDeque<>(10), new ThreadFactory() {
            @Override
            public Thread newThread(Runnable r) {
                Thread t = new Thread(r);
                // 自定义线程名称（便于排查错误）
                t.setName("xxx线程：" + System.currentTimeMillis());
                return t;
            }
        });

// 使用线程池：用execute() 执行操作
executorService.execute(Runnable command);

// 使用完连接池：shutdown() 关闭连接池
executorService.shutdown();
~~~