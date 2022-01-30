# 客户

## 命令行界面

用户可以使用 CLI 查询“危机”模块并与之交互。 

### Transactions

`tx` 命令允许用户与 `crisis` 模块交互。 

```bash
simd tx crisis --help
```

#### invariant-broken

`invariant-broken` 命令在不变量被破坏以停止链时提交证明 

```bash
simd tx crisis invariant-broken [module-name] [invariant-route] [flags]
```

Example:

```bash
simd tx crisis invariant-broken bank total-supply --from=[keyname or address]
```
