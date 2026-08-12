# Ansible 云运维实践 - 阿里云 ECS

基于阿里云 ECS 3 节点集群，使用 Ansible 实现 Nginx 自动化批量部署。

## 架构

| 节点 | 公网 IP | 私网 IP | 角色 |
|------|---------|---------|------|
| aliyun1 | 8.163.26.172 | 172.24.47.99 | Ansible 控制节点 |
| aliyun2 | 8.138.3.172 | 172.25.112.181 | 被管节点 (cloud1) |
| aliyun3 | 8.138.184.211 | 172.24.47.102 | 被管节点 (cloud2) |

## 执行

```bash
ansible-playbook -i inventory/hosts playbooks/deploy_nginx.yml

## 监控体系（新增）

- 基于 Prometheus + Grafana 构建 3 节点监控体系
- 通过 Ansible 批量部署 Node Exporter 到被管节点
- 导入官方仪表盘 ID 1860，实现 CPU/内存/磁盘/网络可视化监控
- 阿里云安全组管控端口访问（9090/3000/9100）
