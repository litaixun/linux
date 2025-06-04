# CPU 子系统

*功能：限制 CPU 时间分配*

常用控制文件：

* cpu.cfs_period_us：调度周期（默认 100,000 微秒）
* cpu.cfs_quota_us：每个周期内允许使用的时间上限（如 20,000 表示 20% CPU 时间）
* cpu.shares：相对权重（默认 1024），用于多任务竞争时的比例分配（如 A=1024，B=512 → A 获得 2/3 CPU 时间）

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

```bash
#!/usr/bin/env bash
mkdir /sys/fs/cgroup/cpu/service_a
mkdir /sys/fs/cgroup/cpu/service_b

# A 服务权重为 2048
echo 2048 > /sys/fs/cgroup/cpu/service_a/cpu.shares
# B 服务权重为 1024（默认值）
echo 1024 > /sys/fs/cgroup/cpu/service_b/cpu.shares

echo $(pgrep service_a) > /sys/fs/cgroup/cpu/service_a/tasks
echo $(pgrep service_b) > /sys/fs/cgroup/cpu/service_b/tasks
```

    cpu：控制 CPU 时间分配，用于限制资源使用
    cpuacct：统计 CPU 使用量，用于监控和计费
    cpuset：绑定物理 CPU 和内存节点，用于性能优化和隔离

    三者常结合使用，在 k8s 中：
     1. cpu 限制 Pod 的 CPU 配额
     2. cpuacct 统计 Pod 的资源消耗用于计费
     3. cpuset 将关键 Pod 绑定到专用 CPU 避免干扰

