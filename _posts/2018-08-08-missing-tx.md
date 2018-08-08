---
title:  "丢失的Transaction"
date:   2018-08-08 12:00:00 +0800
categories: [ethereum, bug-snapshot]
---

昨天，我的以太坊监听器丢了一个Transaction，交易丢失对于监听器来说是很严重的问题，检查了一早上，但是结果还是毫无收获。

# 回顾
* 这个交易之前和之后的交易都被监听到了，在这期间没有任何人碰过它和它所监听的客户端
* 同一时间，另一个监听器得到了这条记录
* 没有任何log表明，它收到过这个记录
<!--more-->

# 分析
我的监听器结构如下图，十分简单。通过Web3j库的`block filter`与Ethereum客户端交互，取得了需要的`block`变化数据，处理数据然后存入数据库。这些数据获取和处理过程都是线性的，一条数据走完整个流程，才会处理下一条数据。

![my listener ]({{ "/assets/my-listener-structure.png" | absolute_url }})

整个过程有两个`filter`是最值得怀疑的地方。

第一个`filter`由web3j提供，内部实现也很简单，在listener启动时给一个开始的startBlock，然后web3j比较该block是否是最新block，如果不是，连接 获取该块到最新块数据的Observable 和 递归执行这个检查；如果是最新块，返回`onCompleteObservable`, `onCompleteObservable`是我实现的`blockObservable`。反复检查，这部分代码没有发现异常，代码实现如下：
```java
private Observable<EthBlock> catchUpToLatestBlockObservableSync(
            DefaultBlockParameter startBlock, boolean fullTransactionObjects,
            Observable<EthBlock> onCompleteObservable) {

        BigInteger startBlockNumber;
        BigInteger latestBlockNumber;
        try {
            startBlockNumber = getBlockNumber(startBlock);
            latestBlockNumber = getLatestBlockNumber();
        } catch (IOException e) {
            return Observable.error(e);
        }

        if (startBlockNumber.compareTo(latestBlockNumber) > -1) {
            return onCompleteObservable;
        } else {
            return Observable.concat(
                    replayBlocksObservableSync(
                            new DefaultBlockParameterNumber(startBlockNumber),
                            new DefaultBlockParameterNumber(latestBlockNumber),
                            fullTransactionObjects),
                    Observable.defer(() -> catchUpToLatestBlockObservableSync(
                            new DefaultBlockParameterNumber(latestBlockNumber.add(BigInteger.ONE)),
                            fullTransactionObjects,
                            onCompleteObservable)));
        }
    }

private Observable<EthBlock> blockObservable() {
    return ethBlockHashObservable()
        .flatMap(blockHash -> web3j.ethGetBlockByHash(blockHash, true).observable());
}

private Observable<String> ethBlockHashObservable() {
    return Observable.create(subscriber -> {

      try {
        org.web3j.protocol.core.methods.response.EthFilter filter = web3j.ethNewBlockFilter().send();
        BigInteger filterId = filter.getFilterId();

        ScheduledFuture<?> scheduledFuture = scheduledExecutorService
            .scheduleAtFixedRate(() -> {
              try {
                pollFilter(subscriber, filterId);
              } catch (Exception e) {
                subscriber.onError(e);
              }
            }, 0, pollingInterval, TimeUnit.SECONDS);
        subscriber.add(Subscriptions.create(() -> cancel(filterId, scheduledFuture)));
      } catch (IOException e) {
        throwException(e);
      }
    });
  }
```

第二个`filter`是正常的数据过滤，将需要的监听的地址放入一个缓存中，与比较交易中的地址。这里感觉也不像是出错的地方，因为前后交易都被记录了下来。

# 解决

对于这个bug真的毫无头绪。比较了另一个listener的实现，主要有两个地方不同，一是web3j版本，二是它等待了10 confirmations。

暂时能想到的解决是，更新web3j，3.5版本加入了rebuid filter功能，我就不需要自己实现`onCompleteObservable`，可能会增加代码的稳定性。由于这个监听器对于时间比较敏感，暂时不打算加入等待。

# 资料

* [waiting confirmation](https://ethereum.stackexchange.com/questions/319/what-number-of-confirmations-is-considered-secure-in-ethereum)
* [transaction lifecycle](https://freecontent.manning.com/wp-content/uploads/mentalmodel-BuildingEthereumDApps2.png)