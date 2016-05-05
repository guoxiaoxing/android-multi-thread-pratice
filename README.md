#Android多线程开发最佳实践
##1.1 Android中的线程
###1.1.1 线程模型

一个线程需要经历三个生命阶段：开始，执行，结束。线程会在任务执行完毕之后结束，那么为了确保线程的存活，我们会在执行阶段给线程赋予不同的任务，然后在里面添加退出的条件从而确保任务能够执行完毕后退出。

![](https://github.com/guoxiaoxing/android-multi-thread-pratice/blob/master/image/android_perf_5_thread_lifecycle.png)

在很多时候，线程不仅仅是线性执行一系列的任务就结束那么简单的，我们会需要增加一个任务队列，让线程不断的从任务队列中获取任务去进行执行，另外我们还可能在线程执行的任务过程中与其他的线程进行协作。如果这些细节都交给我们自己来处理，这将会是件极其繁琐又容易出错的事情。

![](https://github.com/guoxiaoxing/android-multi-thread-pratice/blob/master/image/android_perf_5_thread_thread.png)

Android一共提供路4种多线程模型

- AsyncTask: 为UI线程与工作线程之间进行快速的切换提供一种简单便捷的机制。适用于当下立即需要启动，但是异步执行的生命周期短暂的使用场景。
- HandlerThread: 为某些回调方法或者等待某些任务的执行设置一个专属的线程，并提供线程任务的调度机制。
- ThreadPool: 把任务分解成不同的单元，分发到各个不同的线程上，进行同时并发处理。
- IntentService: 适合于执行由UI触发的后台Service任务，并可以把后台任务执行的情况通过一定的机制反馈给UI。

###1.1.2 线程与内存

增加并发的线程数会导致内存消耗的增加，平衡好这两者的关系是非常重要的。我们知道，多线程并发访问同一块内存区域有可能带来很多问题，例如读写的权限争夺问题，[ABA问题](http://www.cnblogs.com/549294286/p/3766717.html)等等。为了解决这些问题，我们会需要引入锁的概念。

Android系统中也无法避免因为多线程的引入而导致出现诸如上文提到的种种问题。Android UI对象的创建，更新，销毁等等操作都默认是执行在主线程，但是如果我们在非主线程对UI对象进行操作，程序将可能出现异常甚至是崩溃。

另外，在非UI线程中直接持有UI对象的引用也很可能出现问题。例如Work线程中持有某个UI对象的引用，在Work线程执行完毕之前，UI对象在主线程中被从ViewHierarchy中移除了，这个时候UI对象的任何属性都已经不再可用了，另外对这个UI对象的更新操作也都没有任何意义了，因为它已经从ViewHierarchy中被移除，不再绘制到画面上了。

不仅如此，View对象本身对所属的Activity是有引用关系的，如果工作线程持续保有View的引用，这就可能导致Activity无法完全释放。除了直接显式的引用关系可能导致内存泄露之外，我们还需要特别留意隐式的引用关系也可能导致泄露。例如通常我们会看到在Activity里面定义的一个AsyncTask，这种类型的AsyncTask与外部的Activity是存在隐式引用关系的，只要Task没有结束，引用关系就会一直存在，这很容易导致Activity的泄漏。更糟糕的情况是，它不仅仅发生了内存泄漏，还可能导致程序异常或者崩溃。

为了解决上面的问题，我们需要谨记的原则就是：不要在任何非UI线程里面去持有UI对象的引用。系统为了确保所有的UI对象都只会被UI线程所进行创建，更新，销毁的操作，特地设计了对应的工作机制(当Activity被销毁的时候，由该Activity所触发的非UI线程都将无法对UI对象进行操作，否者就会抛出程序执行异常的错误)来防止UI对象被错误的使用。



##1.2 HandlerThread

为了解决这个问题，Android系统为我们提供了Looper，Handler，MessageQueue来帮助实现上面的线程任务模型：

Looper: 能够确保线程持续存活并且可以不断的从任务队列中获取任务并进行执行。

![](https://github.com/guoxiaoxing/android-multi-thread-pratice/blob/master/image/android_perf_5_thread_looper.png)

Handler: 能够帮助实现队列任务的管理，不仅仅能够把任务插入到队列的头部，尾部，还可以按照一定的时间延迟来确保任务从队列中能够来得及被取消掉。

![](https://github.com/guoxiaoxing/android-multi-thread-pratice/blob/master/image/android_perf_5_thread_handler.png)

MessageQueue: 使用Intent，Message，Runnable作为任务的载体在不同的线程之间进行传递。

![](https://github.com/guoxiaoxing/android-multi-thread-pratice/blob/master/image/android_perf_5_thread_messagequeue.png)

把上面三个组件打包到一起进行协作，这就是HandlerThread

![](https://github.com/guoxiaoxing/android-multi-thread-pratice/blob/master/image/android_perf_5_thread_handlerthread.png)

