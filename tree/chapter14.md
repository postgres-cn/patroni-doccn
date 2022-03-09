# 第14章 发行说明<br>
<b>14.1 Version 2.1.1</b><br>
新功能<br>
支持 ETCD SRV 名称后缀 (David Pavlicek)<br>
Etcd 允许区分同一域下的多个 Etcd 集群，从现在起 Patroni 也支持它。<br>
与新领袖（huiyalin525）一起丰富历史<br>
它将新列添加到patronictl history输出中。<br>
使 CA 包可配置用于集群内 Kubernetes 配置 (Aron Parsons)<br>
默认情况下，Patroni 正在使用/var/run/secrets/kubernetes.io/serviceaccount/ca.crt，这个新功能允许指定自定义kubernetes.cacert.<br>
支持动态注册/注销为 Consul 服务和更改标签 (Tommy Li)<br>
以前它需要 Patroni 重新启动。<br>
Bug修复<br>
避免不必要的 REST API 重新加载 (Alexander Kukushkin)<br>
如果在磁盘上发生更改，先前版本添加了重新加载 REST API 证书的功能。不幸的是，重新加载是在开始后无条件发生的。<br>
etcd.use_proxies设置时不解析集群成员（Alexander）<br>
Patroni 启动时通过查询成员列表来检查 Etcd 集群的健康状况。除此之外，它还尝试解析他们的主机名，这在通过代理使用 Etcd 时是不必要的，并且会导致不必要的警告。<br>
跳过pg_stat_replication中具有NULL值的行(Alexander)<br>
似乎pg_stat_replication视图在replay_lsn、flush_lsn或write_lsn字段中可能包含空值，即使state='streaming'。<br>
<b>14.2 Version 2.1.0</b><br>
此版本增加了与 PostgreSQL v14 的兼容性，使逻辑复制槽能够在故障转移/切换中幸存下来，实现对 REST API 的许可名单的支持，并将日志数量减少到每个心跳一行。<br>
新功能<br>
与 PostgreSQL v14的兼容性 (Alexander Kukushkin)<br>
如果patroni本身未处于“暂停”模式，则取消暂停WAL replay。由于某些参数的更改，例如主节点上的max_connections，可能会“暂停”。<br>
故障转移逻辑槽 (Alexander)<br>
使逻辑复制槽在 PostgreSQL v11+ 上的故障转移/切换中继续存在。复制槽如果从主复制到副本并重新启动，然后pg_replication_slot_advance()函数用于将其向前移动。因此，插槽在故障转移之前已经存在，不应丢失任何事件，但是，某些事件可能会被多次传递。<br>
为 Patroni REST API实施许可名单(Alexander)<br>
如果已配置，则仅允许匹配规则的 IP 调用不安全的端点。除此之外，还可以自动将集群成员的 IP 包含到列表中。<br>
添加了对通过 unix 套接字进行复制连接的支持 (Mohamad El-Rifai)<br>
以前 Patroni 总是使用 TCP 进行复制连接，这可能会导致 SSL 验证出现一些问题。使用 unix 套接字允许免除复制用户的 SSL 验证。<br>
用户定义标签的健康检查 (Arman Jafari Tehrani)<br>
连同预定义标签：可以指定在输出和 REST API 中可见的任意数量的自定义标签。从现在开始，可以在健康检查中使用自定义标签。patronictl list<br>
添加了 Prometheus/metrics端点（Mark Mercado、Michael Banck）<br>
端点公开与/patroni.<br>
减少 Patroni 日志的输出 (Alexander)<br>
当一切正常时，每次运行 HA 循环只会写入一行。<br>
重大变化<br>
旧的永久逻辑复制插槽功能将不再适用于PostgreSQL v10及更旧版本（Alexander）<br>
在执行升级后创建逻辑槽的策略不能保证没有逻辑事件丢失并因此被禁用。<br>
如果节点持有锁，/leader端点始终返回200（Alexander）<br>
提升备集群需要更新负载均衡器的健康检查，不是很方便，容易忘记。为了解决这个问题，我们改变了/leader健康检查端点的行为。它会返回 200 而不考虑集群是正常的还是standby_cluster.<br>
Raft 支持的改进<br>
Raft 流量加密的可靠支持<br>
由于PySyncObj加密支持中的不同问题非常不稳定<br>
处理 Raft 实现中的 DNS 问题 (Alexander)<br>
如果self_addr和/或partner_addrs使用 DNS 名称而不是 IP 进行配置，PySyncObj则在创建对象时仅有效地进行一次解析。当同一个节点使用不同的 IP 重新上线时，它会导致问题。<br>
稳定性改进<br>
与psycopg2-2.9+的兼容性（Alexander）<br>
在psycopg2中，with connection块中忽略autocommit=True，这会中断复制协议连接。<br>
使用 Zookeeper修复过多的HA循环运行(Alexander)<br>
成员 ZNode 的更新引起了连锁反应，并导致连续多次运行 HA 循环。<br>
如果 REST API 证书在磁盘上发生更改，则重新加载 (Michael Todorovic)<br>
如果 REST API 证书文件就地更新，则 Patroni 不会执行重新加载。<br>
如果使用 kerberos 身份验证，则不创建 pgpass 目录 (Kostiantyn Nemchenko)<br>
Kerberos 和密码验证是相互排斥的。<br>
修复了自定义引导程序的小问题（Alexander）<br>
仅当我们进行PITR时，才使用hot_standby=off启动Postgres，并在PITR完成后重新启动。<br>
Bug修复<br>
与kazoo-2.7+的兼容性（Alexander）<br>
由于 Patroni 自己处理重试，它依赖于旧行为，kazoo当没有可用连接时，会立即丢弃对 Zookeeper 集群的请求。<br>
当知道我们通过代理连接时，显式请求 Etcd v3 集群的版本 (Alexander)<br>
Patroni 正在通过 gPRC-gateway 使用 Etcd v3 集群，它取决于集群版本，必须使用不同的端点（/v3、/v3beta、 或/v3alpha）。该版本仅与集群拓扑一起解决，但由于后者在通过代理连接时从未完成。<br>
<b>14.3 Version 2.0.2</b><br>
新功能<br>
能够忽略外部管理的复制槽 (James Coleman)<br>
Patroni 正在尝试删除它不知道的任何复制槽，但肯定存在应该在外部管理复制槽的情况。从现在开始，可以配置不应删除的插槽。<br>
添加了对 REST API 密码套件限制的支持（Gunnar “Nick” Bluth）<br>
它可以通过restapi.ciphers或PATRONI_RESTAPI_CIPHERS环境变量进行配置。<br>
添加了对 REST API 加密 TLS 密钥的支持（Jonathan S. Katz）<br>
它可以通过restapi.keyfile_password或PATRONI_RESTAPI_KEYFILE_PASSWORD环境变量进行配置。<br>
REST API 身份验证凭据的恒定时间比较 (Alex Brasetvik)<br>
使用hmac.compare_digest()而不是==，容易受到定时攻击。<br>
根据复制延迟选择同步节点 (Krishna Sarabu)<br>
如果同步节点上的复制延迟开始超过配置的阈值，则可以将其降级为异步和/或由另一个节点替换。行为受maximum_lag_on_syncnode控制。<br>
稳定性改进<br>
在进行自定义引导时，使用hot_standby=off启动postgres（Igor Yanchenko）<br>
在自定义引导期间，Patroni 正在恢复 basebackup，启动 Postgres，并等待恢复完成。备用数据库上的一些 PostgreSQL 参数不能小于主数据库的参数，如果新值（从 WAL 恢复）高于配置的值，Postgres 会崩溃并停止。为了避免这种行为，我们将在没有hot_standby模式的情况下进行自定义引导。<br>
如果所需的WatchDog不健康，则警告用户 (Nicolas Thauvin)<br>
当WatchDog设备不可写或在required模式下丢失时，无法提升成员。添加了一个警告，以向用户显示在哪里搜索此错误配置。<br>
更详细的单用户模式恢复 (Alexander Kukushkin)<br>
如果 Patroni 注意到 PostgreSQL 没有明确关闭，在某些情况下，崩溃恢复是通过在单用户模式下启动 Postgres 来执行的。可能会发生恢复失败（例如由于磁盘空间不足）但错误被吞下的情况。<br>
添加了与python-consul2模块的兼容性（Alexander、Wilfried Roset）<br>
好东西python-consul几年后就没有维护了，因此有人创建了一个带有新功能和错误修复的分支。<br>
运行patronictl时不使用bypass_api_service（Alexander）<br>
当 K8s pod 在非default命名空间中运行时，它不一定有足够的权限来查询kubernetes端点。在这种情况下，Patroni 会显示警告并忽略bypass_api_service设置。在patronictl警告的情况下有点烦人。<br>
raft.data_dir如果它不存在则创建或确保它是可写的 (Mark Mercado)<br>
提高了用户友好性和可用性。<br>
Bug修复<br>
如果在暂停时失去Leader锁，不要中断重启或升级（亚历山大）<br>
在暂停中，允许在没有锁定的情况下将 postgres 作为主运行。<br>
修复了REST API中的shutdown_request()问题（Nicolas Limage）<br>
为了改进 SSL 连接的处理并延迟握手直到线程启动，Patroni 覆盖了HTTPServer. 该shutdown_request()方法被遗忘了。<br>
修复了使用 Zookeeper时睡眠时间的问题(Alexander)<br>
在运行 HA 代码之间，Patroni 有可能多睡两倍。<br>
修复了os.symlink()引导失败后移动数据目录时的无效调用 (Andrew L'Ecuyer)<br>
如果引导失败 Patroni 正在重命名数据目录、pg_wal 和所有表空间。之后它会更新符号链接，以便文件系统保持一致。由于src和dst参数被交换，符号链接创建失败。<br>
修复了 post_bootstrap() 方法中的错误 (Alexander)<br>
如果没有配置超级用户密码，Patroni 将无法调用post_init脚本，因此整个引导程序都失败了。<br>
修复了备用集群中 pg_rewind 的问题 (Alexander)<br>
如果超级用户名不同于Postgres，则备用集群中的pg_rewin失败，因为连接字符串不包含数据库名称。<br>
仅当使用 Etcd v3 的身份验证显式失败时才退出 (Alexander)<br>
启动时，Patroni 执行 Etcd 集群拓扑的发现，并在必要时进行身份验证。可能会发生其中一个 etcd 服务器不可访问的情况，Patroni 尝试在该服务器上执行身份验证并失败，而不是使用下一个节点重试。<br>
处理psutil cmdline()返回空列表的情况（Alexander）<br>
僵尸进程仍然是postmaster子进程，但它们没有cmdline()<br>
将PATRONI_KUBERNETES_USE_ENDPOINTS环境变量视为布尔值（Alexander）<br>
不这样做会导致无法kubernetes.use_endpoints通过环境禁用。<br>
改进对并发端点更新错误的处理 (Alexander)<br>
Patroni 将显式查询当前端点对象，验证当前 pod 是否仍然持有Leader锁并重复更新。<br>
<b>14.4 Version 2.0.1</b><br>
新功能<br>
如果没有可用的寻呼机，请在patronictl edit-config中使用“更多”作为寻呼机（Pavel Golub）<br>
在Windows上，它将是more.com。除此之外，在requirements.txt中cdiff被更改为ydiff，但为了兼容性，patronictl仍然支持两者。<br>
增加了对raft bind_addr和password的支持（Alexander Kukushkin）<br>
raft.bind_addr在 NAT 后面运行时可能很有用。raft.password启用流量加密（需要该cryptography模块）。<br>
添加了sslpassword连接参数支持 (Kostiantyn Nemchenko)<br>
连接参数是在 PostgreSQL 13 中引入的。<br>
稳定性改进<br>
改变暂停时的行为（Alexander）<br>
bootstrap如果PGDATA目录丢失/为空，Patroni 将不会调用该方法。<br>
Patroni 不会在暂停时因 sysid 不匹配而退出，只会记录警告。<br>
如果 Postgres 未在恢复中运行（接受写入）但 sysid 与初始化密钥不匹配，则节点将不会尝试在暂停模式下获取Leader键。<br>
master_start_timeout执行崩溃恢复时应用(Alexander)<br>
如果 Postgres 在Leader节点上崩溃，Patroni 通过在单用户模式下启动 Postgres 来执行崩溃恢复。在崩溃恢复期间，Leader锁正在更新。如果崩溃恢复没有在master_start_timeout几秒钟内完成，Patroni 将强制停止并释放Leader锁。<br>
从urllib3要求中删除了secure extra（Alexander）<br>
添加它的唯一原因是ipaddress对 python 2.7的依赖。<br>
Bug修复<br>
修复了Kubernetes.update_leader()中的一个bug（Alexander）<br>
当Leader键更新失败时，一个未处理的异常阻止了主的降级。<br>
修复了patronictl使用 RAFT 时挂起的问题（Alexander）<br>
将patronictl与 Patroni 配置一起使用时，应将self_addr添加到partner_addr。<br>
修复了get_guc_value()中的错误（Alexander）<br>
patroni未能在PostgreSQL 12上获取restore_command值，因此获取pg_rewind缺少的WAL不起作用。<br>
<b>14.5 Version 2.0.0</b><br>
此版本增强了与PostgreSQL 13的兼容性，增加了对多个同步备节点的支持，在处理pg_rewind方面有了显著的改进，增加了对纯RAFT上的Etcd v3和Patroni的支持（没有Etcd、Consor或Zookeeper），并且可以选择调用pre_promote（围栏）脚本。<br>
PostgreSQL 13支持<br>
• 在PostgreSQL 13+上升级到standby_leader时不启动on_reload（Alexander Kukushkin）<br>
当升级到standby_leader时，我们会更改primary_conninfo，更新角色并重新加载。由于on_role_change和on_reload实际上是相互重复的，所以Patroni将只调用on_role_change。<br>
• 增加了对gssencmode和channel_binding连接参数的支持（Alexander）<br>
PostgreSQL 12引入了gssencmode和13 channel_binding连接参数，现在如果在PostgreSQL.authentication部分中定义，则可以使用它们。<br>
• 将wal_keep_segments重命名为wal_keep_size（Alexander）<br>
如果出现配置错误（13的wal_keep_segments 和旧版本的wal_keep_size），Patroni将自动调整配置。<br>
• 如果可能的话，在13上使用pg_rewind和--restore-target-wal（Alexander）<br>
在PostgreSQL 13上，用户检查restore_command是否已配置，并告诉pg_rewind使用它。<br>
新功能<br>
• [BETA]在pure RAFT上实施对patroni的支持（Alexander）<br>
这使得运行Patroni无需依赖第三方，如Etcd、Consor或Zookeeper成为可能。对于HA，您必须运行三个patroni节点或两个带有patroni的节点，以及一个带有patronictl的节点。有关更多信息，请查看文档。<br>
• [BETA]通过gPRC-gateway实现对Etcd v3协议的支持（Alexander）<br>
ETCD3.0在四年多前发布，默认情况下ETCD3.4已禁用v2。也有可能从Etcd中完全删除v2，因此我们在patroni中实现了对Etcd v3的支持。为了开始使用它，您必须显式地创建etcd3部分，即patroni配置文件。<br>
• 支持多个同步备节点（Krishna Sarabu）<br>
它允许运行具有多个同步副本的集群。同步副本的最大数量由新参数synchronous_node_count控制。默认情况下，它设置为1，并且在synchronous_mode设置为off时不起作用。<br>
• 增加了调用pre_promote脚本的可能性（Sergey Dudoladov）<br>
与回调不同，pre_promote脚本是在获取leader锁之后，但在升级Postgres之前同步调用的。如果脚本失败或退出时带有非零退出代码，则当前节点将释放leader锁。<br>
• 增加了对配置目录的支持（Floris van Nee）<br>
按字母顺序加载和应用目录中的YAML文件。<br>
• PostgreSQL参数的高级验证（Alexander）<br>
如果当前PostgreSQL版本不支持该特定参数，或者其值不正确，则Patroni将完全删除该参数或尝试修复该值。<br>
• 升级后强制检查点完成时唤醒主线程（Alexander）<br>
副本正在等待通过DCS中leader的ember键指示检查点。密钥通常仅在每个HA循环中更新一次。在不唤醒主线程的情况下，副本的等待时间将比需要的时间长loop_wait秒。<br>
• 在9.6+上使用pg_stat_wal_recevier视图（Alexander）<br>
该视图包含primary_conninfo和primary_slot_name的最新值，而recovery.conf的内容可能已过时。<br>
• 改进了Patroni配置文件中IPv6地址的处理（Mateusz Kowalski）<br>
IPv6地址应该用方括号括起来，但我希望能清楚地表达出来。现在这两种格式都受支持。<br>
• 添加Consul service_tags配置参数（Robert Edström）<br>
它们对于动态服务发现非常有用，例如负载平衡器。<br>
• 为Zookeeper实施SSL支持（Kostiantyn Nemchenko）<br>
它要求kazoo>=2.6.0。<br>
• 自定义引导方法的已实现（无参数）选项（Kostiantyn）<br>
它允许调用wal-g、pgBackRest和其他备份工具，而无需将它们包装到shell脚本中。<br>
• 在失败的初始化之后移动WAL和表空间（Feike Steenbergen）<br>
在执行reinit时，Patroni不仅删除了PGDATA，还删除了符号链接的WAL目录和表空间。现在，move_data_directory()方法将执行类似的工作，即重命名WAL目录和表空间，并更新PGDATA中的符号链接。<br>
改进了pg_rewind支持<br>
• 改进的时间线差异检查（Alexander）<br>
当副本上的重放位置不在切换点之前，或者前一主副本上的检查点记录的结尾与切换点相同时，我们不需要倒带。为了得到检查点记录的结尾，我们使用pg_waldump并解析其输出。<br>
如果pg_rewind抱怨找不到WAL（Alexander）<br>
pg_rewind所需的WAL段可能不再存在于pg_wal目录中，因此pg_rewind无法在分歧点之前找到检查点位置。从PostgreSQL 13开始，pg_rewind可以使用restore_command获取丢失的WAL。对于较旧的PostgreSQL版本，Patroni解析倒带尝试失败的错误，并尝试通过自己调用restore_command来获取丢失的WAL。<br>
• 检测备用集群中的新时间线，并在必要时触发倒带/重新初始化（Alexander）<br>
standby_cluster与主集群分离，因此不会立即知道Leader选举和时间线切换。为了检测事实，standby_leader定期检查pg_wal中的新历史文件。<br>
• 缩短和美化历史日志输出（Alexander）<br>
当Patroni试图找出pg_rewind的必要性时，它可以将主历史文件的内容写入日志。历史记录文件随着每次故障切换/切换而不断增长，最终开始占用太多的行，其中大部分都不是很有用。与显示原始数据不同，Patroni只显示当前副本时间线之前的3行和之后的2行。<br>
K8s的改进<br>
• 摆脱kubernetes python模块（Alexander）<br>
官方的python kubernetes客户端包含大量自动生成的代码，因此非常繁重。Patroni只使用了K8S API端点的一小部分，实现对它们的支持并不困难。<br>
• 使绕过kubernetes服务成为可能（Alexander）<br>
在K8s上运行时，patroni通常通过kubernetes服务与K8s API通信，其地址在KUBERNETES_SERVICE_HOST环境变量中公开。与任何其他服务一样，kubernetes服务由kube-proxy处理，而kube-proxy又根据配置依赖于用户空间程序或iptables进行流量路由。跳过中间组件并直接连接到K8s主节点使我们能够实施更好的重试策略，并在K8s主节点升级时降低Postgres降级的风险。<br>
• 同步一个Patroni集群的所有POD的HA循环（Alexander）<br>
不这样做会将故障检测时间从ttl增加到ttl+loop_wait。<br>
• 在K8s上的子集地址中填充references和nodename（Alexander）<br>
一些load-balancers依赖于此信息。<br>
• 修复update_leader()中可能的争用条件（Alexander）<br>
在Patroni之外同时更新leader configmap或端点可能会导致update_leader（）调用失败。在这种情况下Patroni重新检查当前节点是否仍然拥有leader锁，并重复更新。<br>
• 明确禁止修补non-existent配置（Alexander）<br>
对于kubernetes以外的DCS，由于cluster.config为None，PATCH调用失败，出现异常，但在kubernetes上，它很高兴地创建了配置注释，并阻止在引导完成后写入引导配置。<br>
• 修正暂停时的错误（Alexander）<br>
当缺少leader键时，复制副本正在删除primary_conninfo并重新启动Postgres，但它们不应执行任何操作。<br>
REST API的改进<br>
•延迟TLS握手，直到工作线程启动（Alexander，Ben Harris）<br>
如果TLS握手是在API线程中完成的，并且client-side没有发送任何数据，则API线程将被阻塞（risking DoS）。<br>
•独立于REST API中的客户端证书进行检查basic-auth（Alexander）<br>
以前只验证了客户端证书。独立进行两次检查是绝对有效的（use-case）。<br>
•在选项请求的HTTP头之后写入双CRLF（Sergey Burladyan）<br>
HAProxy对一个CRLF很满意，而Consul health-check则抱怨连接中断和意外的EOF。<br>
•GET/cluster正在显示Zookeeper的过时成员信息（Alexander）<br>
端点使用的是Patroni内部群集视图。对于Patroni本身来说，它不会引起任何问题，但是当我们暴露在外部世界时，我们需要显示最新的信息，尤其是复制延迟。<br>
•修复了备用集群的健康检查（Alexander）<br>
主设备的GET /standby-leader和standby_leader的GET /master错误地响应了200。<br>
•执行DELETE /switchover (Alexander)<br>
REST API调用删除切换的任务计划。<br>
•创建 /readiness就绪和 /liveness端点（Alexander）<br>
当K8s服务与标签选择器一起使用时，它们有助于从子集地址中消除“unhealthy”的POD。<br>
•增强的GET/replica和GET/async RESTAPI 健康检查（Krishna，Alexander）<br>
检查现在支持可选关键字？lag=&lt;max lag&gt;，并且仅当滞后小于提供的值时才会响应200。如果依赖此功能，请记住，关于leader上的WAL位置的信息仅每隔loop_wait秒更新一次！<br>
•在RESTAPI响应中添加了对用户定义的HTTP头的支持（Yogesh Sharma）<br>
如果从浏览器发出请求，此功能可能会很有用。<br>
Patronictl 改进<br>
•不试图在patronictl pause中调用不存在的leader（Alexander）<br>
暂停K8s上没有leader的集群时，patronictl显示无法访问成员“None”的警告。<br>
• 处理成员conn_url丢失的情况（Alexander）<br>
在K8s上，pod可能没有必要的注释，因为Patroni尚未运行。它使patronictl失败了。<br>
• 增加了打印ASCII群集拓扑的能力（Maxim Fedotov，Alexander）<br>
了解级联复制的集群概况非常有用<br>
• 执行patronictl flush switchover（Alexander）<br>
在此之前patronictl flush仅支持取消restart的任务计划。<br>
错误修正<br>
•使用现有PGDATA引导集群时出现属性错误（Krishna）<br>
尝试create/update /history键时，Patroni正在访问尚未在DCS中创建的ClusterConfig对象。<br>
• 改进了Consul中的异常处理（Alexander）<br>
touch_member()方法中未处理的异常导致整个Patroni进程崩溃。<br>
• 为post_init脚本强制执行synchronous_commit=local（Alexander）<br>
在创建用户(replication, rewind)时，Patroni已经在这样做了，但在post_init的情况下没有这样做是一个疏忽。因此，如果脚本没有自己在内部执行，那么synchronous_mode中的引导就无法完成。<br>
• Consul pool manager中的maxsize增加（ponvenkates）<br>
在默认情况下（size=1），会生成一些警告。<br>
• Patroni错误地报告Postgres正在运行（Alexander）<br>
例如，当Postgres由于磁盘不足错误而崩溃时，状态未更新。<br>
• 将*放入pgpass，而不是缺少或空值（Alexander）<br>
例如，如果未指定standby_cluster.port，则生成的pgpass文件不正确。<br>
• 跳过具有特殊角色的leader节点上的物理复制slot创建（Krishna）<br>
当名称包含特殊字符如“-”（例如“abc-us-1”）时，Patroni似乎正在为leader节点创建一个休眠slot（定义slots时）。<br>
• 避免在自定义引导中删除不存在的 pg_hba.conf（Krishna）<br>
如果在自定义引导后pg_hba.conf恰好位于pgdata目录之外，则Patroni失败。<br>
<b>14.6 Version 1.6.5</b><br>
新功能<br>
• 主停止超时（Krishna Sarabu）<br>
停止Postgres时允许Patroni等待的秒数。仅在启用synchronous_mode时有效。当设置为大于0的值且启用synchronous_mode时，如果停止操作的运行时间超过master_stop_timeout设置的值，则Patroni向postmaster发送SIGKILL。根据您的耐久性/可用性权衡设置值。如果参数未设置或设置为非正值，master_stop_timeout不起作用。<br>
• 不使用主服务器的名称创建永久物理slot (Alexander Kukushkin)<br>
当复制副本关闭时，主循环（WAL）段是一个常见问题。现在我们有了一个很好的静态集群解决方案，具有固定数量的节点和永不更改的名称。您只需要列出插槽中所有节点的名称，这样当节点关闭（未在DCS中注册）时，主节点将不会删除slot。<br>
• 配置验证程序初稿（Igor Yanchenko）<br>
使用patroni --validate-config patroni.yaml来验证Patroni配置。<br>
• 配置时间线历史记录最大长度的可能性（Krishna）<br>
Patroni将故障切换/切换的历史记录写入DCS中的/history键。随着时间的推移，这个键的大小会变大，但在大多数情况下，只有最后几行是有趣的。max_timeline_history参数允许指定要在DCS中保留的最大时间线历史项目数。<br>
• Kazoo 2.7.0兼容性（Danyal Prout）<br>
Kazoo的一些non-public方法更改了签名，但Patroni依赖于这些签名。<br>
Patronictl 的改进<br>
• 显示成员标签（Kostiantyn Nemchenko，Alexander）<br>
标记是为每个节点单独配置的，没有简单的方法可以获得它们的概述。<br>
• 提高成员产出（Alexander）<br>
冗余集群名称不再显示在每一行上，只显示在表头中。<br>
• 如果明确指定了配置文件但未找到，则失败（Kaarel Moppel）<br>
以前，patronictl只报告DEBUG消息。<br>
• 解决了未初始化K8s Pod破坏的问题patronictl（Alexander）<br>
Patroni依赖K8s上的某些pod注释。当其中一个Patroni Pod停止或启动时，还没有有效的注释，并且patronictl异常失败。<br>
稳定性改进<br>
• 如果对K8s API服务器的列表调用失败，则应用1秒回退（Alexander）<br>
这主要是为了避免淹没日志，但也有助于防止主线程不足。<br>
• 如果K8s API返回retry-after HTTP头，请重试（Alexander）<br>
如果K8s API服务器的请求太多，它可能会要求重试<br>
• 从postmaster清除KUBERNETES_ 环境变量（Feike Steenbergen）<br>
PostgreSQL不需要KUBERNETES_环境变量，但是将它们公开给postmaster也会将它们公开给后端和常规数据库用户（例如使用pl/perl）。<br>
• 重新初始化时清理表空间（Krishna）<br>
在reinit期间，Patroni只删除PGDATA并保留用户定义表空间目录。这导致了Patroni对其进行控制。以前解决该问题的方法是实现自定义引导脚本。<br>
• 升级发生后显式执行CHECKPOINT（Alexander）<br>
它有助于缩短新主设备可用于pg_rewind之前的时间。<br>
• Etcd成员的智能刷新（Alexander）<br>
如果patroni未能在Etcd集群的所有成员上执行请求，则patroni将re-check A或SRV记录中的IPs/hosts更改，然后重试。<br>
• 跳过pg_controldata中缺失的值（Feike）<br>
尝试使用与PGDATA不匹配的版本的二进制文件时，缺少值。不管怎样，Patroni都会尝试启动Postgres，Postgres会抱怨主版本不匹配，并因错误而中止。<br>
错误修正<br>
• 需要时禁用Consul 的SSL验证（Julien Riou）<br>
从urllib3的某个版本开始，cert_reqs必须显式设置为ssl.cert_NONE，以便有效地禁用ssl验证。<br>
• 避免在HA循环的每个周期上打开复制连接（Alexander）<br>
回归在1.6.4中引入。<br>
• 调用on_role_change失败的主上的回调（Alexander）<br>
在某些情况下，这可能导致虚拟IP仍然连接到旧的主IP。回归在1.4.5中引入。<br>
• 如果postgres在成功pg_rewind后启动，则重置倒带状态（Alexander）<br>
由于这个错误，Patroni在暂停模式下手动启动并关闭postgres。<br>
• 检查recovery.conf时将recovery_min_apply_delay转换为ms<br>
如果在早于12的PostgreSQL上配置了recovery_min_apply_delay，则patroni将无限期地重新启动复制副本。<br>
• PyInstaller兼容性（Alexander）<br>
PyInstaller将Python应用程序冻结打包为独立的可执行文件。当我们切换到spawn方法而不是fork进行多处理时，兼容性被破坏。<br>
<b>14.7 Version 1.6.4</b><br>
新功能<br>
• 为patronictl reinit实现了--wait选项（Igor Yanchenko）<br>
如果Patronictl使用了--wait选项，将等待reinit完成。<br>
• 进一步改进Windows支持（Igor Yanchenko、Alexander Kukushkin）<br>
1.	用于集成测试的所有shell脚本都用python重写<br>
2.	pg_ctl kill将用于在非posix系统上停止postgres<br>
3.	不尝试使用unix域套接字<br>
稳定性改进<br>
• 确保unix_socket_directories和stats_temp_directory存在（Igor）<br>
在启动Patroni和Postgres时，请确保unix_socket_directories和stats_temp_directory存在或尝试创建它们。如果创建失败，则Patroni将退出。<br>
• 确保postgresql.pgpass位于patroni具有写访问权限的位置（Igor）<br>
如果它没有写访问权限，我将异常退出。<br>
• 默认情况下禁用Consul服务健康检查（Kostiantyn Nemchenko）<br>
即使在网络出现小问题的情况下，失败的serfHealth也会导致与节点相关的所有会话失效。因此，leader键的丢失比ttl早得多，这会导致副本的不必要重新启动，并可能导致主降级。<br>
• 为与K8s API的连接配置tcp keepalives（Alexander）<br>
如果我们在TTL秒后没有从socket中获得任何信息，则可以认为它已死亡。<br>
• 避免在用户创建时记录密码（Alexander）<br>
如果密码被拒绝，或者日志被配置为verbose，或者根本没有配置，则可能会将密码写入postgres日志。为了避免这种情况，在尝试创建/更新用户之前，Patroni会将log_statement、log_min_duration_statement和log_min_error_statement更改为一些安全值。<br>
错误修正<br>
• 在级联副本上使用restore_command从standby_cluster配置（Alexander）<br>
standby_leader从功能存在的一开始就已经在这样做了。在副本上不这样做可能会阻止它们赶上备集群leader。<br>
• 由备用群集报告的更新时间线（Alexander）<br>
在时间线切换的情况下，备用群集从主群集正确复制，但patronictl报告的是旧的时间线。<br>
在副本上执行恢复参数验证时，如果在patroni配置中未定义恢复参数，而是在postgresql.auto.conf或postgresql.conf以外的文件中定义恢复参数，则patroni将跳过archive_cleanup_command，promote_trigger_file，recovery_end_command，recovery_min_apply_delay和restore_command。<br>
• 改进postgresql参数的处理，名称中使用句点（Alexander）<br>
这样的参数可以通过扩展来定义，其中单位不一定是字符串。更改该值可能需要重新启动（例如pg_stat_statements.max）。<br>
• 改进shutdown期间的异常处理（Alexander）<br>
shutdown期间，patroni试图更新其在DCS中的状态。如果无法访问DCS，则可能引发异常。缺少异常处理导致记录器线程无法停止。<br>
<b>14.8 Version 1.6.3</b><br>
错误修正<br>
• 运行pg_rewind时不暴露密码（Alexander Kukushkin）<br>
Bug是在#1301中引入的<br>
• 将postgresql.authentication中指定的连接参数应用于pg_basebackup和自定义副本创建方法（Alexander）<br>
它们依赖于url-like连接字符串，因此从未应用参数。<br>
<b>14.9 Version 1.6.2</b><br>
新特性<br>
• 增加patroni --version（Igor Yanchenko）<br>
它打印当前版本的Patenti并退出。<br>
• 为所有http请求设置user-agent http头（Alexander Kukushkin）<br>
Patenti通过http协议与Concur、Etcd和Kubernetes API进行通信。拥有一个专门设计的user-agent（例如：Patroni/1.6.2 Python/3.6.8 Linux）对于调试和监视可能很有用。<br>
• 可以为异常回溯配置日志级别（Igor）<br>
如果你设置log.traceback_level=DEBUG则只有当log.level=DEBUG时，才能看到回溯。<br>
默认行为保持不变。<br>
稳定性改进<br>
• 搜索配置文件所需的模块时，避免导入所有DCS模块（Alexander）<br>
如果我们只需要例如Zookeeper，则无需为Etcd、Consor和Kubernetes导入模块。它有助于减少内存使用，并解决信息消息导入smth失败的问题。<br>
• 从明确需求中删除python requests模块（Alexander）<br>
它没有用于任何重要的用途，但在发布新版本的urllib3时会引起很多问题。<br>
• 改进处理etcd.hosts作为逗号分隔字符串而不是YAML数组写入（Igor）<br>
以前，以格式host1:port1,host2:port2（逗号后的空格字符）编写时失败。<br>
可用性改进<br>
• 不强迫用户从patronictl的空列表中选择成员（Igor）<br>
如果用户提供了错误的集群名称，我们将引发异常，而不是要求从空列表中选择成员。<br>
• 如果REST API无法绑定，则使错误消息更有帮助（Igor）<br>
对于没有经验的用户来说，可能很难从Python 堆栈跟踪中找出问题所在。<br>
错误修正<br>
• 固定计算wal_buffers（Alexander）<br>
PostgreSQL 11中的基本单位已从8KB块更改为字节。<br>
• 仅在PostgreSQL 10+上使用primary_conninfo中的passfile（Alexander）<br>
在旧版本上，除非安装了最新版本的libpq，否则不能保证passfile能够工作。<br>
<b>14.10 Version 1.6.1</b><br>
新特性<br>
• 添加了PATRONICTL_CONFIG_FILE环境变量（msvechla）<br>
它允许环境变量为patronictl提供--config-file参数。<br>
• 添加patronictl history（Alexander Kukushkin）<br>
它显示故障切换/切换的历史记录。<br>
• 执行pg_rewind时在PGOPTIONS中通过-c statement_timeout=0（Alexander Kukushkin）<br>
当服务器上的statement_timeout设置为某个较小的值，并且pg_rewind执行的某个语句被取消时，它可以防止出现这种情况<br>
• 允许较低的PostgreSQL配置值（Soulou）<br>
Patroni不允许将某些PostgreSQL配置参数设置为小于某些固定值。现在允许的最小值更小，默认值没有更改。<br>
• 允许基于证书的身份验证（Jonathan S.Katz）<br>
此功能为超级用户、复制用户和rewind 用户启用基于证书的身份验证，并允许用户指定要连接的sslmode。<br>
• 使用primary_conninfo中的passfile而不是密码（Alexander Kukushkin）<br>
它允许避免在postgresql.conf上设置600个权限<br>
• 执行pg_ctl reload而不考虑配置更改（Alexander Kukushkin）<br>
有些配置文件可能不受patroni控制。当有人通过restapi或向patroni进程发送SIGHUP进行重新加载时，通常是希望Postgres也会被重新加载。以前，当patroni配置的postgresql部分没有更改时，就不会发生这种情况。<br>
• 比较所有恢复参数，而不仅仅是primary_conninfo（Alexander Kukushkin）<br>
以前，check_recovery_conf() 方法只检查primary_conninfo是否已更改，从不考虑所有其他恢复参数。<br>
• 使应用某些恢复参数而无需重新启动成为可能（Alexander Kukushkin）<br>
从PostgreSQL 12开始，无需重新启动即可更改以下恢复参数：archive_cleanup_command, promote_trigger_file, recovery_end_command, and recovery_min_apply_delay。在未来的Postgres发行版中，该列表将被扩展，并且Patroni将自动支持该列表。<br>
• 在线更改use_slots（Alexander Kukushkin）<br>
以前它需要重新启动Patenti并手动移除。<br>
• 启动Postgres时仅删除PATRONI_前缀的环境变量（Cody Coons）<br>
它将解决运行不同的外部数据包装器的许多问题。<br>
稳定性改进<br>
• 使用K8s API时使用LIST + WATCH（Alexander Kukushkin）<br>
它允许有效地接收对象更改（pods, endpoints/configmaps），并减少K8s主节点上的压力。<br>
• 在引导过程中PGDATA不为空时改进工作流（Alexander Kukushkin）<br>
根据initdb源代码，当只存在lost+found和.dotfiles时，它可能会考虑PGDATA空。现在，我也这么做了。如果PGDATA恰好为非空，同时从pg_controldata的角度来看无效，则Patroni将投诉并退出。<br>
• 避免在每个HA循环上调用昂贵的os.listdir()（Alexander Kukushkin）<br>
当系统处于IO压力下时，os.listdir() 可能需要几秒钟（甚至几分钟）才能执行，严重影响了patroni的HA循环。由于缺少更新，这甚至可能导致leader键从DCS中消失。有一种更好、更便宜的方法来检查PGDATA是否为空。现在，我们检查PGDATA中是否存在global/pg_control文件。<br>
• 日志基础设施的一些改进（Alexander Kukushkin）<br>
以前有可能在关机时丢失最后几行日志，因为日志线程是守护进程线程。<br>
• 在Python3.4+上使用spawn多处理启动方法（Maciej Kowalczyk）<br>
Python中的一个已知问题是线程和多处理不能很好地混合。建议从默认方法fork切换到spawn。不这样做可能会导致Postmaster启动流程挂起，并且Patroni会无限期地报告INFO: restarting after failure in progress，而Postgres实际上已启动并运行<br>
REST API的改进<br>
• 使在REST API中检查客户端证书成为可能（Alexander Kukushkin）<br>
如果verify_client设置为required，则Patroni将检查所有REST API调用的客户端证书。将其设置为optional时，将检查所有不安全REST API端点的客户端证书。<br>
• 如果Postgres未运行，则返回GET /replica健康检查请求的响应代码503（Alexander Anikin）<br>
Postgres在开始接受客户端连接之前可能会花费大量时间进行恢复。<br>
• 实现 /history和 /cluster端点（Alexander Kukushkin）<br>
/history端点显示DCS中history键的内容。/cluster端点显示所有集群成员和一些服务信息，如挂起的和计划任务的重新启动或切换。<br>
Etcd支持的改进<br>
• 重试Etcd RAFT内部错误（Alexander Kukushkin）<br>
关闭Etcd节点时，它会发送（response code=300，data='etcdserver:server stopped'），这会导致patroni降级主。<br>
• 不过早放弃Etcd请求重试（Alexander Kukushkin）<br>
当出现一些网络问题时，patroni会很快耗尽Etcd节点列表，并在不使用整个节点的情况下放弃retry_timeout，这可能会导致主节点降级。<br>
错误修正<br>
• 向pg_rewind用户授予执行权限时禁用 synchronous_commit（kremius）<br>
如果引导是使用synchronous_mode_strict:true完成的，GRANT EXECUTE语句由于非同步节点可用而无限期等待。<br>
• 修复python 3.7上的内存泄漏（Alexander Kukushkin）<br>
patroni使用ThreadingMixIn处理REST API请求，默认情况下，python 3.7使线程为每个非守护进程请求生成。<br>
• 修复异步操作中的竞争条件（Alexander Kukushkin）<br>
恢复停止的Postgres的尝试有可能会覆盖condenictl reinit--force。结果是，当basebackup运行时，Patroni试图启动Postgres。<br>
• 修复postmaster_start_time（）方法中的争用条件（Alexander Kukushkin）<br>
如果该方法是从RESTAPI线程执行的，则需要创建一个单独的游标对象。<br>
• 修复了名称包含大写字母的同步待机无法升级的问题（Alexander Kukushkin）<br>
我们将名称转换为小写，因为Postgres在将（application_name）与（synchronous_standby_names）中的值进行比较时也在执行相同的操作。<br>
• 在启动新的回调进程之前，杀死所有子进程（Alexander Kukushkin）<br>
如果不这样做，则很难在bash中实现回调，并最终导致两个回调同时运行的情况。<br>
• 修复“启动失败”问题（Alexander Kukushkin）<br>
在某些情况下，Postgres状态可能设置为“start failed”，尽管Postgres正在启动和运行。<br>
<b>14.11 Version 1.6.0</b><br>
此版本增加了与PostgreSQL 12的兼容性，使得在PostgreSQL 11及更新版本上无需超级用户即可运行pg_rewind，并支持IPv6。<br>
新功能<br>
• Psycopg2已从要求中删除，必须独立安装（Alexander Kukushkin）<br>
从2.8.0开始，psycopg2被分为两个不同的包：psycopg2和psycopg2 binary，可以同时安装到文件系统的同一位置。为了减少依赖问题，我们让用户选择如何安装它。有几个选项可用，请查阅文档。<br>
• 与PostgreSQL 12的兼容性（Alexander Kukushkin）<br>
从PostgreSQL 12开始，不再有recovery.conf，所有以前的恢复参数都转换为GUC。为了防止ALTER SYSTEM SET primary_conninfo或类似情况，patroni将解析postgresql.auto.conf并从中删除所有备用和恢复参数。patroni配置保持向后兼容。例如，尽管restore_command是一个GUC，但仍然可以在postgresql.recovery_conf.restore_command部分中指定它，并且对于postgresql 12，Patroni将把它写入postgresql.conf。<br>
• 在PostgreSQL 11及更高版本上无需超级用户即可使用pg_rewind（Alexander Kukushkin）<br>
如果要使用此功能，请在用户配置文件的postgresql.authentication.rewind部分中定义username和password。对于已经存在的集群，您必须手动创建用户并GRANT EXECUTE对一些函数的权限。您可以在PostgreSQL文档中找到更多详细信息。<br>
• 对副本上的实际primary_conninfo 值和期望primary_conninfo值进行智能比较（Alexander Kukushkin）<br>
当您将一个已经存在的（主备用）集群转换为一个由Patroni管理的集群时，这可能有助于避免副本重新启动。<br>
• IPv6支持（Alexander Kukushkin）<br>
有两个主要问题。Patroni REST API服务仅在0.0.0.0上侦听，api_url和conn_url中使用的IPv6 IP地址没有正确引用。<br>
• Kerberos支持（Ajith Vilas、Alexander Kukushkin）<br>
这使得在Postgres节点之间使用Kerberos身份验证而不是在Patroni配置文件中定义密码成为可能。<br>
• 管理pg_ident.conf (Alexander Kukushkin)<br>
此功能的工作原理与pg_hba.conf类似：如果postgresql.pg_ident是在配置文件或DCS中定义的，则Patroni 会将其值写入pg_ident.conf，但是，如果postgresql.parameters.ident_file是定义的，Patroni 会假定pg_ident是从外部管理的，不会更新文件。<br>
REST API的改进<br>
• 新增 /health端点（Wilfried Roset）<br>
只有在PostgreSQL正在运行时，它才会返回HTTP状态代码<br>
• 添加 /read-only和 /read-write读写端点（Julien Riou）<br>
/read-only端点支持在副本和主副本之间平衡读取。/read-write端点是/primary、/leader和/master的别名。<br>
• 使用SSLContext包装REST API套接字（Julien Riou）<br>
ssl.wrap_socket()的使用已被弃用，并且仍然允许使用像TLS 1.1这样的即将弃用的协议。<br>
日志改进<br>
• Two-step 日志<br>
所有日志消息首先写入内存队列，然后从单独的线程异步刷新到stderr或文件中。最大队列大小有限（可配置）。如果达到限制，Patroni将开始丢失日志，这仍然比阻塞HA循环要好。<br>
• 为GET/OPTIONS API调用和延迟启用调试日志记录（Jan Tomsa）<br>
它将有助于调试HAProxy、Concur或其他决定哪个节点是主/副本的工具执行的运行状况检查。<br>
• 重试时捕获日志异常（Daniel Kucera）<br>
当达到尝试次数或超时，记录最后一个异常。当与DCS的通信失败时，它有望帮助调试一些问题。<br>
Patronictl 改进<br>
• 增强计划切换和重启的对话（Rafia Sabih）<br>
以前的对话没有考虑到预定的行动，因此具有误导性。<br>
• 检查配置文件是否存在（Wilfried Roset）<br>
当给定的文件名不存在时，请详细说明配置文件，而不要默默忽略（这可能导致误解）。<br>
• 为EDITOR添加回退值（Wilfried Roset）<br>
当未定义EDITOR环境变量时，patronictl edit config因PatroniCtlException而失败。新的策略是尝试editor和而不是vi，这应该在大多数系统上都可用。<br>
Consul支持改进<br>
• 允许指定Consul一致性模式（Jan Tomsa）<br>
您可以在此处阅读有关一致性模式的更多信息。<br>
• 在SIGHUP上重新加载Consul配置(Cameron Daniel, Alexander Kukushkin)<br>
当有人更改令牌的值时，它特别有用。<br>
错误修正<br>
• 修复切换/故障切换中的bug（Sharon Thomas）<br>
如果REST API不可访问，并且我们使用DCS作为回退，则变量scheduled_at可能未定义。<br>
• 自定义引导期间在pg_hba.conf中打开对本地主机的信任（Alexander Kukushkin）<br>
以前它只对unix_socket开放，这导致了很多错误：FATAL: no pg_hba.conf<br>
entry for replication connection from host "127.0.0.1", user "replicator"。<br>
• 即使前leader领先，也可以认为同步节点是健康的。（Alexander Kukushkin）<br>
如果主节点失去对DCS的访问，它将以只读方式重新启动Postgres，但其他节点可能仍然可以通过REST API访问旧的主节点。这种情况导致同步待机无法升级，因为旧的主服务器在同步待机之前报告WAL位置。<br>
• 备用群集错误修复（Alexander Kukushkin）<br>
当备用主机无法访问时，可以在备用集群中引导副本，并进行一些其他小修复。<br>
<b>14.12 Version 1.5.6</b><br>
新功能<br>
• 通过一组代理支持etcd集群的工作（Alexander Kukushkin）<br>
etcd集群可能无法直接访问，而是通过一组代理进行访问。在这种情况下，Patroni将不执行etcd拓扑发现，而只是通过代理主机进行循环。行为由（etcd.use_proxies）控制。<br>
• 更改节点上的角色时的回调行为（Alexander）<br>
如果角色已从主节点角色或备用集群leader角色更改为备节点角色或从备节点角色更改为备用集群leader角色，则将不再调用on_restart callback以支持on_role_change callback。<br>
• 改变我们开始postgres的方式（Alexander）<br>
使用multiprocessing.Process而不是执行自身和multiprocessing.Pipe将postmaster pid传输到patroni进程。在我们使用管道之前，是什么让postmaster进程关闭了stdin。<br>
错误修复<br>
• REST API为备用集群leader返回的修复角色（Alexander）<br>
它错误地返回了replica，而不是standby_leader<br>
• 如果无法终止，请等待回调结束（Julien Tachoires）<br>
Patroni没有足够的权限终止在sudo what下运行的取消新回调的回调脚本。如果无法终止正在运行的脚本，patroni将等待它完成，然后运行下一个回调。<br>
• 通过dcs.get_cluster方法减少锁定时间（Alexander）<br>
由于锁定被保持，DCS的慢度会影响REST API运行状况检查，从而导致误报。<br>
• 当pg_wal/pg_xlog是符号链接时，改进PGDATA的清理（Julien）<br>
在这种情况下，patroni将显式地从目标目录中删除文件。<br>
• 删除 os.path.relpath不必要的用法 (Ants Aasma)<br>
这取决于是否能够解析工作目录，如果在一个后来与文件系统断开链接的目录中启动了Patroni，那么将失败什么。<br>
• 与Etcd通信时不强制执行ssl版本（Alexander）<br>
由于一些未知的原因，debian和ubuntu上的python3 etcd不基于最新版本的软件包，因此它强制执行etcd v3不支持的TLSv1。我们在Patroni方面解决了这个问题。<br>
<b>14.13 Version 1.5.5</b><br>
这个版本引入了自动重新安装前主节点的可能性，改进了patronictl list输出并修复了一些错误。<br>
新功能<br>
•添加对PATRONI_ETCD_PROTOCOL、PATRONI_ETCD_USERNAME和PATRONI_ETCD_PASSWORD环境变量的支持（Étienne M）<br>
在此之前，只能在配置文件中配置它们，或将其作为PATRONI_ETCD_URL的一部分进行配置，这并不总是方便的。<br>
• 可以自动控制前主节点（Alexander Kukushkin）<br>
如果pg_rewind被禁用或无法使用，则由于时间线不一致，以前的主节点可能无法作为新副本启动。在这种情况下，修复它的唯一方法是删除数据目录并重新初始化。可以通过设置postgresql.remove_data_directory_on_diverged_timelines来更改此行为。设置后，Patroni将删除数据目录，并自动重新初始化前主节点目录。<br>
• 在用户列表中显示有关时间线的信息（Alexander）<br>
它有助于检测过时的副本。除此之外，如果端口值不是默认值，或者在同一主机上运行多个成员，则主机将包括“：{port}”。<br>
• 创建与 $SCOPE-config端点关联的无头服务（Alexander）<br>
“config”端点保存有关集群范围的Patroni和Postgres配置、历史文件的信息，最后但最重要的是，它保存着initialize键。重新启动或升级Kubernetes主节点时，它会删除没有服务的端点。无头服务将防止其被移除。<br>
错误修复<br>
• 调整leader watch阻塞查询的读取超时（Alexander）<br>
根据Consul文档，实际响应超时会增加一小部分随机的额外等待时间，并将其添加到提供的最大等待时间中，以分散任何并发请求的唤醒时间。它加起来等于最大持续时间的等待/16额外时间。在我们的例子中，我们添加了wait/15或1秒，这取决于哪个更大。<br>
• 不为wal-only备用集群将primary_conninfo写入recovery.conf（Alexander）<br>
尽管在standby_cluster配置中既没有定义主机也没有定义端口，但Patroni将primary_conninfo放入了recovery.conf中，这是无用的，并且会产生很多错误。<br>
<b>14.14 Version 1.5.4</b><br>
此版本实现了灵活的日志记录，并修复了许多错误<br>
新功能<br>
• 改进日志基础（Alexander Kukushkin、Lucas Capistrant、Alexander Anikin）<br>
日志配置不仅可以从环境变量配置，还可以从Patroni配置文件配置。它可以在运行时通过更新配置并重新加载或向patroni进程发送消息来更改日志记录配置。默认情况下，Patroni将日志写入stderr，但现在可以将日志直接写入文件，并在文件达到一定大小时进行轮转。除此之外，还增加了对自定义dateformat的支持，以及对每个python模块的日志级别进行微调的可能性。<br>
• leader选举期间考虑当前的时间表（Alexander Kukushkin）<br>
节点可能认为自己是最健康的节点，尽管它目前不在最新的已知时间线上。在某些情况下，我们希望避免升级此类节点，这可以通过将check_timeline参数设置为true来实现（默认行为保持不变）。<br>
• 放宽对超级用户凭据的要求<br>
Libpq允许在不明确指定用户名和密码的情况下打开连接。根据情况，它依赖于pg_hba.conf中的pgpass文件或信任身份验证方法。由于pg_rewind也使用libpq，因此其工作方式相同。<br>
• 实现了通过环境变量配置Consul 服务注册和检查间隔的可能性（Alexander Kukushkin）<br>
1.5.0版中增加了Consul服务注册，但到目前为止，只能通过Patroni.yaml启用。<br>
稳定性改进<br>
• 自定义引导过程中将archive_mode 设置为off（Alexander Kukushkin）<br>
我们希望避免归档WAL和历史文件，直到集群完全正常运行。如果自定义引导涉及pg_upgrade，那么它真的很有帮助。<br>
• 在启动时加载全局配置时应用5秒后退（Alexander Kukushkin）<br>
这有助于在Patroni刚启动时避免锤击（DCS）。<br>
• 减少关机时生成的错误消息量（Alexander Kukushkin）<br>
他们是无害的，但相当烦人，有时还很吓人。<br>
• 在创建时显式保护recovery.conf的rw perms（Lucas）<br>
我们不希望任何人patroni/postgres用户读取此文件，因为它包含复制用户和密码。<br>
• 将HTTPServer异常重定向到记录器（Julien Riou）<br>
默认情况下，这些异常记录在标准输出上，与常规日志混淆。<br>
错误修复<br>
• 已去除pg_ctl程序通向标准装置的标准管道（Cody Coons）<br>
从主Patroni进程继承stderr允许查看所有Postgres日志以及所有用户日志。这在容器环境中非常有用，因为patroni和Postgres日志可以使用标准工具（docker日志、kubectl等）使用。除此之外，此更改还修复了一个错误，即当postgres在stderr中写入一些警告时，Patroni无法捕获postmaster pid。<br>
• 以Go time格式设置Consul服务检查注销超时（Pavel Kirillov）<br>
没有明确提到时间单位注册失败。<br>
• 放松对备用集群配置的检查（Dmitry Dolgov，Alexander Kukushkin）<br>
它只接受字符串作为有效值，因此无法将端口指定为整数并将create_replica_methods创建为列表。<br>
<b>14.15 Version 1.5.3</b><br>
兼容性和错误修复版本。<br>
• 提高python3对抗zookeeper时的稳定性（Alexander Kukushkin）<br>
loop_wait的改变导致Patroni与zookeeper断开连接，再也无法重新连接。<br>
• 修复与postgres 9.3不兼容的问题（Alexander）<br>
打开复制连接时，我们应该指定replication=1，因为9.3不理解replication='database'。<br>
• 确保每个HA循环至少刷新一次Consul会话，并改进Consul会话异常的处理（Alexander）<br>
重新启动本地consul代理将使与节点相关的所有会话无效。未按时调用会话刷新以及未正确处理会话错误导致主节点降级。<br>
<b>14.16 Version 1.5.2</b><br>
兼容性和错误修复版本。<br>
• 与kazoo-2.6.0的兼容性（Alexander Kukushkin）<br>
为了确保在适当的超时情况下执行请求，Patroni从python kazoo模块重新定义了create_connection方法。kazoo的上一个版本稍微改变了create_connection方法的调用方式。<br>
• 修复Consul集群失去leader时的Patroni崩溃（Alexander）<br>
崩溃是由于touch_member方法的不正确实现造成的，它应该返回布尔值，并且不会引发任何异常。<br>
<b>14.17 Version 1.5.1</b><br>
此版本实现了对永久复制插槽的支持，添加了对PGA的支持，并修复了大量错误。<br>
新功能<br>
• 永久复制slots（Alexander Kukushkin）<br>
永久复制slots在故障切换/切换时保留，也就是说，新主服务器上的Patroni将在执行升级后立即创建已配置的复制slots。slots可以在patronictl edit-config的帮助下进行配置。初始配置也可以在bootstrap.dcs中完成。<br>
• 增加pgbackrest支持（Yogesh Sharma）<br>
pgBackrest可以在现有$PGDATA文件夹中恢复，这允许快速恢复自上次备份以来未更改的文件，为支持此功能，引入了新参数keep_data。有关其他示例，请参见副本创建方法一节。<br>
错误修复<br>
• “standby cluster”工作流中的一些错误修复（Alexander）<br>
请看https://github.com/zalando/patroni/pull/823更多细节。<br>
• 修复群集管理暂停且无法访问DCS时的REST API运行状况检查（Alexander）<br>
回归引入的 https://github.com/zalando/patroni/commit/90cf930036a9d5249265af15d2b787ec7517cf57<br>
<b>14.18 Version 1.5.0</b><br>
此版本使Patroni HA cluster能够在备用集群模式下运行，引入了在Windows上运行的实验性支持，并提供了一个新的配置参数以在Consul中注册PostgreSQL服务。<br>
新功能<br>
• 备用集群（Dmitry Dolgov）<br>
一个或多个用户节点可以形成一个备用集群，该集群与主集群（即在另一个数据中心）并行运行，并由从主集群中复制的副本组成。备用集群中的所有PostgreSQL节点都是副本；其中一个副本选择自己直接从远程主节点进行复制，而其他副本则以级联方式从远程主机进行复制。有关此功能的更详细描述和一些配置示例，请参见此处。<br>
• Consul注册服务（Pavel Kirillov, Alexander Kukushkin）<br>
如果启用了Consul配置中的register_service参数，则节点将使用名称scope和标记master、replica或standby-leader注册服务<br>
• 实验性Windows支持（Pavel Golub）<br>
从现在起，在Windows上运行Patroni是可能的，尽管Windows支持是全新的，并且没有得到像Linux那样多的实际测试。我们欢迎您的反馈！<br>
Patronictl 的改进<br>
• 添加patronictl -k/–insecure标志和对restapi证书的支持（Wilfried Roset）<br>
在过去，如果REST API受到自签名证书的保护，则patronictl将无法验证这些证书。无法禁用该验证。现在可以将patronictl配置为完全跳过证书验证，或者在配置的ctl:部分中提供CA和客户端证书。<br>
• 从patronictl切换/故障转移输出中排除具有nofailover标记的成员（Alexander Anikin）<br>
以前，在通过patronictl执行交互式切换或故障切换时，这些成员被错误地推荐为候选成员。<br>
稳定性改进<br>
• 避免解析pg_controldata的non-key-value输出行（Alexander Anikin）<br>
在某些情况下pg_controldata输出不带冒号字符的行。这将在解析pg_controldata输出的用户代码中触发错误，隐藏实际问题；通常，在常规输出之前pg_controldata显示的警告中会发出此类行，即当二进制主版本与PostgreSQL数据目录不匹配时。<br>
• 在leader选举期间向错误消息中添加成员名称（Jan Mussler）<br>
在leader选举期间，Patroni连接到集群的所有已知成员并请求其状态。该状态会写入用户日志，并包括成员名。以前，如果无法访问该成员，则错误消息不会指示其名称，只包含URL。<br>
• 创建复制slot后立即保留WAL位置（Alexander Kukushkin）<br>
从9.6开始，pg_create_physical_replication_slot函数提供了一个额外的布尔参数immediately_reserve。当设置为false这也是默认值时，slot在收到第一个客户端连接之前不会保留WAL位置，这可能会在插槽创建和初始客户端连接之间的时间窗口中丢失客户端所需的某些段。<br>
• 修复严格同步复制中的错误（Alexander Kukushkin）<br>
在使用synchronous_mode_strict: true运行时，在某些情况下，Patroni会将*intosynchronous_standby_names，从而将大多数复制连接的同步状态更改为潜在状态。以前，Patroni无法在这种curcuimstances下选择同步候选者，因为它只考虑状态为async的候选者。<br>
<b>14.19 Version 1.4.6</b><br>
Bug修复和稳定性改进<br>
此版本修复了非主节点的Patroni API /master终端端返回200的关键问题。这是一个报告问题，没有真正的发生脑裂，但在某些情况下，客户端可能被定向到只读节点。<br>
• 重置is_leader状态为降级（Alexander Kukushkin，Olii Kliuk）<br>
确保降级的集群成员停止响应/master API调用中的代码200。<br>
• 将新的“cluster_unlocked”字段添加到API输出（Dmitry Dolgov）<br>
此字段指示集群是否正在运行主节点。当除了其中一个复制副本之外，无法查询任何其他节点时，可以使用该方法。<br>
<b>14.20 Version 1.4.5</b><br>
新功能<br>
• 在应用新的postgres配置时改进日志记录（Don Seiler）<br>
Patroni记录更改的参数名称和值。<br>
• Python 3.7兼容性（Christoph Berg）<br>
async是python3.7中的保留关键字。<br>
• 当一个成员关闭时，在DCS中将状态设置为“stopped”（Tony Sorrentino）<br>
这将在“patronictl list”命令中将成员状态显示为“stopped”。<br>
• 改进stale postmaster.pid与正在运行的进程匹配时记录的消息（Ants Aasma）<br>
前一个是无可置疑的。<br>
• 实现patronictl reload功能（Don Seiler）<br>
在此之前，只能通过调用REST API或向用户进程发送SIGHUP信号来重新加载配置。<br>
• 作为副本启动时，从controldata获取并应用一些参数（Alexander Kukushkin）<br>
在全局配置中设置的max_connections和一些其他参数的值可能低于主节点实际使用的值；发生这种情况时，复制副本无法启动，应手动修复。Patroni现在通过读取和应用pg_controldata中的值、启动postgres并设置pending_restart标志来解决这个问题。<br>
• 如果已设置，则在启动postgres时使用LD_LIBRARY_PATH（Chris Fraser）<br>
当启动Postgres时，Patroni正在通过PATH，LC_ALL和LANG env Var如果已设置。现在它对LD_LIBRARY_PATH执行相同的操作。如果有人将PostgreSQL安装到非标准位置，应该会有所帮助。<br>
• 将create_replica_method重命名为create_replica_methods（Dmitry Dolgov）<br>
为了清楚地表明它实际上是一个数组。为了向后兼容，仍然支持旧名称。<br>
Bug修复和稳定性改进<br>
• 修复由于pg_rewind处于暂停状态而导致副本启动的条件（Oleksii Kliukin）<br>
避免启动之前已执行pg_rewind的复制副本。<br>
• 仅当更新锁定成功时才对主运行状况检查作出响应（Alexander）<br>
如果DC已分区，防止patroni在前（降级）主节点报告自己是主。<br>
• 修复与新consul模块的兼容性（Alexander）<br>
从v1.1.0开始，python-consul更改了内部API，并开始使用list而不是dict来传递查询参数。<br>
• 在关机期间从Patroni REST API线程捕获异常（Alexander）<br>
这些未捕获的异常使PostgreSQL在关闭时保持运行。<br>
• 只有在Postgres以主节点身份运行时才执行崩溃恢复（Alexander）<br>
要求pg_controldata报告“生产中”或“关闭”或“故障恢复中”。在所有其他情况下，无需进行故障恢复。<br>
• 改进配置错误的处理（Henning Jacobs，Alexander）<br>
通过更新Patroni配置文件并向patroni进程发送SIGHUP，可以在运行时更改许多参数（包括restapi.listen）。当某些参数接收到无效值时，此修复程序消除了“restapi”线程中的模糊异常。<br>
<b>14.21 Version 1.4.4</b><br>
稳定性改进<br>
• 修正 poll_failover_result 中的竞争条件（Alexander Kukushkin）<br>
它既不直接影响故障切换，也不直接影响切换，但在一些罕见的情况下，它过早地报告成功，当前任leader释放锁时，会生成一条Failed over to “None”消息，而不是Failed over to “desired-node”消息。<br>
• 将Postgres参数名称视为不区分大小写（Alexander）<br>
大多数Postgres参数都有snake_case名称，但有三个例外：DateStyle、IntervalStyle和TimeZone。Postgres在不同情况下写入时接受这些参数（例如。e.g.timezone = ‘some/tzn’）；但是，patroni无法在pg_settings中找到这些参数名称的不区分大小写的匹配项，因此忽略了这些参数。<br>
• 如果连接到正在运行的postgres且集群未初始化，则中止启动（Alexander）<br>
Patroni可以将自己连接到已经运行的Postgres实例。在访问复制副本之前，必须在主节点上开始运行Patroni。<br>
• 修复 patronictl scaffold的行为（Alexander）<br>
将dict对象传递给touch_member而不是json编码的字符串，DCS实现将负责对其进行编码。<br>
• 如果未能在暂停中更新leader键，则不降级主（Alexander）<br>
在维护过程中，DCS可能会在继续响应读取请求的同时开始失败写入请求。在这种情况下，在无法更新DCS中的Leader锁后，Patroni通常将Postgres主节点置于read-only模式。<br>
• Patroni注意到新的postmaster流程时同步复制slots（Alexander）<br>
如果Postgres已重新启动，则patroni必须确保复制slots列表符合其预期。<br>
• 在暂停结束后验证sysid和同步复制slots（Alexander）<br>
在维护模式期间，可能会发生数据目录被完全重写的情况，因此我们必须确保数据库系统标识符仍然属于我们的集群，并且复制插槽与patroni的期望同步。<br>
• 修复在存在postmaster锁文件的数据目录上不运行Postgres的可能故障（Alexander）<br>
检测postmaster锁定文件中PID的重用。如果在docker容器中运行Patroni和Postgres，则更可能遇到此类问题。<br>
• 改进DCS意外擦除的保护（Alexander）<br>
在这种情况下，Patroni有很多逻辑来防止故障转移；它还可以恢复所有的键；然而，在此更改之前，/config键的意外删除是关闭暂停模式1个HA循环。<br>
• 遇到无效的系统ID时不要退出（Oleksii Kliukin）<br>
当集群系统ID为空或未通过验证检查时，请勿退出。在这种情况下，集群很可能需要一个reinit；在结果消息中提及它。避免终止patroni，否则它不会发生。<br>
与Kubernetes 1.10的兼容性+<br>
• 增加了空 subsets检查（Cody Coons）<br>
Kubernetes 1.10.0+开始返回Endpoints.Subset，并将其设置为None，而不是[]<br>
Bootstrap改进<br>
• 将删除recovery.conf设置为可选（Brad Nicholson）<br>
如果bootstrap.<custom_bootstrap_method_name>.keep_existing_recovery_conf已定义并设置为True，则patroni不会删除现有recovery.conf文件。这在使用pgBackRest等工具从备份中引导时非常有用，这些工具可以为您生成适当的recovery.conf。<br>
• 允许basebackup内置方法的选项（Oleksii）<br>
现在可以通过在配置中定义basebackup部分为内置basebackup方法提供选项，类似于为自定义副本创建方法定义这些选项的方式。不同之处在于basebackup部分接受的格式：由于pg_basebackup同时接受–key=value和–key选项，因此该部分的内容可以是键值对的字典，或者是一个元素字典的列表，或者只是键（对于不接受值的选项）。有关其他示例，请参见副本创建方法一节。<br>
<b>14.22 Version 1.4.3</b><br>
日志改进<br>
• 从环境变量中配置日志级别（Andy Newton、Keyvan Hedayati）<br>
PATRONI_LOGLEVEL-设置一般日志级别PATRONI_REQUESTS_LOGLEVEL-设置所有HTTP请求的日志级别，例如Kubernetes API调用请参阅Python日志记录文档<br>
<https://docs.python.org/3.6/library/logging.html#levels>获取可能的日志级别的名称<br>
稳定性改进和错误修复<br>
• 观察超时时不重新发现etcd集群拓扑（Alexander Kukushkin）<br>
如果我们在etcd配置中只有一台主机，而实际上该主机不可访问，那么Patroni就开始发现集群拓扑，但从未成功过。相反，它应该只切换到下一个可用节点。<br>
• 在自定义引导后将bootstrap.pg_hba的内容写入pg_hba.conf（Alexander）<br>
现在，它的行为类似于initdb的常规引导<br>
• 单用户模式正在等待用户输入，但从未完成（Alexander）<br>
引入的https://github.com/zalando/patroni/pull/576<br>
<b>14.23 Version 1.4.2</b><br>
Patronictl 改进<br>
• 将计划failover 重命名为计划switchover （Alexander Kukushkin）<br>
failover 和 switchover 功能在版本1.4中是分开的，但patronictl list仍然报告计划的failover ，而不是计划switchover。<br>
• 显示有关挂起的重新启动的信息（Alexander）<br>
为了应用某些配置更改，有时需要重新启动postgres。在REST API中以及在将节点状态写入DCS时，Patroni已经给出了关于这一点的提示，但是没有简单的方法来显示它。<br>
• 使show-config与配置文件中的cluster_name一起工作（Alexander）<br>
它的工作原理与patronictl edit-config类似<br>
稳定性改进<br>
• 避免在引导过程中调用pg_controldata（Alexander）<br>
在initdb或自定义引导期间，当pgdata不为空但pg_controldata尚未写入时，会有一个时间窗口。在这种情况下，pg_controldata调用失败并显示错误消息。<br>
• 处理从psutil引发的异常（Alexander）<br>
每次调用cmdline ()方法时，都会读取和分析cmdline。可能正在检查的进程已经消失，在这种情况下NoSuchProcess被提出。<br>
Kubernetes支持改进<br>
• 不接受来自k8s API的错误（Alexander）<br>
对Kubernetes API的调用可能会由于不同的原因而失败。在某些情况下，应该重试此类调用，在其他一些情况下，我们应该记录错误消息和异常堆栈跟踪。此处的更改将有助于调试Kubernetes权限问题。<br>
• 更新Kubernetes示例Dockerfile以从主分支安装Patroni（Maciej Szulik）<br>
在此之前，它使用的是feature/k8s，这已经过时了。<br>
• 添加适当的RBAC以在k8s上运行patroni（Maciej）<br>
添加分配给集群POD的服务帐户、仅持有必要权限的角色以及连接服务帐户和角色的rolebinding。<br>
<b>14.24 Version 1.4.1</b><br>
修复patronictl<br>
• 在要故障切换到的成员，建议列表中不显示当前的leader。(Alexander Kukushkin)<br>
当集群中有leader时，patronictl failover仍然可以工作，并且应该将其从可以故障切换到的成员列表中排除。<br>
• 使patronictl switchover与旧Patroni api兼容（Alexander）<br>
若POST/SWITCHOVERT REST API调用失败，状态代码为501，则它将再次执行该操作，但会对/failover端点执行该操作。<br>
<b>14.25 Version 1.4</b><br>
    此版本增加了对使用 Kubernetes 作为 DCS 的支持，允许在 Kubernetes 中将 Patroni 作为云原生代理运行，而无需额外部署 Etcd、Zookeeper 或 Consul。<br>
升级通知<br>
    通过 pip 安装 Patroni 将不再引入依赖项（例如 Etcd、Zookeper、Consul 或 Kubernetes 的库，或对 AWS 的支持）。为了启用它们，您需要在 pip install 命令中明确列出它们，例如pip install pausei[etcd,kubernetes]。<br>
Kubernetes 支持<br>
实施基于 Kubernetes 的 DCS。端点元数据用于存储配置和leader 键。pods 定义中的元数据字段用于存储与成员相关的数据。除了使用 端点之外，Patroni 还支持 ConfigMaps。您可以[在文档]<br>
(https://patroni.readthedocs.io/en/latest/kubernetes.html#kubernetes)的[Kubernetes 章节中]<br>
(https://patroni.readthedocs.io/en/latest/kubernetes.html#kubernetes)找到有关此功能的更多信息<br>
稳定性改进<br>
- 将 postmaster 进程分解为一个单独的对象（Ants Aasma）<br>
    该对象通过 pid 和启动时间识别正在运行的 postmaster 进程，并简化了对 postmaster 在我们背后重新启动或 postgres 目录从文件系统中消失的情况的检测（和解决）。<br>
- 最大限度地减少 Patroni 在 HA 循环的每个循环中发出的 SELECT 数量（Alexander Kukushkin）<br>
      在 HA 循环的每次迭代中，Patroni 需要知道恢复状态和绝对 wal 位置。从现在开始，Patroni 将只运行单个 SELECT 来获取此信息，而不是在副本上运行两个，在主服务器上运行三个。<br>
- 仅当我们有锁时才在关机时移除Leader键（Ants）<br>
无条件删除会产生不必要的和误导性的例外情况。<br>
保护的改进<br>
- 将版本命令添加到patronictl (Ants)<br>
    它将显示已安装的 Patroni 的版本和运行的 Patroni 实例的版本（如果指定了集群名称）。<br>
- 为某些守护程序命令设置可选的指定 cluster_name 参数（Alexander，Ants）<br>
    如果patronictl 使用带有`scope`定义的通常的Patroni 配置文件，它将起作用。<br>
- 显示有关计划switchover和维护模式的信息 (Alexander)<br>
    在此之前，只能从 Patroni 日志或直接从 DCS 获取此信息。<br>
- 改进`patronictl reinit`（Alexander）<br>
    有时在 Patroni 忙于其他操作时拒绝继续，即尝试启动 postgres。switchover 没有提供任何命令来取消这种长时间运行的操作，唯一的（危险的）解决方法是手动删除数据目录。reinit的新实现在继续 reinit 之前强制取消其他长时间运行的操作。<br>
-实现——patronictl pause和patronictl resume中的--wait标志（Alexander）<br>
    它将`patronictl`等待直到请求的操作被集群中的所有节点确认。这种行为是通过`pause`为 DCS 中的每个节点以及通过 REST API公开标志来实现的。<br>
- 将`patronictl failover`重命名为`patronictl switchover`（Alexander）<br>
    前者`failover`实际上只能做一个切换；它拒绝在没有leader的情况下继续在集群中运行。<br>
- 改变`patronictl failover` 的行为（Alexander）<br>
    即使没有Leader，它也能工作，但在这种情况下，您必须明确指定一个应该成为新Leader的节点。<br>
公开有关时间线和历史的信息<br>
- 在 DCS 中和通过 API 公开当前时间线 (Alexander)<br>
    存储有关集群每个成员的当前时间线的信息。此信息可通过 API 访问并存储在 DCS 中<br>
- 在 DCS的 /history 键中存储提升历史 (Alexander) <br>
  此外，在DCS的/history键中存储包含相应升级时间戳的时间线历史记录，并在每次提升时更新。<br>
添加端点以获取同步和异步副本<br>
- 添加新的 /sync 和 /async 端点（Alexander、Oleksii Kliukin）<br>
    这些端点（也可以作为 /synchronous 和 /asynchronous 访问）仅针对同步和异步副本相应地返回 200 （不包括标记为noloadbalance 的那些）。<br>
允许 Etcd 的多个主机<br>
- 向 Etcd 配置添加新的hosts参数 (Alexander) <br>
    此参数应包含将用于发现和填充正在运行的 etcd 集群成员列表的hosts 的初始列表。如果在工作期间由于某种原因发现的hosts 列表已用完（该列表中没有可用的主机），Patroni 将从hosts参数返回到初始列表。<br>
<b>14.26 Version 1.3.6</b><br>
稳定性改进<br>
- 检查 postgres 是否正在运行时验证进程启动时间。（Ants Aasma）<br>
    在没有清理 postmaster.pid 的崩溃之后，可能会有一个具有相同 pid 的新进程，导致 is_running() 的误报，这将导致各种不良行为。<br>
- 当我们丢失数据目录时，在引导之前关闭 postgresql (ainlolcat)<br>
    当主节点上的数据目录被强制删除时，postgres进程仍可以保持活动状态一段时间，并防止在该主节点上创建的副本启动或复制。修复程序使patroni缓存postmaster pid及其开始时间，并允许它终止旧的postmaster，以防在删除相应的数据目录后它仍在运行。<br>
- 如果 postgres 主节点死亡，则在单用户模式下执行故障恢复 (Alexander Kukushkin)<br>
    如果postgres没有完全关闭，立即作为备启动是不安全的，并且不可能运行`pg_rewind`。单用户故障恢复仅在`pg_rewind`已启用或目前没有主时生效。<br>
Consul改进<br>
- 使为 Consul 提供数据中心配置成为可能（Vilius Okockis，Alexander）<br>
    在此之前，Patroni 始终与其运行的主机的数据中心进行通信。<br>
- 始终在 X-Consul-Token http 标头中发送令牌（Alexander）<br>
    如果`consul.token`在 Patroni 配置中定义，我们将始终在“X-Consul-Token”http 标头中发送它。python-consul 模块尝试与 Consul REST API“一致”，它不接受令牌作为[会话 API](https://www.consul.io/api/session.html)的查询参数，但它仍然适用于“X-Consul-Token”标头。<br>
- 如果提供的值小于可能的最小值，则调整会话 TTL（Stas Fomin，Alexander）<br>
    Patroni 配置中提供的 TTL 可能小于 Consul 支持的最小值。在这种情况下，Consul 代理无法创建新会话。没有会话 Patroni 无法在 Consul KV 存储中创建成员和Leader 键，从而导致集群不健康。<br>
其他改进<br>
- 通过环境变量`PATRONI_LOGFORMAT`定义自定义日志格式（Stas）<br>
    如果系统logger已添加时间戳和 Patroni 日志中的其他类似字段，则允许禁用它们（通常在 Patroni 作为服务运行时）。<br>
<b>14.27 Version 1.3.5</b><br>
错误修正<br>
- 如果删除了数据目录，则将角色设置为“uninitialized”（Alexander Kukushkin）<br>
    如果节点作为主节点运行，它会阻止故障转移。<br>
稳定性提升<br>
- 如果我们尝试启动 postgres 并失败，请尝试在单用户模式下运行 postmaster (Alexander)<br>
    通常，当作为主节点运行的节点终止并且时间线出现分歧时，会发生此类问题。如果recovery.conf定义了restore_command，postgres很有可能中止启动并保持controldata不变。这使得无法使用pg_rewind，这需要一个干净的shutdown。<br>
领事改进<br>
- 可以在创建会话时指定健康检查 (Alexander)<br>
    如果未指定，Consul 将使用“serfHealth”。一方面，它允许快速检测孤立的主节点，但另一方面，它使 Patroni 无法容忍短暂的网络延迟。<br>
错误修正<br>
- 修复 Python 3 上的WatchDog（Ants Aasma）<br>
    对 ioctl() 调用接口的误解。如果 mutable=False 那么 fcntl.ioctl() 实际上返回 arg 缓冲区。这意外地在 Python2 上起作用，因为 int 和 str 比较没有返回错误。错误报告实际上是通过在 Python2 上引发 IOError 和在 Python3 上引发 OSError 来完成的。<br>
<b>14.28 Version 1.3.4</b><br>
不同的 Consul 改进<br>
- 将consul令牌作为标头传递（Andrew Colin Kissa）<br>
标头现在是将令牌传递给 <br>
consul [API](https://www.consul.io/api/index.html#authentication)的首选方式。<br>
- Consul的高级配置 (Alexander Kukushkin) <br>
指定`scheme`、`token`客户端和 CA 证书<br>
[详细信息的]<br>
(https://patroni.readthedocs.io/en/latest/SETTINGS.html#consul-settings)可能性。<br>
- 与 python-consul-0.7.1 及更高版本的兼容性（Alexander）<br>
    新的 python-consul 模块改变了一些方法的签名<br>
- “Could not take out TTL lock”消息从未被记录（Alexander）<br>
    不是一个严重的错误，但缺乏适当的日志记录会使出现问题时的调查复杂化。<br>
使用 quote_ident 引用 synchronous_standby_names<br>
- 将`synchronous_standby_names`写入`postgresql.conf`其值必须加引号（Alexander）<br>
    如果没有正确引用，PostgreSQL 将有效禁用同步复制并继续工作。<br>
围绕暂停状态的不同错误修正，主要与WatchDog有关（Alexander）<br>
- 如果WatchDog不活动，则不要发送保活<br>
- 避免在暂停模式下激活WatchDog<br>
- 在暂停模式下设置正确的 postgres 状态<br>
- 如果 postgres 停止，不要尝试从 API 运行查询<br>
<b>14.29 Version 1.3.3</b><br>
Bug修复<br>
- 即使启用了 synchronous_mode_strict，在升级后不久同步复制也被禁用 (Alexander Kukushkin)<br>
- `pg_ident.conf`如果从备份恢复后丢失，则创建空文件（Alexander）<br>
- 在`pg_hba.conf`中开放访问所有数据库，不仅是 postgres (Franco Bellagamba)<br>
<b>14.30 Version 1.3.2</b><br>
错误修正<br>
- patronictl edit-config 不适用于 ZooKeeper (Alexander Kukushkin)<br>
<b>14.31 Version 1.3.1</b><br>
错误修正<br>
- 由于`_MemberStatus`中的更改，通过 API 进行的故障转移已中断(Alexander Kukushkin) <br>
<b>14.32 Version 1.3</b><br>
    1.3 版本增加了自定义引导程序的可能性，显着改进了对 pg_rewind 的支持，增强了同步模式支持，增加了对增加了patronictl对配置的编辑，并在 Linux 上实现了WatchDog支持。此外，这是第一个与 PostgreSQL 10 一起正常工作的版本。<br>
升级通知<br>
    新版本的 Patroni 没有已知的兼容性问题。版本 1.2 的配置应该无需任何更改即可工作。可以通过安装新软件包并重新启动 Patroni（将导致 PostgreSQL 重新启动）或先将Patroni 置于[暂停模式](https://patroni.readthedocs.io/en/latest/pause.html#pause)然后在集群中的所有节点上重新启动 Patroni（处于暂停模式的 Patroni 不会尝试停止）来进行升级/启动PostgreSQL)，最后从暂停模式恢复。<br>
自定义引导程序<br>
- 使引导集群的过程可配置 (Alexander Kukushkin)<br>
允许自定义引导脚本，而不是`initdb`在初始化集群中的第一个节点时。bootstrap 命令接收集群的名称和数据目录的路径。生成的集群可以配置为执行恢复，从而可以从备份引导并进行时间点恢复。有关此功能的更详细说明，请参阅[文档页面]<br>
(https://patroni.readthedocs.io/en/latest/replica_bootstrap.html#custom-bootstrap)。<br>
更智能的 pg_rewind 支持<br>
- 通过查看与当前 master的时间线差异来决定是否运行 pg_rewind (Alexander) <br>
    以前，Patroni 有一组固定的条件来触发 pg_rewind，即在启动之前的主节点时，当为集群中的每个其他节点切换到指定节点时，或者当存在带有 nofailover 标记的副本时。所有这些情况都有一个共同点，即某些副本可能领先于新的主节点。在某些情况下， pg_rewind 什么都不做，在其他一些情况下，它在必要时不运行。Patroni 不依赖于这个有限的规则列表，而是比较主和副本的 WAL 位置（使用流复制协议），以便可靠地决定副本是否需要切换。<br>
严格的同步复制模式<br>
- 通过添加严格模式来增强同步复制支持（James Sewell，Alexander）<br>
    通常，当`synchronous_mode`启用并且没有副本附加到主时，Patroni 将禁用同步复制以保持主服务器可用于写入。该`synchronous_mode_strict`选项更改为，当它被设置时，Patroni 不会在缺少副本的情况下禁用同步复制，从而有效地阻止所有客户端向主服务器写入数据。除了防止由于自动故障转移导致任何数据丢失的同步模式保证之外，严格模式还确保每次写入要么持久存储在两个节点上，要么在集群中只有一个节点时完全不发生。<br>
使用patronictl 进行配置编辑<br>
- 将配置编辑添加到patronictl (Ants Aasma, Alexander)<br>
    添加patronictl 对存储在 DCS 中的动态集群配置进行编辑的能力。支持从命令行指定参数/值、调用 $EDITOR 或从 yaml 文件应用配置。<br>
Linux WatchDog支持<br>
- 实现对 Linux  的WatchDog支持(Ants)<br>
    支持 Linux 软件WatchDog，以便在 Patroni 没有运行或没有响应的节点重新启动（例如，由于高负载） Linux 软件WatchDog重新启动无响应的节点。可以从 Patroni 配置的WatchDog部分配置要使用的WatchDog设备（默认为/dev/watchdog）和模式（on, automatic, off）。您可以从[WatchDog文档中](https://patroni.readthedocs.io/en/latest/watchdog.html#watchdog)获取更多信息。<br>
添加对 PostgreSQL 10 的支持<br>
- Patroni 与迄今为止发布的所有 PostgreSQL 10 测试版兼容，我们预计它会在发布时与 PostgreSQL 10 兼容。<br>
PostgreSQL 相关的小改进<br>
- 通过 Patroni 配置文件或 DCS 中的动态配置定义 pg_hba.conf(Alexander) <br>
    允许在配置部分`pg_hba.conf`的`pg_hba`子部分中定义内容`postgresql`。这简化了`pg_hba.conf`在多个节点上的管理，因为只需要在 DCS 中定义它，而不是记录到每个节点，手动更改它并重新加载配置。<br>
    定义后，本节的内容将`pg_hba.conf`完全取代当前。如果`hba_file`设置了 PostgreSQL 参数，Patroni 会忽略它。<br>
- 支持通过 UNIX 套接字连接到本地 PostgreSQL 集群 (Alexander)<br>
   将`use_unix_socket`选项添加到Patroni 配置的`postgresql`部分。当设置为 true 并且 PostgreSQL`unix_socket_directories`选项不为空时，使 Patroni 能够使用它的第一个值连接到本地 PostgreSQL 集群。如果`unix_socket_directories`未定义，Patroni 将采用其默认值并完全省略`host`PostgreSQL 连接字符串中的参数。<br>
- 支持在重新加载时更改超级用户和复制凭据（Alexander）<br>
- 支持在 PostgreSQL 数据目录之外存储配置文件 (@jouir)<br>
    添加新的配置`postgresql`配置指令`config_dir`。它默认为数据目录，并且必须可由 Patroni 写入。<br>
错误修复和稳定性改进<br>
- 处理 EtcdEventIndexCleared 和 EtcdWatcherCleared 异常 (Alexander)<br>
    通过避免无用的重试，在 Etcd 结束监视操作时更快地恢复。<br>
- 消除 Etcd 故障时的错误旋转并减少日志垃圾邮件 (Ants)<br>
    避免在第二次和后续 Etcd 连接失败时立即重试并在日志中发出堆栈跟踪。<br>
- 分叉 PostgreSQL 进程时导出语言环境变量 (Oleksii Kliukin)<br>
    在使用 NLS 构建的 PostgreSQL 的非英语语言环境中，避免postmaster 在启动期间成为多线程的致命错误。<br>
- 删除复制槽时的额外检查（Alexander）<br>
    在某些情况下，WAL 发送方会阻止 Patroni 删除复制槽。<br>
- 将复制槽名称截断为 63 (NAMEDATALEN - 1) 个字符以符合 PostgreSQL 命名规则 (Nick Scott)<br>
- 修复了导致从 Patroni  打开到 PostgreSQL 集群的额外连接的竞争条件(Alexander)<br>
- 当节点以空数据目录重新启动时释放Leader键 (Alex Kerney)<br>
- 在没有Leader的情况下运行引导程序时将异步执行程序设置为繁忙 (Alexander)<br>
    不这样做可能会导致错误，指出节点属于不同的集群，因为 Patroni 继续进行正常的业务，同时由不需要Leader存在于集群中的引导方法引导。<br>
- 改进 WAL-E 副本创建方法（Joar Wandborg，Alexander）。<br>
- 在解析 WAL-E 基础备份时使用 csv.DictReader，接受带有空格分隔的日期和时间的 ISO 日期。<br>
- 支持从副本中获取当前 WAL 位置以估计要恢复的 WAL 数量。以前，用于调用仅在主节点上可用的系统信息函数的代码。<br>
<b>14.33 Version 1.2</b><br>
    此版本对同步复制的处理进行了重大改进，使启动过程和故障转移更加可靠，添加了 PostgreSQL 9.6 支持并修复了大量错误。此外，包括这些发行说明在内的文档已移至 (https://patroni.readthedocs.io/)。<br>
同步复制<br>
- 添加同步复制支持。（Ants Aasma）<br>
    添加一个新的配置变量`synchronous_mode`。启用后，`synchronous_standby_names`只要有可用的健康备库，Patroni 就会设法启用同步复制。启用同步模式后，Patroni 将仅自动故障转移到同步备用服务器。这实际上意味着在这种情况下不会丢失任何用户可见的事务。有关详细说明和实现细节，请参阅 [功能文档](https://patroni.readthedocs.io/en/latest/replication_modes.html#synchronous-mode)。<br>
可靠性改进<br>
- 当 PostgreSQL 不是 100% 健康时，不尝试更新Leader 键。当Leader 键更新失败时立即降级。 （Alexander Kukushkin）<br>
- 从克隆新副本的目标列表中排除不健康的节点(Alexander)<br>
- 为 Consul 实施重试和超时策略，类似于对 Etcd 的做法。(Alexander)<br>
- 将`--dcs`和`--config-file`应用到`patronictl`的所有选项。(Alexander)<br>
- 将所有 postgres 参数写入 postgresql.conf。(Alexander)<br>
    它允许启动由patroni配置的PostgreSQL，只需pg_ctl。<br>
- 当配置中没有用户时避免异常。（Kirill Pushkin）<br>
- 允许暂停不健康的集群。在此修复之前，`patronictl`如果它尝试暂停不健康节点，则将退出。(Alexander)<br>
- 改进领导观察功能。(Alexander)<br>
    以前，副本一直在监视Leader 键（休眠直到超时或Leader 键更改）。通过此更改，他们仅在副本的 PostgreSQL 处于`running`状态时进行监视，而不会在它停止/启动或重新启动 PostgreSQL时进行监视。<br>
- 将 SIGCHILD 作为 PID 1 处理时，避免遇到竞争条件。（Alexander）<br>
    以前在 Docker 容器中运行时可能会发生竞争条件，因为 Patroni 中的同一个进程既产生了新进程，又从它们处理了 SIGCHILD。此更改为 Patroni 使用 fork/execs，并使原始 PID 1 进程负责处理来自子进程的信号。<br>
- 修复 WAL-E 还原。（Oleksii Kliukin）<br>
    以前 WAL-E 恢复使用该`no_master`标志来完全避免咨询主控，这使得 Patroni 总是选择从 WAL 恢复而不是`pg_basebackup`. 此更改将其还原为 的原始含义`no_master`，即如果 master 未运行，则可以选择 Patroni WAL-E 还原作为复制方法。通过检查传递给方法的连接字符串来检查后者。此外，它使重试机制更加健壮并处理其他细节。<br>
- 实现异步 DNS 解析器缓存。(Alexander)<br>
    避免在 DNS 暂时不可用时失败（例如，由于节点接收的流量过多）。<br>
- 实现启动状态和主启动超时。（Ants, Alexander)<br>
    以前`pg_ctl`等待超时，然后高兴地考虑考虑运行 PostgreSQL。这导致 PostgreSQL 在列表中显示为正在运行，而实际上并没有运行，并导致竞争条件，从而导致故障转移或崩溃恢复，或因故障转移和错过倒带而中断的崩溃恢复。此更改添加了一个`master_start_timeout`参数并为主 HA 循环引入了一个新状态：`starting`。当`master_start_timeout`是 0 时，一旦有故障转移候选者，当主节点崩溃时，我们将立即进行故障转移。否则，Patroni 将在超时期间尝试在 master 上启动 PostgreSQL 后等待；当它到期时，它会在可能的情况下进行故障转移。即使在超时到期之前，手动故障转移请求也将在主服务器崩溃期间得到满足。<br>
    将`timeout`参数引入`restart`API 端点和`patronictl`. 当它被设置并且重启时间超过超时时间时，PostgreSQL 被认为是不健康的，其他节点有资格获得领导者锁。<br>
- 修复`pg_rewind`暂停模式下的行为。(Ants)<br>
    当 Patroni 认为它需要倒带但无法倒带（即`pg_rewind`不存在）时，避免在暂停模式下不必要的重新启动。如果 pg_rewind相关用户配置部分中缺少超级用户身份验证，则返回超级用户（默认操作系统用户）的默认libpq值。<br>
- 序列化回调执行。当新的回调即将运行时，杀死前一个相同类型的回调。修复运行回调时产生僵尸进程的问题。(Alexander)<br>
- 避免在 DCS 中设置Leader键但更新此Leader键失败时提升前主。(Alexander)<br>
    这避免了当前主节点与 Etcd 和其他 DCS 中允许“不一致读取”的少数节点一起分区时继续保持其角色的问题。<br>
各种各样的<br>
- `post_init`在引导程序上添加配置选项。（Alejandro Martínez）<br>
    Patroni 将在`initdb`为新集群运行和启动 PostgreSQL后立即调用此选项的脚本参数。脚本接收超级用户的连接URL，并将PGPASSFILE设置为指向包含密码的.pgpass文件。如果脚本失败，Patroni 初始化也会失败。它对于在新集群中添加新用户或创建扩展很有用。<br>
- 实现 PostgreSQL 9.6 支持。(Alexander)<br>
    使用wal_level=replica作为hot_standby的同义词，避免在从一个更改为另一个时挂起重新启动标志。(Alexander)<br>
文档改进<br>
- 添加 Patroni 主[循环工作流图]<br>
(https://raw.githubusercontent.com/zalando/patroni/master/docs/ha_loop_diagram.png)。（Alejandro，Alexander）<br>
- 改进自述文件，添加 Helm 图表和发布说明链接。（Lauri Apple）<br>
- 将 Patroni 文档移至阅读文档。 最新文档可在 https://patroni.readthedocs.io 获得。(Oleksii)<br>
    使文档可以从不同的设备（包括智能手机）轻松查看和搜索。<br>
- 将包移动到语义版本控制。(Oleksii)<br>
    Patroni 将遵循major.minor.patch 版本架构，以避免在小但关键的错误修正上发布新的次要版本。我们只会发布包含所有补丁的次要版本的发行说明。<br>
<b>14.34 Version 1.1</b><br>
    此版本通过引入暂停模式改进了 Patroni 集群的管理，通过计划和有条件的重启改进了维护，使 Patroni 与 Etcd 或 Zookeeper 的交互更具弹性，并大大增强了 patronictl。<br>
升级通知<br>
    从低于 1.0 的版本升级时，请阅读 1.0 版本说明中有关更改凭据和配置格式的信息。<br>
暂停模式<br>
- 引入暂停模式以暂时将 Patroni 与管理 PostgreSQL 实例分离。（Murat Kabilov、Alexander Kukushkin、Oleksii Kliukin）<br>
以前，必须向 Patroni 发送 SIGKILL 信号以停止它而不终止 PostgreSQL。新的暂停模式将 Patroni 从 PostgreSQL 集群范围内分离，而不会终止 Patroni。它类似于 Pacemaker 中的维护模式。Patroni 仍然负责更新 DCS 中的成员和leader键，但它不会在此过程中启动、停止或重新启动 PostgreSQL 服务器。有一些例外，例如，仍然允许手动故障转移、重新初始化和重新启动。您可以阅读[此功能的详细说明]<br>
(https://patroni.readthedocs.io/en/latest/pause.html#pause)。<br>
此外，padomictl 支持新的`pause`和`resume`命令来切换暂停模式。<br>
计划重启和有条件重启<br>
- 向重启 API 命令添加条件(Oleksii) <br>
    此更改通过添加几个可以验证以进行重新启动的条件来增强 Patroni 重新启动。条件包括在 PostgreSQL 角色是主服务器或副本时重新启动、检查 PostgreSQL 版本号或仅在需要重新启动以应用配置更改时才重新启动。<br>
- 添加计划重启 (Oleksii)<br>
    现在可以安排将来重新启动。每个节点仅支持一次计划重启。如果不再需要，可以清除计划的重启。支持计划重启和有条件重启的组合，例如，可以在夜间计划次要 PostgreSQL 升级，仅重启运行过时次要版本的实例，而无需向管理脚本添加 postgres 特定的逻辑。<br>
- 将有条件的和预定的重启支持添加到patronictl (Murat)。<br>
    守护者重启支持几个新选项。还有patriotictl flush命令来清理计划的动作。<br>
强大的 DCS 交互<br>
- 根据 loop_wait设置 Kazoo 超时 (Alexander) <br>
    最初，ping_timeout 和connect_timeout 值是根据协商会话超时计算得出的。未考虑 Patroni loop_wait。因此，单次重试可能需要比会话超时更多的时间，从而迫使 Patroni 释放锁定并降级。<br>
    此更改将 ping 和连接超时设置为 loop_wait 值的一半，加快检测连接问题并留出足够的时间在丢失锁定之前重试连接尝试。<br>
- 仅在原始请求成功后更新 Etcd 拓扑（Alexander）<br>
    推迟更新客户端已知的 Etcd 拓扑，直到原始请求之后。在检索集群拓扑时，根据 Etcd 集群中已知的节点数实现重试超时。这使得我们的客户更喜欢获得请求的结果，而不是拥有最新的节点列表。<br>
    面对网络问题，这两项更改都使 Patroni 与 DCS 的连接更加可靠。<br>
Patronictl，监控和配置<br>
- 通过 API返回有关流式副本的信息 (Feike Steenbergen) <br>
以前，没有可靠的方法向 Patroni 查询无法流式传输更改（例如，由于连接问题）的 PostgreSQL 实例。此更改通过 /patroni 端点公开 pg_stat_replication 的内容。<br>
- 添加patronictl scaffold 命令(Oleksii)<br>
    在 Etcd 中添加创建集群结构的命令。集群是用用户指定的 sysid 和leader创建的，leader和members键都是持久的。此命令可用于创建所谓的无主配置，其中 Patroni 集群仅由副本组成，从不知道 Patroni 的外部主节点复制。随后，可以删除leader 键，提升 Patroni 节点之一并用基于 Patroni 的 HA 集群替换原来的主节点。<br>
- 添加配置选项`bin_dir`以定位 PostgreSQL 二进制文件 (Ants Aasma)<br>
    当 Linux 发行版支持同时安装多个 PostgreSQL 版本时，能够显式指定 PostgreSQL 二进制文件的位置很有用。<br>
- 允许使用`custom_conf`覆盖配置文件路径(Alejandro Martínez)<br>
允许自定义配置文件路径，这将由 Patroni 管理，[详细信息]<br>
(https://patroni.readthedocs.io/en/latest/SETTINGS.html#postgresql-settings)。<br>
错误修复和代码改进<br>
- 使 Patroni 兼容 PostgreSQL 10 及以上的新版本模式（Feike）<br>
    在根据 PostgreSQL 版本进行有条件重启时，请确保 Patroni 理解 2 位版本号。<br>
- 使用 pkgutil 查找 DCS 模块 (Alexander)<br>
    使用专用的 python 模块而不是手动遍历目录来查找 DCS 模块。<br>
- 启动 Patroni时始终调用 on_start 回调 (Alexander) <br>
    以前，Patroni 在附加到具有正确角色的已运行节点时不会调用任何回调。由于回调通常用于路由客户端连接，这可能导致无法在连接路由方案中注册正在运行的节点。通过此修复，即使连接到已运行的节点，Patroni 也会调用 on_start 回调。<br>
- 不要丢弃活动复制槽（Murat、Oleksii）<br>
    避免删除 master 上的活动物理复制槽。无论如何，PostgreSQL 不能删除这样的插槽。此更改使在主服务器上运行非 Patroni 管理的副本/消费者成为可能。<br>
- 在 PostgreSQL 实例启动期间关闭 Patroni 连接 (Alexander)<br>
    当 PostgreSQL 节点启动时，强制 Patroni 关闭所有以前的连接。如果 postmaster 被 SIGKILL 杀死，则避免重用以前的连接的陷阱。<br>
- 从member名称构造槽名称时替换无效字符（Ants）<br>
    确保不符合槽命名规则的备用名称不会导致槽创建和备用启动失败。将插槽名称中的破折号替换为下划线，并将插槽名称中不允许的所有其他字符替换为其 unicode 代码点。<br>
<b>14.35 Version 1.0</b><br>
    此版本引入了全局动态配置，允许动态更改整个 HA 集群的 PostgreSQL 和 Patroni 配置参数。它还提供了许多错误修正。<br>
升级通知<br>
    从 v0.90 或更低版本升级时，请始终在 master 之前升级所有副本。由于我们不再在 DCS 中存储复制凭据，旧副本将无法连接到新主服务器。<br>
动态配置<br>
- 实现动态全局配置 (Alexander Kukushkin)<br>
    引入新的 REST API 端点 /config 以提供应为整个 HA 集群（主节点和所有副本）全局设置的 PostgreSQL 和 Patroni 配置参数。这些参数在 DCS 中设置，并且在许多情况下可以在不中断 PostgreSQL 或 Patroni 的情况下应用。当某些值需要重新启动 PostgreSQL 时，Patroni 会通过 API 设置一个称为“待重启”的特殊标志。在这种情况下，应通过 API 手动发出重新启动。<br>
    Patroni SIGHUP 或 POST 到 /reload 将使它重新读取配置文件。<br>
有关 哪些参数可以更改以及处理差异配置源的顺序的详细信息，请参阅[动态配置]<br>
(https://patroni.readthedocs.io/en/latest/dynamic_configuration.html#dynamic-configuration)。<br>
自 v0.90 以来，配置文件格式已更改。Patroni 仍然与旧的配置文件兼容，但为了利用引导程序参数，需要对其进行更改。鼓励用户通过参考[动态配置文档页面]<br>
(https://patroni.readthedocs.io/en/latest/dynamic_configuration.html#dynamic-configuration)来更新它们。<br>
更灵活的配置<br>
- 使 postgresql 配置和数据库名称 Patroni 连接到可配置 (Misja Hoebe)<br>
    引入database和config_base_name配置参数。其中，可以使用 PipelineDB 和其他 PostgreSQL fork 运行 Patroni。<br>
- 实现通过环境配置一些 Patroni 配置参数的可能性（Alexander）<br>
这些包括范围、节点名称和命名空间，以及秘密，使在动态环境（即 Kubernetes）中运行 Patroni 变得更容易。请参阅[支持的环境变量]<br>
(https://patroni.readthedocs.io/en/latest/ENVIRONMENT.html#environment)以获取更多详细信息。<br>
- 更新内置的 Patroni docker 容器以利用基于环境的配置 (Feike Steenbergen)。<br>
- 将 Zookeeper 支持添加到 Patroni docker 镜像 (Alexander)<br>
- 拆分 Zookeeper 和 Exhibitor 配置选项（Alexander）<br>
- 让patronictl 重用来自Patroni 的代码来读取配置（Alexander）<br>
    这允许patronictl 利用基于环境的配置。<br>
- 在 primary_conninfo 中将应用程序名称设置为节点名称(Alexander) <br>
    这简化了给定节点的同步复制的识别和配置。<br>
稳定性、安全性和可用性改进<br>
- 重置 sysid 并且在恢复正在进行的备份时不调用 pg_controldata (Alexander)<br>
    此更改减少了 Patroni API 运行状况检查在从备份中对该节点进行冗长初始化期间产生的噪音量。<br>
- 修复一堆 pg_rewind 角落案例（Alexander）<br>
    如果源集群不是主集群，请避免运行 pg_rewind。<br>
    此外，避免在倒带失败时删除数据目录，除非新参数remove_data_directory_on_rewind_failure设置为 true。默认情况下它是false的。<br>
- 从 DCS中的复制连接字符串中删除密码 (Alexander) <br>
    以前，Patroni 始终使用 DCS 中 Postgres URL 中的复制凭据。现在已更改为从 Patoni 配置中获取凭据。机密（复制用户名和密码）并不再在 DCS 中公开。<br>
- 修复降级调用周围的异步机制 (Alexander)<br>
    Demote 现在完全异步运行，不会阻止 DCS 交互。<br>
- 如果已配置，则使 Patientictl 始终发送授权标头 (Alexander)<br>
    当 Patroni 被配置为需要对这些请求进行授权时，这允许 Patronictl 发出“受保护的”请求，即重新启动或重新初始化。<br>
- 正确处理 SystemExit 异常 (Alexander)<br>
    避免 Patroni 在收到 SIGTERM 时无法正常停止的问题<br>
- confd (Alexander) 的示例 haproxy 模板<br>
    使用 confide 从 DCS 中的 Patoni 状态生成并动态更改 haproxy 配置<br>
- 改进和重组文档，使其对新用户更友好（Lauri Apple）<br>
- API 必须在 pg_ctl 停止期间报告 role=master (Alexander)<br>
    使回调调用更可靠，尤其是在集群停止的情况下。此外，引入pg_ctl_timeout选项以通过pg_ctl设置启动、停止和重启调用的超时时间。<br>
- 修复 etcd (Alexander) 中的重试逻辑<br>
    使重试更具可预测性和稳健性。<br>
- 使 Zookeeper 代码更能抵御短网络故障 (Alexander)<br>
    减少连接超时，使 Zookeeper 连接尝试更频繁。<br>
<b>14.36 Version 0.90</b><br>
    该版本增加了对Consul，包括一个新的noloadbalance标签，改变的行为clonefrom标签，提高pg_rewind处理和改善patronictl控制程序。<br>
Consul支持<br>
- 实施Consul支持 (Alexander Kukushkin)<br>
    除了 Etcd 和 Zookeeper 之外，Patroni 还对抗 Consul。连接参数可以在 YAML 文件中配置。<br>
新的和改进的标签<br>
- 实现noloadbalance标签 (Alexander)<br>
    此标记使 Patroni 始终返回副本对负载均衡器不可用。<br>
- 更改clonefrom标签的实现（Alexander）<br>
    以前，必须向clonefrom提供节点名称，强制标记副本从特定节点克隆。新的实现使clonefrom成为一个布尔标签：如果它被设置为 true，则副本成为其他副本从中克隆的候选者。当存在多个候选时，副本随机选择一个。<br>
稳定性和安全性改进<br>
- 许多可靠性改进（Alexander）<br>
    删除一些虚假的错误消息，提高故障转移的稳定性，解决一些从 DCS 读取数据、关闭、降级和重新附加前领导者的极端情况。<br>
- 改进系统脚本以避免在停止时杀死 Patroni 子节点（Jan Keirse、Alexander Kukushkin）<br>
    以前，在停止 Patroni 时，systemd也会向 PostgreSQL 发送信号。由于 Patroni 也尝试自行停止 PostgreSQL，因此导致发送到不同的关闭请求（智能关闭，然后是快速关闭）。这导致副本过早断开连接，并且前主节点在降级后无法重新加入。由 Jan 使用 Alexander 先前的研究修复。<br>
- 消除一些前主控在作为副本重新加入之前无法调用 pg_rewind 的情况（Oleksii Kliukin）<br>
    以前，我们只在前主服务器崩溃时调用pg_rewind。只要 pg_rewind 存在于系统中，就将其更改为始终为以前的 master 运行 pg_rewind。这解决了在副本设法获取最新更改之前关闭主服务器的情况（即在“智能”关闭期间）。<br>
- 对单元测试和验收测试的大量改进，特别是支持 Zookeeper 和 Consul (Alexander)。<br>
- 使 Travis CI 更快，并支持对 Zookeeper（Exhibitor）和 Consul（Alexander）运行测试<br>
    在每次提交或拉取请求时，单元测试和验收测试都会针对 Etcd、Zookeeper 和 Consul 自动运行。<br>
- 从 Patroni (Feike Steenbergen) 调用 PostgreSQL 命令之前清除环境变量<br>
    这可以防止通过连接到 Patroni 管理的 PostgreSQL 集群来读取系统环境变量的可能性。<br>
配置和控制更改<br>
- 统一patronictl和Patroni配置（Feike）<br>
    Patroni 可以使用与 Patroni 本身相同的配置文件。<br>
- 启用 Patroni 从环境变量中读取配置 (Oleksii)<br>
    这简化了自动为 Patroni 生成配置，或合并来自不同来源的单个配置。<br>
- API返回的信息中包含数据库系统标识（Feike）<br>
- 为所有可用的 DCS实现delete_cluster (Alexander)<br>
    在patronictl 中启用对Etcd 以外的DCS 的支持。<br>
<b>14.37 Version 0.80</b><br>
    此版本增加了对级联复制的支持，并通过提供预定的故障转移来简化 Patroni 管理。可以将旧版本的 Patroni（特别是 0.78）与该版本结合使用，以便迁移到新版本。请注意，计划故障转移和级联复制相关功能仅适用于 Patroni 0.80 及更高版本。<br>
级联复制<br>
- 添加对patroni 节点的replicatefrom和clonefrom标签的支持。（Oleksii Kliukin）<br>
- 标签replicatefrom允许副本使用任意节点作为源，而不是主节点。该clonefrom做为初始备份相同。它们一起使 Patroni 能够完全支持级联复制。<br>
- 添加对运行复制方法的支持以初始化副本，即使没有正在运行的复制连接 (Oleksii)。<br>
    这对于从存储在 S3 或 FTP 上的快照创建副本很有用。不需要正在运行的复制连接的复制方法应该在 yaml 配置中提供no_master: true。如果存在复制连接，这些脚本仍将按顺序调用。<br>
Patronictl、API 和 DCS 改进<br>
- 计划任务failover (Feike Steenbergen)。<br>
    failover可以被安排在未来的某个时间发生，使用patonictl 或API 调用。<br>
- 在patronictl (Feike) 中添加对dbuser和password参数的支持。<br>
- 将 PostgreSQL 版本添加到健康检查输出 (Feike)。<br>
- 改进对守护程序的 Zookeeper 支持（Oleksandr Shulgin）<br>
- 迁移到 python-etcd 0.43 (Alexander Kukushkin)<br>
配置<br>
- 为 Patroni 添加示例系统配置脚本。(Jan Keirse) <br>
- 修复 Patroni 忽略数据库连接配置文件中指定的超级用户名的问题。（Alexander）<br>
- 通过为 Patroni启动的 postmaster 创建单独的会话 ID 和进程组来修复 CTRL-C 的处理。 (Alexander) <br>
测试<br>
- 添加具有behave验收测试，以检查运行 Patroni的真实场景。（Alexander、Oleksii）<br>
    可以使用behave命令手动启动测试。它们也会针对拉取请求和提交后自动启动。<br>
可以在[项目的 github 页面](https://github.com/zalando/patroni/releases)上找到一些旧版本的发行说明。<br>

[patroni-doccn](https://github.com/postgres-cn/patroni-doccn/blob/main/README.md)
