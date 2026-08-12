## 监控体系架构

基于 Prometheus + Grafana 构建 3 节点可视化监控。

### 架构图

aliyun1 (控制节点)
├── Prometheus (端口 9090) —— 抓取存储指标
└── Grafana (端口 3000) —— 可视化展示
aliyun2/aliyun3 (被管节点)
└── Node Exporter (端口 9100) —— 暴露系统指标

### 组件说明
- **Prometheus**：时序数据库，每 15s 抓取一次 Node Exporter 数据
- **Grafana**：导入官方仪表盘 ID `1860`（Node Exporter Full）
- **Node Exporter**：通过 Ansible 批量安装到 aliyun2/aliyun3

### 访问地址
- Prometheus: http://8.163.26.172:9090
- Grafana: http://8.163.26.172:3000
