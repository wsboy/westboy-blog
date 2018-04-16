# 重试框架SpringRetry

spring-retry 主要实现了**重试**和**熔断**。

![](./img/img-001.png)

> 本文源码版本为：spring-retry:1.2.2.RELEASE

RetryOperations 定义了重试的 API，RetryTemplate 提供了模板实现，线程安全的，同于 Spring 一贯的 API 风格，RetryTemplate 将重试、熔断功能封装到模板中，提供健壮和不易出错的API供大家使用。

首先，RetryOperations 接口 API：

```java
package org.springframework.retry;

public interface RetryOperations {
    <T, E extends Throwable> T execute(RetryCallback<T, E> var1) throws E;

    <T, E extends Throwable> T execute(RetryCallback<T, E> var1, RecoveryCallback<T> var2) throws E;

    <T, E extends Throwable> T execute(RetryCallback<T, E> var1, RetryState var2) throws E, ExhaustedRetryException;

    <T, E extends Throwable> T execute(RetryCallback<T, E> var1, RecoveryCallback<T> var2, RetryState var3) throws E;
}
```
解释一下各个参数：
* RetryCallback：定义需重试的业务服务;
* RecoveryCallback：当重试超过最大重试时间或最大重试次数后可以调用 RecoveryCallback 进行恢复；
* RetryState：定义有状态重试。

那什么时候需重试？spring-retry 是当抛出相关异常后执行重试策略，定义重试策略时需要定义需要重试的异常（如因远程调用失败的可以重试、而因入参校对失败不应该重试）。只读操作可以重试，幂等写操作可以重试，但是非幂等写操作不能重试，重试可能导致脏写，或产生重复数据。


重试策略有哪些呢？

![](./img/img-002.png)

RetryPolicy 提供了如下策略实现：
* SimpleRetryPolicy：固定次数重试策略，默认重试最大次数为3次，RetryTemplate 默认使用的策略；
* ExpressionRetryPolicy：使用表达式匹配重试策略。
* TimeoutRetryPolicy：超时时间重试策略，默认超时时间为1秒，在指定的超时时间内允许重试；
* ExceptionClassifierRetryPolicy：设置不同异常的重试策略，类似组合重试策略，区别在于这里只区分不同异常的重试；
* CompositeRetryPolicy：组合重试策略，有两种组合方式，乐观组合重试策略是指顺序遍历配置的组合策略如果有一个策略允许重试就可以重试，悲观组合重试策略则只要有一个策略不允许重试就不可以重试。
* CircuitBreakerRetryPolicy：有熔断功能的重试策略，需设置3个参数 openTimeout、resetTimeout 和 delegate，稍后详细介绍该策略；
* NeverRetryPolicy：只允许调用 RetryCallback 一次，不允许重试；
* AlwaysRetryPolicy：允许无限重试，直到成功，此方式逻辑不当会导致死循环。

重试时的退避策略是什么？是立即重试还是等待一段时间后重试，比如是网络错误，立即重试将导致立即失败，最好等待一小段时间后重试，还要防止很多服务同时重试导致 DDos。

![](./img/img-003.png)

BackOffPolicy 提供了如下策略实现： 
* ExponentialBackOffPolicy：指数退避策略，需设置参数 sleeper、initialInterval、maxInterval 和 multiplier，initialInterval 指定初始休眠时间，默认100毫秒，maxInterval 指定最大休眠时间，默认30秒，multiplier 指定乘数，即下一次休眠时间为当前休眠时间 * multiplier；
* ExponentialRandomBackOffPolicy：随机指数退避策略，引入随机乘数，之前说过固定乘数可能会引起很多服务同时重试导致 DDos，使用随机休眠时间来避免这种情况。
* FixedBackOffPolicy：固定时间的退避策略，需设置参数 sleeper 和 backOffPeriod，sleeper 指定等待策略，默认是 Thread.sleep，即线程休眠，backOffPeriod 指定休眠时间，默认1秒；
* UniformRandomBackOffPolicy：随机时间退避策略，需设置 sleeper、minBackOffPeriod 和 maxBackOffPeriod，该策略在 [minBackOffPeriod, maxBackOffPeriod] 之间取一个随机休眠时间，minBackOffPeriod 默认500毫秒，maxBackOffPeriod 默认1500毫秒；
* NoBackOffPolicy：无退避算法策略，即当重试时是立即重试。