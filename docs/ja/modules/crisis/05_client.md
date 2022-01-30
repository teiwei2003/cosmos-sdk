# お客様

## コマンドラインインターフェイス

ユーザーはCLIを使用して、[危機]モジュールを照会および操作できます。 

### Transactions

`tx`コマンドを使用すると、ユーザーは` crisis`モジュールを操作できます。

```bash
simd tx crisis --help
```

#### invariant-broken

`invariant-broken`コマンドは、チェーンを停止するために不変条件が壊れたときに証明を送信します

```bash
simd tx crisis invariant-broken [module-name] [invariant-route] [flags]
```

例:

```bash
simd tx crisis invariant-broken bank total-supply --from=[keyname or address]
```
