---
title: 响应式开发Webflux的基本使用一
date: 2021-10-09 01:02:31
tags: [SpringBoot]
---

# 响应式开发Webflux的基本使用
## 首先导入pom.xml
```
	<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-webflux</artifactId>
		</dependency>
```
## 理解概念
Mono(返回0或1个元素)
Flux(返回0-n个元素)

<!--more-->

### 返回的常用方法

#### Mono 常用的方法
```
public abstract class Mono<T> implements CorePublisher<T> {
    static final BiPredicate EQUALS_BIPREDICATE = Object::equals;
 
    public Mono() {
    }
 
    public static <T> Mono<T> create(Consumer<MonoSink<T>> callback) {  //创建Mono对象
 
    public static Mono<Long> delay(Duration duration) {
    public static Mono<Long> delay(Duration duration, Scheduler timer) {
 
    public static <T> Mono<T> empty() {  //内容为空，订阅时执行Operators.complete操作
        return MonoEmpty.instance();
    }
 
 
    public static <T> Mono<T> never() {  //内容为空，订阅时不执行任何操作
        return MonoNever.instance();
    }
 
 
    public static <T> Mono<T> error(Throwable error) {
    public static <T> Mono<T> error(Supplier<? extends Throwable> errorSupplier) {
 
    @SafeVarargs
    public static <T> Mono<T> first(Mono<? extends T>... monos) {
    public static <T> Mono<T> first(Iterable<? extends Mono<? extends T>> monos) {
 
 
    static <T> Mono<Void> empty(Publisher<T> source) {  //方法权限default，只能在同一个包中调用
        Mono<Void> then = ignoreElements(source);
        return then;
    }
 
    public static <T> Mono<T> ignoreElements(Publisher<T> source) {
    public static Mono<Context> subscriberContext() {
 
 
 
***************
just 操作：立刻创建Mono对象
 
    public static <T> Mono<T> just(T data) {                   //创建不为空的Mono对象
 
    public static <T> Mono<T> justOrEmpty(@Nullable Optional<? extends T> data) {
    public static <T> Mono<T> justOrEmpty(@Nullable T data) {  //创建包含0或者1个元素的对象
 
 
 
***************
defer 操作：延时创建Mono对象，当subscribe方法调用时，才会创建对象
 
    public static <T> Mono<T> defer(Supplier<? extends Mono<? extends T>> supplier) {
    public static <T> Mono<T> deferWithContext(Function<Context, ? extends Mono<? extends T>> supplier) {
 
 
 
***************
from 操作
 
    public static <T> Mono<T> from(Publisher<? extends T> source) {
    public static <I> Mono<I> fromDirect(Publisher<? extends I> source) {
 
    public static <T> Mono<T> fromSupplier(Supplier<? extends T> supplier) {
 
    public static <T> Mono<T> fromRunnable(Runnable runnable) {
    public static <T> Mono<T> fromCallable(Callable<? extends T> supplier) {
 
    public static <T> Mono<T> fromCompletionStage(CompletionStage<? extends T> completionStage) {
    public static <T> Mono<T> fromCompletionStage(Supplier<? extends CompletionStage<? extends T>> stageSupplier) {
 
    public static <T> Mono<T> fromFuture(CompletableFuture<? extends T> future) {
    public static <T> Mono<T> fromFuture(Supplier<? extends CompletableFuture<? extends T>> futureSupplier) {
 
 
 
***************
sequenceEqual 操作：比较对象流是否相等
 
    public static <T> Mono<Boolean> sequenceEqual(Publisher<? extends T> source1, Publisher<? extends T> source2) {
    public static <T> Mono<Boolean> sequenceEqual(Publisher<? extends T> source1, Publisher<? extends T> source2, BiPredicate<? super T, ? super T> isEqual) {
    public static <T> Mono<Boolean> sequenceEqual(Publisher<? extends T> source1, Publisher<? extends T> source2, BiPredicate<? super T, ? super T> isEqual, int prefetch) {
 
 
 
***************
using 操作
 
    public static <T, D> Mono<T> using(Callable<? extends D> resourceSupplier, Function<? super D, ? extends Mono<? extends T>> sourceSupplier, Consumer<? super D> resourceCleanup, boolean eager) {
                                 //callable返回初始的数据源，function对初始的数据源进行操作
                                 //consumer执行清理操作
                                 //eager为true（默认）：consumer在subscribe之前调用
                                 //eager为false：consumer在subscribe之后调用
 
    public static <T, D> Mono<T> using(Callable<? extends D> resourceSupplier, Function<? super D, ? extends Mono<? extends T>> sourceSupplier, Consumer<? super D> resourceCleanup) {
        return using(resourceSupplier, sourceSupplier, resourceCleanup, true);
    }                            //consumer操作默认在subscribe之前调用
 
    public static <T, D> Mono<T> usingWhen(Publisher<D> resourceSupplier, Function<? super D, ? extends Mono<? extends T>> resourceClosure, Function<? super D, ? extends Publisher<?>> asyncCleanup) {
    public static <T, D> Mono<T> usingWhen(Publisher<D> resourceSupplier, Function<? super D, ? extends Mono<? extends T>> resourceClosure, Function<? super D, ? extends Publisher<?>> asyncComplete, BiFunction<? super D, ? super Throwable, ? extends Publisher<?>> asyncError, Function<? super D, ? extends Publisher<?>> asyncCancel) {
 
 
 
***************
when 操作：当被调用的时候执行预设的操作
 
    public static Mono<Void> when(Publisher<?>... sources) {
    public static Mono<Void> when(Iterable<? extends Publisher<?>> sources) {
 
    public static Mono<Void> whenDelayError(Iterable<? extends Publisher<?>> sources) {
    public static Mono<Void> whenDelayError(Publisher<?>... sources) {
 
 
 
***************
zip 操作：压缩对象流
 
    public static <T1, T2> Mono<Tuple2<T1, T2>> zip(Mono<? extends T1> p1, Mono<? extends T2> p2) {
    public static <T1, T2, O> Mono<O> zip(Mono<? extends T1> p1, Mono<? extends T2> p2, BiFunction<? super T1, ? super T2, ? extends O> combinator) {
    public static <T1, T2, T3> Mono<Tuple3<T1, T2, T3>> zip(Mono<? extends T1> p1, Mono<? extends T2> p2, Mono<? extends T3> p3) {
    public static <T1, T2, T3, T4> Mono<Tuple4<T1, T2, T3, T4>> zip(Mono<? extends T1> p1, Mono<? extends T2> p2, Mono<? extends T3> p3, Mono<? extends T4> p4) {
    public static <T1, T2, T3, T4, T5> Mono<Tuple5<T1, T2, T3, T4, T5>> zip(Mono<? extends T1> p1, Mono<? extends T2> p2, Mono<? extends T3> p3, Mono<? extends T4> p4, Mono<? extends T5> p5) {
    public static <T1, T2, T3, T4, T5, T6> Mono<Tuple6<T1, T2, T3, T4, T5, T6>> zip(Mono<? extends T1> p1, Mono<? extends T2> p2, Mono<? extends T3> p3, Mono<? extends T4> p4, Mono<? extends T5> p5, Mono<? extends T6> p6) {
    public static <T1, T2, T3, T4, T5, T6, T7> Mono<Tuple7<T1, T2, T3, T4, T5, T6, T7>> zip(Mono<? extends T1> p1, Mono<? extends T2> p2, Mono<? extends T3> p3, Mono<? extends T4> p4, Mono<? extends T5> p5, Mono<? extends T6> p6, Mono<? extends T7> p7) {
    public static <T1, T2, T3, T4, T5, T6, T7, T8> Mono<Tuple8<T1, T2, T3, T4, T5, T6, T7, T8>> zip(Mono<? extends T1> p1, Mono<? extends T2> p2, Mono<? extends T3> p3, Mono<? extends T4> p4, Mono<? extends T5> p5, Mono<? extends T6> p6, Mono<? extends T7> p7, Mono<? extends T8> p8) {
 
    public static <R> Mono<R> zip(Iterable<? extends Mono<?>> monos, Function<? super Object[], ? extends R> combinator) {
    public static <R> Mono<R> zip(Function<? super Object[], ? extends R> combinator, Mono<?>... monos) {
 
 
    public static <T1, T2> Mono<Tuple2<T1, T2>> zipDelayError(Mono<? extends T1> p1, Mono<? extends T2> p2) {
    public static <T1, T2, T3> Mono<Tuple3<T1, T2, T3>> zipDelayError(Mono<? extends T1> p1, Mono<? extends T2> p2, Mono<? extends T3> p3) {
    public static <T1, T2, T3, T4> Mono<Tuple4<T1, T2, T3, T4>> zipDelayError(Mono<? extends T1> p1, Mono<? extends T2> p2, Mono<? extends T3> p3, Mono<? extends T4> p4) {
    public static <T1, T2, T3, T4, T5> Mono<Tuple5<T1, T2, T3, T4, T5>> zipDelayError(Mono<? extends T1> p1, Mono<? extends T2> p2, Mono<? extends T3> p3, Mono<? extends T4> p4, Mono<? extends T5> p5) {
    public static <T1, T2, T3, T4, T5, T6> Mono<Tuple6<T1, T2, T3, T4, T5, T6>> zipDelayError(Mono<? extends T1> p1, Mono<? extends T2> p2, Mono<? extends T3> p3, Mono<? extends T4> p4, Mono<? extends T5> p5, Mono<? extends T6> p6) {
    public static <T1, T2, T3, T4, T5, T6, T7> Mono<Tuple7<T1, T2, T3, T4, T5, T6, T7>> zipDelayError(Mono<? extends T1> p1, Mono<? extends T2> p2, Mono<? extends T3> p3, Mono<? extends T4> p4, Mono<? extends T5> p5, Mono<? extends T6> p6, Mono<? extends T7> p7) {
    public static <T1, T2, T3, T4, T5, T6, T7, T8> Mono<Tuple8<T1, T2, T3, T4, T5, T6, T7, T8>> zipDelayError(Mono<? extends T1> p1, Mono<? extends T2> p2, Mono<? extends T3> p3, Mono<? extends T4> p4, Mono<? extends T5> p5, Mono<? extends T6> p6, Mono<? extends T7> p7, Mono<? extends T8> p8) {
 
    public static <R> Mono<R> zipDelayError(Iterable<? extends Mono<?>> monos, Function<? super Object[], ? extends R> combinator) {
    public static <R> Mono<R> zipDelayError(Function<? super Object[], ? extends R> combinator, Mono<?>... monos) {
 
 
 
***************
flatMap 操作
 
    public final <R> Mono<R> flatMap(Function<? super T, ? extends Mono<? extends R>> transformer) {
 
    public final <R> Flux<R> flatMapMany(Function<? super T, ? extends Publisher<? extends R>> mapper) {
    public final <R> Flux<R> flatMapMany(Function<? super T, ? extends Publisher<? extends R>> mapperOnNext, Function<? super Throwable, ? extends Publisher<? extends R>> mapperOnError, Supplier<? extends Publisher<? extends R>> mapperOnComplete) {
 
    public final <R> Flux<R> flatMapIterable(Function<? super T, ? extends Iterable<? extends R>> mapper) {
 
 
***************
map 操作
 
    public final <R> Mono<R> map(Function<? super T, ? extends R> mapper) {
 
 
 
***************
transform 操作
 
    public final <V> Mono<V> transform(Function<? super Mono<T>, ? extends Publisher<V>> transformer) {
 
    public final <V> Mono<V> transformDeferred(Function<? super Mono<T>, ? extends Publisher<V>> transformer) {
 
 
 
***************
filter 操作
 
    public final Mono<T> filter(Predicate<? super T> tester) {
    public final Mono<T> filterWhen(Function<? super T, ? extends Publisher<Boolean>> asyncPredicate) {
 
 
 
***************
do 操作
 
    public final Mono<T> doFirst(Runnable onFirst) {
    public final Mono<T> doFinally(Consumer<SignalType> onFinally) {
 
    public final Mono<T> doOnCancel(Runnable onCancel) {
    public final <R> Mono<T> doOnDiscard(Class<R> type, Consumer<? super R> discardHook) {
    public final Mono<T> doOnNext(Consumer<? super T> onNext) {
    public final Mono<T> doOnSuccess(Consumer<? super T> onSuccess) {
    public final Mono<T> doOnEach(Consumer<? super Signal<T>> signalConsumer) {
 
    public final Mono<T> doOnError(Consumer<? super Throwable> onError) {
    public final <E extends Throwable> Mono<T> doOnError(Class<E> exceptionType, Consumer<? super E> onError) {
    public final Mono<T> doOnError(Predicate<? super Throwable> predicate, Consumer<? super Throwable> onError) {
 
    public final Mono<T> doOnRequest(LongConsumer consumer) {
    public final Mono<T> doOnSubscribe(Consumer<? super Subscription> onSubscribe) {
 
    public final Mono<T> doOnTerminate(Runnable onTerminate) {
    public final Mono<T> doAfterTerminate(Runnable afterTerminate) {
 
 
 
***************
block 操作
 
    @Nullable
    public T block() {
 
    @Nullable
    public T block(Duration timeout) {
 
    public Optional<T> blockOptional() {
    public Optional<T> blockOptional(Duration timeout) {
 
 
 
***************
cache 操作
 
    public final Mono<T> cache() {
    public final Mono<T> cache(Duration ttl) {
    public final Mono<T> cache(Duration ttl, Scheduler timer) {
    public final Mono<T> cache(Function<? super T, Duration> ttlForValue, Function<Throwable, Duration> ttlForError, Supplier<Duration> ttlForEmpty) {
 
 
 
***************
checkpoint 操作
 
    public final Mono<T> checkpoint() {
    public final Mono<T> checkpoint(String description) {
    public final Mono<T> checkpoint(@Nullable String description, boolean forceStackTrace) {
 
 
***************
delay 操作
 
    public final Mono<T> delayElement(Duration delay) {
    public final Mono<T> delayElement(Duration delay, Scheduler timer) {
 
    public final Mono<T> delayUntil(Function<? super T, ? extends Publisher<?>> triggerProvider) {
 
    public final Mono<T> delaySubscription(Duration delay) {
    public final Mono<T> delaySubscription(Duration delay, Scheduler timer) {
    public final <U> Mono<T> delaySubscription(Publisher<U> subscriptionDelay) {
 
 
***************
elapsed 操作
 
    public final Mono<Tuple2<Long, T>> elapsed() {
    public final Mono<Tuple2<Long, T>> elapsed(Scheduler scheduler) {
 
 
 
***************
expand 操作
 
    public final Flux<T> expandDeep(Function<? super T, ? extends Publisher<? extends T>> expander, int capacityHint) {
    public final Flux<T> expandDeep(Function<? super T, ? extends Publisher<? extends T>> expander) {
    public final Flux<T> expand(Function<? super T, ? extends Publisher<? extends T>> expander, int capacityHint) {
    public final Flux<T> expand(Function<? super T, ? extends Publisher<? extends T>> expander) {
 
 
 
***************
log 操作
 
    public final Mono<T> log() {
        return this.log((String)null, Level.INFO);
    }
 
    public final Mono<T> log(@Nullable String category) {
        return this.log(category, Level.INFO);
    }
 
    public final Mono<T> log(@Nullable String category, Level level, SignalType... options) {
        return this.log(category, level, false, options);
    }
 
    public final Mono<T> log(@Nullable String category, Level level, boolean showOperatorLine, SignalType... options) {
        SignalLogger<T> log = new SignalLogger(this, category, level, showOperatorLine, options);
        return this instanceof Fuseable ? onAssembly(new MonoLogFuseable(this, log)) : onAssembly(new MonoLog(this, log));
    }
 
    public final Mono<T> log(Logger logger) {
        return this.log(logger, Level.INFO, false);
    }
 
    public final Mono<T> log(Logger logger, Level level, boolean showOperatorLine, SignalType... options) {
 
 
 
***************
onError 操作
 
    public final Mono<T> onErrorContinue(BiConsumer<Throwable, Object> errorConsumer) {
    public final <E extends Throwable> Mono<T> onErrorContinue(Class<E> type, BiConsumer<Throwable, Object> errorConsumer) {
    public final <E extends Throwable> Mono<T> onErrorContinue(Predicate<E> errorPredicate, BiConsumer<Throwable, Object> errorConsumer) {
 
 
    public final Mono<T> onErrorStop() {
 
    public final Mono<T> onErrorMap(Predicate<? super Throwable> predicate, Function<? super Throwable, ? extends Throwable> mapper) {
    public final Mono<T> onErrorMap(Function<? super Throwable, ? extends Throwable> mapper) {
    public final <E extends Throwable> Mono<T> onErrorMap(Class<E> type, Function<? super E, ? extends Throwable> mapper) {
 
 
    public final Mono<T> onErrorResume(Function<? super Throwable, ? extends Mono<? extends T>> fallback) {
    public final <E extends Throwable> Mono<T> onErrorResume(Class<E> type, Function<? super E, ? extends Mono<? extends T>> fallback) {
    public final Mono<T> onErrorResume(Predicate<? super Throwable> predicate, Function<? super Throwable, ? extends Mono<? extends T>> fallback) {
 
    public final Mono<T> onErrorReturn(T fallback) {
    public final <E extends Throwable> Mono<T> onErrorReturn(Class<E> type, T fallbackValue) {
    public final Mono<T> onErrorReturn(Predicate<? super Throwable> predicate, T fallbackValue) {
 
 
 
***************
publish 操作
 
    public final <R> Mono<R> publish(Function<? super Mono<T>, ? extends Mono<? extends R>> transform) {
 
    public final Mono<T> publishOn(Scheduler scheduler) {
 
 
 
***************
repeat 操作
 
    public final Flux<T> repeat() {
    public final Flux<T> repeat(BooleanSupplier predicate) {
 
    public final Flux<T> repeat(long numRepeat) {
    public final Flux<T> repeat(long numRepeat, BooleanSupplier predicate) {
 
    public final Flux<T> repeatWhen(Function<Flux<Long>, ? extends Publisher<?>> repeatFactory) {
 
    public final Mono<T> repeatWhenEmpty(Function<Flux<Long>, ? extends Publisher<?>> repeatFactory) {
    public final Mono<T> repeatWhenEmpty(int maxRepeat, Function<Flux<Long>, ? extends Publisher<?>> repeatFactory) {
 
 
***************
retry 操作
 
    public final Mono<T> retry() {
        return this.retry(9223372036854775807L);
    }
 
    public final Mono<T> retry(long numRetries) {
        return onAssembly(new MonoRetry(this, numRetries));
    }
 
    public final Mono<T> retryWhen(Retry retrySpec) {
        return onAssembly(new MonoRetryWhen(this, retrySpec));
    }
 
 
 
***************
take 操作
 
    public final Mono<T> take(Duration duration) {
    public final Mono<T> take(Duration duration, Scheduler timer) {
    public final Mono<T> takeUntilOther(Publisher<?> other) {
 
 
***************
then 操作
 
    public final Mono<Void> then() {
 
    public final <V> Mono<V> then(Mono<V> other) {
    public final <V> Mono<V> thenReturn(V value) {
 
    public final Mono<Void> thenEmpty(Publisher<Void> other) {
 
    public final <V> Flux<V> thenMany(Publisher<V> other) {
 
 
 
***************
timeout 操作
 
    public final Mono<T> timeout(Duration timeout) {
    public final Mono<T> timeout(Duration timeout, Mono<? extends T> fallback) {
    public final Mono<T> timeout(Duration timeout, Scheduler timer) {
    public final Mono<T> timeout(Duration timeout, @Nullable Mono<? extends T> fallback, Scheduler timer) {
 
    public final <U> Mono<T> timeout(Publisher<U> firstTimeout) {
    public final <U> Mono<T> timeout(Publisher<U> firstTimeout, Mono<? extends T> fallback) {
 
 
 
***************
timestamp 操作
 
    public final Mono<Tuple2<Long, T>> timestamp() {
    public final Mono<Tuple2<Long, T>> timestamp(Scheduler scheduler) {
 
 
 
***************
zipwhen、zipwith 操作
 
    public final <T2> Mono<Tuple2<T, T2>> zipWhen(Function<T, Mono<? extends T2>> rightGenerator) {
    public final <T2, O> Mono<O> zipWhen(Function<T, Mono<? extends T2>> rightGenerator, BiFunction<T, T2, O> combinator) {
 
    public final <T2> Mono<Tuple2<T, T2>> zipWith(Mono<? extends T2> other) {
    public final <T2, O> Mono<O> zipWith(Mono<? extends T2> other, BiFunction<? super T, ? super T2, ? extends O> combinator) {
 
 
***************
subscribe 操作
 
    public final Disposable subscribe() {
 
    public final Disposable subscribe(Consumer<? super T> consumer) {
    public final Disposable subscribe(@Nullable Consumer<? super T> consumer, Consumer<? super Throwable> errorConsumer) {
    public final Disposable subscribe(@Nullable Consumer<? super T> consumer, @Nullable Consumer<? super Throwable> errorConsumer, @Nullable Runnable completeConsumer) {
    public final Disposable subscribe(@Nullable Consumer<? super T> consumer, @Nullable Consumer<? super Throwable> errorConsumer, @Nullable Runnable completeConsumer, @Nullable Consumer<? super Subscription> subscriptionConsumer) {
    public final Disposable subscribe(@Nullable Consumer<? super T> consumer, @Nullable Consumer<? super Throwable> errorConsumer, @Nullable Runnable completeConsumer, @Nullable Context initialContext) {
 
    public final void subscribe(Subscriber<? super T> actual) {
    public abstract void subscribe(CoreSubscriber<? super T> var1);
 
    public final Mono<T> subscriberContext(Context mergeContext) {
    public final Mono<T> subscriberContext(Function<Context, Context> doOnContext) {
 
    public final Mono<T> subscribeOn(Scheduler scheduler) {
    public final <E extends Subscriber<? super T>> E subscribeWith(E subscriber) {
 
 
 
***************
其他操作
 
    public final Mono<T> ignoreElement() {                //忽略Mono内容
    public final Mono<Boolean> hasElement() {             //判断是否含有元素
 
    public final Mono<T> defaultIfEmpty(T defaultV) {     //如果当前对象为空，设置默认的zhidefaultValue
    public final Mono<T> switchIfEmpty(Mono<? extends T> alternate) {  //当前对象为空，返回新的Mono对象 alternate
 
 
    public final Flux<T> flux() {                   //将Mono对象转换为 flux对象
    public final CompletableFuture<T> toFuture() {  //转换为CompleteableFuture对象
    public final MonoProcessor<T> toProcessor() {   //转换为MonoProcessor
 
 
    public final Flux<T> concatWith(Publisher<? extends T> other) {    //拼接，返回Flux对象
    public final Flux<T> mergeWith(Publisher<? extends T> other) {     //合并，返回Flux对象
 
 
    public final <E> Mono<E> cast(Class<E> clazz) {  //将存储元素转换为指定的类型E
    public final <U> Mono<U> ofType(Class<U> clazz) {
        Objects.requireNonNull(clazz, "clazz");
        return this.filter((o) -> {
            return clazz.isAssignableFrom(o.getClass());
        }).cast(clazz);
    }
 
    public final Mono<Void> and(Publisher<?> other) {
        if (this instanceof MonoWhen) {
            MonoWhen o = (MonoWhen)this;
            Mono<Void> result = o.whenAdditionalSource(other);
            if (result != null) {
                return result;
            }
        }
 
        return when(this, other);
    }
 
    public final Mono<T> or(Mono<? extends T> other) {
        if (this instanceof MonoFirst) {
            MonoFirst<T> a = (MonoFirst)this;
            Mono<T> result = a.orAdditionalSource(other);
            if (result != null) {
                return result;
            }
        }
 
        return first(this, other);
    }
 
 
    public final Mono<T> cancelOn(Scheduler scheduler) {
 
    public final Mono<T> name(String name) {                 //设置名称
    public final Mono<T> tag(String key, String value) {     //设置标签
 
 
    public final Mono<Signal<T>> materialize() {
    public final <X> Mono<X> dematerialize() {
 
    public final Mono<T> hide() {
    public final Mono<T> single() {
    public final Mono<T> metrics() {
    public final Mono<T> onTerminateDetach() {
 
    public final <P> P as(Function<? super Mono<T>, P> transformer) {
    public final <R> Mono<R> handle(BiConsumer<? super T, SynchronousSink<R>> handler) {
 
 
    protected static <T> Mono<T> onAssembly(Mono<T> source) {
        Function<Publisher, Publisher> hook = Hooks.onEachOperatorHook;
        if (hook != null) {
            source = (Mono)hook.apply(source);
        }
 
        if (Hooks.GLOBAL_TRACE) {
            AssemblySnapshot stacktrace = new AssemblySnapshot((String)null, (Supplier)Traces.callSiteSupplierFactory.get());
            source = (Mono)Hooks.addAssemblyInfo(source, stacktrace);
        }
 
        return source;
    }
 
 
    static <T> Mono<T> doOnSignal(Mono<T> source, @Nullable Consumer<? super Subscription> onSubscribe, @Nullable Consumer<? super T> onNext, @Nullable LongConsumer onRequest, @Nullable Runnable onCancel) {
    static <T> Mono<T> doOnTerminalSignal(Mono<T> source, @Nullable Consumer<? super T> onSuccess, @Nullable Consumer<? super Throwable> onError, @Nullable BiConsumer<? super T, Throwable> onAfterTerminate) {
 
 
    static <T> Mono<T> wrap(Publisher<T> source, boolean enforceMonoContract) {
    static <T> BiPredicate<? super T, ? super T> equalsBiPredicate() {
    public String toString() {

```

