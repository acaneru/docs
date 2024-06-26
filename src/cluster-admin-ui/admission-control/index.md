# 准入控制

准入控制负责对用户创建的特定资源对象进行验证、修改，以保证集群的正确性和稳定性。准入控制分为两类：
* 验证控制器：根据验证规则来检验用户的资源对象，拒绝非法的资源对象的创建/修改。
* 变更控制器：根据变更规则来对资源对象进行修改、验证，既可以将资源对象的某些字段设为规定值，也可以拒绝资源对象的创建/修改。

在菜单**准入控制**下，你可以查看、管理验证规则和变更规则。

## 参考

[准入控制](../../admission-control/index.md)
