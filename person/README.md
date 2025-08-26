# 个人知识库

主要存放个人知识库

## 二级目录预览

```markdown
* 编程语言
  * PHP
  * Go
  * Python
  * Java
  * C++
  * Shell
  * Web-Frontend

* 数据库
  * MySQL
  * PostgreSQL
  * NoSQL
  * 大数据技术

* 系统架构
  * Docker
  * Kubernetes
  * Nginx
  * 中间件
  * 微服务架构

* 开发工具
  * Git
  * 正则表达式
  * IDE工具
  * CI/CD工具

* 计算机科学
  * 算法
  * 数据结构
  * 操作系统
  * 计算机网络
  * 数学基础

* 运维监控
  * 测试运维
  * ELK
  * Prometheus
  * 云原生技术

* 前沿技术
  * 人工智能
  * 区块链
  * 物联网
```

## 归档目录（三级）

```markdown
# 技术知识库归档目录结构

## 1. 编程语言
### 1.1 前端语言
- JavaScript/TypeScript
- WebAssembly
- CSS预处理器(Sass/Less)

### 1.2 后端语言
- Java生态(Spring/MyBatis)
- Go生态(Gin/Beego)
- Python生态(Django/Flask)
- PHP生态(Laravel/ThinkPHP)

### 1.3 系统语言
- C++标准库
- Rust特性
- Shell脚本进阶

## 2. 数据库系统
### 2.1 关系型数据库
- MySQL(索引优化/事务隔离)
- PostgreSQL(扩展/JSON支持)
- Oracle(PL/SQL/性能调优)

### 2.2 NoSQL数据库
- MongoDB(聚合管道)
- Redis(数据结构/持久化)
- Elasticsearch(搜索语法)

### 2.3 大数据技术
- Hadoop生态
- Spark核心原理
- Flink流处理

## 3. 系统架构
### 3.1 分布式系统
- CAP理论
- 一致性算法(Paxos/Raft)
- 服务发现机制

### 3.2 微服务架构
- 服务网格(Service Mesh)
- 熔断限流(Hystrix/Sentinel)
- 分布式事务(Seata)

### 3.3 云原生
- Kubernetes调度原理
- Docker核心原理
- Serverless架构

## 4. 开发工具链
### 4.1 版本控制
- Git高级用法
- Git工作流(Git Flow)
- 代码审查(Gerrit)

### 4.2 开发环境
- VS Code插件开发
- IntelliJ插件体系
- 终端工具(Tmux/Zsh)

## 5. 计算机基础
### 5.1 算法与数据结构
- 排序算法复杂度分析
- 动态规划范式
- 图论算法

### 5.2 操作系统
- Linux内核原理
- 进程调度算法
- 内存管理机制

### 5.3 计算机网络
- TCP/IP协议栈
- HTTP/2特性
- QUIC协议原理

## 6. 运维体系
### 6.1 监控告警
- PromQL语法
- Grafana仪表盘
- 日志收集架构

### 6.2 自动化运维
- Ansible剧本
- Terraform架构
- 混沌工程实践
```

## 归档指南

### 知识库归档规范

#### 1. 文件命名规则
- 编程语言：`lang-[语言]-[主题].md`
  - 示例：`lang-go-goroutine.md`
- 数据库：`db-[类型]-[主题].md`
  - 示例：`db-mysql-index.md`
- 系统架构：`arch-[领域]-[主题].md`
  - 示例：`arch-cloud-k8s-scheduler.md`

### 2. 内容结构模板

```markdown
# [标题]

## 1. 核心概念
- 关键定义
- 基本原理公式（如算法复杂度）：
  $$O(n) = \sum_{i=1}^{n} c_i$$

## 2. 技术细节

// 示例代码片段
func main() {
    fmt.Println("Hello World")
}


## 3. 实践案例

| 场景   | 解决方案   | 效果       |
| ------ | ---------- | ---------- |
| 高并发 | 连接池优化 | QPS提升40% |

## 4. 相关资源
- 链接
- 推荐书籍
```

### 模板选择建议：
1. **技术原理类**（如Swoole工作原理）→ 技术原理解析模板  
2. **工具使用类**（如Git/FFmpeg）→ 语言速查手册模板  
3. **系统设计类**（如电商架构）→ 系统设计文档模板  
4. **前沿技术类**（如AI模型）→ 机器学习实验模板  
5. **运维部署类**（如Docker）→ DevOps运维模板

> 所有模板均在 **_template** 目录下


## 3. 版本管理策略

- 使用Git进行版本控制
- 主干分支：`main`
- 修改分支：`feat/[功能]-[日期]`
- 版本标签：`v[年].[月]`（如v2023.08）

## 4. 知识图谱构建
1. 建立概念关联：

   ```mermaid
   graph LR
   A[微服务] --> B[服务发现]
   A --> C[负载均衡]
   B --> D[Consul]
   ```

2. 技术栈依赖关系：
   
   ```mermaid
   graph TD
   Web --> Frontend
   Web --> Backend
   Backend --> Database
   ```

## 5. 更新机制

- 每周技术复盘时更新
- 重大技术突破即时归档
- 每季度进行知识库体检


## 题外话

这个结构具有以下改进：
1. 二级目录增加了新技术领域（如JavaScript/TypeScript、Kubernetes等）
2. 归档目录细化到具体技术点（如MySQL索引优化、Flink流处理等）
3. 归档指南提供了具体的操作规范（命名规则、内容模板等）
4. 增加了版本管理和知识图谱构建指导
5. 所有技术领域都保持了可扩展性，便于后续添加新内容

需要特别说明的数学表达式示例：
- 算法复杂度公式：$O(n \log n) = \sum_{i=1}^{n} \log i$
- 分布式系统一致性公式：$P_{consistency} = \frac{N_{agree}}{N_{total}}$