#### Flux 常用的方法

```
public abstract class Flux<T> implements CorePublisher<T> {
    static final BiFunction TUPLE2_BIFUNCTION = Tuples::of;
    static final Supplier LIST_SUPPLIER = ArrayList::new;
    static final Supplier SET_SUPPLIER = HashSet::new;
    static final BooleanSupplier ALWAYS_BOOLEAN_SUPPLIER = () -> {
        return true;
    };
    static final BiPredicate OBJECT_EQUAL = Object::equals;
    static final Function IDENTITY_FUNCTION = Function.identity();
 
    public Flux() {
    }
 
 
    public static <T> Flux<T> empty() {
    public static <T> Flux<T> never() {
    public static Flux<Integer> range(int start, int count) {
 
 
 
*****************
just 操作
 
    public static <T> Flux<T> just(T... data) {
    public static <T> Flux<T> just(T data) {
 
 
 
*****************
defer 操作
 
    public static <T> Flux<T> defer(Supplier<? extends Publisher<T>> supplier) {
    public static <T> Flux<T> deferWithContext(Function<Context, ? extends Publisher<T>> supplier) {
 
 
 
*****************
from 操作
 
    public static <T> Flux<T> from(Publisher<? extends T> source) {
 
    public static <T> Flux<T> fromArray(T[] array) {
    public static <T> Flux<T> fromIterable(Iterable<? extends T> it) {
 
    public static <T> Flux<T> fromStream(Stream<? extends T> s) {
    public static <T> Flux<T> fromStream(Supplier<Stream<? extends T>> streamSupplier) {
 
 
*****************
merge 操作
 
    public static <I> Flux<I> merge(Iterable<? extends Publisher<? extends I>> sources) {
 
    public static <T> Flux<T> merge(Publisher<? extends Publisher<? extends T>> source) {
    public static <T> Flux<T> merge(Publisher<? extends Publisher<? extends T>> source, int concurrency) {
    public static <T> Flux<T> merge(Publisher<? extends Publisher<? extends T>> source, int concurrency, int prefetch) {
    public static <I> Flux<I> merge(Publisher<? extends I>... sources) {
    public static <I> Flux<I> merge(int prefetch, Publisher<? extends I>... sources) {
 
    public static <I> Flux<I> mergeDelayError(int prefetch, Publisher<? extends I>... sources) {
 
    public static <I extends Comparable<? super I>> Flux<I> mergeOrdered(Publisher<? extends I>... sources) {
    public static <T> Flux<T> mergeOrdered(Comparator<? super T> comparator, Publisher<? extends T>... sources) {
    public static <T> Flux<T> mergeOrdered(int prefetch, Comparator<? super T> comparator, Publisher<? extends T>... sources) {
 
    public static <T> Flux<T> mergeSequential(Publisher<? extends Publisher<? extends T>> sources) {
    public static <T> Flux<T> mergeSequential(Publisher<? extends Publisher<? extends T>> sources, int maxConcurrency, int prefetch) {
    public static <I> Flux<I> mergeSequential(Publisher<? extends I>... sources) {
    public static <I> Flux<I> mergeSequential(int prefetch, Publisher<? extends I>... sources) {
    public static <I> Flux<I> mergeSequential(Iterable<? extends Publisher<? extends I>> sources) {
    public static <I> Flux<I> mergeSequential(Iterable<? extends Publisher<? extends I>> sources, int maxConcurrency, int prefetch) {
 
    public static <T> Flux<T> mergeSequentialDelayError(Publisher<? extends Publisher<? extends T>> sources, int maxConcurrency, int prefetch) {
    public static <I> Flux<I> mergeSequentialDelayError(int prefetch, Publisher<? extends I>... sources) {
    public static <I> Flux<I> mergeSequentialDelayError(Iterable<? extends Publisher<? extends I>> sources, int maxConcurrency, int prefetch) {
 
 
    static <I> Flux<I> merge(int prefetch, boolean delayError, Publisher<? extends I>... sources) {
    static <I> Flux<I> mergeSequential(int prefetch, boolean delayError, Publisher<? extends I>... sources) {
    static <T> Flux<T> mergeSequential(Publisher<? extends Publisher<? extends T>> sources, boolean delayError, int maxConcurrency, int prefetch) {
    static <I> Flux<I> mergeSequential(Iterable<? extends Publisher<? extends I>> sources, boolean delayError, int maxConcurrency, int prefetch) {
                                      //方法权限为default，只能在同一个包中使用
 
 
 
*****************
zip 操作
 
    public static <T1, T2, O> Flux<O> zip(Publisher<? extends T1> source1, Publisher<? extends T2> source2, BiFunction<? super T1, ? super T2, ? extends O> combinator) {
    public static <T1, T2> Flux<Tuple2<T1, T2>> zip(Publisher<? extends T1> source1, Publisher<? extends T2> source2) {
    public static <T1, T2, T3> Flux<Tuple3<T1, T2, T3>> zip(Publisher<? extends T1> source1, Publisher<? extends T2> source2, Publisher<? extends T3> source3) {
    public static <T1, T2, T3, T4> Flux<Tuple4<T1, T2, T3, T4>> zip(Publisher<? extends T1> source1, Publisher<? extends T2> source2, Publisher<? extends T3> source3, Publisher<? extends T4> source4) {
    public static <T1, T2, T3, T4, T5> Flux<Tuple5<T1, T2, T3, T4, T5>> zip(Publisher<? extends T1> source1, Publisher<? extends T2> source2, Publisher<? extends T3> source3, Publisher<? extends T4> source4, Publisher<? extends T5> source5) {
    public static <T1, T2, T3, T4, T5, T6> Flux<Tuple6<T1, T2, T3, T4, T5, T6>> zip(Publisher<? extends T1> source1, Publisher<? extends T2> source2, Publisher<? extends T3> source3, Publisher<? extends T4> source4, Publisher<? extends T5> source5, Publisher<? extends T6> source6) {
    public static <T1, T2, T3, T4, T5, T6, T7> Flux<Tuple7<T1, T2, T3, T4, T5, T6, T7>> zip(Publisher<? extends T1> source1, Publisher<? extends T2> source2, Publisher<? extends T3> source3, Publisher<? extends T4> source4, Publisher<? extends T5> source5, Publisher<? extends T6> source6, Publisher<? extends T7> source7) {
    public static <T1, T2, T3, T4, T5, T6, T7, T8> Flux<Tuple8<T1, T2, T3, T4, T5, T6, T7, T8>> zip(Publisher<? extends T1> source1, Publisher<? extends T2> source2, Publisher<? extends T3> source3, Publisher<? extends T4> source4, Publisher<? extends T5> source5, Publisher<? extends T6> source6, Publisher<? extends T7> source7, Publisher<? extends T8> source8) {
 
    public static <O> Flux<O> zip(Iterable<? extends Publisher<?>> sources, Function<? super Object[], ? extends O> combinator) {
    public static <O> Flux<O> zip(Iterable<? extends Publisher<?>> sources, int prefetch, Function<? super Object[], ? extends O> combinator) {
 
    public static <I, O> Flux<O> zip(Function<? super Object[], ? extends O> combinator, Publisher<? extends I>... sources) {
    public static <I, O> Flux<O> zip(Function<? super Object[], ? extends O> combinator, int prefetch, Publisher<? extends I>... sources) {
 
    public static <TUPLE extends Tuple2, V> Flux<V> zip(Publisher<? extends Publisher<?>> sources, final Function<? super TUPLE, ? extends V> combinator) {
 
 
 
*****************
combineLatest 操作
 
    public static <T, V> Flux<V> combineLatest(Function<Object[], V> combinator, Publisher<? extends T>... sources) {
    public static <T, V> Flux<V> combineLatest(Function<Object[], V> combinator, int prefetch, Publisher<? extends T>... sources) {
 
    public static <T1, T2, V> Flux<V> combineLatest(Publisher<? extends T1> source1, Publisher<? extends T2> source2, BiFunction<? super T1, ? super T2, ? extends V> combinator) {
    public static <T1, T2, T3, V> Flux<V> combineLatest(Publisher<? extends T1> source1, Publisher<? extends T2> source2, Publisher<? extends T3> source3, Function<Object[], V> combinator) {
    public static <T1, T2, T3, T4, V> Flux<V> combineLatest(Publisher<? extends T1> source1, Publisher<? extends T2> source2, Publisher<? extends T3> source3, Publisher<? extends T4> source4, Function<Object[], V> combinator) {
    public static <T1, T2, T3, T4, T5, V> Flux<V> combineLatest(Publisher<? extends T1> source1, Publisher<? extends T2> source2, Publisher<? extends T3> source3, Publisher<? extends T4> source4, Publisher<? extends T5> source5, Function<Object[], V> combinator) {
    public static <T1, T2, T3, T4, T5, T6, V> Flux<V> combineLatest(Publisher<? extends T1> source1, Publisher<? extends T2> source2, Publisher<? extends T3> source3, Publisher<? extends T4> source4, Publisher<? extends T5> source5, Publisher<? extends T6> source6, Function<Object[], V> combinator) {
 
    public static <T, V> Flux<V> combineLatest(Iterable<? extends Publisher<? extends T>> sources, Function<Object[], V> combinator) {
    public static <T, V> Flux<V> combineLatest(Iterable<? extends Publisher<? extends T>> sources, int prefetch, Function<Object[], V> combinator) {
 
 
 
*****************
concat 操作
 
    public static <T> Flux<T> concat(Iterable<? extends Publisher<? extends T>> sources) {
    public static <T> Flux<T> concat(Publisher<? extends Publisher<? extends T>> sources) {
    public static <T> Flux<T> concat(Publisher<? extends Publisher<? extends T>> sources, int prefetch) {
    public static <T> Flux<T> concat(Publisher<? extends T>... sources) {
 
    public static <T> Flux<T> concatDelayError(Publisher<? extends Publisher<? extends T>> sources) {
    public static <T> Flux<T> concatDelayError(Publisher<? extends Publisher<? extends T>> sources, int prefetch) {
    public static <T> Flux<T> concatDelayError(Publisher<? extends Publisher<? extends T>> sources, boolean delayUntilEnd, int prefetch) {
    public static <T> Flux<T> concatDelayError(Publisher<? extends T>... sources) {
 
 
    public final Flux<T> concatWithValues(T... values) {
    public final Flux<T> concatWith(Publisher<? extends T> other) {
 
    public final <V> Flux<V> concatMap(Function<? super T, ? extends Publisher<? extends V>> mapper) {
    public final <V> Flux<V> concatMap(Function<? super T, ? extends Publisher<? extends V>> mapper, int prefetch) {
 
    public final <V> Flux<V> concatMapDelayError(Function<? super T, ? extends Publisher<? extends V>> mapper) {
    public final <V> Flux<V> concatMapDelayError(Function<? super T, ? extends Publisher<? extends V>> mapper, int prefetch) {
    public final <V> Flux<V> concatMapDelayError(Function<? super T, ? extends Publisher<? extends V>> mapper, boolean delayUntilEnd, int prefetch) {
 
    public final <R> Flux<R> concatMapIterable(Function<? super T, ? extends Iterable<? extends R>> mapper) {
    public final <R> Flux<R> concatMapIterable(Function<? super T, ? extends Iterable<? extends R>> mapper, int prefetch) {
 
 
 
 
*****************
create 操作
 
    public static <T> Flux<T> create(Consumer<? super FluxSink<T>> emitter) {
    public static <T> Flux<T> create(Consumer<? super FluxSink<T>> emitter, OverflowStrategy backpressure) {
 
 
 
*****************
push 操作
 
    public static <T> Flux<T> push(Consumer<? super FluxSink<T>> emitter) {
    public static <T> Flux<T> push(Consumer<? super FluxSink<T>> emitter, OverflowStrategy backpressure) {
 
 
 
*****************
error 操作
 
 
    public static <T> Flux<T> error(Throwable error) {
    public static <T> Flux<T> error(Supplier<? extends Throwable> errorSupplier) {
    public static <O> Flux<O> error(Throwable throwable, boolean whenRequested) {
 
 
 
*****************
first 操作
 
    public static <I> Flux<I> first(Publisher<? extends I>... sources) {
    public static <I> Flux<I> first(Iterable<? extends Publisher<? extends I>> sources) {
 
 
 
*****************
generate 操作
 
    public static <T> Flux<T> generate(Consumer<SynchronousSink<T>> generator) {
    public static <T, S> Flux<T> generate(Callable<S> stateSupplier, BiFunction<S, SynchronousSink<T>, S> generator) {
    public static <T, S> Flux<T> generate(Callable<S> stateSupplier, BiFunction<S, SynchronousSink<T>, S> generator, Consumer<? super S> stateConsumer) {
 
 
 
*****************
interval 操作
 
    public static Flux<Long> interval(Duration period) {
    public static Flux<Long> interval(Duration delay, Duration period) {
    public static Flux<Long> interval(Duration period, Scheduler timer) {
    public static Flux<Long> interval(Duration delay, Duration period, Scheduler timer) {
 
 
 
*****************
using 操作
 
    public static <T, D> Flux<T> using(Callable<? extends D> resourceSupplier, Function<? super D, ? extends Publisher<? extends T>> sourceSupplier, Consumer<? super D> resourceCleanup) {
    public static <T, D> Flux<T> using(Callable<? extends D> resourceSupplier, Function<? super D, ? extends Publisher<? extends T>> sourceSupplier, Consumer<? super D> resourceCleanup, boolean eager) {
 
    public static <T, D> Flux<T> usingWhen(Publisher<D> resourceSupplier, Function<? super D, ? extends Publisher<? extends T>> resourceClosure, Function<? super D, ? extends Publisher<?>> asyncCleanup) {
    public static <T, D> Flux<T> usingWhen(Publisher<D> resourceSupplier, Function<? super D, ? extends Publisher<? extends T>> resourceClosure, Function<? super D, ? extends Publisher<?>> asyncComplete, BiFunction<? super D, ? super Throwable, ? extends Publisher<?>> asyncError, Function<? super D, ? extends Publisher<?>> asyncCancel) {
 
 
 
*****************
switchOnNext 操作
 
    public static <T> Flux<T> switchOnNext(Publisher<? extends Publisher<? extends T>> mergedPublishers) {
    public static <T> Flux<T> switchOnNext(Publisher<? extends Publisher<? extends T>> mergedPublishers, int prefetch) {
 
 
 
*****************
flatMap操作
 
    public final <R> Flux<R> flatMap(Function<? super T, ? extends Publisher<? extends R>> mapper) {
    public final <V> Flux<V> flatMap(Function<? super T, ? extends Publisher<? extends V>> mapper, int concurrency) {
    public final <V> Flux<V> flatMap(Function<? super T, ? extends Publisher<? extends V>> mapper, int concurrency, int prefetch) {
    public final <R> Flux<R> flatMap(@Nullable Function<? super T, ? extends Publisher<? extends R>> mapperOnNext, @Nullable Function<? super Throwable, ? extends Publisher<? extends R>> mapperOnError, @Nullable Supplier<? extends Publisher<? extends R>> mapperOnComplete) {
 
    public final <V> Flux<V> flatMapDelayError(Function<? super T, ? extends Publisher<? extends V>> mapper, int concurrency, int prefetch) {
 
    public final <R> Flux<R> flatMapIterable(Function<? super T, ? extends Iterable<? extends R>> mapper) {
    public final <R> Flux<R> flatMapIterable(Function<? super T, ? extends Iterable<? extends R>> mapper, int prefetch) {
 
    public final <R> Flux<R> flatMapSequential(Function<? super T, ? extends Publisher<? extends R>> mapper) {
    public final <R> Flux<R> flatMapSequential(Function<? super T, ? extends Publisher<? extends R>> mapper, int maxConcurrency) {
    public final <R> Flux<R> flatMapSequential(Function<? super T, ? extends Publisher<? extends R>> mapper, int maxConcurrency, int prefetch) {
 
    public final <R> Flux<R> flatMapSequentialDelayError(Function<? super T, ? extends Publisher<? extends R>> mapper, int maxConcurrency, int prefetch) {
 
    final <V> Flux<V> flatMap(Function<? super T, ? extends Publisher<? extends V>> mapper, boolean delayError, int concurrency, int prefetch) {
    final <R> Flux<R> flatMapSequential(Function<? super T, ? extends Publisher<? extends R>> mapper, boolean delayError, int maxConcurrency, int prefetch) {
                                       //方法权限为default，只能在同一个包中调用
 
 
*****************
map 操作
 
    public final <V> Flux<V> map(Function<? super T, ? extends V> mapper) {
 
 
 
*****************
filter 操作
 
    public final Flux<T> filter(Predicate<? super T> p) {
 
    public final Flux<T> filterWhen(Function<? super T, ? extends Publisher<Boolean>> asyncPredicate) {
    public final Flux<T> filterWhen(Function<? super T, ? extends Publisher<Boolean>> asyncPredicate, int bufferSize) {
 
 
 
*****************
transform 操作
 
    public final <V> Flux<V> transform(Function<? super Flux<T>, ? extends Publisher<V>> transformer) {
    public final <V> Flux<V> transformDeferred(Function<? super Flux<T>, ? extends Publisher<V>> transformer) {
 
 
 
*****************
switch 操作
 
    public final Flux<T> switchIfEmpty(Publisher<? extends T> alternate) {
 
    public final <V> Flux<V> switchMap(Function<? super T, Publisher<? extends V>> fn) {
    public final <V> Flux<V> switchMap(Function<? super T, Publisher<? extends V>> fn, int prefetch) {
 
    public final <V> Flux<V> switchOnFirst(BiFunction<Signal<? extends T>, Flux<T>, Publisher<? extends V>> transformer) {
    public final <V> Flux<V> switchOnFirst(BiFunction<Signal<? extends T>, Flux<T>, Publisher<? extends V>> transformer, boolean cancelSourceOnComplete) {
 
 
 
*****************
toIterable 操作
 
    public final Iterable<T> toIterable() {
    public final Iterable<T> toIterable(int batchSize) {
    public final Iterable<T> toIterable(int batchSize, @Nullable Supplier<Queue<T>> queueProvider) {
 
 
*****************
toStream 操作
 
    public final Stream<T> toStream() {
    public final Stream<T> toStream(int batchSize) {
 
 
 
*****************
single 操作
 
    public final Mono<T> single() {
    public final Mono<T> single(T defaultValue) {
 
    public final Mono<T> singleOrEmpty() {
 
 
 
*****************
collect 操作
 
    public final <E> Mono<E> collect(Supplier<E> containerSupplier, BiConsumer<E, ? super T> collector) {
    public final <R, A> Mono<R> collect(Collector<? super T, A, ? extends R> collector) {
 
    public final Mono<List<T>> collectList() {
    public final Mono<List<T>> collectSortedList() {
    public final Mono<List<T>> collectSortedList(@Nullable Comparator<? super T> comparator) {
 
    public final <K> Mono<Map<K, T>> collectMap(Function<? super T, ? extends K> keyExtractor) {
    public final <K, V> Mono<Map<K, V>> collectMap(Function<? super T, ? extends K> keyExtractor, Function<? super T, ? extends V> valueExtractor) {
    public final <K, V> Mono<Map<K, V>> collectMap(Function<? super T, ? extends K> keyExtractor, Function<? super T, ? extends V> valueExtractor, Supplier<Map<K, V>> mapSupplier) {
 
    public final <K> Mono<Map<K, Collection<T>>> collectMultimap(Function<? super T, ? extends K> keyExtractor) {
    public final <K, V> Mono<Map<K, Collection<V>>> collectMultimap(Function<? super T, ? extends K> keyExtractor, Function<? super T, ? extends V> valueExtractor) {
    public final <K, V> Mono<Map<K, Collection<V>>> collectMultimap(Function<? super T, ? extends K> keyExtractor, Function<? super T, ? extends V> valueExtractor, Supplier<Map<K, Collection<V>>> mapSupplier) {
 
 
 
*****************
distinct 操作
 
    public final Flux<T> distinct() {
 
    public final <V> Flux<T> distinct(Function<? super T, ? extends V> keySelector) {
    public final <V, C extends Collection<? super V>> Flux<T> distinct(Function<? super T, ? extends V> keySelector, Supplier<C> distinctCollectionSupplier) {
    public final <V, C> Flux<T> distinct(Function<? super T, ? extends V> keySelector, Supplier<C> distinctStoreSupplier, BiPredicate<C, V> distinctPredicate, Consumer<C> cleanup) {
 
    public final Flux<T> distinctUntilChanged() {
    public final <V> Flux<T> distinctUntilChanged(Function<? super T, ? extends V> keySelector) {
    public final <V> Flux<T> distinctUntilChanged(Function<? super T, ? extends V> keySelector, BiPredicate<? super V, ? super V> keyComparator) {
 
 
 
*****************
group 操作
 
    public final <K> Flux<GroupedFlux<K, T>> groupBy(Function<? super T, ? extends K> keyMapper) {
    public final <K> Flux<GroupedFlux<K, T>> groupBy(Function<? super T, ? extends K> keyMapper, int prefetch) {
    public final <K, V> Flux<GroupedFlux<K, V>> groupBy(Function<? super T, ? extends K> keyMapper, Function<? super T, ? extends V> valueMapper) {
    public final <K, V> Flux<GroupedFlux<K, V>> groupBy(Function<? super T, ? extends K> keyMapper, Function<? super T, ? extends V> valueMapper, int prefetch) {
 
    public final <TRight, TLeftEnd, TRightEnd, R> Flux<R> groupJoin(Publisher<? extends TRight> other, Function<? super T, ? extends Publisher<TLeftEnd>> leftEnd, Function<? super TRight, ? extends Publisher<TRightEnd>> rightEnd, BiFunction<? super T, ? super Flux<TRight>, ? extends R> resultSelector) {
 
 
 
*****************
sort 操作
 
    public final Flux<T> sort() {
    public final Flux<T> sort(Comparator<? super T> sortFunction) {
 
 
 
*****************
parallel 操作
 
    public final ParallelFlux<T> parallel() {
    public final ParallelFlux<T> parallel(int parallelism) {
    public final ParallelFlux<T> parallel(int parallelism, int prefetch) {
 
 
*****************
reduce 操作
 
    public final Mono<T> reduce(BiFunction<T, T, T> aggregator) {
    public final <A> Mono<A> reduce(A initial, BiFunction<A, ? super T, A> accumulator) {
 
    public final <A> Mono<A> reduceWith(Supplier<A> initial, BiFunction<A, ? super T, A> accumulator) {
 
 
 
*****************
zipWith 操作
 
    public final <T2> Flux<Tuple2<T, T2>> zipWith(Publisher<? extends T2> source2) {
    public final <T2, V> Flux<V> zipWith(Publisher<? extends T2> source2, BiFunction<? super T, ? super T2, ? extends V> combinator) {
    public final <T2, V> Flux<V> zipWith(Publisher<? extends T2> source2, int prefetch, BiFunction<? super T, ? super T2, ? extends V> combinator) {
    public final <T2> Flux<Tuple2<T, T2>> zipWith(Publisher<? extends T2> source2, int prefetch) {
 
    public final <T2> Flux<Tuple2<T, T2>> zipWithIterable(Iterable<? extends T2> iterable) {
    public final <T2, V> Flux<V> zipWithIterable(Iterable<? extends T2> iterable, BiFunction<? super T, ? super T2, ? extends V> zipper) {
 
 
 
*****************
mergeWith 操作
 
    public final Flux<T> mergeWith(Publisher<? extends T> other) {
    public final Flux<T> mergeOrderedWith(Publisher<? extends T> other, Comparator<? super T> otherComparator) {
 
 
 
*****************
do 操作
 
    public final Flux<T> doFirst(Runnable onFirst) {
    public final Flux<T> doFinally(Consumer<SignalType> onFinally) {
 
    public final Flux<T> doOnCancel(Runnable onCancel) {
    public final Flux<T> doOnComplete(Runnable onComplete) {
    public final Flux<T> doOnTerminate(Runnable onTerminate) {
    public final Flux<T> doAfterTerminate(Runnable afterTerminate) {
    public final <R> Flux<T> doOnDiscard(Class<R> type, Consumer<? super R> discardHook) {
 
    public final Flux<T> doOnEach(Consumer<? super Signal<T>> signalConsumer) {
    public final Flux<T> doOnNext(Consumer<? super T> onNext) {
    public final Flux<T> doOnRequest(LongConsumer consumer) {
    public final Flux<T> doOnSubscribe(Consumer<? super Subscription> onSubscribe) {
 
    public final Flux<T> doOnError(Consumer<? super Throwable> onError) {
    public final <E extends Throwable> Flux<T> doOnError(Class<E> exceptionType, Consumer<? super E> onError) {
    public final Flux<T> doOnError(Predicate<? super Throwable> predicate, Consumer<? super Throwable> onError) {
 
    static <T> Flux<T> doOnSignal(Flux<T> source, @Nullable Consumer<? super Subscription> onSubscribe, @Nullable Consumer<? super T> onNext, @Nullable Consumer<? super Throwable> onError, @Nullable Runnable onComplete, @Nullable Runnable onAfterTerminate, @Nullable LongConsumer onRequest, @Nullable Runnable onCancel) {
                                 //方法权限为default，同一个包中调用
 
 
 
*****************
elementAt 操作
 
    public final Mono<T> elementAt(int index) {
    public final Mono<T> elementAt(int index, T defaultValue) {
 
 
 
*****************
last 操作
 
    public final Mono<T> last() {
    public final Mono<T> last(T defaultValue) {
 
 
 
*****************
index 操作
 
    public final Flux<Tuple2<Long, T>> index() {
    public final <I> Flux<I> index(BiFunction<? super Long, ? super T, ? extends I> indexMapper) {
 
 
 
*****************
repeat 操作
 
    public final Flux<T> repeat() {
    public final Flux<T> repeat(BooleanSupplier predicate) {
    public final Flux<T> repeat(long numRepeat) {
    public final Flux<T> repeat(long numRepeat, BooleanSupplier predicate) {
 
    public final Flux<T> repeatWhen(Function<Flux<Long>, ? extends Publisher<?>> repeatFactory) {
 
 
*****************
retry 操作
 
    public final Flux<T> retry() {
    public final Flux<T> retry(long numRetries) {
 
    public final Flux<T> retryWhen(Retry retrySpec) {
 
 
*****************
replay 操作
 
    public final ConnectableFlux<T> replay() {
    public final ConnectableFlux<T> replay(int history) {
    public final ConnectableFlux<T> replay(Duration ttl) {
    public final ConnectableFlux<T> replay(int history, Duration ttl) {
    public final ConnectableFlux<T> replay(Duration ttl, Scheduler timer) {
    public final ConnectableFlux<T> replay(int history, Duration ttl, Scheduler timer) {
 
 
 
*****************
skip 操作
 
    public final Flux<T> skip(long skipped) {
    public final Flux<T> skip(Duration timespan) {
    public final Flux<T> skip(Duration timespan, Scheduler timer) {
 
    public final Flux<T> skipLast(int n) {
 
    public final Flux<T> skipUntil(Predicate<? super T> untilPredicate) {
    public final Flux<T> skipUntilOther(Publisher<?> other) {
 
    public final Flux<T> skipWhile(Predicate<? super T> skipPredicate) {
 
 
 
*****************
then 操作
 
    public final Mono<Void> then() {
    public final <V> Mono<V> then(Mono<V> other) {
 
    public final Mono<Void> thenEmpty(Publisher<Void> other) {
    public final <V> Flux<V> thenMany(Publisher<V> other) {
 
 
 
*****************
elapsed 操作
 
    public final Flux<Tuple2<Long, T>> elapsed() {
    public final Flux<Tuple2<Long, T>> elapsed(Scheduler scheduler) {
 
 
 
*****************
expand 操作
 
    public final Flux<T> expandDeep(Function<? super T, ? extends Publisher<? extends T>> expander, int capacityHint) {
    public final Flux<T> expandDeep(Function<? super T, ? extends Publisher<? extends T>> expander) {
 
    public final Flux<T> expand(Function<? super T, ? extends Publisher<? extends T>> expander, int capacityHint) {
    public final Flux<T> expand(Function<? super T, ? extends Publisher<? extends T>> expander) {
 
 
 
*****************
limitRate 操作
 
    public final Flux<T> limitRate(int prefetchRate) {
    public final Flux<T> limitRate(int highTide, int lowTide) {
 
    public final Flux<T> limitRequest(long requestCap) {
 
 
 
*****************
log 操作
 
    public final Flux<T> log() {
        return this.log((String)null, Level.INFO);
    }
 
    public final Flux<T> log(String category) {
    public final Flux<T> log(@Nullable String category, Level level, SignalType... options) {
    public final Flux<T> log(@Nullable String category, Level level, boolean showOperatorLine, SignalType... options) {
 
    public final Flux<T> log(Logger logger) {
    public final Flux<T> log(Logger logger, Level level, boolean showOperatorLine, SignalType... options) {
 
 
 
*****************
buffer 操作
 
    public final Flux<List<T>> buffer() {
    public final Flux<List<T>> buffer(int maxSize) {
    public final Flux<List<T>> buffer(Publisher<?> other) {
 
    public final <C extends Collection<? super T>> Flux<C> buffer(int maxSize, Supplier<C> bufferSupplier) {
    public final <C extends Collection<? super T>> Flux<C> buffer(int maxSize, int skip, Supplier<C> bufferSupplier) {
    public final <C extends Collection<? super T>> Flux<C> buffer(Publisher<?> other, Supplier<C> bufferSupplier) {
 
    public final Flux<List<T>> buffer(Duration bufferingTimespan) {
    public final Flux<List<T>> buffer(Duration bufferingTimespan, Duration openBufferEvery) {
    public final Flux<List<T>> buffer(Duration bufferingTimespan, Scheduler timer) {
    public final Flux<List<T>> buffer(Duration bufferingTimespan, Duration openBufferEvery, Scheduler timer) {
 
    public final Flux<List<T>> bufferTimeout(int maxSize, Duration maxTime) {
    public final Flux<List<T>> bufferTimeout(int maxSize, Duration maxTime, Scheduler timer) {
    public final <C extends Collection<? super T>> Flux<C> bufferTimeout(int maxSize, Duration maxTime, Supplier<C> bufferSupplier) {
    public final <C extends Collection<? super T>> Flux<C> bufferTimeout(int maxSize, Duration maxTime, Scheduler timer, Supplier<C> bufferSupplier) {
 
 
    public final Flux<List<T>> bufferUntil(Predicate<? super T> predicate) {
    public final Flux<List<T>> bufferUntil(Predicate<? super T> predicate, boolean cutBefore) {
 
    public final <V> Flux<List<T>> bufferUntilChanged() {
    public final <V> Flux<List<T>> bufferUntilChanged(Function<? super T, ? extends V> keySelector) {
    public final <V> Flux<List<T>> bufferUntilChanged(Function<? super T, ? extends V> keySelector, BiPredicate<? super V, ? super V> keyComparator) {
 
    public final Flux<List<T>> bufferWhile(Predicate<? super T> predicate) {
 
    public final <U, V> Flux<List<T>> bufferWhen(Publisher<U> bucketOpening, Function<? super U, ? extends Publisher<V>> closeSelector) {
    public final <U, V, C extends Collection<? super T>> Flux<C> bufferWhen(Publisher<U> bucketOpening, Function<? super U, ? extends Publisher<V>> closeSelector, Supplier<C> bufferSupplier) {
 
 
 
*****************
cache 操作
 
    public final Flux<T> cache() {
    public final Flux<T> cache(int history) {
    public final Flux<T> cache(Duration ttl) {
    public final Flux<T> cache(Duration ttl, Scheduler timer) {
    public final Flux<T> cache(int history, Duration ttl) {
    public final Flux<T> cache(int history, Duration ttl, Scheduler timer) {
 
 
 
*****************
delay 操作
 
    public final Flux<T> delayElements(Duration delay) {
    public final Flux<T> delayElements(Duration delay, Scheduler timer) {
 
    public final Flux<T> delaySequence(Duration delay) {
    public final Flux<T> delaySequence(Duration delay, Scheduler timer) {
 
    public final Flux<T> delayUntil(Function<? super T, ? extends Publisher<?>> triggerProvider) {
 
    public final Flux<T> delaySubscription(Duration delay) {
    public final Flux<T> delaySubscription(Duration delay, Scheduler timer) {
    public final <U> Flux<T> delaySubscription(Publisher<U> subscriptionDelay) {
 
 
 
*****************
onBackpressure 操作
 
    public final Flux<T> onBackpressureBuffer() {
    public final Flux<T> onBackpressureBuffer(int maxSize) {
 
    public final Flux<T> onBackpressureBuffer(int maxSize, Consumer<? super T> onOverflow) {
    public final Flux<T> onBackpressureBuffer(int maxSize, BufferOverflowStrategy bufferOverflowStrategy) {
    public final Flux<T> onBackpressureBuffer(int maxSize, Consumer<? super T> onBufferOverflow, BufferOverflowStrategy bufferOverflowStrategy) {
    public final Flux<T> onBackpressureBuffer(Duration ttl, int maxSize, Consumer<? super T> onBufferEviction) {
    public final Flux<T> onBackpressureBuffer(Duration ttl, int maxSize, Consumer<? super T> onBufferEviction, Scheduler scheduler) {
 
    public final Flux<T> onBackpressureDrop() {
    public final Flux<T> onBackpressureDrop(Consumer<? super T> onDropped) {
    public final Flux<T> onBackpressureError() {
    public final Flux<T> onBackpressureLatest() {
 
 
*****************
onError 操作
 
    public final Flux<T> onErrorContinue(BiConsumer<Throwable, Object> errorConsumer) {
    public final <E extends Throwable> Flux<T> onErrorContinue(Class<E> type, BiConsumer<Throwable, Object> errorConsumer) {
    public final <E extends Throwable> Flux<T> onErrorContinue(Predicate<E> errorPredicate, BiConsumer<Throwable, Object> errorConsumer) {
 
    public final Flux<T> onErrorStop() {
 
    public final Flux<T> onErrorMap(Function<? super Throwable, ? extends Throwable> mapper) {
    public final <E extends Throwable> Flux<T> onErrorMap(Class<E> type, Function<? super E, ? extends Throwable> mapper) {
    public final Flux<T> onErrorMap(Predicate<? super Throwable> predicate, Function<? super Throwable, ? extends Throwable> mapper) {
 
    public final Flux<T> onErrorResume(Function<? super Throwable, ? extends Publisher<? extends T>> fallback) {
    public final <E extends Throwable> Flux<T> onErrorResume(Class<E> type, Function<? super E, ? extends Publisher<? extends T>> fallback) {
    public final Flux<T> onErrorResume(Predicate<? super Throwable> predicate, Function<? super Throwable, ? extends Publisher<? extends T>> fallback) {
 
    public final Flux<T> onErrorReturn(T fallbackValue) {
    public final <E extends Throwable> Flux<T> onErrorReturn(Class<E> type, T fallbackValue) {
    public final Flux<T> onErrorReturn(Predicate<? super Throwable> predicate, T fallbackValue) {
 
 
 
*****************
publish 操作
 
    public final ConnectableFlux<T> publish() {
    public final ConnectableFlux<T> publish(int prefetch) {
    public final <R> Flux<R> publish(Function<? super Flux<T>, ? extends Publisher<? extends R>> transform) {
    public final <R> Flux<R> publish(Function<? super Flux<T>, ? extends Publisher<? extends R>> transform, int prefetch) {
 
    public final Mono<T> publishNext() {
 
    public final Flux<T> publishOn(Scheduler scheduler) {
    public final Flux<T> publishOn(Scheduler scheduler, int prefetch) {
    public final Flux<T> publishOn(Scheduler scheduler, boolean delayError, int prefetch) {
 
    final Flux<T> publishOn(Scheduler scheduler, boolean delayError, int prefetch, int lowTide) {
                           //方法权限为default，只能在同一个包中使用
 
 
*****************
sample 操作
 
    public final Flux<T> sample(Duration timespan) {
    public final <U> Flux<T> sample(Publisher<U> sampler) {
 
    public final Flux<T> sampleFirst(Duration timespan) {
    public final <U> Flux<T> sampleFirst(Function<? super T, ? extends Publisher<U>> samplerFactory) {
 
    public final <U> Flux<T> sampleTimeout(Function<? super T, ? extends Publisher<U>> throttlerFactory) {
    public final <U> Flux<T> sampleTimeout(Function<? super T, ? extends Publisher<U>> throttlerFactory, int maxConcurrency) {
 
 
 
*****************
scan 操作
 
    public final Flux<T> scan(BiFunction<T, T, T> accumulator) {
    public final <A> Flux<A> scan(A initial, BiFunction<A, ? super T, A> accumulator) {
 
    public final <A> Flux<A> scanWith(Supplier<A> initial, BiFunction<A, ? super T, A> accumulator) {
 
 
 
*****************
startWith 操作
 
 
    public final Flux<T> startWith(Iterable<? extends T> iterable) {
    public final Flux<T> startWith(T... values) {
    public final Flux<T> startWith(Publisher<? extends T> publisher) {
 
 
 
*****************
take 操作
 
    public final Flux<T> take(long n) {
 
    public final Flux<T> take(Duration timespan) {
    public final Flux<T> take(Duration timespan, Scheduler timer) {
 
    public final Flux<T> takeLast(int n) {
 
    public final Flux<T> takeUntil(Predicate<? super T> predicate) {
    public final Flux<T> takeUntilOther(Publisher<?> other) {
 
    public final Flux<T> takeWhile(Predicate<? super T> continuePredicate) {
 
 
 
*****************
timeout 操作
 
    public final Flux<T> timeout(Duration timeout) {
    public final Flux<T> timeout(Duration timeout, @Nullable Publisher<? extends T> fallback) {
    public final Flux<T> timeout(Duration timeout, Scheduler timer) {
    public final Flux<T> timeout(Duration timeout, @Nullable Publisher<? extends T> fallback, Scheduler timer) {
 
    public final <U> Flux<T> timeout(Publisher<U> firstTimeout) {
    public final <U, V> Flux<T> timeout(Publisher<U> firstTimeout, Function<? super T, ? extends Publisher<V>> nextTimeoutFactory) {
    private final <U, V> Flux<T> timeout(Publisher<U> firstTimeout, Function<? super T, ? extends Publisher<V>> nextTimeoutFactory, String timeoutDescription) {
    public final <U, V> Flux<T> timeout(Publisher<U> firstTimeout, Function<? super T, ? extends Publisher<V>> nextTimeoutFactory, Publisher<? extends T> fallback) {
 
 
 
*****************
timestamp 操作
 
    public final Flux<Tuple2<Long, T>> timestamp() {
    public final Flux<Tuple2<Long, T>> timestamp(Scheduler scheduler) {
 
 
 
*****************
window 操作
 
    public final Flux<Flux<T>> window(int maxSize) {
    public final Flux<Flux<T>> window(int maxSize, int skip) {
 
    public final Flux<Flux<T>> window(Publisher<?> boundary) {
 
    public final Flux<Flux<T>> window(Duration windowingTimespan) {
    public final Flux<Flux<T>> window(Duration windowingTimespan, Duration openWindowEvery) {
    public final Flux<Flux<T>> window(Duration windowingTimespan, Scheduler timer) {
    public final Flux<Flux<T>> window(Duration windowingTimespan, Duration openWindowEvery, Scheduler timer) {
 
    public final Flux<Flux<T>> windowTimeout(int maxSize, Duration maxTime) {
    public final Flux<Flux<T>> windowTimeout(int maxSize, Duration maxTime, Scheduler timer) {
 
    public final Flux<Flux<T>> windowUntil(Predicate<T> boundaryTrigger) {
    public final Flux<Flux<T>> windowUntil(Predicate<T> boundaryTrigger, boolean cutBefore) {
    public final Flux<Flux<T>> windowUntil(Predicate<T> boundaryTrigger, boolean cutBefore, int prefetch) {
 
    public final <V> Flux<Flux<T>> windowUntilChanged() {
    public final <V> Flux<Flux<T>> windowUntilChanged(Function<? super T, ? super V> keySelector) {
    public final <V> Flux<Flux<T>> windowUntilChanged(Function<? super T, ? extends V> keySelector, BiPredicate<? super V, ? super V> keyComparator) {
 
    public final Flux<Flux<T>> windowWhile(Predicate<T> inclusionPredicate) {
    public final Flux<Flux<T>> windowWhile(Predicate<T> inclusionPredicate, int prefetch) {
 
    public final <U, V> Flux<Flux<T>> windowWhen(Publisher<U> bucketOpening, Function<? super U, ? extends Publisher<V>> closeSelector) {
 
 
 
*****************
checkpoint 操作
 
    public final Flux<T> checkpoint() {
    public final Flux<T> checkpoint(String description) {
    public final Flux<T> checkpoint(@Nullable String description, boolean forceStackTrace) {
 
 
 
*****************
block 操作
 
    public final T blockFirst() {
    public final T blockFirst(Duration timeout) {
 
    public final T blockLast() {
    public final T blockLast(Duration timeout) {
 
 
 
*****************
subscribe 操作
 
    public final Disposable subscribe() {
    public final Disposable subscribe(Consumer<? super T> consumer) {
    public final Disposable subscribe(@Nullable Consumer<? super T> consumer, Consumer<? super Throwable> errorConsumer) {
    public final Disposable subscribe(@Nullable Consumer<? super T> consumer, @Nullable Consumer<? super Throwable> errorConsumer, @Nullable Runnable completeConsumer) {
    public final Disposable subscribe(@Nullable Consumer<? super T> consumer, @Nullable Consumer<? super Throwable> errorConsumer, @Nullable Runnable completeConsumer, @Nullable Consumer<? super Subscription> subscriptionConsumer) {
    public final Disposable subscribe(@Nullable Consumer<? super T> consumer, @Nullable Consumer<? super Throwable> errorConsumer, @Nullable Runnable completeConsumer, @Nullable Context initialContext) {
 
    public final void subscribe(Subscriber<? super T> actual) {
    public abstract void subscribe(CoreSubscriber<? super T> var1);
 
    public final Flux<T> subscriberContext(Context mergeContext) {
    public final Flux<T> subscriberContext(Function<Context, Context> doOnContext) {
 
    public final Flux<T> subscribeOn(Scheduler scheduler) {
    public final Flux<T> subscribeOn(Scheduler scheduler, boolean requestOnSeparateThread) {
 
    public final <E extends Subscriber<? super T>> E subscribeWith(E subscriber) {
 
 
 
*****************
其他操作
 
    public final Mono<T> next() {
    public final Mono<Long> count() {
    public final Flux<T> defaultIfEmpty(T defaultV) {
 
    public final Mono<Boolean> all(Predicate<? super T> predicate) {
    public final Mono<Boolean> any(Predicate<? super T> predicate) {
 
    public final Mono<Boolean> hasElement(T value) {
    public final Mono<Boolean> hasElements() {
 
    public final Mono<T> ignoreElements() {
 
    public final <U, R> Flux<R> withLatestFrom(Publisher<? extends U> other, BiFunction<? super T, ? super U, ? extends R> resultSelector) {
 
    public final <TRight, TLeftEnd, TRightEnd, R> Flux<R> join(Publisher<? extends TRight> other, Function<? super T, ? extends Publisher<TLeftEnd>> leftEnd, Function<? super TRight, ? extends Publisher<TRightEnd>> rightEnd, BiFunction<? super T, ? super TRight, ? extends R> resultSelector) {
 
    public final <P> P as(Function<? super Flux<T>, P> transformer) {
 
    public final Flux<Signal<T>> materialize() {
    public final <X> Flux<X> dematerialize() {
 
    public final Flux<T> cancelOn(Scheduler scheduler) {
    public final Flux<T> onTerminateDetach() {
 
    public final Flux<T> or(Publisher<? extends T> other) {
 
    public Flux<T> hide() {
    public final <R> Flux<R> handle(BiConsumer<? super T, SynchronousSink<R>> handler) {
 
    public final <E> Flux<E> cast(Class<E> clazz) {
    public final <U> Flux<U> ofType(Class<U> clazz) {
 
    public int getPrefetch() {
    public final Flux<T> share() {
 
    public final Flux<T> metrics() {
    public final Flux<T> name(String name) {
    public final Flux<T> tag(String key, String value) {
 
    static <O> Supplier<Set<O>> hashSetSupplier() {
    static <O> Supplier<List<O>> listSupplier() {
    static <U, V> BiPredicate<U, V> equalPredicate() {
    static <T> Function<T, T> identityFunction() {
    static <A, B> BiFunction<A, B, Tuple2<A, B>> tuple2Function() {
 
    static BooleanSupplier countingBooleanSupplier(final BooleanSupplier predicate, final long max) {
    static <O> Predicate<O> countingPredicate(final Predicate<O> predicate, final long max) {
 
    static <I> Flux<I> wrap(Publisher<? extends I> source) {
    static <T> Mono<T> wrapToMono(Callable<T> supplier) {
 
    protected static <T> Flux<T> onAssembly(Flux<T> source) {
    protected static <T> ConnectableFlux<T> onAssembly(ConnectableFlux<T> source) {
 
    public String toString() {

```

### 测试Demo

#### 响应式输出 逐行输出给前端
```
  @GetMapping(value = "index", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    private Flux<String> flux() {
        return Flux
                .fromStream(IntStream.range(1, 5).mapToObj(i -> {
                    try {
                        TimeUnit.SECONDS.sleep(1);
                    } catch (InterruptedException ignored) {
                    }
                    return "flux data--" + i;
                }));
    }
```

#### 常规输出

```
  public Mono<String> hello() {
        return Mono.just("ces1");
    }

    public Flux<String> hello2() {
        ArrayList<String> list = new ArrayList<>();
        list.add("小明");
        list.add("小东");
        list.add("小强");
        return Flux.fromIterable(list);
    }
```

#### 常规输出2
```
 @GetMapping("/hello")
    private Mono<String> hello() {   // 【改】返回类型为Mono<String>
        return Mono.just("Welcome to reactive world ~");     // 【改】使用Mono.just生成响应式数据
    }


    @GetMapping("/hello1")
    private Mono<String> hello1() {
        return helloService.hello();
    }

    @GetMapping("/hello2")
    private Flux<String> hello2() {
        return helloService.hello2();
    }
```
