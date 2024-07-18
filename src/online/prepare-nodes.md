# 准备节点

我们支持在 Ubuntu 20.04, 22.04 上安装 K8s，其他版本将会适时提供支持。

## 目的

1. 使用 ansible 对集群服务器做基本的配置;
2. 获取并审查节点基本信息，以保证接下来的安装工作能够正确执行。

## 前提条件

完成 [准备 inventory](./prepare-inventory.md) 中的工作。

确认 inventory 可用：

```bash
# 进入为此次安装准备的 inventory 目录
cd ~/ansible/$T9K_CLUSTER 

# 确认 server 列表
ansible-inventory -i inventory/inventory.ini --list

# 测试可访问
ansible all -m ping -i inventory/inventory.ini
```

## 准备节点

安装 K8s 之前，需要做如下准备工作：

1. 禁用 Ubuntu 自动更新；
1. 设置自动同步系统时钟。

<aside class="note">
<div class="title">注意</div>

如果为临时测试目的，可跳过 “禁止 Ubuntu 自动更新” 和 “设置自动同步系统时钟”。

</aside>

### 获取节点信息

运行脚本：

```bash
ansible-playbook ../ks-clusters/t9k-playbooks/0-gather-information.yml \
  -i inventory/inventory.ini \
  --become -K
```

运行结束后，可在 ansible 控制节点中查看保存的信息：

```bash
ls /tmp/facts/
```

特别地，确认 GPU、网络设备等信息是否符合预期。

### 禁用 Ubuntu 自动更新

<aside class="note warning">
<div class="title">注意</div>

该脚本中包含重启节点的操作。

</aside>

运行脚本：

```bash
ansible-playbook ../ks-clusters/t9k-playbooks/1-disable-auto-upgrade.yml \
  -i inventory/inventory.ini \
  --become -K
```

### 设置时钟同步

运行脚本：

```bash
ansible-playbook ../ks-clusters/t9k-playbooks/2-sync-time.yml \
  -i inventory/inventory.ini \
  --become -K \
  -e <chrony_server_ip> \
  -e chrony_client_ip_range=<chrony_client_ip_range_1>,<chrony_client_ip_range_2>
```

其中的变量说明如下：

1. `chrony_server_ip`：运行 chrony server 节点的 IP 地址。
1. `chrony_client_ip_range_1`：chrony 生效的 IP 地址网段，例如 `1.2.3.4/24`，可设置一个或多个网段，使用逗号分割。

也可以直接在 YAML 中设置变量（在 ks-clusters/t9k-playbooks/group_vars/all/all.yml 中）：

```bash
chrony_server_ip: <chrony_server_ip> # 1.2.3.4
chrony_client_ip_range:
- <chrony_client_ip_range_1> # 1.2.3.4/24
- <chrony_client_ip_range_2> # 100.0.0.1/8
```

## 下一步

准备好节点之后，我们可进行 [安装 K8s](./k8s-index.md) 的工作。