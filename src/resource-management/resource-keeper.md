# Resource Keeper

Resource Keeper 是一种资源回收机制。当一些特定类型的工作负载处于空闲状态超过一定时间之后，Resource Keeper 将工作负载标记为暂停状态并回收其占用的计算资源，用户需要时可手动重启工作负载。

目前，Resource Keeper 支持以下三种类型的工作负载：

1. Notebook
1. Tensorboard
1. Explorer

## 运行状态

查看 Resource Keeper 的运行状态：

```bash
$ kubectl get pod -n t9k-system -l component=resource-keeper
NAME                               READY   STATUS    RESTARTS   AGE
resource-keeper-5d86dfff8f-k5fwd   1/1     Running   0          7d1h
```

查看 Resource Keeper 的日志：

```bash
$ kubectl logs  -n t9k-system -l component=resource-keeper --tail=-1
I1 10/12 09:33:55 init.go:119 resource-keeper/base [Flag is set] name=config-file value=policy.yaml
I1 10/12 09:33:55 init.go:119 resource-keeper/base [Flag is set] name=configmap-name value=resource-keeper-policy-config
I1 10/12 09:33:55 init.go:119 resource-keeper/base [Flag is set] name=configmap-namespace value=t9k-system
I1 10/12 09:33:55 init.go:119 resource-keeper/base [Flag is set] name=show-error-trace value=true
I1 10/12 09:33:55 init.go:119 resource-keeper/base [Flag is set] name=v value=2
I2 10/12 09:33:55 init.go:125 resource-keeper/base [Working directory] dir=/app
I0 10/12 09:33:55 init.go:157 resource-keeper/base [Initialized] name=resource-keeper
…
```

如果日志过多，可通过命令行参数 --tail=N 查看最近的 N 行日志，并通过命令行参数 -f 持续观察日志输出：

```bash
$ kubectl logs  -n t9k-system -l component=resource-keeper --tail=100 -f
I0 10/19 10:38:56 resourcekeeper.go:67 resource-keeper/explorer [List resources] namespace=dev number=0
I0 10/19 10:38:56 resourcekeeper.go:67 resource-keeper/tensorboard [List resources] namespace=demo number=7
I0 10/19 10:38:56 resourcekeeper.go:67 resource-keeper/explorer [List resources] namespace=test-project number=4
```

## 查看配置

查看 Resource Keeper 配置文件：

```bash
$ kubectl -n t9k-system get cm resource-keeper-policy-config -o yaml
```

配置文件示例：

```yaml
apiVersion: v1
data:
  policy.yaml: |-
    policy:
      notebook:
        scanInterval: 300
        idleTimeout: 86400
        namespaceSelector:
          matchLabels:
            tensorstack.dev/resource-keeper: true
          matchExpressions: []
      tensorboard:
        scanInterval: 300
        idleTimeout: 86400
        namespaceSelector:
          matchLabels:
            tensorstack.dev/resource-keeper: true
          matchExpressions: []
      explorer:
        scanInterval: 300
        idleTimeout: 86400
        namespaceSelector:
          matchLabels:
            tensorstack.dev/resource-keeper: true
          matchExpressions: []
kind: ConfigMap
metadata:
  name: resource-keeper-policy-config
  namespace: t9k-system
```

管理员可以对每种类型的工作负载进行独立的资源回收配置，配置参数有：

* `scanInterval`：类型 int，单位秒。每隔多长时间检查一次 Notebook 的 status。默认 300 秒。
* `idleTimeout`：当工作负载空闲时间超过 idleTimeout，回收资源。默认 86400 秒即 24 小时。
* `namespaceSelector`：仅针对符合条件的 namespace 下的工作负载执行资源回收。具体语法见 <a target="_blank" rel="noopener noreferrer" href="https://github.com/kubernetes/apimachinery/blob/v0.18.8/pkg/apis/meta/v1/types.go#L1065">K8s LabelSelector</a>。
    * namespaceSelector 为空或者为 null 均表示匹配所有 namespace；
    * 如果希望匹配 no namespace，可删除整个 notebook 配置，即禁用 notebook 资源回收
    * 一般不需要修改

