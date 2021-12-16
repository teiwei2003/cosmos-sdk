# 子空间

`Subspace` 是参数存储的前缀子空间。每个模块使用
参数存储将采用“子空间”来隔离访问权限。

## 钥匙

参数键是人类可读的字母数字字符串。密钥的参数
`"ExampleParameter"` 存储在 `[]byte("SubspaceName" + "/" + "ExampleParameter")` 下，
其中“SubspaceName”是子空间的名称。

子键是与主参数键一起使用的辅助参数键。
子密钥可用于在运行时进行分组或动态参数密钥生成。

## 密钥表

所有将使用的参数键都应该在编译时注册
时间。 `KeyTable` 本质上是一个 `map[string]attribute`，其中 `string` 是一个参数键。

目前，`attribute` 包含一个 `reflect.Type`，表示参数
type 来检查提供的键和值是否兼容和注册，以及一个函数 `ValueValidatorFn` 来验证值。

只有主键必须在“KeyTable”上注册。子键继承
主键的属性。

## 参数集

模块通常将参数定义为 proto 消息。生成的结构体可以实现
`ParamSet` 接口与以下方法一起使用:

* `KeyTable.RegisterParamSet()`:注册结构体中的所有参数
* `Subspace.{Get, Set}ParamSet()`:从结构体中获取和设置

为了使用`GetParamSet()`，实现者应该是一个指针。 