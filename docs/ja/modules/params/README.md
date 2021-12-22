# `参数`

## 摘要

包 params 提供了一个全局可用的参数存储。

有两种主要类型，Keeper 和 Subspace。 子空间是一个独立的命名空间
paramstore，其中键以预配置的空间名称为前缀。 守门员有一个
访问所有现有空间的权限。

子空间可以由需要私有参数存储的个体守护者使用
其他饲养员无法修改。 params Keeper 可用于向 `x/gov` 路由器添加路由，以便在提案通过时修改任何参数。

以下内容解释了如何将 params 模块用于 master 和 user 模块。

## 内容

1. **[Keeper](01_keeper.md)**
2. **[子空间](02_subspace.md)**
     - [密钥](02_subspace.md#key)
     - [密钥表](02_subspace.md#keytable)
     - [参数集](02_subspace.md#paramset) 