## 修改配置

运行下列命令可以修改 Resource Keeper 配置文件：

```bash
$ kubectl -n t9k-system edit cm resource-keeper-policy-config
```

如果需要针对一种工作负载禁用资源回收机制，以 Notebook 为例，将配置文件中的 `policy.notebook` 字段全部删除即可：

```yaml
apiVersion: v1
data:
  policy.yaml: |-
    policy:
      # notebook:
      #   scanInterval: 300
      #   idleTimeout: 86400
      #   namespaceSelector:
      #     matchLabels:
      #       tensorstack.dev/resource-keeper: true
      #     matchExpressions: []
      tensorboard:
        scanInterval: 300
        idleTimeout: 86400
        namespaceSelector:
          matchLabels:
            tensorstack.dev/resource-keeper: true
          matchExpressions: []
      explorer:
        scanInterval: 300
        idleTimeout: 86400
        namespaceSelector:
          matchLabels:
            tensorstack.dev/resource-keeper: true
          matchExpressions: []
kind: ConfigMap
metadata:
  name: resource-keeper-policy-config
  namespace: t9k-system
```

如果需要针对一种工作负载使其资源回收机制对所有的 namespace 生效，以 Notebook 为例，将配置文件中的 `policy.notebook.namespaceSelector` 字段删除即可：

```yaml
apiVersion: v1
data:
  policy.yaml: |-
    policy:
      notebook:
        scanInterval: 300
        idleTimeout: 86400
        # namespaceSelector:
        #   matchLabels:
        #     tensorstack.dev/resource-keeper: true
        #   matchExpressions: []
      tensorboard:
        scanInterval: 300
        idleTimeout: 86400
        namespaceSelector:
          matchLabels:
            tensorstack.dev/resource-keeper: true
          matchExpressions: []
      explorer:
        scanInterval: 300
        idleTimeout: 86400
        namespaceSelector:
          matchLabels:
            tensorstack.dev/resource-keeper: true
          matchExpressions: []
kind: ConfigMap
metadata:
  name: resource-keeper-policy-config
  namespace: t9k-system
```

修改配置文件并保存后，新的配置将立即生效，可以通过查看 Resource Keeper 的日志确认：

```bash
$ kubectl -n t9k-system logs -l component=resource-keeper --tail=100 -f
I0 10/19 11:07:56 conf.go:71 resource-keeper/conf [Received ConfigMap watch event] configmap=t9k-system/resource-keeper-policy-config eventType="MODIFIED"
I0 10/19 11:07:56 main.go:60 resource-keeper/monitoring [Reloaded ConfigMap configuration] policy={"explorer":{"scanInterval":300,"idleTimeout":86400,"namespaceSelector":{"matchLabels":{"tensorstack.dev/resource-keeper":"true"}}},"notebook":{"scanInterval":300,"idleTimeout":86400,"namespaceSelector":{"matchLabels":{"tensorstack.dev/resource-keeper":"true"}}},"tensorboard":{"scanInterval":300,"idleTimeout":86400,"namespaceSelector":{"matchLabels":{"tensorstack.dev/resource-keeper":"true"}}}}
I0 10/19 11:07:56 main.go:94 resource-keeper/explorer [Received a new policy configuration, and it is the same as before then do nothing]
I0 10/19 11:07:56 main.go:107 resource-keeper/notebook [Received a new policy configuration, then update the resource keeper] newPolicy={"scanInterval":300,"idleTimeout":86400,"namespaceSelector":{"matchLabels":{"tensorstack.dev/resource-keeper":"true"}}} prevPolicy={"scanInterval":300,"idleTimeout":86402,"namespaceSelector":{"matchLabels":{"tensorstack.dev/resource-keeper":"true"}}}
I0 10/19 11:07:56 main.go:94 resource-keeper/tensorboard [Received a new policy configuration, and it is the same as before then do nothing]
```