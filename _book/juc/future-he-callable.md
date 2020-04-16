# Future和Callable

### 为什么需要Future

---

在`JDK 1.5`之前我们进行异步编程的时候，通常会有两种做法，实例化`Thrad`或者实现`Runnable`接口。但是在实际使用中我们会发现， 上面的编程方式会出现两个问题。

1. 不能抛出自定义异常。这通常要求我们在线程执行代码中手动进行`try...catch代码块`，比如下面这段代码。

   ```java
   new Thread(() -> {
   
               try {
                   int k = 1/0;
               } catch (Exception e) {
                   e.printStackTrace();
               }
           }).start();
   ```

   

2. 不能返回数据,这点稍微有点麻烦，通常我们需要定义全局变量或者使用线程间通信来完成，比如下面这段代码。

   ```java
   	final StringBuffer message = new StringBuffer() ;
   	Thread thread = new Thread(() -> {
           message.append("HELLO");
       });
       thread.start();
       thread.join();
   
       System.out.println(message.toString());
   ```

   

但是在`JDK1.5`之后`JAVA`通过引入Callable和Future来解决这两个问题。

### Callable

----

`Callable`接口和`Runnable`接口类似。在Callable中之定义了一个方法。`public T call() throw Exception;`我们可以看到，`Callable`是支持返回数据和抛出异常的。

### Future

---

Future接口可以理解成是一个占位符，表示的是异步操作的结果，这个结果在异步线程执行之后可以获取到，它是一个未来的结果。这个结果可能是你预期的值，也可能是线程执行过程中‘抛出的异常。

### Future和Callable的使用

---

```java
    Callable<String> callable = () -> "Hello";

    ExecutorService executorService = Executors.newFixedThreadPool(1);
    Future<String> future = executorService.submit(callable);
    System.out.println(future.get());
```



先看如上代码，我向线程池中提交了`Callable`接口的任务，并且得到了返回值`Future`，最终通过`Future.get`来获取线程的执行结果。对比两段代码我们可以发现`Future`和传统线程执行的区别。

1. 没有明显的线程间操作。
2. 不需要定义共享变量，避免了线程之间的资源竞争。
3. 在项目中，我们一般会使用线程池，如果这时候还想获取到执行结果，最方便的做法就是使用`Future`了。

我们再来看下`Future`给我们提供的`Api`。

1. `boolean cancel(boolean mayInterruptIfRunning);`取消当前的任务，在任务已完成，或已被取消，以及存在其他不可被取消的原因的时候，返回`false`，取消成功返回`true`。参数`mayInterruptIfRunning`是布尔类型，表示是否需要取消正在执行的任务，为`true`则会中断当前线程，否则处理中的任务会被允许执行结束。

2. `boolean isCancelled();` 任务状态是否已被取消，调用`Future.cancel`成功后，该方法会返回`true`。

3. `boolean isDone();` 任务状态是否已执行完成,。任务执行结束或者调用`Future.cancel`成功后，该方法会返回`true`。
4. `V get() throws InterruptedException, ExecutionException;` 得到执行结果
5. `V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;` 得到执行结果，并且设置超时时间，需要在规定的时间内返回结果，否则会抛出`TimeOutException`



