# CPU 统计子系统

*功能：统计控制组的 CPU 使用情况（不限制，仅监控）*

常用控制文件：

* cpuacct.usage：总 CPU 使用时间（纳秒）
* cpuacct.usage_percpu：按 CPU 核心统计的使用时间（纳秒）
* cpuacct.stat：按用户态（user）和内核态（system）统计的 CPU 使用时间（时钟周期）

典型场景：

* 资源计费（如按 CPU 使用量向租户收费）
* 性能分析（找出高 CPU 消耗的进程组）

