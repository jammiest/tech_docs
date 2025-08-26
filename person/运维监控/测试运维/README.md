# 测试与运维（TestOps）

TestOps 是测试（Testing）和运维（Operations）的结合，强调通过自动化、协作和持续改进来提高软件质量和交付效率。

## 1. 测试金字塔与策略

### 测试金字塔模型

$$
\text{测试金字塔} = \begin{cases}
\text{单元测试} & \text{(70-80%)} \\
\text{集成测试} & \text{(15-20%)} \\
\text{端到端测试} & \text{(5-10%)}
\end{cases}
$$

### 测试类型矩阵

| 测试类型 | 执行频率 | 执行速度 | 维护成本 | 故障定位 |
|---------|----------|----------|----------|----------|
| 单元测试 | 非常高 | 毫秒级 | 低 | 精确 |
| 集成测试 | 高 | 秒级 | 中 | 较精确 |
| 系统测试 | 中 | 分钟级 | 高 | 较模糊 |
| UI测试 | 低 | 分钟级 | 非常高 | 模糊 |

## 2. 自动化测试框架

### 流行测试框架比较

| 框架 | 语言 | 类型 | 特点 |
|------|------|------|------|
| JUnit | Java | 单元测试 | 注解驱动, 生命周期管理 |
| pytest | Python | 全栈测试 | 插件丰富, 简洁语法 |
| Selenium | 多语言 | UI测试 | 浏览器自动化 |
| Cypress | JavaScript | E2E测试 | 实时重载, 时间旅行 |
| Robot | Python | 验收测试 | 关键字驱动, 易读报告 |

### 测试代码示例

```python
# pytest 示例
import pytest

@pytest.mark.parametrize("input,expected", [
    (3, 9),
    (5, 25),
    (0, 0)
])
def test_square(input, expected):
    assert input**2 == expected
```

## 3. 持续测试与CI/CD集成

### CI/CD 流水线中的测试阶段

```
代码提交 -> 静态分析 -> 单元测试 -> 构建 -> 集成测试 -> 
部署到测试环境 -> 系统测试 -> 安全测试 -> 性能测试 -> 
部署到预生产 -> 验收测试 -> 生产部署
```

### Jenkins Pipeline 示例

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Integration Test') {
            steps {
                sh 'mvn verify -Pintegration'
            }
        }
        stage('Deploy') {
            steps {
                sh 'kubectl apply -f k8s/'
            }
        }
    }
}
```

## 4. 测试环境管理

### 环境配置策略

| 环境类型 | 用途 | 数据 | 稳定性 |
|----------|------|------|--------|
| 开发环境 | 功能开发 | 模拟数据 | 低 |
| 测试环境 | 功能测试 | 清洗过的生产数据 | 中 |
| 预生产环境 | 验收测试 | 生产数据副本 | 高 |
| 生产环境 | 真实用户 | 真实数据 | 最高 |

### 基础设施即代码（IaC）示例

```terraform
# Terraform 配置测试环境
resource "aws_instance" "test_server" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.medium"
  
  tags = {
    Name = "Test-Environment"
    Env  = "Test"
  }
}
```

## 5. 监控与可观测性

### 监控指标公式

$$
\text{系统健康度} = \frac{\text{成功请求数}}{\text{总请求数}} \times 100\%
$$

$$
\text{MTTR} = \frac{\sum \text{故障修复时间}}{\text{故障次数}}
$$

### 监控层级

1. **基础设施层**：CPU、内存、磁盘、网络
2. **应用层**：响应时间、错误率、吞吐量
3. **业务层**：转化率、用户活跃度、交易量

### Prometheus 查询示例

```promql
# 计算错误率
sum(rate(http_requests_total{status=~"5.."}[5m])) 
/ 
sum(rate(http_requests_total[5m]))
```

## 6. 性能测试方法论

### 性能测试类型

| 测试类型 | 用户负载 | 目的 |
|----------|----------|------|
| 基准测试 | 单用户 | 建立性能基线 |
| 负载测试 | 典型负载 | 验证系统容量 |
| 压力测试 | 超过容量 | 找出系统极限 |
| 耐久测试 | 长时间运行 | 检查内存泄漏 |

### JMeter 测试计划要素

```
Test Plan
├─ Thread Group (虚拟用户组)
│  ├─ Sampler (HTTP请求等)
│  ├─ Logic Controller (逻辑控制)
│  ├─ Listener (结果收集)
├─ Config Element (配置元素)
├─ Timer (定时器)
```

## 7. 混沌工程与可靠性测试

### 混沌实验原则

1. 定义稳态指标（如错误率 < 0.1%）
2. 假设系统在特定条件下能保持稳态
3. 注入故障（如网络分区、服务终止）
4. 验证假设是否成立

### 常见故障注入

```bash
# 模拟网络延迟
tc qdisc add dev eth0 root netem delay 100ms

# 随机杀死进程
while true; do kill -9 $(ps aux | grep service | awk '{print $2}' | shuf -n 1); sleep 30; done
```

## 8. 测试数据管理

### 测试数据生成策略

```python
# 使用 Faker 生成测试数据
from faker import Faker

fake = Faker()
test_users = [{
    'name': fake.name(),
    'email': fake.email(),
    'address': fake.address()
} for _ in range(100)]
```

### 数据脱敏技术

```sql
-- SQL 数据脱敏示例
UPDATE customers 
SET 
    ssn = CONCAT('XXX-XX-', RIGHT(ssn, 4)),
    phone = CONCAT('(XXX) XXX-', RIGHT(phone, 4))
WHERE id > 0;
```

## 9. 质量门禁与度量

### 质量指标公式

$$
\text{测试覆盖率} = \frac{\text{被测试代码行数}}{\text{总代码行数}} \times 100\%
$$

$$
\text{缺陷密度} = \frac{\text{发现缺陷数}}{\text{代码千行数}}
$$

### SonarQube 质量阈表示例

```yaml
# sonar-project.properties
sonar.qualitygate.wait=true
sonar.qualitygate.timeout=300

# 质量阈条件
sonar.qualitygate.conditions=\
  coverage > 80%,\
  duplicated_lines < 3%,\
  bugs < 10
```

## 10. 测试运维工具链

### 现代TestOps工具栈

```
版本控制: Git
CI/CD: Jenkins, GitHub Actions
测试框架: JUnit, pytest, Selenium
虚拟化: Docker, Kubernetes
监控: Prometheus, Grafana
日志: ELK Stack
APM: New Relic, Datadog
混沌工程: Chaos Monkey, Gremlin
协作: Jira, TestRail
```

通过实施TestOps，团队可以实现更快的反馈循环、更高的测试效率和更可靠的软件交付，最终实现DevOps的"持续测试"目标。