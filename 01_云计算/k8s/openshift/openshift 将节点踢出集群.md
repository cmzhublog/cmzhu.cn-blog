## Openshift 将节点踢出集群

当您使用 CLI 删除节点时，节点对象会从 Kubernetes 中删除，但该节点上存在的 pod 不会被删除。任何未由复制控制器支持的裸机 pod 都无法从 OpenShift Container Platform 访问。由复制控制器支持的 Pod 会重新调度到其他可用的节点。您必须删除本地清单 pod。

### 步骤

通过完成以下步骤，从裸机上运行的 OpenShift Container Platform 集群中删除节点：

1. 将节点标记为不可调度。

   ```terminal
   $ oc adm cordon <node_name>
   ```

2. 排空节点上的所有 pod：

   ```terminal
   $ oc adm drain <node_name> --force=true
   ```

   如果节点离线或者无响应，此步骤可能会失败。即使节点没有响应，它仍然在运行写入共享存储的工作负载。为了避免数据崩溃，请在进行操作前关闭物理硬件。

3. 从集群中删除节点：

   ```terminal
   $ oc delete node <node_name>
   ```

   虽然节点对象现已从集群中删除，但它仍然可在重启后或 kubelet 服务重启后重新加入集群。要永久删除该节点及其所有数据，您必须[弃用该节点](https://access.redhat.com/solutions/84663)。

4. 如果您关闭了物理硬件，请重新打开它以便节点可以重新加入集群。