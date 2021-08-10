# 主备同步
![主备同步示意图](https://static001.geekbang.org/resource/image/a6/a3/a66c154c1bc51e071dd2cc8c1d6ca6a3.png)

1. 在备库B上通过change master命令设置主库的IP、端口、用户名、密码，以及从哪个位置开始请求binlog；
2. 在备库B上执行start slave命令，备库会启动两个线程，即io_thread和sql_thread，io_thread负责与主库建立连接；
3. 主库A校验用户名和密码后，开始按照备库B请求的位置，从本地读取binlog，通过dump_thread发送给B；
4. 备库B获得binlog后，写到本地文件relay log；
5. sql_thread读取relay log，解析日志并执行。