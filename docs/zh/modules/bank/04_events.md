# 事件

bank 模块发出以下事件: 

## Handlers

### MsgSend

| Type     | Attribute Key | Attribute Value    |
| -------- | ------------- | ------------------ |
| transfer | recipient     | {recipientAddress} |
| transfer | amount        | {amount}           |
| message  | module        | bank               |
| message  | action        | send               |
| message  | sender        | {senderAddress}    |

### MsgMultiSend

| Type     | Attribute Key | Attribute Value    |
| -------- | ------------- | ------------------ |
| transfer | recipient     | {recipientAddress} |
| transfer | amount        | {amount}           |
| message  | module        | bank               |
| message  | action        | multisend          |
| message  | sender        | {senderAddress}    |

## Keeper events

除了处理程序事件之外，银行管理员将在调用以下方法(或最终调用它们的任何方法)时产生事件 

### MintCoins

```json
{
  "type": "coinbase",
  "attributes": [
    {
      "key": "minter",
      "value": "{{sdk.AccAddress of the module minting coins}}",
      "index": true
    },
    {
      "key": "amount",
      "value": "{{sdk.Coins being minted}}",
      "index": true
    }
  ]
}
```

```json
{
  "type": "coin_received",
  "attributes": [
    {
      "key": "receiver",
      "value": "{{sdk.AccAddress of the module minting coins}}",
      "index": true
    },
    {
      "key": "amount",
      "value": "{{sdk.Coins being received}}",
      "index": true
    }
  ]
}
```

### BurnCoins

```json
{
  "type": "burn",
  "attributes": [
    {
      "key": "burner",
      "value": "{{sdk.AccAddress of the module burning coins}}",
      "index": true
    },
    {
      "key": "amount",
      "value": "{{sdk.Coins being burned}}",
      "index": true
    }
  ]
}
```

```json
{
  "type": "coin_spent",
  "attributes": [
    {
      "key": "spender",
      "value": "{{sdk.AccAddress of the module burning coins}}",
      "index": true
    },
    {
      "key": "amount",
      "value": "{{sdk.Coins being burned}}",
      "index": true
    }
  ]
}
```

### addCoins

```json
{
  "type": "coin_received",
  "attributes": [
    {
      "key": "receiver",
      "value": "{{sdk.AccAddress of the address beneficiary of the coins}}",
      "index": true
    },
    {
      "key": "amount",
      "value": "{{sdk.Coins being received}}",
      "index": true
    }
  ]
}
```

### subUnlockedCoins/DelegateCoins

```json
{
  "type": "coin_spent",
  "attributes": [
    {
      "key": "spender",
      "value": "{{sdk.AccAddress of the address which is spending coins}}",
      "index": true
    },
    {
      "key": "amount",
      "value": "{{sdk.Coins being spent}}",
      "index": true
    }
  ]
}
```
