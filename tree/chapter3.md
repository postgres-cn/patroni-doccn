# 第3章 Patroni REST API <br>
Patroni具有丰富的REST API，Patronictl自身在领导者竞赛中使用了patronictl工具，以执行故障转移/切换/重新初始化/重新启动/重新加载，通过HAProxy或任何其他类型的负载平衡器来执行HTTP健康检查 ，当然也可以用于监视。在下面，您将找到Patroni REST API端点的列表。<br>
<b>3.1 健康检查端点</b><br>
对于所有运行状况检查，GET请求Patroni返回一个JSON文档以及该节点的状态以及HTTP状态代码。如果您不需要或不需要JSON文档，则可以考虑使用OPTIONS方法而不是GET。<br>
• 仅当Patroni节点作为领导者运行时，对Patroni REST API的以下请求将返回HTTP状态代码200：<br>
&nbsp;&nbsp;–GET/<br>
&nbsp;&nbsp;–GET/master<br>
&nbsp;&nbsp;–GET/primary<br>
&nbsp;&nbsp;–GET/read-write<br>
• GET /standby-leader：仅当Patroni 节点在备用集群中作为leader 运行时才返回HTTP 状态代码200。<br>
• GET /leader：当Patroni 节点有leader 锁时，返回HTTP 状态码200。 与前两个端点的主要区别在于它没有考虑 PostgreSQL 是作为主服务器还是作为后备领导者运行。<br>
• GET /replica: replica健康检查端点。 仅当Patroni节点处于running状态，角色为replica且未设置noloadbalance标签时，才返回HTTP状态码200。<br>
• GET /replica?lag=<max-lag>：replica检查端点。 除了从replica检查外，它还检查复制延迟并仅在低于指定值时返回状态代码 200。 出于性能原因，来自 DCS 的关键 cluster.last_leader_operation 用于 Leader wal 位置和replica上的计算延迟。 max-lag 可以以字节（整数）或人类可读的值指定，例如 16kB、64MB、1GB。<br>
&nbsp;&nbsp;– GET /replica?lag=1048576 <br>
&nbsp;&nbsp;– GET /replica?lag=1024kB <br>
&nbsp;&nbsp;– GET /replica?lag=10MB <br>
&nbsp;&nbsp;– GET /replica?lag=1GB<br>
• GET /replica?tag_key1=value1&tag_key2=value2：replica检查端点。 此外，它还将在 yaml 的标签部分检查用户定义的标签 key1 和 key2 及其各自的值 配置管理。 如果没有为实例定义标签，或者 yaml 配置中的值与查询值不匹配，它将返回 HTTP 状态代码 503。<br>
在以下请求中，由于我们正在检查leader或standby-leader的状态，因此 Patroni 不会应用任何用户定义的标签，它们将被忽略。<br>
• GET /?tag_key1=value1&tag_key2=value2 <br>
• GET /master?tag_key1=value1&tag_key2=value2 <br>
• GET /leader?tag_key1=value1&tag_key2=value2 <br>
• GET /primary?tag_key1=value1&tag_key2=value2 <br>
• GET /read-write?tag_key1=value1&tag_key2=value2 <br>
• GET /standby_leader?tag_key1=value1&tag_key2=value2 <br>
• GET /standby-leader?tag_key1=value1&tag_key2=value2<br>
• GET /read-only：与上述端点类似，但也包括主节点<br>
• GET /synchronous 或GET /sync：仅当Patroni 节点作为同步备用节点运行时才返回HTTP 状态代码200。<br>
• GET /asynchronous 或 GET /async：仅当 Patroni 节点作为异步备用节点运行时才返回 HTTP 状态代码 200<br>
• GET /asynchronous?lag=<max-lag> 或 GET /async?lag=<max-lag>：异步待机 检查端点。 除了检查异步或异步之外，它还检查复制延迟并仅在低于指定值时返回状态代码 200。 出于性能原因，来自 DCS 的关键 cluster.last_leader_operation 用于 Leader wal 位置和replica上的计算延迟。 max-lag 可以以字节（整数）或人类可读的值指定，例如 16KB、64MB、1GB。<br>
&nbsp;&nbsp;– GET /async?lag=1048576 <br>
&nbsp;&nbsp;– GET /async?lag=1024kB<br>
&nbsp;&nbsp;– GET /async?lag=10MB <br>
&nbsp;&nbsp;– GET /async?lag=1GB<br>
• GET /health：仅当PostgreSQL 启动并运行时才返回HTTP 状态代码200。<br>
• GET /liveness：始终返回 HTTP 状态代码 200，它仅表示 Patroni 正在运行。 可用于 livenessProbe。<br>
• GET /readiness：当Patroni 节点作为领导者运行或PostgreSQL 启动并运行时，返回HTTP 状态代码200。 当无法使用 Kubernetes 端点进行领导选举 (OpenShift) 时，端点可用于 readinessProbe。<br>
readiness 和liveness 端点是非常轻量的不需要执行SQL。 探针的配置方式应使其在引导密钥到期时的某个时间开始失效。默认值ttl为30秒，示例探针如下所示：<br>
<table width="500" border="1"><tr><th align="left" >
readinessProbe: <br>
&nbsp;&nbsp;httpGet: <br>
&nbsp;&nbsp;&nbsp;&nbsp;scheme: HTTP <br>
&nbsp;&nbsp;&nbsp;&nbsp;path: /readiness <br>
&nbsp;&nbsp;&nbsp;&nbsp;port: 8008 <br>
&nbsp;&nbsp;initialDelaySeconds: 3 <br>
&nbsp;&nbsp;periodSeconds: 10 <br>
&nbsp;&nbsp;timeoutSeconds: 5 <br>
&nbsp;&nbsp;successThreshold: 1 <br>
&nbsp;&nbsp;failureThreshold: 3 <br>
livenessProbe: <br>
&nbsp;&nbsp;httpGet: <br>
&nbsp;&nbsp;&nbsp;&nbsp;scheme: HTTP <br>
&nbsp;&nbsp;&nbsp;&nbsp;path: /liveness <br>
&nbsp;&nbsp;&nbsp;&nbsp;port: 8008 <br>
&nbsp;&nbsp;initialDelaySeconds: 3 <br>
&nbsp;&nbsp;periodSeconds: 10 <br>
&nbsp;&nbsp;timeoutSeconds: 5 <br>
&nbsp;&nbsp;successThreshold: 1 <br>
&nbsp;&nbsp;failureThreshold: 3<br>
</th></tr></table><br>
<b>3.2 监控端点</b><br>
GET /patroni 由 Patroni 在leader选举期间使用。 您的监控系统也可以使用它。此端点生成的 JSON 文档与健康检查端点生成的 JSON 具有相同的结构。<br>
<table width="500" border="1"><tr><th align="left" >
$ curl -s http://localhost:8008/patroni | jq . <br>
{<br>
&nbsp;&nbsp;"state": "running",<br>
&nbsp;&nbsp;"postmaster_start_time": "2019-09-24 09:22:32.555 CEST",<br>
&nbsp;&nbsp;"role": "master",<br>
&nbsp;&nbsp;"server_version": 110005,<br>
&nbsp;&nbsp;"cluster_unlocked": false,<br>
&nbsp;&nbsp;"xlog": {<br>
&nbsp;&nbsp;&nbsp;&nbsp;"location": 25624640<br>
&nbsp;&nbsp;},<br>
&nbsp;&nbsp;"timeline": 3,<br>
&nbsp;&nbsp;"database_system_identifier": "6739877027151648096",<br>
&nbsp;&nbsp;"patroni": {<br>
&nbsp;&nbsp;&nbsp;&nbsp;"version": "1.6.0",<br>
&nbsp;&nbsp;&nbsp;&nbsp;"scope": "batman"<br>
&nbsp;&nbsp;}<br>
}<br>
</th></tr></table><br>
<b>3.3 集群状态端点</b><br>
• GET /cluster 端点生成描述当前集群拓扑和状态的 JSON 文档：<br>
<table width="500" border="1"><tr><th align="left" >
$ curl -s http://localhost:8008/cluster | jq .{<br>
&nbsp;&nbsp;&nbsp;&nbsp;"members": [{<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"name": "postgresql0", <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"host": "127.0.0.1", <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"port": 5432, <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"role": "leader", <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"state": "running", <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"api_url": "http://127.0.0.1:8008/patroni", <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"timeline": 5, <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"tags": { <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"clonefrom": true <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;} <br>
&nbsp;&nbsp;&nbsp;&nbsp;}, <br>
&nbsp;&nbsp;&nbsp;&nbsp;{ <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"name": "postgresql1", <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"host": "127.0.0.1", <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"port": 5433, <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"role": "replica",<br> 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"state": "running", <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"api_url": "http://127.0.0.1:8009/patroni", <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"timeline": 5, <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"tags": { <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"clonefrom": true <br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"lag": 0 <br>
&nbsp;&nbsp;&nbsp;&nbsp;} <br>
&nbsp;&nbsp;], <br>
&nbsp;&nbsp;"scheduled_switchover": { <br>
&nbsp;&nbsp;&nbsp;&nbsp;"at": "2019-09-24T10:36:00+02:00", <br>
&nbsp;&nbsp;&nbsp;&nbsp;"from": "postgresql0" <br>
&nbsp;&nbsp;} <br>
}
</th></tr></table><br>
• GET /history 端点提供了有关集群切换/故障切换历史的视图。 格式与 pg_wal 目录中的历史文件的内容非常相似。唯一的区别是显示新时间线创建时间的时间戳字段。<br>
<table width="500" border="1"><tr><th align="left" >
$ curl -s http://localhost:8008/history | jq . <br>
[ <br>
&nbsp;&nbsp;[ <br>
&nbsp;&nbsp;&nbsp;&nbsp;1, <br>
&nbsp;&nbsp;&nbsp;&nbsp;25623960, <br>
&nbsp;&nbsp;&nbsp;&nbsp;"no recovery target specified", <br>
&nbsp;&nbsp;&nbsp;&nbsp;"2019-09-23T16:57:57+02:00" <br>
&nbsp;&nbsp;], <br>
&nbsp;&nbsp;[ <br>
&nbsp;&nbsp;&nbsp;&nbsp;2, <br>
&nbsp;&nbsp;&nbsp;&nbsp;25624344, <br>
&nbsp;&nbsp;&nbsp;&nbsp;"no recovery target specified", <br>
&nbsp;&nbsp;&nbsp;&nbsp;"2019-09-24T09:22:33+02:00" <br>
&nbsp;&nbsp;], <br>
&nbsp;&nbsp;[ <br>
&nbsp;&nbsp;&nbsp;&nbsp;3, <br>
&nbsp;&nbsp;&nbsp;&nbsp;25624752, <br>
&nbsp;&nbsp;&nbsp;&nbsp;"no recovery target specified",<br>
&nbsp;&nbsp;&nbsp;&nbsp;"2019-09-24T09:26:15+02:00"<br>
&nbsp;&nbsp;], <br>
&nbsp;&nbsp;[ <br>
&nbsp;&nbsp;&nbsp;&nbsp;4, <br>
&nbsp;&nbsp;&nbsp;&nbsp;50331856, <br>
&nbsp;&nbsp;&nbsp;&nbsp;"no recovery target specified", <br>
&nbsp;&nbsp;&nbsp;&nbsp;"2019-09-24T09:35:52+02:00"<br>
&nbsp;&nbsp;] <br>
]
</th></tr></table><br>
<b>3.4配置端点</b><br>
GET /config：获取动态配置的当前版本：<br>
<table width="500" border="1"><tr><th align="left" >
$ curl -s localhost:8008/config | jq .<br>
{<br>
&nbsp;&nbsp;"ttl": 30,<br>
&nbsp;&nbsp;"loop_wait": 10,<br>
&nbsp;&nbsp;"retry_timeout": 10,<br>
&nbsp;&nbsp;"maximum_lag_on_failover": 1048576,<br>
&nbsp;&nbsp;"postgresql": {<br>
&nbsp;&nbsp;&nbsp;&nbsp;"use_slots": true,<br>
&nbsp;&nbsp;&nbsp;&nbsp;"use_pg_rewind": true,<br>
&nbsp;&nbsp;&nbsp;&nbsp;"parameters": {<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"hot_standby": "on",<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"wal_log_hints": "on",<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"wal_level": "hot_standby",<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"max_wal_senders": 5,<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"max_replication_slots": 5,<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"max_connections": "100"<br>
&nbsp;&nbsp;&nbsp;&nbsp;}<br>
&nbsp;&nbsp;}<br>
}
</th></tr></table><br>
PATCH /config：更改现有配置。<br>
<table width="500" border="1"><tr><th align="left" >
$ curl -s -XPATCH -d \<br>
'{"loop_wait":5,"ttl":20,"postgresql":{"parameters":{"max_connections":"101"}}<br>
˓→}' \<br>
http://localhost:8008/config | jq .<br>
{<br>
&nbsp;&nbsp;"ttl": 20,<br>
&nbsp;&nbsp;"loop_wait": 5,<br>
&nbsp;&nbsp;"maximum_lag_on_failover": 1048576,<br>
&nbsp;&nbsp;"retry_timeout": 10,<br>
&nbsp;&nbsp;"postgresql": {<br>
&nbsp;&nbsp;&nbsp;&nbsp;"use_slots": true,<br>
&nbsp;&nbsp;&nbsp;&nbsp;"use_pg_rewind": true,<br>
&nbsp;&nbsp;&nbsp;&nbsp;"parameters": {<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"hot_standby": "on",<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"wal_log_hints": "on",<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"wal_level": "hot_standby",<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"max_wal_senders": 5,<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"max_replication_slots": 5,<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"max_connections": "101"<br>
&nbsp;&nbsp;&nbsp;&nbsp;}<br>
&nbsp;&nbsp;}<br>
}
</th></tr></table><br>
上面的REST API调用修补了现有配置，并返回了新配置。让我们检查节点是否处理了此配置。 首先，它应该每 5 秒开始打印日志行（loop_wait=5）。 “max_connections”的改变需要重启，因此应显示“pending_restart”标志：<br>
<table width="500" border="1"><tr><th align="left" >
$ curl -s http://localhost:8008/patroni | jq .{<br>
&nbsp;&nbsp;"pending_restart": true,<br>
&nbsp;&nbsp;"database_system_identifier": "6287881213849985952",<br>
&nbsp;&nbsp;"postmaster_start_time": "2016-06-13 13:13:05.211 CEST",<br>
&nbsp;&nbsp;"xlog": {<br>
&nbsp;&nbsp;&nbsp;&nbsp;"location": 2197818976<br>
&nbsp;&nbsp;},<br>
&nbsp;&nbsp;"patroni": {<br>
&nbsp;&nbsp;&nbsp;&nbsp;"scope": "batman",<br>
&nbsp;&nbsp;&nbsp;&nbsp;"version": "1.0"<br>
&nbsp;&nbsp;},<br>
&nbsp;&nbsp;"state": "running",<br>
&nbsp;&nbsp;"role": "master",<br>
&nbsp;&nbsp;"server_version": 90503<br>
}
</th></tr></table><br>
删除参数：<br>
如果你想删除（重置）一些设置，只需用 null 修补它：<br>
<table width="500" border="1"><tr><th align="left" >
$ curl -s -XPATCH -d \<br>
	'{"postgresql":{"parameters":{"max_connections":null}}}' \<br>
