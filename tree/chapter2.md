<font size="48"><b>第2章Patroni配置</b></font><br>
Patroni配置存储在DCS（分布式配置存储）中。有3种类型的配置：<br>
<b>•动态配置。</b>这些选项可随时在DCS中设置。如果更改的选项不是启动配置的一部分，则它们将异步（在下一个唤醒周期中）应用于每个节点，然后重新加载。如果节点需要重新启动以应用配置（对于具有context postmaster的选项，如果其值已更改），则会在members.data JSON中设置一个特殊标志pending_restart标记。此外，节点状态还通过显示“restart_pending”：true来表明这一点。<br>
<b>•本地配置(patroni.yml). </b>这些选项在配置文件中定义，优先于动态配置。可以通过向Patroni进程发送SIGHUP、执行POST/reload REST-API请求或执行patronictl reload.在运行时（无需重新启动Patroni）更改和重新加载Patroni.yml。<br>
<b>•环境配置。</b>可以使用环境变量设置/覆盖某些“本地”配置参数。当您在动态环境中运行并且事先不知道某些参数时（例如，在docker内部运行时，不可能知道外部IP地址），环境配置非常有用。	<br>
本地配置可以是单个YAML文件或目录。当它是一个目录时，该目录中的所有YAML文件将按排序顺序逐个加载。如果在多个文件中定义了一个键，则以最后一个文件中出现的键为准。<br>
某些PostgreSQL参数在主数据库和副本数据库上必须具有相同的值。对于这些，在本地用户配置文件中或通过环境变量设置的值均无效。要更改或设置其值，必须更改DCS中的共享配置。以下是此类参数的实际列表以及默认值：<br>
• max_connections: 100 <br>
• max_locks_per_transaction: 64 <br>
• max_worker_processes: 8 <br>
• max_prepared_transactions: 0 <br>
• wal_level: hot_standby <br>
• wal_log_hints: on <br>
• track_commit_timestamp: off<br>
对于以下参数，PostgreSQL在主副本和所有副本之间不需要相等的值。但是考虑到副本随时可能成为主数据库的可能性，对它们进行不同的设置并没有多大意义。因此，Patroni将其值限制为动态配置。<br>
• max_wal_senders: 5 <br>
• max_replication_slots: 5 <br>
• wal_keep_segments: 8 <br>
• wal_keep_size: 128MB<br>
对这些参数进行验证，以确保其正常或满足最小值。<br>
Patroni还控制着其他一些Postgres参数：<br>
• listen_addresses -是从postgresql.listen或从PATRONI_POSTGRESQL_LISTEN环境变量设置的<br>
• port - 通过postgresql.listen或PATRONI_POSTGRESQL_LISTEN环境设置变量<br>
• cluster_name - 从scope或PATRONI_SCOPE环境变量设置<br>
• hot_standby: on （切换为备机时仍然提供查询服务）<br>
为了安全起见，上述列表中的参数未写入postgresql.conf中，而是作为参数列表传递给pg_ctl start，它们具有最高优先级，甚至在ALTER SYSTEM以上。<br>
应用本地或动态配置选项时，将执行以下操作：<br>
•节点首先检查是否存在postgresql.base.conf或是否设置了custom_conf参数。<br>
•如果设置了custom_conf参数，它将把在其上指定的文件作为基本配置，忽略postgresql.base.conf和postgresql.conf。<br>
•如果未设置custom_conf参数且存在postgresql.base.conf，则它包含重命名的“original”配置，并将用作基本配置。<br>
•如果没有custom_conf或postgresql.base.conf，则取原始的postgresql.conf并重命名为postgresql.base.conf。<br>
•动态选项（上述例外情况除外）转储到postgresql.conf中，并在postgresql.conf中为所用的基本配置（postgresql.base.conf或custom_conf上的内容）设置一个include。因此，我们将能够应用新选项，而无需重新读取配置文件以检查是否不包括include。<br>
•使用命令行覆盖对Patroni管理集群至关重要的一些参数。<br>
•如果某些需要重新启动的选项发生更改（我们应查看pg_settings中的上下文以及这些选项的实际值），则会设置给定节点的pending_restart标志。此标志在任何重新启动时重置。<br>
参数将按以下顺序应用（运行时具有最高优先级）：<br>
1.从文件postgresql.base.conf（或从custom_conf文件（如果已设置））加载参数<br>
2.从文件postgresql.conf加载参数<br>
3.从文件postgresql.auto.conf加载参数<br>
4.使用 -o –name=value 设置运行时参数<br>
这允许对所有节点进行配置（2），使用ALTER SYSTEM对特定节点进行配置（3），并确保执行对运行Patroni至关重要的参数（4），同时为配置工具留出空间，以便直接管理postgresql.conf而不涉及Patroni（1）。<br>
此外，以下Patroni配置选项只能动态更改：<br>
• ttl: 30 <br>
• loop_wait: 10 <br>
• retry_timeouts: 10 <br>
• maximum_lag_on_failover: 1048576 <br>
• max_timelines_history: 0 <br>
• check_timeline: false <br>
• postgresql.use_slots: true<br>
更改这些选项后，Patroni将读取存储在DCS中的配置的相关部分，并更改其运行时值。<br>
每次更改配置时，Patroni节点都会将DCS选项的状态转储到磁盘，并将其转储到位于Postgres数据目录中的文件patoni.dynamic.json中。如果DCS中完全没有这些选项或这些选项无效，则仅允许主机从磁盘转储恢复这些选项。<br>
