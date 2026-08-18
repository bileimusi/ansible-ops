# Ansible 云运维实践 - 阿里云 ECS

基于阿里云 ECS 3 节点 Ubuntu 22.04 集群，使用 Ansible 实现 Nginx 自动化批量部署与 GitHub Actions CI/CD 持续交付。

## 架构

| 节点 | 公网 IP | 私网 IP | 角色 |
|:---|:---|:---|:---|
| aliyun1 | 8.163.26.172 | 172.24.47.99 | Ansible 控制节点 |
| aliyun2 | 8.138.3.172 | 172.25.112.181 | 被管节点 (cloud1) |
| aliyun3 | 8.138.184.211 | 172.24.47.102 | 被管节点 (cloud2) |

## 仓库结构
ansible-ops/
├── inventory/
│   └── hosts              # Ansible 主机清单
├── playbooks/
│   └── deploy_nginx.yml   # Nginx 批量部署剧本
├── .github/
│   └── workflows/
│       └── deploy.yml     # GitHub Actions CI/CD 工作流
└── README.md
plain

## 前置条件

1. 控制节点安装 Ansible
   ```bash
   sudo apt update && sudo apt install ansible -y
所有节点配置 SSH 免密登录

bash
ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa
ssh-copy-id root@目标IP
被管节点配置 sudo 免密（NOPASSWD）
执行部署

bash
cd ~/ansible-ops
ansible-playbook -i inventory/hosts playbooks/deploy_nginx.yml

CI/CD 流水线
基于 GitHub Actions /ˈɡɪthʌb ˈækʃənz/（GitHub 自动化动作）实现代码提交后自动触发部署
工作流覆盖：代码拉取 → SSH 连通测试 → Ansible Playbook 执行 → 部署结果验证
部署耗时 80 秒内完成

踩坑记录
地域不一致：aliyun2 默认创建到了华东 1（杭州），需释放重建到华南 3（广州），否则内网不通

Ubuntu 22.04 云镜像禁用 root 密码登录：云镜像默认 PermitRootLogin prohibit-password + PasswordAuthentication no，需通过阿里云 Workbench 登录后修改 sshd_config 及 sshd_config.d/ 子配置

SSH 子配置覆盖：sshd_config.d/ 目录下的配置优先级高于主配置，追加配置到 99-allow-root.conf 确保最后加载
安全组端口未开放：默认安全组只放行 22/3389，需手动添加 HTTP(80) 端口规则

GitHub 下载超时：国内服务器从 GitHub 直接下载二进制包大概率超时，改用 apt 官方源或国内镜像
监控体系（关联项目）

基于 Prometheus /prəˈmiːθiəs/ + Grafana /ˈɡræfənə/ 构建 3 节点监控体系

通过 Ansible 批量部署 Node Exporter /noʊd ɪkˈspɔːrtər/（节点指标导出器）到被管节点
导入官方仪表盘 ID 1860，实现 CPU / 内存 / 磁盘 / 网络可视化监控

阿里云安全组管控端口访问（9090 / 3000 / 9100）

监控项目独立仓库：github.com/bileimusi/prometheus-monitoring

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

## 核心文件

### Inventory 主机清单 /ˈɪnvəntɔːri/（因-文-托-瑞）

```ini
[webservers]
cloud1 ansible_host=172.25.112.181 ansible_user=root
cloud2 ansible_host=172.24.47.102 ansible_user=root

[all:vars]
ansible_python_interpreter=/usr/bin/python3

Playbook 部署剧本

---
- name: Deploy Nginx to Cloud Servers
  hosts: webservers
  become: yes

  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
      changed_when: false

    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Deploy custom index.html
      copy:
        content: "Hello World! Deployed by Ansible on {{ inventory_hostname }}."
        dest: /var/www/html/index.html
        owner: root
        group: root
        mode: '0644'

    - name: Ensure Nginx is running
      service:
        name: nginx
        state: started
        enabled: yes
