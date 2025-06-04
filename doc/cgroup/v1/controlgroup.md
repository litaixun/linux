# 1 什么是 cgroup
cgroup 是 linux 实现的一种资源限制和隔离机制，通过各个子控制组，可以针对不同进程组实施不同的资源隔离和限制。

# 2 怎么使用
cgroup 是通过 vfs 暴露控制接口的，用户只需要修改对应的文件，即可配置 cgroup，一般情况下，cgroup 通过 systemd 自动挂载到 tmpfs 中。
```shell
# mount | grep cgroup
tmpfs on /sys/fs/cgroup type tmpfs (ro,nosuid,nodev,noexec,mode=755)
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-agent,name=systemd)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,hugetlb)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,net_cls,net_prio)
cgroup on /sys/fs/cgroup/memory,blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio,memory)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,perf_event)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpu,cpuacct)
cgroup on /sys/fs/cgroup/rdma type cgroup (rw,nosuid,nodev,noexec,relatime,rdma)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,freezer)
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,pids)
```

## 2.1 为什么要挂载到 tmpfs
1. cgroups 的控制文件（如 CPU 配额、内存限制等）需要频繁读写，使用 tmpfs 可以避免磁盘 IO 延迟，提升性能
2. cgroups 的配置是动态的，仅在系统运行时生效（如进程生命周期内），无需将数据持久化到磁盘
3. tmpfs 是 Linux 内核内置的文件系统，无需额外驱动或配置，与 cgroups 集成良好，确保稳定性和兼容性

### 2.1.1 什么是 tmpfs 
*tmpfs 是一种基于内存（RAM）或交换空间（swap）的文件系统，数据存储在内存中，读写速度极快，具有如下特点：*
1. 高速读写：数据存储在内存中，读写速度远高于磁盘文件系统（如 ext4、xfs）
2. 动态容量：容量可根据需求动态调整（受限于内存和 swap 空间），无需预先分配固定大小
3. 非持久化：数据仅存在于内存中，系统重启或卸载后数据丢失，适合临时数据存储
4. 内存管理：内核自动管理内存使用，支持数据交换到磁盘，避免占用过多物理内存
5. 支持权限控制：可像普通文件系统一样设置文件权限（如 chmod），保障数据安全