#Android多线程开发最佳实践
##1.1 Android中的线程
一个线程需要经历三个生命阶段：开始，执行，结束。线程会在任务执行完毕之后结束，那么为了确保线程的存活，我们会在执行阶段给线程赋予不同的任务，然后在里面添加退出的条件从而确保任务能够执行完毕后退出。

![](https://github.com/guoxiaoxing/android-multi-thread-pratice/blob/master/image/android_perf_5_thread_lifecycle.png)

在很多时候，线程不仅仅是线性执行一系列的任务就结束那么简单的，我们会需要增加一个任务队列，让线程不断的从任务队列中获取任务去进行执行，另外我们还可能在线程执行的任务过程中与其他的线程进行协作。如果这些细节都交给我们自己来处理，这将会是件极其繁琐又容易出错的事情。

![](https://github.com/guoxiaoxing/android-multi-thread-pratice/blob/master/image/android_perf_5_thread_thread.png)

Android一共提供路4种多线程模型

- AsyncTask: 为UI线程与工作线程之间进行快速的切换提供一种简单便捷的机制。适用于当下立即需要启动，但是异步执行的生命周期短暂的使用场景。
- HandlerThread: 为某些回调方法或者等待某些任务的执行设置一个专属的线程，并提供线程任务的调度机制。
- ThreadPool: 把任务分解成不同的单元，分发到各个不同的线程上，进行同时并发处理。
- IntentService: 适合于执行由UI触发的后台Service任务，并可以把后台任务执行的情况通过一定的机制反馈给UI。

##1.2 HandlerTHread

为了解决这个问题，Android系统为我们提供了Looper，Handler，MessageQueue来帮助实现上面的线程任务模型：

Looper: 能够确保线程持续存活并且可以不断的从任务队列中获取任务并进行执行。

![](https://github.com/guoxiaoxing/android-multi-thread-pratice/blob/master/image/android_perf_5_thread_looper.png)

Handler: 能够帮助实现队列任务的管理，不仅仅能够把任务插入到队列的头部，尾部，还可以按照一定的时间延迟来确保任务从队列中能够来得及被取消掉。

![](https://github.com/guoxiaoxing/android-multi-thread-pratice/blob/master/image/android_perf_5_thread_handler.png)

MessageQueue: 使用Intent，Message，Runnable作为任务的载体在不同的线程之间进行传递。

![](https://github.com/guoxiaoxing/android-multi-thread-pratice/blob/master/image/android_perf_5_thread_messagequeue.png)

把上面三个组件打包到一起进行协作，这就是HandlerThread

![](https://github.com/guoxiaoxing/android-multi-thread-pratice/blob/master/image/android_perf_5_thread_handlerthread.png)

