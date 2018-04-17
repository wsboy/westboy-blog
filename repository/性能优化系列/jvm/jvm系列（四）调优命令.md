常用的jvm调优命令如下：

* jps
* jstack
* jstat

下面我们详细介绍一下以上常用调优命令。

## jps(Java Virtual Machine Process Status Tool)

用于输出VM中运行的进程状态信息，语法格式如下：

```shell
# jps [options] [hostid]
```

如果不指定hostid就默认为当前主机或服务器, 命令行参数选项说明如下：

* -q 只输出PID
* -m 输出传入main方法的参数
* -l 输出main方法所在类的全限名和main方法的参数
* -v 输出传入JVM的参数

```shell
# jps
21407 Bootstrap
25170 Jps
25046 Bootstrap
# jps -q
21407
25233
25046
# jps -m
21407 Bootstrap start
25180 Jps -m
25046 Bootstrap start
# jps -l
25190 sun.tools.jps.Jps
21407 org.apache.catalina.startup.Bootstrap
25046 org.apache.catalina.startup.Bootstrap
# jps -ml
21407 org.apache.catalina.startup.Bootstrap start
25046 org.apache.catalina.startup.Bootstrap start
25210 sun.tools.jps.Jps -ml
# jps -mlv
21407 org.apache.catalina.startup.Bootstrap start #java main方法和main方法的参数
	-Djava.util.logging.config.file=/home/webapps/tomcat-store/conf/logging.properties #jvm参数
	......
25222 sun.tools.jps.Jps -mlv
	......
25046 org.apache.catalina.startup.Bootstrap start 
	-Djava.util.logging.config.file=/home/webapps/tomcat-test/conf/logging.properties 
	......
```

———— ☆☆☆ —— 返回 -> [JVM](index.md) <- 目录 —— ☆☆☆ ————

