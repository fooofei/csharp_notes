# C#开始使用多线程 - Console

①	C#中使用多线程的一般方法是
```
// 使用了无参的线程函数
Thread thread = new Thread(new ThreadStart(threadFunc));
thread.IsBackground = true;
thread.Start();
```
②	C#使用多线程的一个诡异的方法是：使用委托的异步调用，系统自动创建后台线程。
```
如：new ThreadStart(threadFunc).BeginInvoke(null,null);
```
③	若是处理简单的事情就不需要自己创建线程了，可以使用Windows线程池，如 `ThreadPool.QueueUserWorkItem`。 我倾向于使用线程池。

# C#开始使用多线程 - WinForm

①	WinForm中创建多线程与Console无异。只是，UI线程突出的WinForm会遇到在子线程中访问控件的困难。

②	在多线程的线程函数中访问控件，方法有

	a. 调用this.Invoke(new [访问控件的方法的委托],[参数列表]) （注意使用try-catch包裹，不使用会造成程序结束时，引发不能访问已释放的对象的异常）, 该委托所对应的方法中不需要使用this.InvokeRequired。

    b. 直接调用访问控件的方法，形式如setText(string msg);不需要调用委托;但是在访问控件的方法，如setText方法的开始处，要加上this.InvokeRequired,形如：
```
if(this.InvokeRequired)
{ 
    try
    {
        this.Invoke(new [该方法的委托],[参数列表]);
    }
    catch (Exception ) { }
    return;
}
```
③	在多线程的线程函数中，当多个线程共享代码段时，并且该代码段访问UI控件时，格外注意访问UI代码段的同步问题。可使用lock解决。
    
    a. lock加在循环内部。
    
    b. lock括弧内的对象一定要是类的静态对象，使用this指针时，在主线程关闭子线程还在运行时会造成子线程无法结束而程序进程无法退出。

# 前台线程和后台线程

前台线程和后台线程的区别在于，当我们子线程还在运行时，窗口关闭，即主线程（UI线程）退出，子线程的运行情况区别，是结束还是运行。

在Console中，前台线程和后台线程区分很明显。如，有代码：
```
static void Main(string[] args)
{
    Thread thread = new Thread(new ThreadStart(threadFunc));
    thread.Start();
}
```

好。我们这是创建了一个线程，线程创建好了，我们的main函数可不会阻塞。因此，我们的主线程已经over。这时候，就区分前台线程和后台线程的区别了。

若为前台线程，则会一直等线程执行完毕，整个程序才over，若为后台线程，则现在就over，你会看到子线程执行了几下下就不执行了。

那么，如何设置前台和后台呢？在创建线程之后，调用 thread.IsBackground = true; 即可。

通过此方式证明，委托的异步调用也是启动了后台线程。

在WinForm中，前台线程和后台线程就不那么明显了。

当为前台线程时，的确程序进程结束的时间会比当为后台线程时更晚一些，但是也不会产生要等待子线程结束程序才关闭或者子线程就不结束的问题。

当为后台线程时，在子线程运行时关闭程序，程序结束的很快，运行良好。

2014_05_14
