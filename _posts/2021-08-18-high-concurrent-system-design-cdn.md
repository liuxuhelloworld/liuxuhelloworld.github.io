# 什么是CDN(Content Distribution Network)？
CDN，内容分发网络，通过将静态资源分发到位于多个地理位置机房中的服务器上，然后基于就近访问来加快静态资源的访问速度。

# 从静态资源请求到CDN节点
1. 对静态资源请求做DNS解析，得到CNAME记录，映射到CDN域名；
2. 通过GSLB(Global Server Load Balance)将请求映射到就近的CDN节点。

![CDN域名解析示意图](https://static001.geekbang.org/resource/image/fc/01/fcc357ff674b4abdc00dc9c33cbf9a01.jpg)