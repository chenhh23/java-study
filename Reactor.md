### Reference
- [IBM developer](https://www.ibm.com/developerworks/cn/java/j-cn-with-reactor-response-encode/index.html)
- [官方用户指南](http://projectreactor.io/docs/core/release/reference/docs/index.html)

#### Reactor简介
反应式编程来源于数据流和变化的传播，意味着由底层的执行模型负责通过数据流来自动传播变化。使用反应式流时采用的则是推的方式，即常见的发布者-订阅者模式。当发布者有新的数据产生时，这些数据会被推送到订阅者来进行处理。在反应式流上可以添加各种不同的操作来对数据进行处理，形成数据处理链。这个以声明式的方式添加的处理链只在订阅者进行订阅操作时才会真正执行。反应式流中第一个重要概念是负压（backpressure）。在基本的消息推送模式中，当消息发布者产生数据的速度过快时，会使得消息订阅者的处理速度无法跟上产生的速度，从而给订阅者造成很大的压力。当压力过大时，有可能造成订阅者本身的奔溃，所产生的级联效应甚至可能造成整个系统的瘫痪。负压的作用在于提供一种从订阅者到生产者的反馈渠道。订阅者可以通过 request()方法来声明其一次所能处理的消息数量，而生产者就只会产生相应数量的消息，直到下一次 request()方法调用。这实际上变成了推拉结合的模式。

#### Jar包依赖

```
        <dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-core</artifactId>
            <version>3.0.5.RELEASE</version>
        </dependency>
```

#### Flux
Flux 表示的是包含 0 到 N 个元素的异步序列。在该序列中可以包含三种不同类型的消息通知：正常的包含元素的消息、序列结束的消息和序列出错的消息。当消息通知产生时，订阅者中对应的方法 onNext(), onComplete()和 onError()会被调用。

##### Flux 类的静态方法
- just()：可以指定序列中包含的全部元素。创建出来的 Flux 序列在发布这些元素之后会自动结束。
fromArray()，fromIterable()和 fromStream()：可以从一个数组、Iterable 对象或 Stream 对象中创建 -Flux 对象。
- empty()：创建一个不包含任何元素，只发布结束消息的序列。
- error(Throwable error)：创建一个只包含错误消息的序列。
- never()：创建一个不包含任何消息通知的序列。
- range(int start, int count)：创建包含从 start 起始的 count 个数量的 Integer 对象的序列。
- interval(Duration period)和 interval(Duration delay, Duration period)：创建一个包含了从 0 开始递增的 Long 对象的序列。其中包含的元素按照指定的间隔来发布。除了间隔时间之外，还可以指定起始元素发布之前的延迟时间。
- intervalMillis(long period)和 intervalMillis(long delay, long period)：与 interval()方法的作用相同，只不过该方法通过毫秒数来指定时间间隔和延迟时间。
- generate() 通过同步和逐一的方式来产生 Flux 序列，逐一生成的含义是在具体的生成逻辑中，next()方法只能最多被调用一次。序列的生成可能是有状态的，需要用到某些状态对象。此时可以使用 generate()方法的另外一种形式 `generate(Callable<S> stateSupplier, BiFunction<S,SynchronousSink<T>,S> generator)`，其中 stateSupplier 用来提供初始的状态对象。
- create(): create()方法与 generate()方法的不同之处在于所使用的是 FluxSink 对象。FluxSink 支持同步和异步的消息产生，并且可以在一次调用中产生多个元素。




#### Mono
Mono 表示的是包含 0 或者 1 个元素的异步序列。该序列中同样可以包含与 Flux 相同的三种类型的消息通知。在该序列中可以包含三种不同类型的消息通知：正常的包含元素的消息、序列结束的消息和序列出错的消息, Mono 类中也包含了一些与 Flux 类中相同的静态方法包括 just()，empty()，error()和 never()等。

#### 操作符
- buffer(int maxSize) 所包含的元素的最大数量
- bufferMillis(long duration) 收集的时间间隔
- bufferUntil(Predicate p) bufferUntil 会一直收集直到 Predicate 返回为 true
- bufferWhile(Predicate p) 只有当 Predicate 返回 true 时才会收集, 一旦值为 false，会立即开始下一次收集。
- filter(Predicate p) 对流中包含的元素进行过滤，只留下满足 Predicate 指定条件的元素
- window(int maxSize) 把当前流中的元素收集到另外的 Flux 序列中，因此返回值类型是 <pre> Flux<Flux<T>> </pre>
- zipWith(Flux flux) 把当前流中的元素与另外一个流中的元素按照一对一的方式进行合并
- take(long n)，take(Duration timespan)和 takeMillis(long timespan)：按照指定的数量或时间间隔来提取。
- takeLast(long n)：提取流中的最后 N 个元素。
- takeUntil(Predicate<? super T> predicate)：提取元素直到 Predicate 返回 true。
- takeWhile(Predicate<? super T> continuePredicate)： 当 Predicate 返回 true 时才进行提取。
- takeUntilOther(Publisher<?> other)：提取元素直到另外一个流开始产生元素。
- reduce(BiFunction f) 对流中包含的所有元素进行累积操作，得到一个包含计算结果的 Mono 序列
- reduceWith(A a, BiFunction f) 从初始值开始对流中包含的所有元素进行累积操作，得到一个包含计算结果的 Mono 序列
- merge(Flux ... fluxs) 把多个流合并成一个 Flux 序列, 按照元素产生的顺序来merge
- mergeSequential(Flux... fluxs) 把多个流合并成一个 Flux 序列, 按照订阅顺序来merge
- flatMap 把流中的每个元素转换成一个流，再把所有流中的元素进行合并, 按照流产生元素的顺序
- flatMapSequential 把流中的每个元素转换成一个流，再把所有流中的元素进行合并, 按照流被flat的顺序
- combineLatest 把所有流中的最新产生的元素合并成一个新的元素，作为返回结果流中的元素。

#### 消息处理
- onErrorReturn() 出现错误时返回默认值
- switchOnError() 出现错误时使用另外的流
- onErrorResumeWith(Exception e) 根据不同的异常类型来选择要使用的产生元素的流
- retry(int size) 使用 retry 操作符进行重试

#### 调度器
- 当前线程，通过 Schedulers.immediate()方法来创建。
- 单一的可复用的线程，通过 Schedulers.single()方法来创建。
- 使用弹性的线程池，通过 Schedulers.elastic()方法来创建。线程池中的线程是可以复用的。当所需要时，新的线程会被创建。如果一个线程闲置太长时间，则会被销毁。该调度器适用于 I/O 操作相关的流的处理。
- 使用对并行操作优化的线程池，通过 Schedulers.parallel()方法来创建。其中的线程数量取决于 CPU 的核的数量。该调度器适用于计算密集型的流的处理。
- 使用支持任务调度的调度器，通过 Schedulers.timer()方法来创建。
- 从已有的 ExecutorService 对象中创建调度器，通过 Schedulers.fromExecutorService()方法来创建。

通过 publishOn()和 subscribeOn()方法可以切换执行操作的调度器。其中 publishOn()方法切换的是操作符的执行方式，而 subscribeOn()方法切换的是产生流中元素时的执行方式。

#### 热序列
冷序列的含义是不论订阅者在何时订阅该序列，总是能收到序列中产生的全部消息。而与之对应的热序列，则是在持续不断地产生消息，订阅者只能获取到在其订阅之后产生的消息。

通过 publish()方法把一个 Flux 对象转换成 ConnectableFlux 对象。方法 autoConnect()的作用是当 ConnectableFlux 对象有一个订阅者时就开始产生消息。代码 source.subscribe()的作用是订阅该 ConnectableFlux 对象，让其开始产生数据。接着当前线程睡眠 5 秒钟，第二个订阅者此时只能获得到该序列中的后 5 个元素，因此所输出的是数字 5 到 9。

```
final Flux<Long> source = Flux.intervalMillis(1000)
        .take(10)
        .publish()
        .autoConnect();
source.subscribe();
Thread.sleep(5000);
source
        .toStream()
        .forEach(System.out::println);
```