http://localhost:8008/config | jq .<br>
{<br>
&nbsp;&nbsp;"ttl": 20,<br>
&nbsp;&nbsp;"loop_wait": 5,<br>
&nbsp;&nbsp;"retry_timeout": 10,<br>
&nbsp;&nbsp;"maximum_lag_on_failover": 1048576,<br>
&nbsp;&nbsp;"postgresql": {<br>
&nbsp;&nbsp;&nbsp;&nbsp;"use_slots": true,<br>
&nbsp;&nbsp;&nbsp;&nbsp;"use_pg_rewind": true,<br>
&nbsp;&nbsp;&nbsp;&nbsp;"parameters": {<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"hot_standby": "on",<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"unix_socket_directories": ".",<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"wal_level": "hot_standby",<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"wal_log_hints": "on",<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"max_wal_senders": 5,<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"max_replication_slots": 5<br>
&nbsp;&nbsp;&nbsp;&nbsp;}<br>
&nbsp;&nbsp;}<br>
}
</th></tr></table><br>
上面的调用从动态配置中删除 postgresql.parameters.max_connections.PUT /config: 也可以无条件地执行现有动态配置的完全重写：<br>
<table width="500" border="1"><tr><th align="left" >
$ curl -s -XPUT -d \<br>
	'{"maximum_lag_on_failover":1048576,"retry_timeout":10,"postgresql":{"use_<br>
