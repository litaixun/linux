# CPU 绑核子系统

*功能：将进程绑定到特定 CPU 核心和内存节点（NUMA）*

核心机制：

* CPU 亲和性：限制进程只能在指定 CPU 上运行
* 内存本地化：强制进程使用指定的内存节点（减少跨 NUMA 节点访问延迟）

常用控制文件：

* cpuset.cpus：允许使用的 CPU 列表（如 0-1 表示 CPU0 和 CPU1）
* cpuset.mems：允许使用的内存节点列表（如 0 表示内存节点 0）
* cpuset.memory_migrate：是否允许内存页在节点间迁移（1 = 允许）
* cpuset.cpu_exclusive：是否独占 CPU（1 = 独占，不与其他组共享 CPU）

典型场景：
* 高性能计算：将关键任务绑定到特定 CPU 避免干扰 
* NUMA 优化：将内存密集型应用绑定到就近的内存节点（减少延迟） 
* 隔离系统进程（如将 kswapd 限制在特定 CPU 上）

