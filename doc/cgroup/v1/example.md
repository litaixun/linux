# cpu

*案例一：限制进程 cpu 使用率上限为 2mc*

```bash
#!/usr/bin/env bash

# 创建控制组
mkdir /sys/fs/cgroup/cpu/mygroup

# 设置配额（20% of 1 CPU）
echo 20000 > /sys/fs/cgroup/cpu/mygroup/cpu.cfs_quota_us  # 20ms
echo 100000 > /sys/fs/cgroup/cpu/mygroup/cpu.cfs_period_us # 100ms

# 将进程加入控制组
echo $PID > /sys/fs/cgroup/cpu/mygroup/tasks
```

*案例二：限制 containerd cpu 使用率上限为 4c*

```bash
#!/usr/bin/env bash

# 创建控制组
mkdir /sys/fs/cgroup/cpu/containerd

# 设置配额为 4c
echo 400000 > /sys/fs/cgroup/cpu/containerd/cpu.cfs_quota_us  # 400ms
echo 100000 > /sys/fs/cgroup/cpu/containerd/cpu.cfs_period_us # 100ms

# 将 containerd 加入控制组
echo $(pgrep containerd) > /sys/fs/cgroup/cpu/containerd/tasks
```

*案例三：配置进程间 cpu 使用权重*

```bash
#!/usr/bin/env bash

mkdir /sys/fs/cgroup/cpu/container_a
mkdir /sys/fs/cgroup/cpu/container_b

# A 服务权重为 2048
echo 2048 > /sys/fs/cgroup/cpu/container_a/cpu.shares
# B 服务权重为 1024（默认值）
echo 1024 > /sys/fs/cgroup/cpu/container_b/cpu.shares

echo $(pgrep service_a) > /sys/fs/cgroup/cpu/container_a/tasks
echo $(pgrep service_b) > /sys/fs/cgroup/cpu/container_b/tasks
```

# cpuset

*案例一：redis 内存本地化*

```bash
#!/usr/bin/env bash

# 查看系统 NUMA 拓扑
numactl --hardware
# 输出示例:
# available: 2 nodes (0-1)
# node 0 cpus: 0 1 2 3
# node 1 cpus: 4 5 6 7

# 创建控制组
mkdir /sys/fs/cgroup/cpuset/redis

# 绑定到 node 0 的 CPU
echo 0-3 > /sys/fs/cgroup/cpuset/redis/cpuset.cpus

# 仅使用 node 0 的内存
echo 0 > /sys/fs/cgroup/cpuset/redis/cpuset.mems

# 启动 redis，将其 pid 加入到上述控制组
redis-server --bind 127.0.0.1 &
echo $! > /sys/fs/cgroup/cpuset/redis/tasks

# 测试验证
# 未优化
time redis-benchmark -c 100 -n 100000 set key value
# 优化后
time redis-benchmark -c 100 -n 100000 set key value
# 通常延迟降低 10-30%
```

*案例二：将 mysql 绑定到专用 cpu 核心*

```bash
#!/usr/bin/env bash

# 查看系统 NUMA 拓扑
numactl --hardware
# 输出示例:
# available: 2 nodes (0-1)
# node 0 cpus: 0 1 2 3
# node 1 cpus: 4 5 6 7

# 创建控制组
mkdir /sys/fs/cgroup/cpuset/mysql

# 绑定到 node 1 的 CPU
echo 4-7 > /sys/fs/cgroup/cpuset/mysql/cpuset.cpus

# 仅使用 node 1 的内存
echo 1 > /sys/fs/cgroup/cpuset/mysql/cpuset.mems

# 启用 CPU 独占模式
echo 1 > /sys/fs/cgroup/cpuset/mysql/cpuset.cpu_exclusive

# 启动 mysql，将其 pid 加入到上述控制组
systemctl start mysql
echo $(pgrep mysql) > /sys/fs/cgroup/cpuset/mysql/tasks

# 测试验证
# 检查 mysql 进程是否仅在 CPU 4-7 上运行
ps -o pid,args,psr -p $(pgrep mysql)
```

