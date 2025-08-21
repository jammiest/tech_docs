# 操作系统概述

操作系统(Operating System, OS)是管理计算机硬件与软件资源的系统软件，为用户和应用程序提供基本的服务和接口。

## 操作系统核心功能

### 1. 进程管理
- **进程(Process)**：程序的执行实例
- **线程(Thread)**：轻量级进程，共享同一地址空间
- **调度算法**：
  - 先来先服务(FCFS)
  - 短作业优先(SJF)
  - 轮转调度(Round Robin)
  - 多级反馈队列(MLFQ)

### 2. 内存管理
- **虚拟内存**：$$虚拟地址空间大小 = 2^n$$，n为地址总线位数
- **分页(Paging)**：$$物理地址 = 页表[虚拟页号] + 页内偏移$$
- **分段(Segmentation)**
- **页面置换算法**：
  - 先进先出(FIFO)
  - 最近最少使用(LRU)
  - 最优置换(OPT)

### 3. 文件系统
- **文件分配方法**：
  - 连续分配
  - 链式分配
  - 索引分配
- **目录结构**：
  - 单级目录
  - 两级目录
  - 树形目录
  - 图形目录

### 4. 设备管理
- **I/O子系统**：
  - 程序控制I/O
  - 中断驱动I/O
  - DMA(直接内存访问)
- **设备驱动程序**

### 5. 安全与保护
- 访问控制矩阵
- 能力表
- 访问控制列表(ACL)

## 进程调度算法比较

| 算法名称       | 特点                          | 平均等待时间 | 适用场景           |
|---------------|-----------------------------|-------------|-------------------|
| FCFS          | 简单，非抢占式                  | 高          | 批处理系统          |
| SJF           | 最短作业优先                   | 最低         | 短作业占优环境       |
| Round Robin   | 时间片轮转，公平                | 中等         | 交互式系统          |
| Priority      | 优先级调度                    | 取决于优先级   | 实时系统           |
| MLFQ          | 多级反馈，结合多种策略            | 较低         | 通用系统           |

## 内存管理关键公式

1. **有效访问时间(EAT)**：
   $$EAT = (1 - p) \times 内存访问时间 + p \times 页面错误处理时间$$
   其中p为缺页率

2. **工作集模型**：
   $$W(t, \Delta) = \{页面i | 页面i在时间区间[t-\Delta, t]被访问\}$$

3. **Belady异常**：
   某些页面置换算法(FIFO)在增加页框数时可能导致缺页率上升

## 进程同步经典问题

### 生产者-消费者问题
```c
// 使用信号量的解决方案
semaphore mutex = 1;    // 互斥信号量
semaphore empty = N;    // 空缓冲区数量
semaphore full = 0;     // 满缓冲区数量

void producer() {
    while (true) {
        item = produce_item();
        wait(empty);
        wait(mutex);
        insert_item(item);
        signal(mutex);
        signal(full);
    }
}

void consumer() {
    while (true) {
        wait(full);
        wait(mutex);
        item = remove_item();
        signal(mutex);
        signal(empty);
        consume_item(item);
    }
}
```

### 读者-写者问题
```c
// 读者优先解决方案
semaphore rw_mutex = 1;  // 读写互斥
semaphore mutex = 1;     // 保护read_count
int read_count = 0;

void writer() {
    wait(rw_mutex);
    // 执行写操作
    signal(rw_mutex);
}

void reader() {
    wait(mutex);
    read_count++;
    if (read_count == 1)
        wait(rw_mutex);
    signal(mutex);
    
    // 执行读操作
    
    wait(mutex);
    read_count--;
    if (read_count == 0)
        signal(rw_mutex);
    signal(mutex);
}
```

## 死锁处理

### 死锁必要条件
1. 互斥条件
2. 占有并等待
3. 非抢占条件
4. 循环等待条件

### 死锁处理方法
1. **预防**：破坏四个必要条件之一
2. **避免**：银行家算法
   - 安全状态判断
   - 资源分配图算法
3. **检测与恢复**：
   - 资源分配图检测
   - 进程终止或资源抢占

## 现代操作系统特性

1. **多核处理**：对称多处理(SMP)、非一致内存访问(NUMA)
2. **虚拟化**：虚拟机监控程序(Hypervisor)
3. **分布式系统**：分布式文件系统、集群计算
4. **实时系统**：硬实时、软实时
5. **微内核架构**：将核心功能最小化

## 性能指标

1. **吞吐量(Throughput)**：单位时间完成的进程数
2. **周转时间(Turnaround Time)**：$$T_{周转} = T_{完成} - T_{到达}$$
3. **响应时间(Response Time)**：$$T_{响应} = T_{首次响应} - T_{到达}$$
4. **CPU利用率**：$$利用率 = \frac{忙时间}{总时间} \times 100\%$$

操作系统是计算机系统的核心，理解其原理和机制对于开发高效、可靠的软件至关重要。