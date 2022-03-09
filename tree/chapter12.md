<font size="48"><b>第12章 将 Patroni 与 Kubernetes 结合使用</b></font><br>
Patroni 可以使用 Kubernetes 对象来存储集群状态并管理leader键。这使得它能够在没有任何一致性存储的 Kubernetes 环境中运行 Postgres，即不需要运行额外的 Etcd 部署。Patroni 可以使用两种不同类型的 Kubernetes 对象来存储leader和配置键，它们使用kubernetes.use_endpoints或PATRONI_KUBERNETES_USE_ENDPOINTS 环境变量进行配置。<br>
<b>12.1使用 Endpoints</b><br>
尽管这是推荐的模式，但出于兼容性的原因，默认情况下它是关闭的。启用时，Patroni将集群配置和leader键存储在它创建的各个端点的metadata: annotations字段中。更改leader比使用ConfigMaps更安全，因为包含leader 信息的注释和指向正在运行的leader的实际地址都会一次性同时更新。<br>
<b>12.2使用 ConfigMaps</b><br>
在这种模式下，Patroni 将创建 ConfigMaps 而不是 Endpoints 并将键存储在这些 ConfigMaps 的元数据中。更改leader至少需要两次更新，一个更新到leader ConfigMap，另一个更新到相应的端点。<br>
有两种方法可以将流量引导到 Postgres master：<br>
•使用Patroni 提供的回调脚本<br>
•配置 Kubernetes Postgres 服务以使用带有role_label的标签选择器（在patroni配置中配置）。<br>
请注意，在某些情况下，例如，在 OpenShift（OpenShift是红帽的云开发平台即服务）上运行时，除了使用 ConfigMaps 之外别无选择。<br>
<b>12.3配置</b><br>
Patroni Kubernetes设置 和 环境变量在文档中进行了描述。<br>
<b>12.4示例</b><br>
•Patroni 存储库的kubernetes文件夹包含 Docker 镜像、Kubernetes 清单和回调脚本的示例，以便测试 Patroni Kubernetes 设置。请注意，在当前状态下，由于权限问题，它将无法使用 PersistentVolumes。<br>
•您可以在Spilo Project 中找到可以使用 Persistent Volumes 的全功能 Docker 镜像 。<br>
•Helm chart 用于部署使用Kubernetes运行的Patroni配置的Spilo映像。<br>
•为了使用 Patroni 和 Spilo 大规模运行您的数据库集群，请查看 postgres-operator项目。它实现了操作员模式来管理 Spilo 集群。<br>
