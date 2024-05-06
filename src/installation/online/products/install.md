# 安装

## 安装产品

产品列表及其最新版本见：[T9k Releases](https://docs.google.com/document/d/13aBfNmEYTysJJS_S7bMPPZkTqms6ZrUkVL6bv-IaRPc/edit)

登录到 Registry：

```bash
# 要求 helm version >= v3.8.0
helm registry login tsz.io

# 在完成安装后，可以通过以下命令登出
helm registry logout tsz.io
```

从 <a target="_blank" rel="noopener noreferrer" href="https://github.com/t9k/ks-clusters/tree/master/values">ks-clusters</a> 中获取 values-sample.yaml，并根据文件中的注释进行修改，其中标注了 MUST 的项目必须修改。

安装单个产品：

```bash
# --version 指定的参数为 Helm Chart 的版本，如果省略则安装最新版本
helm install t9k-core \
   oci://tsz.io/t9kcharts/t9k-core \
   -f values.yaml \
   -n t9k-system \
   --version <version>
```

## 基本检查

等待并确认 K8s 集群中所有的 Pod 都正常工作：

* 如果执行了 Pre-Pull Image 的操作，根据集群性能不同，通常需要等待 3-5 分钟
* 如果没有执行 Pre-Pull Image 的操作，根据集群网络环境，通常需要等待 30-60 分钟

```bash
# 持续查看 K8s 集群中的所有 Pod 状态
kubectl get pod -A -w

# 查看 K8s 集群中是否有异常状态的 Pod
kubectl get pod -A -o wide | grep -Eiv "running|complete"
```

查看所有的 helm chart releases：

```bash
$ helm list -A -d
NAME                            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                                   APP VERSION
elasticsearch-single            t9k-monitoring  1               2023-11-19 04:42:24.939067616 +0000 UTC deployed        elasticsearch-7.13.4                    7.13.4
t9k-gatekeeper                  t9k-system      2               2023-11-19 04:47:12.871874737 +0000 UTC deployed        gatekeeper-3.11.0                       v3.11.0
t9k-core                        t9k-system      1               2023-11-19 04:52:52.591086929 +0000 UTC deployed        t9k-core-1.78.3                         1.78.3
t9k-scheduler                   t9k-system      1               2023-11-19 04:53:22.047545558 +0000 UTC deployed        t9k-scheduler-1.78.4                    1.78.4
t9k-csi-s3                      t9k-system      1               2023-11-19 04:53:46.694820382 +0000 UTC deployed        t9k-csi-s3-1.78.3                       1.78.3
t9k-jobs                        t9k-system      1               2023-11-19 04:54:12.858122721 +0000 UTC deployed        t9k-jobs-1.78.4                         1.78.4
t9k-services                    t9k-system      1               2023-11-19 04:54:36.863984918 +0000 UTC deployed        t9k-services-1.78.4                     1.78.4
t9k-landing-page                t9k-system      1               2023-11-19 04:55:00.60533111 +0000 UTC  deployed        t9k-landing-page-1.78.4                 1.78.4
t9k-security-console            t9k-system      1               2023-11-19 04:55:19.309728043 +0000 UTC deployed        t9k-security-console-1.78.5             1.78.5
t9k-notebook                    t9k-system      1               2023-11-19 04:55:54.230482157 +0000 UTC deployed        t9k-notebook-1.78.4                     1.78.4
t9k-monitoring                  t9k-monitoring  1               2023-11-19 04:56:12.617506927 +0000 UTC deployed        t9k-monitoring-1.78.5                   1.78.5
t9k-build-console               t9k-system      1               2023-11-19 04:57:19.251309469 +0000 UTC deployed        t9k-build-console-1.78.5                1.78.5
t9k-deploy-console              t9k-system      1               2023-11-19 04:57:36.088260359 +0000 UTC deployed        t9k-deploy-console-1.78.4               1.78.4
t9k-workflow-manager            t9k-system      1               2023-11-19 04:57:56.56433641 +0000 UTC  deployed        t9k-workflow-manager-1.78.4             1.78.4
t9k-asset-hub                   t9k-system      1               2023-11-19 04:58:28.991306879 +0000 UTC deployed        t9k-asset-hub-1.78.4                    1.78.4
t9k-experiment-management       t9k-system      1               2023-11-19 04:58:49.350846324 +0000 UTC deployed        t9k-experiment-management-1.78.4        1.78.4
t9k-cluster-admin               t9k-system      1               2023-11-19 06:02:45.082613774 +0000 UTC deployed        t9k-cluster-admin-1.78.8                1.78.8
t9k-aistore                     t9k-system      3               2023-11-19 06:37:17.947109956 +0000 UTC deployed        t9k-aistore-1.78.5                      1.78.5
```