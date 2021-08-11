# 为什么需要发号器？
因为数据库分库分表后，简单的使用自增id不能满足全局唯一性。

为什么不使用UUID？
- UUID占用的空间较大
- 我们希望生成的id具有单调递增性，一方面写入性能更好，另一方面业务可以根据ID排序
- 我们希望生成的id可以具备业务含义，这样通过反解ID可以排查问题

# Snowflake算法
Snowflake算法的核心思想是将64bit的二进制数字分成若干部分，每一部分存储有特定含义的数据，由此生成全局唯一的有序id：

![Snowflake算法示意图](https://static001.geekbang.org/resource/image/2d/8d/2dee7e8e227a339f8f3cb6e7b47c0c8d.jpg)

一般的，你可以根据需要调整Snowflake算法，定制自己的发号器实现。