˓→slots":true,"use_pg_rewind":true,"parameters":{"hot_standby":"on","wal_log_hints":<br>
˓→"on","wal_level":"hot_standby","unix_socket_directories":".","max_wal_senders":5}},<br>
˓→"loop_wait":3,"ttl":20}' \<br>
http://localhost:8008/config | jq .<br>
{<br>
&nbsp;&nbsp;"ttl": 20,<br>
&nbsp;&nbsp;"maximum_lag_on_failover": 1048576,<br>
&nbsp;&nbsp;"retry_timeout": 10,<br>
&nbsp;&nbsp;"postgresql": {<br>
&nbsp;&nbsp;&nbsp;&nbsp;"use_slots": true,<br>
&nbsp;&nbsp;&nbsp;&nbsp;"parameters": {<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"hot_standby": "on",<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"unix_socket_directories": ".",<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"wal_level": "hot_standby",<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"wal_log_hints": "on",<br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"max_wal_senders": 5<br>
&nbsp;&nbsp;&nbsp;&nbsp;},<br>
&nbsp;&nbsp;&nbsp;&nbsp;"use_pg_rewind": true<br>
&nbsp;&nbsp;},<br>
&nbsp;&nbsp;"loop_wait": 3<br>
}
</th></tr></table><br>
<b>3.5 切换和故障切换端点</b><br>
POST /switchover 或 POST /failover。 这些端点彼此非常相似。 但是有一些细微的差异： <br>
1. 故障转移端点允许在没有健康节点时执行手动故障转移，但同时不允许调度切换。<br>
2. 切换端点相反。 它仅在集群健康（有领导者）时才起作用，并允许在给定时间安排切换。 <br>
如果要在特定时间安排切换，则必须在POST请求的JSON正文中至少指定leader 或candidate 字段以及可选的scheduled_at字段。<br>
示例：执行到特定节点的故障转移：<br>
<table border="1"><tr><th align="left">
$ curl -s http://localhost:8009/failover -XPOST -d '{"candidate":"postgresql1"}'<br>
Successfully failed over to "postgresql1"
</th></tr></table><br>
示例：在指定时间调度一个切换，在一个集群中从leader到一些健康的副本节点上：<br>
<table border="1"><tr><th align="left">
$ curl -s http://localhost:8008/switchover -XPOST -d \<br>
'{"leader":"postgresql0","scheduled_at":"2019-09-24T12:00+00"}'<br>
Switchover scheduled
</th></tr></table><br>
根据情况，请求可能会以不同的 HTTP 状态代码和内容结束。 成功完成切换或故障转移后，将返回状态码 200。 如果成功安排了切换，Patroni 将返回 HTTP 状态代码 202。如果出现问题，将在响应正文中返回错误状态代码（400、412 或 503 之一）以及一些详细信息。 更多信息请查看patroni/api.py:do_POST_failover() 方法的源代码。
• DELETE /switchover：删除计划切换
POST /switchover 和POST /failover 端点被分别用于 patronictl switchover 和patronictl failover
 DELETE /switchover 用于 patronictl flush &lt;cluster-name&gt; switchover。
