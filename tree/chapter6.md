# 第6章  环境配置设置<br>
可以使用系统环境变量覆盖 Patroni 配置文件中定义的一些配置参数。 本文档列出了 Patroni 处理的所有环境变量。 通过这些变量设置的值始终优先于在 Patroni 配置文件中设置的值。<br>
<b>6.1 全局/通用</b><br>
• PATRONI_CONFIGURATION：可以通过PATRONI_CONFIGURATION 环境变量为Patroni 设置整个配置。 在这种情况下，不会考虑任何其他环境变量！<br>
• PATRONI_NAME：当前运行Patroni 实例的节点的名称。 对于集群必须是唯一的。<br>
• PATRONI_NAMESPACE：Patroni保留有关集群的信息在配置存储的路径中。 默认值：“/service”<br>
• PATRONI_SCOPE：集群名称<br>
<b>6.2 日志</b><br>
• PATRONI_LOG_LEVEL：设置一般日志记录级别。 默认值为 INFO（请参阅 Python 日志记录文档 16.6.2）<br>
• PATRONI_LOG_TRACEBACK_LEVEL：设置回溯可见的级别。 默认值为错误。 如果您只想在启用 PATRONI_LOG_LEVEL=DEBUG 时查看回溯，请将其设置为 DEBUG。<br>
• PATRONI_LOG_FORMAT：设置日志格式字符串。 默认值为 %(asctime)s %(levelname)s: %(message)s（参见 LogRecord 属性）<br>
• PATRONI_LOG_DATEFORMAT：设置日期时间格式字符串。 （请参阅 formatTime() 文档 https://docs.python.org/3.6/library/logging.html#logging.Formatter.formatTime）<br>
• PATRONI_LOG_MAX_QUEUE_SIZE：Patroni 使用两步日志记录。 日志记录被写入内存队列，并且有一个单独的线程将它们从队列中拉出并写入 stderr 或文件。 内部队列的最大大小默认限制为 1000 条记录，足以保留过去 1h20m 的日志。<br>
• PATRONI_LOG_DIR：应用程序日志写入的目录。 该目录必须存在并且可由执行 Patroni 的用户写入。 如果您设置此 env 变量，默认情况下应用程序将保留 4 个 25MB 的日志。 您可以使用 PATRONI_LOG_FILE_NUM 和 PATRONI_LOG_FILE_SIZE 调整这些保留值（请参见下文）。<br>
• PATRONI_LOG_FILE_NUM：要保留的应用程序日志的数量。<br>
• PATRONI_LOG_FILE_SIZE：触发日志滚动的patroni.log 文件的大小（以字节为单位）。 <br>
• PATRONI_LOG_LOGGERS：重新定义每个python 模块的日志记录级别。 示例 PATRONI_LOG_LOGGERS="{patroni.postmaster: WARNING, urllib3: DEBUG}"<br>
<b>6.3 引导配置</b><br>
可以在新集群成功初始化后立即创建新的数据库用户。 此过程由以下变量定义：<br>
• PATRONI_&lt;username&gt;_PASSWORD=’&lt;password&gt;’<br>
• PATRONI_&lt;username&gt;_OPTIONS=’list,of,options’<br>
示例：定义 PATRONI_admin_PASSWORD=strongpasswd 和 PATRONI_admin_OPTIONS='createrole,createdb' 将导致使用密码 strongpasswd 创建用户 admin，允许创建其他用户和数据库。<br>
<b>6.4 Consul</b><br>
• PATRONI_CONSUL_HOST：Consul 本地代理的主机：端口。 <br>
• PATRONI_CONSUL_URL：Consul 本地代理的 URL，格式为：http(s)://host:port <br>
• PATRONI_CONSUL_PORT：（可选）Consul端口<br>
• PATRONI_CONSUL_SCHEME：（可选）http 或 https，默认为 http <br>
• PATRONI_CONSUL_TOKEN：（可选）ACL 令牌 <br>
• PATRONI_CONSUL_VERIFY：（可选）是否验证 HTTPS 请求的 SSL 证书 <br>
• PATRONI_CONSUL_CACERT：（可选）ca 证书。如果存在，它将启用验证。 <br>
• PATRONI_CONSUL_CERT：（可选）包含客户端证书的文件 <br>
• PATRONI_CONSUL_KEY：（可选）带有客户端密钥的文件。如果密钥是证书的一部分，则可以为空。 <br>
• PATRONI_CONSUL_DC：（可选）要与之通信的数据中心。默认情况下使用主机的数据中心。 <br>
• PATRONI_CONSUL_CONSISTENCY：（可选）选择consul 一致性模式。可能的值为 default、consistent 或 stale（consul API 参考中的更多详细信息） <br>
• PATRONI_CONSUL_CHECKS：（可选）用于会话的Consul 健康检查列表。默认情况下使用空列表。<br>
• PATRONI_CONSUL_REGISTER_SERVICE：（可选）是否使用scope参数定义的名称注册服务，并根据节点的角色标记master、replica或standby-leader。默认为false <br>
• PATRONI_CONSUL_SERVICE_CHECK_INTERVAL：（可选）对注册的 url 执行健康检查的频率<br>
<b>6.5 Etcd</b><br>
• PATRONI_ETCD_PROXY：etcd 的代理网址。如果您使用代理连接到 etcd，请使用此参数而不是 PATRONI_ETCD_URL <br>
• PATRONI_ETCD_URL：etcd 的url，格式为：http(s)://(username:password@)host:port • PATRONI_ETCD_HOSTS：etcd 端点列表，格式为‘host1:port1’、‘host2:port2’等。 . . <br>
• PATRONI_ETCD_USE_PROXIES：如果此参数设置为true，Patroni将把主机视为一个代理列表，不会执行etcd集群的拓扑发现，而是坚持使用固定的主机列表。<br>
• PATRONI_ETCD_PROTOCOL：http 或 https，如果未指定，则使用 http。如果指定了 url 或proxy - 将从他们那里获取协议。 <br>
• PATRONI_ETCD_HOST：etcd端点格式：host:port。<br>
• PATRONI_ETCD_SRV：搜索 SRV 记录以进行集群自动发现的域。 Patroni 将尝试查询指定域的这些 SRV 服务名称（按该顺序直到第一次成功）：_etcd-client-ssl、_etcd-client、_etcd-ssl、_etcd、_etcd-server-ssl、_etcd-server。如果检索到 _etcd-server-ssl 或 _etcd-server 的 SRV 记录，则使用 ETCD 对等协议查询 ETCD 以获取可用成员。否则将使用 SRV 记录中的主机。 <br>
• PATRONI_ETCD_SRV_SUFFIX：配置发现期间查询的SRV 名称的后缀。使用此标志来区分同一域下的多个 etcd 集群。仅适用 与 PATRONI_ETCD_SRV 结合使用。例如，如果设置了 PATRONI_ETCD_SRV_SUFFIX=foo 和 PATRONI_ETCD_SRV=example.org，则会进行以下 DNS SRV 查询：_etcd-client-ssl-foo._tcp.example.com（对于每个可能的 ETCD SRV 服务名称，依此类推）。 <br>
• PATRONI_ETCD_USERNAME：用于etcd 身份验证的用户名。 <br>
• PATRONI_ETCD_PASSWORD：etcd 认证的密码。 <br>
• PATRONI_ETCD_CACERT：ca 证书。如果存在，它将启用验证。 <br>
• PATRONI_ETCD_CERT：包含客户端证书的文件。 <br>
• PATRONI_ETCD_KEY：包含客户端密钥的文件。如果密钥是证书的一部分，则可以为空。<br>
<b>6.6 Etcdv3</b><br>
Etcdv3 的环境名称与 Etcd 类似，您只需要在变量名称中使用 ETCD3 而不是 ETCD。 示例：PATRONI_ETCD3_HOST、PATRONI_ETCD3_CACERT 等。<br>
<table border="1"><tr><th align="left">
警告：使用协议版本2创建的密钥在协议版本3中不可见，反之亦然，<br>
因此仅通过更新Patroni配置就无法从Etcd切换到Etcdv3。
</th></tr></table><br>
<b>6.7 ZooKeeper</b><br>
• PATRONI_ZOOKEEPER_HOSTS：ZooKeeper 集群成员的逗号分隔列表：“‘host1:port1’,‘host2:port2’等。 . . ’”。 引用每个实体很重要！ <br>
• PATRONI_ZOOKEEPER_USE_SSL：（可选）是否使用SSL默认为false 如果设置为false，将忽略所有SSL特定参数。<br>
• PATRONI_ZOOKEEPER_CACERT：（可选）CA 证书。 如果存在，它将启用验证。 <br>
• PATRONI_ZOOKEEPER_CERT：（可选）带有客户端证书的文件。 <br>
• PATRONI_ZOOKEEPER_KEY：（可选）带有客户端密钥的文件。 <br>
• PATRONI_ZOOKEEPER_KEY_PASSWORD：（可选）客户端密钥密码。 <br>
• PATRONI_ZOOKEEPER_VERIFY：（可选）是否验证证书。 默认为true。<br>
<table border="1"><tr><th align="left">
注意：需要安装kazoo>=2.6.0才能支持SSL。
</th></tr></table><br>
<b>6.8 Exhibitor</b><br>
• PATRONI_EXHIBITOR_HOSTS：Exhibitor (ZooKeeper) 节点的初始列表，格式为：‘host1、host2 等。 . . ’。 每当 Exhibitor (ZooKeeper) 集群拓扑发生变化时，此列表就会自动更新。 <br>
• PATRONI_EXHIBITOR_PORT：Exhibitor端口。<br>
<b>6.9 Kubernetes</b><br>
• PATRONI_KUBERNETES_BYPASS_API_SERVICE：（可选）在与Kubernetes API 通信时，Patroni 通常依赖于kubernetes 服务，其地址通过KUBERNETES_SERVICE_HOST 环境变量暴露在pod 中。如果 PATRONI_KUBERNETES_BYPASS_API_SERVICE 设置为 true，Patroni 将解析服务背后的 API 节点列表并直接连接到它们。 <br>
• PATRONI_KUBERNETES_NAMESPACE：（可选）运行Patroni pod 的Kubernetes 命名空间。默认值为default。 <br>
• PATRONI_KUBERNETES_LABELS：格式为{label1: value1, label2: value2} 的标签。这些标签将用于查找与当前集群关联的现有对象（Pod 和端点或ConfigMap）。此外，Patroni 将在它创建的每个对象（端点或 ConfigMap）上设置它们。 <br>
• PATRONI_KUBERNETES_SCOPE_LABEL：（可选）包含集群名称的标签名称。默认值为集群名称。 <br>
• PATRONI_KUBERNETES_ROLE_LABEL：（可选）包含Postgres 角色（主或副本）的标签名称。 Patroni 会在它运行的 pod 上设置这个标签。默认值为 role。 <br>
• PATRONI_KUBERNETES_USE_ENDPOINTS：（可选）如果设置为true，Patroni 将使用Endpoints 而不是ConfigMaps 来运行leader 选举并保持集群状态。 <br>
• PATRONI_KUBERNETES_POD_IP：（可选）正在运行的Pod Patroni 的IP 地址。启用PATRONI_KUBERNETES_USE_ENDPOINTS 时需要此值，用于在提升Pod 的PostgreSQL 时填充leader 端点子集。<br>
• PATRONI_KUBERNETES_PORTS：（可选）如果服务对象具有端口名称，则端点对象中必须出现相同的名称，否则服务将无法工作。 例如，如果您的服务定义为 {Kind: Service, spec: {ports: [{name: postgresql, port: 5432, targetPort: 5432}]}}，那么你必须设置 PATRONI_KUBERNETES_PORTS='[{"name": "postgresql", "port": 5432}]' 并且 Patroni 将使用它来更新领导者端点的子集。 仅当设置了 PATRONI_KUBERNETES_USE_ENDPOINTS 时才使用此参数。 <br>
• PATRONI_KUBERNETES_CACERT：（可选）指定带有CA_BUNDLE 文件的文件，其中包含在验证Kubernetes API SSL 证书时要使用的可信CA 证书。 如果未提供，Patroni将使用 ServiceAccount 密钥提供的值。<br>
<b>6.10 Raft</b><br>
• PATRONI_RAFT_SELF_ADDR：ip:port 监听 Raft 连接。 self_addr 必须可以从集群的其他节点访问。 如果不设置，节点将不参与共识。 <br>
• PATRONI_RAFT_BIND_ADDR：（可选）ip:port 用于侦听 Raft 连接。 如果未指定，将使用 self_addr。 <br>
• PATRONI_RAFT_PARTNER_ADDRS：集群中其他 Patroni 节点的列表，格式为“'ip1:port1'， 'ip2:port2'"。引用每个实体很重要！ <br>
• PATRONI_RAFT_DATA_DIR：存放Raft 日志和快照的目录。 如果未指定，则使用当前工作目录。 <br>
• PATRONI_RAFT_PASSWORD：（可选）使用指定的密码对Raft通信进行加密，需要使用Python中的cryptography模块。<br>
<b>6.11 PostgreSQL</b><br>
• PATRONI_POSTGRESQL_LISTEN：Postgres监听的IP地址+ 端口号。 允许使用多个逗号分隔的地址，只要端口组件用冒号附加到最后一个地址之后，如 listen: 127.0.0.1,127.0.0.2:5432。 Patroni 将使用此列表中的第一个地址来建立到 PostgreSQL 节点的本地连接。 <br>
• PATRONI_POSTGRESQL_CONNECT_ADDRESS：IP 地址+ 端口，通过该端口可以从其他节点和应用程序访问 Postgres。 <br>
• PATRONI_POSTGRESQL_DATA_DIR：Postgres 数据目录的位置，可以是现有的，也可以是由Patroni 初始化的。 <br>
• PATRONI_POSTGRESQL_CONFIG_DIR：Postgres 配置目录的位置，默认为数据目录。 必须可由 Patroni 写入。<br>
• PATRONI_POSTGRESQL_BIN_DIR：PostgreSQL 二进制文件的路径。 (pg_ctl, pg_rewind, pg_basebackup,postgres) 默认值是一个空字符串，意味着 PATH 环境变量将用于查找可执行文件。 <br>
• PATRONI_POSTGRESQL_PGPASS：.pgpass 密码文件的路径。 Patroni 在执行 pg_basebackup 之前和在其他一些情况下创建这个文件。 该位置必须可由 Patroni 写入。 <br>
• PATRONI_REPLICATION_USERNAME：复制用户名； 用户将在初始化期间创建。 副本将使用此用户通过流式复制访问主服务器 <br>
• PATRONI_REPLICATION_PASSWORD：复制密码； 用户将在初始化期间创建。<br>
• PATRONI_REPLICATION_SSLMODE：（可选）映射到sslmode连接参数，它允许客户端指定与服务器的 TLS 协商模式的类型。有关每种模式如何工作的更多信息，请访问PostgreSQL 文档。默认模式是prefer。<br>
• PATRONI_REPLICATION_SSLKEY：（可选）映射到sslkey连接参数，该参数指定与客户端证书一起使用的密钥的位置。<br>
• PATRONI_REPLICATION_SSLPASSWORD：（可选）映射到sslpassword连接参数，该参数指定PATRONI_REPLICATION_SSLKEY.<br>
• PATRONI_REPLICATION_SSLCERT：（可选）映射到sslcert连接参数，该参数指定客户端证书的位置。<br>
• PATRONI_REPLICATION_SSLROOTCERT：（可选）映射到sslrootcert连接参数，指定包含一个或多个证书颁发机构（CA）证书的文件的位置，客户端将使用该证书来验证服务器的证书。<br>
• PATRONI_REPLICATION_SSLCRL：（可选）映射到sslcrl连接参数，该参数指定包含证书吊销列表的文件的位置。客户端将拒绝连接到此列表中存在证书的任何服务器。<br>
• PATRONI_REPLICATION_GSSENCMODE：（可选）映射到gssencmode连接参数，该参数确定是否或以何种优先级与服务器协商安全 GSS TCP/IP 连接<br>
• PATRONI_REPLICATION_CHANNEL_BINDING：（可选）映射到channel_binding连接参数，该参数控制客户端对通道绑定的使用。<br>
• PATRONI_SUPERUSER_USERNAME：超级用户的名称，在初始化 (initdb) 期间设置，稍后由 Patroni 用于连接到 postgres。这个用户也被 pg_rewind 使用。<br>
• PATRONI_SUPERUSER_PASSWORD：超级用户的密码，在初始化 (initdb) 期间设置。<br>
• PATRONI_SUPERUSER_SSLMODE：（可选）映射到sslmode连接参数，它允许客户端指定与服务器的 TLS 协商模式的类型。有关每种模式如何工作的更多信息，请访问PostgreSQL 文档。默认模式是prefer。<br>
• PATRONI_SUPERUSER_SSLKEY：（可选）映射到sslkey连接参数，该参数指定与客户端证书一起使用的密钥的位置。<br>
• PATRONI_SUPERUSER_SSLPASSWORD：（可选）映射到sslpassword连接参数，该参数指定PATRONI_SUPERUSER_SSLKEY.<br>
• PATRONI_SUPERUSER_SSLCERT：（可选）映射到sslcert连接参数，该参数指定客户端证书的位置。<br>
• PATRONI_SUPERUSER_SSLROOTCERT：（可选）映射到sslrootcert连接参数，该参数指定包含客户端将用于验证服务器证书的一个或多个证书颁发机构 (CA) 证书的文件的位置。<br>
• PATRONI_SUPERUSER_SSLCRL：（可选）映射到sslcrl连接参数，该参数指定包含证书吊销列表的文件的位置。客户端将拒绝连接到此列表中存在证书的任何服务器。<br>
• PATRONI_SUPERUSER_GSSENCMODE：（可选）映射到gssencmode连接参数，该参数决定是否或以何种优先级与服务器协商安全 GSS TCP/IP 连接<br>
• PATRONI_SUPERUSER_CHANNEL_BINDING：（可选）映射到channel_binding连接参数，该参数控制客户端对通道绑定的使用。<br>
• PATRONI_REWIND_USERNAME : 使用pg_rewind的用户名;在 postgres 11版本以上初始化期间创建用户，并授予所有必要的权限。<br>
• PATRONI_REWIND_PASSWORD : 使用pg_rewind的用户密码; 在初始化过程中创建用户时被创建。<br>
• PATRONI_REWIND_SSLMODE：（可选）映射到sslmode连接参数，它允许客户端指定与服务器的 TLS 协商模式的类型。有关每种模式如何工作的更多信息，请访问PostgreSQL 文档。默认模式是prefer。<br>
• PATRONI_REWIND_SSLKEY：（可选）映射到sslkey连接参数，该参数指定与客户端证书一起使用的密钥的位置。<br>
• PATRONI_REWIND_SSLPASSWORD：（可选）映射到sslpassword连接参数，该参数指定PATRONI_REWIND_SSLKEY.<br>
• PATRONI_REWIND_SSLCERT：（可选）映射到sslcert连接参数，该参数指定客户端证书的位置。<br>
• PATRONI_REWIND_SSLROOTCERT： (可选) 映射sslrootcert连接参数，指定包含一个或多个证书颁发机构（CA）证书的文件的位置，客户端将使用这些证书验证服务器的证书。<br>
• PATRONI_REWIND_SSLCRL：（可选）映射到sslcrl连接参数，该参数指定包含证书吊销列表的文件的位置。客户端将拒绝连接到此列表中存在证书的任何服务器。<br>
• PATRONI_REWIND_GSSENCMODE：（可选）映射到gssencmode连接参数，该参数确定是否或以何种优先级与服务器协商安全 GSS TCP/IP 连接<br>
• PATRONI_REWIND_CHANNEL_BINDING：（可选）映射到channel_binding连接参数，该参数控制客户端对通道绑定的使用。<br>
<b>6.12 REST API</b><br>
• PATRONI_RESTAPI_CONNECT_ADDRESS：访问 REST API 的 IP 地址和端口。<br>
• PATRONI_RESTAPI_LISTEN：Patroni 将侦听的 IP 地址和端口，为 HAProxy 提供健康检查信息。<br>
• PATRONI_RESTAPI_USERNAME：用于保护不安全 REST API 端点的基本身份验证用户名。<br>
• PATRONI_RESTAPI_PASSWORD：用于保护不安全 REST API 端点的基本身份验证密码。<br>
• PATRONI_RESTAPI_CERTFILE：指定带有 PEM 格式证书的文件。如果未指定 certfile 或将其留空，则 API 服务器将在没有 SSL 的情况下工作。<br>
• PATRONI_RESTAPI_KEYFILE：指定具有 PEM 格式的密钥的文件。<br>
• PATRONI_RESTAPI_KEYFILE_PASSWORD：指定用于解密密钥文件的密码。<br>
• PATRONI_RESTAPI_CAFILE：指定带有 CA_BUNDLE 的文件，其中包含在验证客户端证书时要使用的受信任 CA 的证书。<br>
• PATRONI_RESTAPI_CIPHERS：（可选）指定允许的密码套件（例如“ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES128-GCM -SHA256:!SSLv1:!SSLv2:!SSLv3:!TLSv1:!TLSv1.1”）<br>
• PATRONI_RESTAPI_VERIFY_CLIENT：（none默认），optional或required。当设置为none 时，REST API 不会检查客户端证书时。当required所有 REST API 调用都需要客户端证书时。当optional所有不安全的 REST API 端点都需要客户端证书时。当required使用时，如果证书签名验证成功，则客户端验证成功。对于optional客户端证书将只检查PUT，POST，PATCH，和DELETE请求。<br>
• PATRONI_RESTAPI_ALLOWLIST：（可选）：指定允许调用不安全REST API端点的主机集。单个元素可以是主机名、IP地址或使用CIDR表示法的网络地址。默认情况下，将使用allow all。如果设置了allowlist 或 allowlist_include_members，则拒绝未包含的任何内容。<br>
• PATRONI_RESTAPI_ALLOWLIST_INCLUDE_MEMBERS：（可选）：如果设置为true它允许从在 DCS 中注册的其他集群成员访问不安全的 REST API 端点（IP 地址或主机名取自成员api_url）。请注意，操作系统可能会使用不同的 IP 进行传出连接。<br>
• PATRONI_RESTAPI_HTTP_EXTRA_HEADERS：（可选）HTTP 标头让 REST API 服务器通过 HTTP 响应传递附加信息。<br>
• PATRONI_RESTAPI_HTTPS_EXTRA_HEADERS：（可选）HTTPS 标头让 REST API 服务器在启用 TLS 时通过 HTTP 响应传递附加信息。这也将传递设置在http_extra_headers.<br>
<b>6.13 CTL</b><br>
• PATRONICTL_CONFIG_FILE：配置文件的位置。<br>
• PATRONI_CTL_INSECURE：允许连接到 REST API 而无需验证 SSL 证书。<br>
• PATRONI_CTL_CACERT：指定带有 CA_BUNDLE 文件的文件或带有可信 CA 证书的目录，以在验证 REST API SSL 证书时使用。如果没有提供，pavictl 将使用为 REST API“cafile”参数提供的值。<br>
• PATRONI_CTL_CERTFILE：指定具有 PEM 格式的客户端证书的文件。如果没有提供，pavictl 将使用为 REST API“certfile”参数提供的值。<br>
• PATRONI_CTL_KEYFILE：指定具有 PEM 格式的客户端密钥的文件。如果没有提供，pavictl 将使用为 REST API “keyfile”参数提供的值。<br>

[patroni-doccn](https://github.com/postgres-cn/patroni-doccn/blob/main/README.md)
