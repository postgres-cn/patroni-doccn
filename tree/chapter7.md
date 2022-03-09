# 第7章 YAML 配置设置<br>
<b>7.1 动态配置设置</b><br>
动态配置存储在DCS中(分布式配置存储）并应用到所有集群中的节点上。一些参数，例如像 loop_wait, ttl, postgresql.parameters.max_connections, postgresql.parameters.max_worker_processes 等等仅能通过动态配置被设置。一些其他参数像postgresql.listen, postgresql.data_dir 仅能在本地被设置，如在Patroni配置文件或者通过配置变量。在大多数情况下，本地配置将覆盖动态配置。 为了更改动态配置，您可以使用patronictl edit-config工具或Patroni REST API<br>
• loop_wait：循环将休眠的秒数。默认值：10<br>
• ttl：获取leader key的TTL（以秒为单位）。可以将其视为启动自动故障转移过程之前的时间长度。默认值：30<br>
• retry_timeout：DCS和PostgreSQL操作重试超时（秒）。在低于此值时，不会因为DCS或网络问题不会导致Patroni降级。默认值: 10<br>
• maximum_lag_on_failover：跟随者可以延迟的最大字节数，以便能够参加领导人选举。<br>
• maximum_lag_on_syncnode：同步节点在被认为是不健康的候选者并被健康的异步节点交换之前可能滞后的最大字节数。如果有多个从库，Patroni将使用max replica lsn，否则它将使用leader的当前wal lsn。默认值为-1，当该值设置为0或更低时，用户不会采取操作来交换不健康的同步节点。请设置足够高的值，以便在高交易量期间，Patroni不会频繁地交换同步节点。<br>
• max_timelines_history : DCS 中保存的最大时间线历史项目数。默认值：0。当设置为 0 时，它会在 DCS 中保留完整的历史记录。<br>
• master_start_timeout：在触发故障转移之前允许主服务器从故障中恢复的时间（以秒为单位）。默认值为 300 秒。如果可能，设置为 0 时故障转移会在检测到崩溃后立即完成。使用异步复制时，故障转移可能会导致事务丢失。master 故障的最坏情况故障转移时间是：loop_wait + master_start_timeout + loop_wait，除非 master_start_timeout 为零，在这种情况下它只是 loop_wait。根据您的持久性/可用性权衡设置值。<br>
• master_stop_timeout : Patroni 停止 Postgres 时允许等待的秒数，仅在启用 synchronous_mode 时有效。当设置为 > 0 并且启用了 synchronous_mode 时，如果停止操作的运行时间超过了 master_stop_timeout 设置的值，则 Patroni 会向 postmaster 发送 SIGKILL。根据您的持久性/可用性权衡设置值。如果该参数未设置或设置 <= 0，则 master_stop_timeout 不适用。<br>
• synchronous_mode : 打开同步复制模式。在这种模式下，将选择一个副本作为同步副本，只有最新的领导者和同步副本才能参与领导人选举。同步模式确保成功提交的事务不会在故障转移时丢失，代价是在用户无法确保事务持久性的情况下失去写入可用性。有关详细信息，请参阅复制模式文档。<br>
• synchronous_mode_strict：如果没有可用的同步副本，则防止禁用同步复制，从而阻止所有客户端写入主服务器。有关详细信息，请参阅复制模式文档。<br>
• PostgreSQL：<br>
&nbsp;&nbsp;–use_pg_rewind : 是否使用 pg_rewind。默认为false。<br>
&nbsp;&nbsp;–use_slots : 是否使用复制槽。在 PostgreSQL 9.4+ 上默认为true。<br>
&nbsp;&nbsp;–recovery_conf：配置follower写入recovery.conf 的附加配置设置。PostgreSQL 12 中不再有 recovery.conf，但您可以继续使用此部分，因为 Patroni 透明地处理它。<br>
&nbsp;&nbsp;–parameters：Postgres 的配置设置列表。<br>
• Standby_cluster：如果定义了这个部分，我们想要引导一个备用集群。<br>
&nbsp;&nbsp;–host : 远程主机的地址<br>
&nbsp;&nbsp;–port : 远程主机的端口<br>
&nbsp;&nbsp;–primary_slot_name：远程主服务器上用于复制的插槽。此参数是可选的，默认值来自实例名称（请参阅函数slot_name_from_member_name）。<br>
&nbsp;&nbsp;–create_replica_methods：可用于从远程主机引导备用leader的有序方法列表可能与PostgreSQL中定义的列表不同。<br>
&nbsp;&nbsp;–restore_command：将WAL记录从远程主机还原到备用leader的命令，可以与PostgreSQL中定义的列表不同。<br>
&nbsp;&nbsp;–archive_cleanup_command : standby leader的清理命令<br>
&nbsp;&nbsp;–recovery_min_apply_delay : 等待多长时间将WAL记录应用到备用leader上<br>
• slots：定义永久复制槽。这些插槽将在切换/故障切换期间保留。逻辑槽通过重启从主节点复制到备用节点，之后它们的位置每隔loop_wait秒（如有必要）前进。通过libpq连接并使用倒带或超级用户凭据复制逻辑插槽文件（请参阅postgresql.authentication部分）。副本上的逻辑槽位置总是有可能比以前的主位置稍晚，因此应用程序应该准备好在故障转移后第二次可以接收到一些消息。最简单的方法是跟踪confirmed_flush_lsn。启用永久逻辑复制槽需要postgresql.use_slots进行设置，并且还将自动启用hot_standby_feedback. 由于逻辑复制槽的故障转移在 PostgreSQL 9.6 及更早版本上是不安全的，并且 PostgreSQL 10 版缺少一些重要功能，因此该功能仅适用于 PostgreSQL 11及以上版本。<br>
&nbsp;&nbsp;– my_slot_name：复制槽的名称。 如果永久插槽名称与当前主插槽的名称匹配，则不会创建该插槽。操作员有责任确保由Patroni 自动为成员创建的复制槽和永久复制槽之间的名称没有冲突<br>
&nbsp;&nbsp;&nbsp;&nbsp;-- type：插槽类型。可能是physical或logical。如果插槽是合乎逻辑的，则必须另外定义database和plugin。<br>
&nbsp;&nbsp;&nbsp;&nbsp;-- database：应在其中创建逻辑槽的数据库名称。<br>
&nbsp;&nbsp;&nbsp;&nbsp;-- plugin : 逻辑插槽的插件名称。<br>
• ignore_slots：Patroni 应忽略匹配插槽的复制插槽属性集列表。此配置/功能/等。当某些复制槽在 Patroni 之外管理时非常有用。匹配属性的任何子集都会导致插槽被忽略。<br>
&nbsp;&nbsp;– name：复制槽的名称。<br>
&nbsp;&nbsp;– type：插槽类型。可以physical或logical。如果插槽是逻辑的，您可以另外定义database和/或plugin.<br>
&nbsp;&nbsp;– database：数据库名称（匹配logical插槽时）。<br>
&nbsp;&nbsp;– plugin : 逻辑解码插件（匹配logical插槽时）。<br>
注意：slots是一个 hashmap 而ignore_slots是一个数组。例如：<br>
<table border="1"><tr><th align="left">
slots:<br>
&nbsp;&nbsp;permanent_logical_slot_name:<br>
&nbsp;&nbsp;&nbsp;&nbsp;type: logical<br>
&nbsp;&nbsp;&nbsp;&nbsp;database: my_db<br>
&nbsp;&nbsp;&nbsp;&nbsp;plugin: test_decoding<br>
&nbsp;&nbsp;permanent_physical_slot_name:<br>
&nbsp;&nbsp;&nbsp;&nbsp;type: physical<br>
&nbsp;&nbsp;...<br>
ignore_slots:<br>
&nbsp;&nbsp;- name: ignored_logical_slot_name<br>
&nbsp;&nbsp;&nbsp;&nbsp;type: logical<br>
&nbsp;&nbsp;&nbsp;&nbsp;database: my_db<br>
&nbsp;&nbsp;&nbsp;&nbsp;plugin: test_decoding<br>
&nbsp;&nbsp;- name: ignored_physical_slot_name<br>
&nbsp;&nbsp;&nbsp;&nbsp;type: physical<br>
&nbsp;&nbsp;...
</th></tr></table><br>
<b>7.2 全局/通用</b><br>
• name : 主机名。对于集群必须是唯一的。<br>
• namespace：配置存储区内的路径，Patroni 将保存有关群集的信息。默认值：“/service”<br>
• scope：集群名称<br>
<b>7.3 Log</b><br>
• level：设置一般日志记录级别。默认值为INFO（请参阅Python 日志记录文档）<br>
• traceback_level：设置回溯可见的级别。默认是 ERROR。如果只想在启用时查看回溯，请将其设置为DEBUG，同时log.level=DEBUG。<br>
• format：设置日志格式字符串。默认值为%(asctime)s %(levelname)s: %(message)s（参见LogRecord 属性）<br>
• dateformat : 设置日期时间格式字符串。（请参阅formatTime() 文档）<br>
• max_queue_size：Patroni 使用两步日志记录。日志记录被写入内存队列，并且有一个单独的线程将它们从队列中拉出并写入 stderr 或文件。内部队列的最大大小默认限制为1000条记录，足以保留过去 1h20m 的日志。<br>
• dir : 将应用程序日志写入的目录。该目录必须存在并且可由执行 Patroni 的用户写入。如果设置此值，默认情况下应用程序将保留 4 个 25MB 的日志。您可以使用file_num和file_size调整这些保留值（见下文）。<br>
• file_num：要保留的应用程序日志的数量。<br>
• file_size：触发日志滚动的patoni.log 文件的大小（以字节为单位）。<br>
• loggers：此部分允许重新定义每个 python 模块的日志记录级别<br>
• Patroni.postmaster: WARNING<br>
• urllib3：DEBUG<br>
<b>7.4 引导配置</b><br>
• bootstrap：<br>
&nbsp;&nbsp;-dcs：在初始化一个集群后，将配置信息写入给定的配置存储 /&lt;namespace&gt;/&lt;scope&gt;/config 中。集群的全局动态配置。在 bootstrap.dcs 中可以设置动态配置中描述的任何参数，在用户初始化（引导）新集群后，它将把这个部分写入配置存储的/&lt;namespace&gt;/&lt;scope&gt;/config。所有以后的bootstrap.dcs 不会有任何效果！如果要更改它们，请使用patronictl edit-config或Patroni REST API<br>
&nbsp;&nbsp;-method：用于引导此集群的自定义脚本。有关详细信息，请参阅自定义引导程序方法文档。initdb指定何时恢复为默认initdb命令。initdb当method 配置文件中没有参数时也会触发。<br>
&nbsp;&nbsp;-initdb：列出要传递给 initdb 的选项。<br>
&nbsp;&nbsp;&nbsp;&nbsp;--data-checksums: 在9.3版本时使用pg_rewind，需要打开。<br>
&nbsp;&nbsp;&nbsp;&nbsp;--encoding: UTF8:默认新数据库的编码。<br>
&nbsp;&nbsp;&nbsp;&nbsp;--locale: UTF8: 新数据库的默认区域设置。<br>
&nbsp;&nbsp;-pg_hba：您应该添加到 pg_hba.conf 的行列表。<br>
&nbsp;&nbsp;&nbsp;&nbsp;--host all all 0.0.0.0/0 md5.<br>
&nbsp;&nbsp;&nbsp;&nbsp;--host replication replicator 127.0.0.1/32 md5: 复制所需要的。<br>
&nbsp;&nbsp;-users : 初始化新集群后需要创建的一些额外用户<br>
&nbsp;&nbsp;&nbsp;&nbsp;--admin : 用户名<br>
&nbsp;&nbsp;&nbsp;&nbsp;--password：zalando：<br>
&nbsp;&nbsp;&nbsp;&nbsp;--options : CREATE USER 语句的选项列表<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;--- createrole<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;--- createdb<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;--- post_bootstrap或post_init：将在初始化集群后执行的附加脚本。该脚本接收一个连接字符串 URL（使用集群超级用户作为用户名）。PGPASSFILE 变量设置为 pgpass 文件的位置。<br>
<b>7.5 Consul</b><br>
大多数参数是可选的，但您必须指定主机或网址之一<br>
• host：Consul的本地代理，host:port。<br>
• url : Consul 本地代理的 url，格式为：http(s)://host:port。<br>
• port：（可选）Consul端口。<br>
• scheme：（可选）http或https，默认为http。<br>
• token：（可选）ACL 令牌。<br>
• verify：（可选）是否验证 HTTPS 请求的 SSL 证书。<br>
• cacert：（可选）ca 证书。如果存在，它将启用验证。<br>
• cert：（可选）带有客户端证书的文件。<br>
• key：（可选）如果密钥是证书的一部分，则可以为空。<br>
• dc：（可选）要与之通信的数据中心。默认情况下使用主机的数据中心。<br>
• consistency：（可选）选择consul一致性模式。可能的值为default, consistent, 或stale(更多细节在consul API 参考中)<br>
• checks：（可选）用于会话的 Consul 健康检查列表。默认情况下使用空列表。<br>
• register_service：（可选）是否使用由scope参数定义名称以及标签主服务器，副本服务器或备用服务器领导者注册的服务（取决于节点的角色）。模式为false。<br>
• service_tags：（可选）除了角色（master/replica/standby-leader）以外，其他静态标签还可以添加到Consul服务中。默认情况下，使用一个空列表。<br>
• service_check_interval：（可选）对注册的 url 执行健康检查的频率。<br>
token需要有以下ACL权限：<br>
<table border="1"><tr><th align="left">
service_prefix "${scope}" {<br>
&nbsp;&nbsp;policy = "write"<br>
}<br>
key_prefix "${namespace}/${scope}" {<br>
&nbsp;&nbsp;policy = "write"<br>
}<br>
session_prefix "" {<br>
&nbsp;&nbsp;policy = "write"<br>
}
</th></tr></table><br>
<b>7.6 Etcd</b><br>
大多数参数是可选的，但您必须指定host、hosts、url、proxy或srv 之一<br>
• host：etcd的端点，host:port。<br>
• hosts : etcd 端点的列表，格式为 host1:port1,host2:port2,etc... 可以是逗号分隔的字符串或实际的 yaml 列表。<br>
• use_proxies：如果此参数设置为true，Patroni 会将主机视为代理列表，并且不会执行etcd 集群的拓扑发现。<br>
• url : etcd 的 url。<br>
• proxy : etcd 的代理 url。如果您使用代理连接到 etcd，请使用此参数而不是url。<br>
• srv：用于搜索 SRV 记录以进行集群自动发现的域。Patroni 将尝试查询指定域的这些 SRV 服务名称（按此顺序直到第一次成功）：_etcd-client-ssl, _etcd-client, _etcd-ssl, _etcd, _etcd-server-ssl, _etcd-server。如果检索到_etcd-server-ssl或 的SRV 记录，_etcd-server则使用 ETCD 对等协议查询 ETCD 以获取可用成员。否则将使用 SRV 记录中的主机。<br>
• srv_suffix：为查找期间查询的SRV名称配置后缀。使用此标志区分同一域下的多个etcd群集。仅与srv配合使用。例如，如果设置了srv_suffix:foo和srv:example.org，则会进行以下DNS srv查询：_etcd-client-ssl-foo._tcp.example.com（对于每个可能的etcd srv服务名称，依此类推）。<br>
• protocol：（可选）http 或 https，如果未指定，则使用 http。如果指定了url或proxy -将从他们那里获取协议。<br>
• username：（可选）用于 etcd 身份验证的用户名。<br>
• password：（可选）用于 etcd 身份验证的密码。<br>
• cacert：（可选）ca 证书。如果存在，它将启用验证。<br>
• cert：（可选）带有客户端证书的文件。<br>
• key：（可选）带有客户端密钥的文件。如果密钥是cert 的一部分，则可以为空。<br>
<b>7.7 Etcdv3</b><br>
如果您希望Patroni通过协议版本3与Etcd集群一起使用，则需要使用Patroni配置文件中的etcd3 部分。 所有配置参数与etcd相同。<br>
警告：使用协议版本 2 创建的密钥在协议版本 3 中不可见，反之亦然，因此无法仅通过更新 Patroni 配置文件来切换etcd到etcd3。<br>
<b>7.8 ZooKeeper</b><br>
• hosts：ZooKeeper 集群成员列表，格式为：['host1:port1', 'host2:port2', 'etc...']。<br>
• use_ssl：（可选）是否使用 SSL。默认为false. 如果设置为false，则忽略所有 SSL 特定参数。<br>
• cacert：（可选）CA 证书。如果存在，它将启用验证。<br>
• cert：（可选）带有客户端证书的文件。<br>
• key：（可选）带有客户端密钥的文件。<br>
• key_password：（可选）客户端密钥密码。<br>
• verify：（可选）是否验证证书。默认为true.<br>

提示：需要安装kazoo>=2.6.0以支持 SSL。<br>

<b>7.9 Exhibitor</b><br>
• hosts : Exhibitor (ZooKeeper) 节点的初始列表，格式为：'host1,host2,etc...'。每当 Exhibitor (ZooKeeper) 集群拓扑发生变化时，此列表就会自动更新。<br>
• poll_interval：应从 Exhibitor 更新 ZooKeeper 和 Exhibitor 节点列表的频率。<br>
• port：Exhibitor端口。<br>
<b>7.10 Kubernetes</b><br>
• bypass_api_service：（可选）在与 Kubernetes API 通信时，Patroni 通常依赖于kubernetes服务，其地址通过KUBERNETES_SERVICE_HOST环境变量暴露在pod中。如果bypass_api_service设置为true，Patroni 将解析服务背后的 API 节点列表并直接连接到它们。<br>
• namespace：（可选）运行 Patroni pod 的 Kubernetes 命名空间。默认值为default。<br>
• labels：标签格式 {label1: value1, label2: value2}。这些标签将用于查找与当前群集关联的现有对象（Pods和Endpoints或ConfigMaps）。Patroni还将在创建的每个对象（端点或ConfigMap）上设置它们。<br>
• scope_label：（可选）包含集群名称的标签名称。默认值为cluster-name。<br>
• role_label：（可选）包含角色（主或副本）的标签的名称。Patroni将在运行它的Pod上设置此标签。默认为role。<br>
• use_endpoints：（可选）如果设置为 true，Patroni 将使用 Endpoints 而不是 ConfigMaps 来运行领导者选举并保持集群状态。<br>
• pod_ip：（可选）Patroni正在运行中的Pod的IP地址。启用use_endpoints时，此值是必需的；当容器的PostgreSQL提升时，该值用于填充领导者端点子集。<br>
• ports：（可选）如果Service对象具有端口的名称，则必须在Endpoint对象中显示相同的名称，否则服务将无法工作。示例如果你服务定义为 {Kind: Service, spec: {ports: [{name: postgresql, port: 5432, targetPort: 5432}]}}，那么你需要设置kubernetes.ports: [{"name": "postgresql", "port": 5432}] 。Patroni将使用它来更新领导者端点的子集。仅在设置kubernetes.use_endpoints时使用此参数。<br>
• cacert：（可选）指定带有 CA_BUNDLE 文件的文件，其中包含在验证 Kubernetes API SSL 证书时要使用的受信任 CA 的证书。如果未提供，patroni将使用 ServiceAccount 密钥提供的值。<br>
<b>7.11 Raft</b><br>
• self_addr：ip:port监听 Raft 连接。在self_addr必须从集群中的其他节点进行访问。如果不设置，节点将不参与共识。<br>
• bind_addr：（可选）ip:port监听 Raft 连接。如果未指定，self_addr将使用 。<br>
• partner_addrs : 集群中其他 Patroni 节点的列表，格式为：['ip1:port', 'ip2:port', 'etc...']<br>
• data_dir : 存放 Raft 日志和快照的目录。如果未指定，则使用当前工作目录。<br>
• password：（可选）使用指定密码加密 Raft 流量，需要cryptographypython 模块。<br>
关于 Raft 实现的简短常见问题解答<br>
&nbsp;&nbsp;问：如何列出所有提供共识的节点？<br>
&nbsp;&nbsp;答:syncobj_admin -conn host:port -status，其中host:port是一个集群节点的地址<br>
&nbsp;&nbsp;问：作为共识一部分的节点已经消失，我无法为其他节点重复使用相同的 IP。如何从共识中删除这个节点？<br>
&nbsp;&nbsp;答：syncobj_admin -conn host:port -remove host2:port2 ，其中 host2:port2就是你要移除的共识节点的地址。<br>
&nbsp;&nbsp;问：从哪里获得syncobj_admin实用程序？<br>
&nbsp;&nbsp;答: 和pysyncobj模块一起安装（python RAFT实现），是Patroni依赖。<br>
&nbsp;&nbsp;问：是否可以在不加入共识的情况下运行 Patroni 节点？<br>
&nbsp;&nbsp;答：是的，只需raft.self_addr从 Patroni 配置中注释掉或删除即可。<br>
&nbsp;&nbsp;问: Patroni 和 PostgreSQL 可以只在两个节点上运行吗？<br>
&nbsp;&nbsp;答：是的，你可以在第三个节点上运行patroni_raft_controller（没有 Patroni 和 PostgreSQL）。在这样的设置中，可以暂时失去一个节点而不会影响主节点。<br>
<b>7.12 PostgreSQL</b><br>
• PostgreSQL：<br>
&nbsp;&nbsp;-uthentication：<br>
&nbsp;&nbsp;&nbsp;&nbsp;-- superuser：<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---username：超级用户的名称，在初始化（initdb）期间设置，稍后由 Patroni 用于连接到 postgres。<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---password : 超级用户的密码，在初始化 (initdb) 时设置。<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---sslmode：（可选）映射为sslmode连接参数，它允许客户端指定与服务器的 TLS 协商模式的类型。有关每种模式如何工作的更多信息，请访问PostgreSQL 文档。默认模式是prefer。<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---sslkey：（可选）映射为sslkey连接参数，该参数指定与客户端证书一起使用的密钥的位置。<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---sslpassword：（可选）映射为sslpassword连接参数, 它指定sslkey中指定的密钥密码。<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---sslcert：（可选）映射为sslcert连接参数，该参数指定客户端证书的位置。<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---sslrootcert：（可选）映射为sslrootcert连接参数，该参数指定包含客户端将用于验证服务器证书的一个或多个证书颁发机构 (CA) 证书的文件的位置。<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---sslcrl：（可选）映射为sslcrl连接参数，该参数指定包含证书吊销列表的文件的位置。客户端将拒绝连接到此列表中存在证书的任何服务器。<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---gssencmode：（可选）映射为gssencmode连接参数, 它决定是否以什么优先级与服务器协商安全的GSS TCP/IP连接。<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---channel_binding：（可选）映射为channel_binding连接参数，它控制客户端对通道绑定的使用。<br>
&nbsp;&nbsp;&nbsp;&nbsp;--replication：<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---username：复制用户名；用户将在初始化期间创建。副本将使用此用户通过流式复制访问主服务器<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---password：复制用户密码；用户将在初始化期间创建。<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---sslmode：（可选）映射为sslmode连接参数，它允许客户端指定与服务器的 TLS 协商模式的类型。有关每种模式如何工作的更多信息，请访问PostgreSQL 文档。默认模式是prefer。<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---sslkey：（可选）映射为sslkey连接参数，该参数指定与客户端证书一起使用的密钥的位置。<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---sslpassword：（可选）映射为sslpassword连接参数,指定sslkey中指定的密钥密码。<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---sslcert：（可选）映射为sslcert连接参数，该参数指定客户端证书的位置。<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---sslrootcert：（可选）映射为sslrootcert连接参数，该参数指定包含客户端将用于验证服务器证书的一个或多个证书颁发机构 (CA) 证书的文件的位置。<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---sslcrl：（可选）映射为sslcrl连接参数，该参数指定包含证书吊销列表的文件的位置。客户端将拒绝连接到此列表中存在证书的任何服务器。<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---gssencmode：（可选）映射为gssencmode连接参数,它决定是否以什么优先级与服务器协商安全的GSS TCP/IP连接。<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---channel_binding：（可选）映射为channel_binding连接参数，它控制客户端对通道绑定的使用。<br>
&nbsp;&nbsp;&nbsp;&nbsp;--rewind<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---username：使用pg_rewind的用户的用户名; 在postgres 11及以上版本初始化时创建用户，并授予所有必要的权限。<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---password：使用pg_rewind的用户的用户密码; 初始化时创建用户<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---sslmode：（可选）映射为sslmode连接参数，它允许客户端指定与服务器的 TLS 协商模式的类型。有关每种模式如何工作的更多信息，请访问PostgreSQL 文档。默认模式是prefer。<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---sslkey：（可选）映射为sslkey连接参数，该参数指定与客户端证书一起使用的密钥的位置。<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---sslpassword：（可选）映射为sslpassword连接参数,指定sslkey中指定密钥的密码<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---sslcert：（可选）映射为sslcert连接参数，该参数指定客户端证书的位置。<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---sslrootcert：（可选）映射为sslrootcert连接参数，该参数指定包含客户端将用于验证服务器证书的一个或多个证书颁发机构 (CA) 证书的文件的位置。<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---sslcrl：（可选）映射为sslcrl连接参数，该参数指定包含证书吊销列表的文件的位置。客户端将拒绝连接到此列表中存在证书的任何服务器。<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---gssencmode：（可选）映射为gssencmode连接参数,它决定是否以什么优先级与服务器协商安全的GSS TCP/IP连接<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;---channel_binding：（可选）映射为channel_binding连接参数，它控制客户端对通道绑定的使用。<br>
&nbsp;&nbsp;-callbacks : 在某些操作上运行的回调脚本。Patroni 将传递操作、角色和集群名称。（请参阅 scripts/aws.py 作为如何编写它们的示例。）<br>
&nbsp;&nbsp;&nbsp;&nbsp;--on_reload：在触发配置重新加载时运行此脚本。<br>
&nbsp;&nbsp;&nbsp;&nbsp;--on_restart：当 postgres 重新启动时运行这个脚本（不改变角色）。<br>
&nbsp;&nbsp;&nbsp;&nbsp;--on_role_change：当 postgres 被提升或降级时运行这个脚本。<br>
&nbsp;&nbsp;&nbsp;&nbsp;--on_start：在 postgres 启动时运行此脚本。<br>
&nbsp;&nbsp;&nbsp;&nbsp;--on_stop：当 postgres 停止时运行这个脚本。<br>
&nbsp;&nbsp;-connect_address：IP 地址 + 端口，通过该端口可以从其他节点和应用程序访问 Postgres。<br>
• create_replica_methods：将Patroni节点转换为新副本的创建方法的有序列表。“basebackup”是默认方法；假定其他方法引用脚本，每个脚本都配置为自己的配置项。有关更多说明，请参见自定义副本创建方法文档。<br>
&nbsp;&nbsp;-data_dir : Postgres 数据目录的位置，可以是现有的，也可以是由 Patroni 初始化的。<br>
&nbsp;&nbsp;-config_dir : Postgres 配置目录的位置，默认为数据目录。必须可由 Patroni 写入。<br>
&nbsp;&nbsp;-bin_dir：PostgreSQL 二进制文件的路径（pg_ctl、pg_rewind、pg_basebackup、postgres）。默认值是一个空字符串，这意味着 PATH 环境变量将用于查找可执行文件。<br>
&nbsp;&nbsp;-listen：Postgres 监听的IP地址+ 端口号; 如果您正在使用流复制，则必须可以从群集中的其他节点进行访问。只要将端口号附加在最后一个冒号后面，即可使用多个逗号分隔的地址，即监听：127.0.0.1,127.0.0.2：5432。Patroni将使用此列表中的第一个地址建立与PostgreSQL节点的本地连接。<br>
&nbsp;&nbsp;-use_unix_socket：指定Patroni使用unix套接字连接到集群。默认为false。如果unix_socket_directories 被定义，如果Patroni将使用其中的第一个合适的值连接到集群，没有合适的选项，回退到tcp。如果 unix_socket_directories没有定义在 postgresql.parameters中, Patroni将假定应使用默认值，并从连接参数中省略host。<br>
&nbsp;&nbsp;-use_unix_socket_repl：指定 Patroni 应首选使用unix套接字进行复制用户集群连接。默认值为false。如果unix_socket_directories已定义，Patroni 将使用其中的第一个合适的值连接到集群，如果没有合适的值，则回退到 tcp。如果postgresql.parameters中没有指定unix_socket_directories，那么Patroni将假定应该使用默认值，并从连接参数中省略host。<br>
&nbsp;&nbsp;-pgpass : .pgpass密码文件的路径。Patroni 在执行 pg_basebackup、post_init 脚本之前以及在其他一些情况下创建这个文件。该位置必须可由 Patroni 写入。<br>
&nbsp;&nbsp;-recovery_conf：配置follower时写入recovery.conf 的附加配置设置。<br>
&nbsp;&nbsp;-custom_conf：可选定制的postgresql.conf 文件路径, 将用于代替postgresql.base.conf。该文件必须存在于所有群集节点上，并且可由PostgreSQL读取，并将包含在实际的postgresql.conf中。请注意，Patroni不会监视此文件的更改，也不会备份它。但是Patroni自己的配置工具仍然可以覆盖其设置-有关详细信息，请参见动态配置。 <br>
&nbsp;&nbsp;-parameters：Postgres 的配置设置列表。其中许多是复制工作所必需的。<br>
&nbsp;&nbsp;-pg_hba：Patroni 将用于生成pg_hba.conf. 此参数的优先级高于bootstrap.pg_hba。与动态配置一起，它简化了pg_hba.conf.<br>
&nbsp;&nbsp;&nbsp;&nbsp;-- host all all 0.0.0.0/0 md5.<br>
&nbsp;&nbsp;&nbsp;&nbsp;-- host replication replicator 127.0.0.1/32 md5: 复制需要这样一行代码。<br>
&nbsp;&nbsp;-pg_ident：Patroni 将用于生成pg_ident.conf. 与动态配置一起，它简化了pg_ident.conf.<br>
&nbsp;&nbsp;&nbsp;&nbsp;-- mapname1 systemname1 pguser1。<br>
&nbsp;&nbsp;&nbsp;&nbsp;-- mapname1 systemname2 pguser2。<br>
&nbsp;&nbsp;-pg_ctl_timeout : pg_ctl在启动，停止或重新启动时应等待多长时间。 默认值为60秒。<br>
&nbsp;&nbsp;-use_pg_rewind：尝试在先前领导者作为副本加入群集时在先前领导者上使用pg_rewind。<br>
&nbsp;&nbsp;-remove_data_directory_on_rewind_failure：如果这个选项打开，Patroni将移除PostgreSQL数据目录并重新创建副本。否则将尽量跟随一个新领导者，默认为false。<br>
&nbsp;&nbsp;-remove_data_directory_on_diverged_timelines：如果Patroni注意到时间线不同并且以前的主服务器无法从新的主服务器开始流式传输，则Patroni将删除PostgreSQL数据目录并重新创建副本。当无法使用pg_rewind时，此选项很有用。默认值是false。 <br>
&nbsp;&nbsp;-replica_method：对于除 basebackup 之外的每个 create_replica_methods，您将添加一个同名的配置部分。至少，这应该包括“命令”以及要执行的实际脚本的完整路径。其他配置参数将以“parameter=value”的形式传递给脚本。<br>
&nbsp;&nbsp;-pre_promote：在获得领导锁之后，但在副本提升之前，在故障转移期间执行的围栏脚本。如果脚本以非零代码退出，那么Patroni不会升级复制副本并从DCS中删除领导密钥 <br>
<b>7.13 REST API</b><br>
• restapi<br>
&nbsp;&nbsp;-connect_address：IP 地址（或主机名）和端口，用于访问 Patroni 的REST API. 集群的所有成员都必须能够连接到这个地址，所以除非 Patroni 设置是用于本地主机内部的演示，否则这个地址必须是非“localhost”或环回地址（即：“localhost”或“127.0 .0.1”）。它可以作为 HTTP 健康检查的端点（阅读下面关于“listen”REST API 参数的内容），也可以作为用户查询（直接或通过 REST API），以及领导人选举中集群成员进行的健康检查（例如，确定主节点是否仍在运行，或者是否有一个节点的 WAL 位置在执行查询的节点之前；等等）将connect_address放入DCS的成员键中，从而可以将成员名称转换为地址以连接到其REST API。<br>
&nbsp;&nbsp;-listen：Patroni 将为 REST API监听的IP 地址（或主机名）和端口 - 如上所述，还在参与节点之间提供相同的健康检查和集群消息传递。为 HAProxy（或任何其他能够执行 HTTP“OPTION”或“GET”检查的负载均衡器）提供健康检查信息。<br>
&nbsp;&nbsp;-authentication: (可选)<br>
&nbsp;&nbsp;&nbsp;&nbsp;--username : 基本验证的用户名，用于保护不安全的 REST API 端点。<br>
&nbsp;&nbsp;&nbsp;&nbsp;--password : 基本验证的密码，用于保护不安全的 REST API 端点。<br>
&nbsp;&nbsp;-certfile：（可选）：指定带有 PEM 格式证书的文件。如果未指定 certfile 或将其留空，则 API 服务器将在没有 SSL 的情况下工作。<br>
&nbsp;&nbsp;-keyfile：（可选）：指定具有 PEM 格式的密钥的文件。<br>
&nbsp;&nbsp;-keyfile_password：（可选）：指定用于解密密钥文件的密码。<br>
&nbsp;&nbsp;-cafile：（可选）：指定带有 CA_BUNDLE 的文件，其中包含在验证客户端证书时要使用的受信任 CA 的证书。<br>
&nbsp;&nbsp;-ciphers：（可选）：指定允许的密码套件（例如“ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES128-GCM-SHA256:!SSLv1:!SSLv2:!SSLv3:!TLSv1:!TLSv1.1”)<br>
&nbsp;&nbsp;-verify_client : (可选): none(默认),optional或required. 当noneREST API 不会检查客户端证书时。当required所有 REST API 调用都需要客户端证书时。当optional所有不安全的 REST API 端点都需要客户端证书时。当required使用时，如果证书签名验证成功，则客户端验证成功。对于optional客户端证书将只检查PUT，POST，PATCH，和DELETE请求。<br>
&nbsp;&nbsp;-allowlist：（可选）：指定允许调用不安全REST API端点的主机集。单个元素可以是主机名、IP地址或使用CIDR表示法的网络地址。默认情况下，将使用allow all。如果设置了allowlist或allowlist_include_members，则拒绝未包含的任何内容。<br>
&nbsp;&nbsp;-allowlist_include_members：（可选）：如果设置为true它允许从在 DCS 中注册的其他集群成员访问不安全的 REST API 端点（IP 地址或主机名取自成员api_url）。请注意，操作系统可能会使用不同的 IP 进行传出连接。<br>
&nbsp;&nbsp;-http_extra_headers：（可选）：HTTP 标头让 REST API 服务器通过 HTTP 响应传递附加信息。<br>
&nbsp;&nbsp;-https_extra_headers：（可选）：启用TLS后，HTTPS headers可以让REST API服务器通过HTTP响应传递其他信息。这还将传递http_extra_headers中设置的其他信息<br>
这是http_extra_headers和https_extra_headers 的示例：<br>
<table border="1"><tr><th align="left">
restapi:<br>
&nbsp;&nbsp;listen: &lt;listen&gt;<br>
&nbsp;&nbsp;connect_address: &lt;connect_address&gt;<br>
&nbsp;&nbsp;authentication:<br>
&nbsp;&nbsp;&nbsp;&nbsp;username: &lt;username&gt;<br>
&nbsp;&nbsp;&nbsp;&nbsp;password: &lt;password&gt;<br>
&nbsp;&nbsp;http_extra_headers:<br>
&nbsp;&nbsp;&nbsp;&nbsp;'X-Frame-Options': 'SAMEORIGIN'<br>
&nbsp;&nbsp;&nbsp;&nbsp;'X-XSS-Protection': '1; mode=block'<br>
&nbsp;&nbsp;&nbsp;&nbsp;'X-Content-Type-Options': 'nosniff'<br>
&nbsp;&nbsp;cafile: &lt;ca file&gt;<br>
&nbsp;&nbsp;certfile: &lt;cert&gt;<br>
&nbsp;&nbsp;keyfile: &lt;key&gt;<br>
&nbsp;&nbsp;https_extra_headers:<br>
&nbsp;&nbsp;&nbsp;&nbsp;'Strict-Transport-Security': 'max-age=31536000; includeSubDomains'
</th></tr></table><br>
<b>7.14 CTL</b><br>
•ctl : （可选）<br>
&nbsp;&nbsp;-insecure : 允许连接到 REST API 而无需验证 SSL 证书。<br>
&nbsp;&nbsp;-cacert : 指定带有CA_BUNDLE文件的文件或带有受信任CA证书的目录，以在验证REST API SSL证书时使用。如果未提供，则patronictl将使用为REST API “cafile”参数提供的值。<br>
&nbsp;&nbsp;-certfile : 指定带有客户端证书的 PEM 格式的文件。如果没有提供，patronict将使用为 REST API“certfile”参数提供的值。<br>
&nbsp;&nbsp;-keyfile : 指定具有 PEM 格式的客户端密钥的文件。如果没有提供，patronictl将使用为 REST API “keyfile”参数提供的值。<br>
<b>7.15 Watchdog</b><br>
•mode : off, automatic 或者required. 当设置为off ，那么watchdog禁用。当设置为automatic，WatchDog（如果有）将被使用，但如果没有则将被忽略<br>
•device : watchdog设备的路径。默认为 /dev/watchdog.<br>
•Safety_margin : 领导者密钥到期和WatchDog触发之间的安全余量秒数。<br>
<b>7.16 Tags</b><br>
•nofailover : true或false，控制是否允许此节点参与领导者竞赛并成为领导者。默认为false<br>
•clonefrom : true 或false。如果设置为true，其他节点可能更喜欢使用该节点进行引导（从pg_basebackup获取）。如果有多个节点的clonefrom标签设置为true，则将随机选择要从中引导的节点。默认为false。<br>
•noloadbalance : true或false。如果设置true为该节点，则将为REST API 健康检查返回 HTTP 状态代码 503 ，因此将从负载平衡中排除。默认为false。<br>
•replicatefrom : 另一个副本的 IP 地址/主机名。用于支持级联复制。<br>
•nosync : true或false. 如果设置true为该节点将永远不会被选为同步副本。<br>
除了这些预定义的标签，您还可以添加自己的标签：<br>
•key1 : true<br>
•key2 : false<br>
•key3 : 1.4<br>
•key4 : "RandomString"<br>
标签在REST API和用户列表中可见。您还可以使用这些标签检查实例的运行状况。如果没有为实例定义标签，或者如果相应的值与查询值不匹配，它将返回HTTP状态代码503。<br>

[patroni-doccn](https://github.com/postgres-cn/patroni-doccn/blob/main/README.md)
