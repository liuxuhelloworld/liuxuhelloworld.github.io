压力测试指的是在高并发大流量下进行的测试，通过观察系统在大负载下的表现，寻找系统性能的瓶颈点或隐患。

压测注意点：
- 压测应当使用线上环境，但要注意流量隔离
- 压测应当使用线上流量而不是模拟流量
- 压测时应当从多台服务器发起流量

一个自动化的全链路压测平台应当包含以下几个模块：
- 流量构造模块：流量拷贝、流量清洗、流量打压测标
- 压测数据隔离模块：mock服务、影子存储
- 系统健康度检查和压测流量干预模块：逐步加压

![全链路压测系统架构示意图](https://static001.geekbang.org/resource/image/15/ff/1552e524d495bb7e129405578b7907ff.jpg)

