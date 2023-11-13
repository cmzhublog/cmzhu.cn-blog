## kubernetes 修改cgroup 版本

kubernetes 修改 cgroup 版本需要修改kubelet 的cgroup 和修改containerd/docker 的 cgroup 版本;

cgroup 的版本

- cgroupfs : 口头叫法V1
- systemd  : 口头叫法V2

### 修改步骤

1、 修改 kubelet 的cgroup 版本为cgroupfs

- cgroupDriver: cgroupfs
- cgroupDriver: systemd

```bash
### cat /var/lib/kubelet/config.yaml

$ sed -i "s/^cgroupDriver.*/cgroupDriver: cgroupfs/g" /var/lib/kubelet/config.yaml && systemctl restart kubelet

```



2、 修改containerd 的cgroup版本为cgroupfs

```bash
$ sed -i 's#SystemdCgroup.*#SystemdCgroup = false#g' /etc/containerd/config.toml
```

> SystemdCgroup = false 表示V1 版本, SystemdCgroup = true 表示V2 版本