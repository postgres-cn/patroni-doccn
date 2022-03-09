# 第1章 介绍<br>
Patroni起源于Governor在Compose的项目一个分支，它包括许多新功能。<br>
有关使用Patroni基于Docker的部署示例，请参阅目前在Zalando使用的Spilo。有关其他背景信息，请参见：<br>
•PostgreSQL HA与Kubernetes和Patroni，Josh Berkus在KubeCon 2016年的演讲（视频）<br>
•2016年2月Zalando科技博客文章<br>
<b>1.1发展状况</b><br>
Patroni正在积极发展并接受资助。有关更多详细信息，请参阅下面的贡献部分。<br>
我们在此报告新发布的信息。<br>
<b>1.2技术要求和安装</b><br>
Mac OS的先决条件<br>
若要在Mac上安装，请先运行以下命令：<br>
<table border="1"><tr><th>brew install postgresql etcd haproxy libyaml python</th></tr></table><br>
<b>Psycopg2</b><br>
从psycopg2-2.8开始，默认情况下不再安装psycopg2的二进制版本。从源代码安装它需要C编译器和postgres+python开发包。
因为在python中，无法将依赖项指定为psycopg2或psycopg2-binary，所以您必须决定如何安装它。<br>
有几个选项可供选择：<br>
<b>1.使用发行版中的软件包管理器</b><br>
<table border="1"><tr><th>sudo apt-get install python-psycopg2 # install python2 psycopg2 module on Debian/ ˓→Ubuntu<br>
sudo apt-get install python3-psycopg2 # install python3 psycopg2 module on Debian/ ˓→Ubuntu<br>
sudo yum install python-psycopg2 # install python2 psycopg2 on RedHat/Fedora/ ˓→CentOS<br>
</th></tr></table><br>
<b>2.从二进制软件包安装psycopg2</b><br>
<table border="1"><tr><th>pip install psycopg2-binary</th></tr></table><br>
<b>3.从源代码安装psycopg2</b><br>
<table border="1"><tr><th>pip install psycopg2>=2.5.4</th></tr></table><br>
<b>pip的一般安装</b><br>
使用pip安装Patroni:<br>
<table border="1"><tr><th>pip install patroni[dependencies]</th></tr></table><br>
其中依赖项可以为空，也可以由以下一项或多项组成：<br>
<b>etcd or etcd3</b><br>
python-etcd 模块，以便将 Etcd 用作 DCS<br>
<b>consul</b><br>
python-consul 模块，以便将 Consul 用作 DCS<br>
<b>zookeeper</b><br>
kazoo 模块，以便将 Zookeeper 用作 DCS<br>
<b>exhibitor</b><br>
kazoo 模块，以便将参展商用作 DCS（与 Zookeeper 的依赖项相同）<br>
<b>kubernetes</b><br>
kubernetes 模块，以便在 Patroni 中使用 Kubernetes 作为 DCS<br>
<b>raft</b><br>
pysyncobj 模块，以便使用 python Raft 实现作为 DCS<br>
<b>aws</b><br>
boto 以使用 AWS 回调<br>
例如：下面的命令将Patroni与其依赖项，ETCD作为DCS，AWS回调一起安装。<br>
<table border="1"><tr><th>pip install patroni[etcd,aws]</th></tr></table><br>
请注意，应独立于Patroni安装用于创建副本或自定义引导脚本（即WAL-E）的外部工具。<br>
<b>1.3规划PostgreSQL节点数量</b><br>
Patroni/PostgreSQL节点与DCS节点分离（除非Patroni自己实现RAFT），并且因为对最小节点数没有要求。运行由一个主服务器和一个备用服务器组成的集群是非常好的。以后可以添加更多备用节点。<br>
<b>1.4运行和配置</b><br>
如果Patroni存储库是从https://github.com/zalando/patroni中克隆的。即您将需要示例配置文件postgres0.yml和postgres1.yml。如果您使用pip安装了Patroni，您可以从git存储库获取这些文件，并用patroni命令替换下面的./patroni.py。<br>
首先，请从不同的终端执行以下操作：<br>
<table border="1"><tr><th>> etcd --data-dir=data/etcd --enable-v2=true<br>
> ./patroni.py postgres0.yml<br>
> ./patroni.py postgres1.yml
</th></tr></table><br>
然后，将启动高可用性集群。测试YAML文件中的不同设置，以查看集群的行为改变。杀死一些组件以查看系统的行为。添加更多postgres*.yml文件以创建更多的节点组成更大的集群。Patroni提供了一个HAProxy配置，它将为您的应用程序提供一个用于连接到集群领导者（Leader）的端点。配置运行如下：<br>
<table border="1"><tr><th>> haproxy -f haproxy.cfg</th></tr></table><br>
<table border="1"><tr><th>> psql --host 127.0.0.1 --port 5000 postgres</th></tr></table><br>
<b>1.5 YAML配置</b><br>
在第7章有对etcd、consul和ZooKeeper的设置。示例请访问https://github.com/zalando/patroni/blob/master/postgres0.yml获取postgres0.yml的配置信息。<br>
<b>1.6环境配置</b><br>
有关通过环境变量配置的全面信息，请查看第6章。<br>
<b>1.7复制选择</b><br>
Patroni 使用 Postgres 的流式复制，默认情况下是异步的。 Patroni 的异步复制配置允许maximum_lag_on_failover 设置。 此设置可确保如果跟随者超过领导者的一定数量的字节，也不会发生故障转移。应根据业务需求增加或减少此设置。 也可以使用同步复制来获得更好的持久性保证。 有关详细信息，请参阅复制模式文档第10章。<br>
<b>1.8应用程序不应使用超级用户</b><br>
从应用程序连接时，请始终使用非超级用户。Patroni才能正常访问数据库功能。通过使用应用程序中的超级用户，您可以通过设置superuser_reserved_connections来使用整个连接池，包括为超级用户保留的连接。但是如果Patroni由于连接池已满而无法访问主服务器，那么不希望有这种行为。<br>
<b>1.9测试您的HA解决方案</b><br>
测试HA解决方案是一个耗时的过程，有很多变量。考虑到跨平台应用程序，情况尤其如此。您需要经过培训的系统管理员或顾问来完成此工作。这不是我们可以在文档中深入讨论的内容。<br>
也就是说，以下是您应该确保测试的一些基础架构：<br>
•网络（系统前面的网络以及NIC[物理或虚拟]本身）<br>
•磁盘IO<br>
•文件限制（Linux中的nofile）<br>
•RAM。即使您按照建议关闭了oomkiller，RAM的不可用也可能导致问题。<br>
•中央处理器<br>
•虚拟化争用（过度使用虚拟机监控程序）<br>
•任何cgroup限制（可能与上述相关）<br>
•kill-9 postgres进程（postmaster除外！）。这是一个不错的故障模拟。<br>
您不应该做的一件事是在postmaster进程上运行kill-9。这是因为这样做不会模仿任何现实场景。如果您担心您的基础设施不安全，并且攻击者可能会运行kill-9，那么再多的HA进程也无法修复这一问题。攻击者只需再次杀死进程，或以另一种方式造成混乱。<br>

[patroni-doccn](https://github.com/postgres-cn/patroni-doccn/blob/main/README.md)