<b>3.6 重启端点</b><br>
• POST /restart：您可以通过执行 POST /restart 调用在特定节点上重新启动 Postgres。 在 POST 请求的 JSON 正文中，可以选择指定一些重启条件：<br>
&nbsp;&nbsp;– restart_pending：布尔值，如果设置为 true Patroni 将仅在重启挂起时重启 PostgreSQL，以便应用 PostgreSQL 配置中的一些更改。<br>
&nbsp;&nbsp;– role：只有当节点的当前角色与 POST 请求中的角色匹配时才执行重新启动。<br>
&nbsp;&nbsp;– postgres_version：仅当postgres 的当前版本小于POST 请求中指定的版本时才执行重新启动。<br>
&nbsp;&nbsp;– timeout：在 PostgreSQL 开始接受连接之前我们应该等待多长时间。 覆盖 master_start_timeout。<br>
&nbsp;&nbsp;– schedule：带时区的时间戳，安排在将来某个地方重启。<br>
• DELETE /restart：删除计划的重启<br>
POST /restart 和 DELETE /restart 端点分别被patronictl restart 和patronictl flush &lt;cluster-name&gt; restart 使用。<br>
<b>3.7 重载端点</b><br>
POST /reload 调用将命令 Patroni 重新读取和应用配置文件。 这相当于向 Patroni 进程发送 SIGHUP 信号。 如果您更改了一些需要重新启动的 Postgres 参数（如 shared_buffers），您仍然必须通过调用 POST/restart 端点或在patriotictl restart 的帮助下明确地重新启动Postgres。<br>
重新加载端点由patriotictl reload 使用。<br>
<b>3.8 重新初始化端点</b><br>
POST /reinitialize：重新初始化指定节点上的PostgreSQL数据目录。只允许在副本上执行它。 一旦调用，它将删除数据目录并启动pg_basebackup或其他替代副本创建方法。<br>
如果 Patroni 处于尝试恢复（重新启动）故障的 Postgres 的循环中，则调用可能会失败。 为了解决这个问题，可以在请求正文中指定 {"force":true}。<br>
重新初始化端点由patriotictl reinit 使用。<br>

[patroni-doccn](https://github.com/postgres-cn/patroni-doccn/blob/main/README.md)
