<font size="48"><b>第13章 WatchDog支持</b></font><br>
将多个 PostgreSQL 服务器作为主服务器运行可能会因时间线不同而导致事务丢失。这种情况也称为脑裂问题。为了避免裂脑，Patroni 需要确保 PostgreSQL 在 DCS 中的leader键过期后不会接受任何事务提交。在正常情况下，当leader锁更新因任何原因失败时，Patroni 会尝试通过停止 PostgreSQL 来实现这一点。但是，由于各种原因，这可能不会发生：<br>
•Patroni 因错误、内存不足或被系统管理员意外杀死而崩溃。<br>
•关闭 PostgreSQL 太慢了。<br>
•由于系统负载过高、虚拟机管理程序暂停虚拟机或其他基础架构问题，Patroni 无法运行。<br>
为了保证在这些条件下的正确行为，Patroni 支持WatchDog设备。WatchDog设备是软件或硬件机制，当它们在指定的时间范围内没有获得保持活动的心跳时，它们将重置整个系统。这增加了额外的故障安全层，以防通常的 Patroni 脑裂保护机制失败。<br>
Patroni 会在将 PostgreSQL 提升为 master 之前尝试激活WatchDog。如果WatchDog激活失败并且WatchDog模式是required那么节点将拒绝成为主节点。在决定参加leader选举时，Patroni 还将检查WatchDog配置是否允许它成为leader。在将 PostgreSQL 降级后（例如由于手动故障转移），Patroni 将再次禁用WatchDog。当 Patroni 处于暂停状态时，WatchDog也将被禁用。<br>
默认情况下，Patroni 会将WatchDog设置为在 TTL 到期前 5 秒到期。使用默认设置loop_wait=10和ttl=30这使 HA 循环至少有 15 秒 ( ttl- safety_margin- loop_wait) 才能在系统被强制重置之前完成。默认情况下，访问 DCS 配置为在 10 秒后超时。这意味着当 DCS 不可用时，例如由于网络问题，Patroni 和 PostgreSQL 将有至少 5 秒（ttl- safety_margin- loop_wait- retry_timeout）进入所有客户端连接终止的状态。<br>
安全裕度是 Patroni 为leader键更新和WatchDog keepalive之间的时间预留的时间量。Patroni 将在确认leader键更新后立即尝试发送一个 keepalive。如果 Patroni 进程在恰到好处的时刻暂停较长时间，则保活可能会延迟超过安全裕度而不会触发WatchDog。这导致在leader键到期之前WatchDog不会触发的时间窗口，从而使保证无效。为了绝对确保WatchDog在所有情况下都会触发，通过设置safety_margin为 -1 将WatchDog超时设置为TTL 的一半后过期。如果需要此保证，您可能应该增加ttl和/或减少loop_wait和retry_timeout。<br>
目前WatchDog仅支持使用 Linux Watchdog设备接口。<br>
<b>13.1在 Linux 上设置软件WatchDog</b><br>
/dev/watchdog如果 Patroni 可以访问，默认的 Patroni 配置将尝试在 Linux上使用。对于大多数用例，使用内置在Linux内核中的watchdog 软件是足够安全的。<br>
要启用软件WatchDog，请在启动 Patroni 之前以 root 身份发出以下命令：<br>
modprobe softdog<br>
#Replace postgres with the user you will be running patroni under<br>
chown postgres /dev/watchdog<br>
对于测试，通过向modprobe命令行添加soft_noboot=1来禁用重新启动可能会有所帮助。在这种情况下，watchdog 只会在内核环形缓冲区中记录一行，通过dmesg可见。<br>
成功启用后，Patroni 将记录有关WatchDog的信息。<br>
