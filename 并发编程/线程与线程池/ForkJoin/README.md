# ForkJoin

Fork/Join 框架是 Java7 提供了的一个用于并行执行任务的框架，是一个把大任务分割成若干个小任务，最终汇总每个小任务结果后得到大任务结果的框架。我们再通过 Fork 和 Join 这两个单词来理解下 Fork/Join 框架，Fork 就是把一个大任务切分为若干子任务并行的执行，Join 就是合并 这些子任务的执行结果，最后得到这个大任务的结果。比如计算 1+2+。。＋ 10000，可以分割成 10 个子任务，每个子任务分别对 1000 个数进行求和，最终汇总这 10 个子任务的结果。Fork/Join 的运行流程图如下：

![](http://cdn3.infoqstatic.com/statics_s1_20160405-0343u1/resource/articles/fork-join-introduction/zh/resources/21.png)

## 框架原理

我们已经很清楚 Fork/Join 框架的需求了，那么我们可以思考一下，如果让我们来设计一个 Fork/Join 框架，该如何设计？这个思考有助于你理解 Fork/Join 框架的设计。

第一步分割任务。首先我们需要有一个 fork 类来把大任务分割成子任务，有可能子任务还是很大，所以还需要不停的分割，直到分割出的子任务足够小。

第二步执行任务并合并结果。分割的子任务分别放在双端队列里，然后几个启动线程分别从双端队列里获取任务执行。子任务执行完的结果都统一放在一个队列里，启动一个线程从队列里拿数据，然后合并这些数据。

Fork/Join 使用两个类来完成以上两件事情：

- ForkJoinTask：我们要使用 ForkJoin 框架，必须首先创建一个 ForkJoin 任务。它提供在任务中执行 fork()和 join()操作的机制，通常情况下我们不需要直接继承 ForkJoinTask 类，而只需要继承它的子类，Fork/Join 框架提供了以下两个子类:

  - RecursiveAction：用于没有返回结果的任务。
  - RecursiveTask：用于有返回结果的任务。

- ForkJoinPool：ForkJoinTask 需要通过 ForkJoinPool 来执行，任务分割出的子任务会添加到当前工作线程所维护的双端队列中，进入队列的头部。当一个工作线程的队列里暂时没有任务时，它会随机从其他工作线程的队列的尾部获取一个任务。

## Usage

让我们通过一个简单的需求来使用下 Fork／Join 框架，需求是：计算 1+2+3+4 的结果。使用 Fork／Join 框架首先要考虑到的是如何分割任务，如果我们希望每个子任务最多执行两个数的相加，那么我们设置分割的阈值是 2，由于是 4 个 数字相加，所以 Fork／Join 框架会把这个任务 fork 成两个子任务，子任务一负责计算 1+2，子任务二负责计算 3+4，然后再 join 两个子任务的 结果。因为是有结果的任务，所以必须继承 RecursiveTask，实现代码如下：

```java
CountTask leftTask = new CountTask(start, middle);
CountTask rightTask = new CountTask(middle + 1, end);

// 执行子任务
leftTask.fork();
rightTask.fork();

// 等待子任务执行完，并得到其结果
int leftResult = (int) leftTask.join();
int rightResult = (int) rightTask.join();

// 合并子任务
sum = leftResult + rightResult;
```

通过这个例子让我们再来进一步了解 ForkJoinTask，ForkJoinTask 与一般的任务的主要区别在于它需要实现 compute 方法，在这个 方法里，首先需要判断任务是否足够小，如果足够小就直接执行任务。如果不足够小，就必须分割成两个子任务，每个子任务在调用 fork 方法时，又会进入 compute 方法，看看当前子任务是否需要继续分割成孙任务，如果不需要继续分割，则执行当前子任务并返回结果。使用 join 方法会等待子任务执行完并 得到其结果。

### 异常处理

ForkJoinTask 在执行的时候可能会抛出异常，但是我们没办法在主线程里直接捕获异常，所以 ForkJoinTask 提供了 isCompletedAbnormally()方法来检查任务是否已经抛出异常或已经被取消了，并且可以通过 ForkJoinTask 的 getException 方法获取异常。使用如下代码：

```java
if(task.isCompletedAbnormally())
{
    System.out.println(task.getException());
}
```

getException 方法返回 Throwable 对象，如果任务被取消了则返回 CancellationException。如果任务没有完成或者没有抛出异常则返回 null。

## 实现原理

ForkJoinPool 由 ForkJoinTask 数组和 ForkJoinWorkerThread 数组组成，ForkJoinTask 数组负责存放程序提交给 ForkJoinPool 的任务，而 ForkJoinWorkerThread 数组负责执行这些任务。
ForkJoinTask 的 fork 方法实现原理。当我们调用 ForkJoinTask 的 fork 方法时，程序会调用 ForkJoinWorkerThread 的 pushTask 方法异步的执行这个任务，然后立即返回结果。代码如下：

```java
public final ForkJoinTask fork() {
    ((ForkJoinWorkerThread) Thread.currentThread()).pushTask(this);
    return this;
}
```

pushTask 方法把当前任务存放在 ForkJoinTask 数组 queue 里。然后再调用 ForkJoinPool 的 signalWork()方法唤醒或创建一个工作线程来执行任务。代码如下：

```java
final void pushTask(ForkJoinTask t) {
    ForkJoinTask[] q; int s, m;
    if ((q = queue) != null) {    // ignore if queue removed
        long u = (((s = queueTop) & (m = q.length - 1)) << ASHIFT) + ABASE;
        UNSAFE.putOrderedObject(q, u, t);
        queueTop = s + 1;         // or use putOrderedInt
        if ((s -= queueBase) <= 2)
            pool.signalWork();
        else if (s == m)
            growQueue();
    }
}
```

ForkJoinTask 的 join 方法实现原理。Join 方法的主要作用是阻塞当前线程并等待获取结果。让我们一起看看 ForkJoinTask 的 join 方法的实现，代码如下：

```java
public final V join() {
    if (doJoin() != NORMAL)
        return reportResult();
    else
        return getRawResult();
}
private V reportResult() {
        int s; Throwable ex;
        if ((s = status) == CANCELLED)
            throw new CancellationException();
        if (s == EXCEPTIONAL && (ex = getThrowableException()) != null)
                UNSAFE.throwException(ex);
            return getRawResult();
}
```

首先，它调用了 doJoin()方法，通过 doJoin()方法得到当前任务的状态来判断返回什么结果，任务状态有四种：已完成(NORMAL)，被取消(CANCELLED)，信号(SIGNAL)和出现异常(EXCEPTIONAL)。

- 如果任务状态是已完成，则直接返回任务结果。
- 如果任务状态是被取消，则直接抛出 CancellationException。
- 如果任务状态是抛出异常，则直接抛出对应的异常。

让我们再来分析下 doJoin()方法的实现代码：

```java
private int doJoin() {
    Thread t; ForkJoinWorkerThread w; int s; boolean completed;
    if ((t = Thread.currentThread()) instanceof ForkJoinWorkerThread) {
        if ((s = status) < 0)
            return s;
        if ((w = (ForkJoinWorkerThread)t).unpushTask(this)) {
            try {
                completed = exec();
            } catch (Throwable rex) {
                return setExceptionalCompletion(rex);
            }
            if (completed)
                return setCompletion(NORMAL);
        }
        return w.joinTask(this);
    }
    else
        return externalAwaitDone();
}

```

在 doJoin()方法里，首先通过查看任务的状态，看任务是否已经执行完了，如果执行完了，则直接返回任务状态，如果没有执行完，则从任务数组里取出任务并执行。如果任务顺利执行完成了，则设置任务状态为 NORMAL，如果出现异常，则纪录异常，并将任务状态设置为 EXCEPTIONAL。

# Links

- [concurrency-torture-testing-your-code-within-the-java-memory-model](http://zeroturnaround.com/rebellabs/concurrency-torture-testing-your-code-within-the-java-memory-model/)
- https://www.baeldung.com/java-fork-join
- [聊聊并发(八)——Fork/Join 框架介绍](http://www.infoq.com/cn/articles/fork-join-introduction)
