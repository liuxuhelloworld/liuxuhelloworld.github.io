# 级联同步方案
![级联同步方案示意图](https://static001.geekbang.org/resource/image/3a/2b/3a2e08181177529c3229c789c2081b2b.jpg)

![级联同步方案回滚示意图](https://static001.geekbang.org/resource/image/ad/b9/ada8866fda3c3264f495c97c6214ebb9.jpg)

这种方案的优点是简单易实施，业务上基本没有改造的成本；缺点是切写的时候需要短暂地停止写入，业务上是有损的。

# 双写方案
![双写方案示意图](https://static001.geekbang.org/resource/image/ad/30/ad9a4aa37afc39ebe0c91144d5ef7630.jpg)

![双写方案示意图](https://static001.geekbang.org/resource/image/ad/30/ad9a4aa37afc39ebe0c91144d5ef7630.jpg)

这种方案的优点是业务无损，缺点是时间周期比较长，业务有改造成本。