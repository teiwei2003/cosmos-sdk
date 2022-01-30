# カスタムメイド

IBCを使用し、他のチェーンにパケットを送信するようにアプリケーションを構成する方法を学びます。 {まとめ}

このドキュメントは、独自のブロック間チェーンを作成したい開発者向けのガイドとして使用できます。
[ユースケース](https://github.com/cosmos/ics/blob/master/ibc/4_IBC_USECASES.md)の通信プロトコル(IBC)アプリケーションをカスタマイズします。

IBCプロトコルのモジュラー設計により、IBC
アプリケーション開発者は、クライアントの低レベルの詳細を気にする必要はありません。
接続と認証の検証。ただし、下位レベルの簡単な説明
アプリケーション開発者がIBCを高レベルで理解できるように、スタックを提供します
プロトコル。次に、ドキュメントは、アプリケーションに最も関連する抽象化レイヤーについて詳しく説明します
開発者(チャネルとポート)、および独自のカスタムデータパッケージを定義する方法を説明し、
`IBCModule`コールバック。

モジュールがIBCを介して対話できるようにするには、ポートにバインドし、独自のパケットデータと確認構造、およびそれらをエンコード/デコードする方法を定義し、実装する必要があります。
`IBCModule`インターフェース。 IBCアプリケーションの作成方法の詳細については次のとおりです。
モジュールは正しいです。

## 前提条件の読書

- [IBC 概要](./overview.md)) {prereq}
- [IBC デフォルトの統合](./integration.md) {prereq}

## カスタムIBCアプリケーションモジュールを作成する

### `IBCModule`インターフェースとコールバックを実装する

Cosmos SDKは、すべてのIBCモジュールが[`IBCModule`を実装することを期待しています
インターフェース](https://github.com/cosmos/ibc-go/tree/main/modules/core/05-port/types/module.go)。 この
インターフェイスには、IBCがモジュールに実装することを期待するすべてのコールバックが含まれています。 このセクションでは紹介します
チャネルハンドシェイクの実行中にコールバックが呼び出されました。

以下は、モジュールによって実装されることが期待されるチャネルハンドシェイクコールバックです。 

```go
//Called by IBC Handler on MsgOpenInit
func (k Keeper) OnChanOpenInit(ctx sdk.Context,
    order channeltypes.Order,
    connectionHops []string,
    portID string,
    channelID string,
    channelCap *capabilitytypes.Capability,
    counterparty channeltypes.Counterparty,
    version string,
) error {
  //OpenInit must claim the channelCapability that IBC passes into the callback
    if err := k.ClaimCapability(ctx, chanCap, host.ChannelCapabilityPath(portID, channelID)); err != nil {
			return err
	}

  //... do custom initialization logic

  //Use above arguments to determine if we want to abort handshake
  //Examples: Abort if order == UNORDERED,
  //Abort if version is unsupported
    err := checkArguments(args)
    return err
}

//Called by IBC Handler on MsgOpenTry
OnChanOpenTry(
    ctx sdk.Context,
    order channeltypes.Order,
    connectionHops []string,
    portID,
    channelID string,
    channelCap *capabilitytypes.Capability,
    counterparty channeltypes.Counterparty,
    version,
    counterpartyVersion string,
) error {
  //Module may have already claimed capability in OnChanOpenInit in the case of crossing hellos
  //(ie chainA and chainB both call ChanOpenInit before one of them calls ChanOpenTry)
  //If the module can already authenticate the capability then the module already owns it so we don't need to claim
  //Otherwise, module does not have channel capability and we must claim it from IBC
    if !k.AuthenticateCapability(ctx, chanCap, host.ChannelCapabilityPath(portID, channelID)) {
      //Only claim channel capability passed back by IBC module if we do not already own it
        if err := k.scopedKeeper.ClaimCapability(ctx, chanCap, host.ChannelCapabilityPath(portID, channelID)); err != nil {
            return err
        }
    }

  //... do custom initialization logic

  //Use above arguments to determine if we want to abort handshake
    err := checkArguments(args)
    return err
}

//Called by IBC Handler on MsgOpenAck
OnChanOpenAck(
    ctx sdk.Context,
    portID,
    channelID string,
    counterpartyVersion string,
) error {
  //... do custom initialization logic

  //Use above arguments to determine if we want to abort handshake
    err := checkArguments(args)
    return err
}

//Called by IBC Handler on MsgOpenConfirm
OnChanOpenConfirm(
    ctx sdk.Context,
    portID,
    channelID string,
) error {
  //... do custom initialization logic

  //Use above arguments to determine if we want to abort handshake
    err := checkArguments(args)
    return err
}
```

The channel closing handshake will also invoke module callbacks that can return errors to abort the
closing handshake. Closing a channel is a 2-step handshake, the initiating chain calls
`ChanCloseInit` and the finalizing chain calls `ChanCloseConfirm`.

```go
//Called by IBC Handler on MsgCloseInit
OnChanCloseInit(
    ctx sdk.Context,
    portID,
    channelID string,
) error {
  //... do custom finalization logic

  //Use above arguments to determine if we want to abort handshake
    err := checkArguments(args)
    return err
}

//Called by IBC Handler on MsgCloseConfirm
OnChanCloseConfirm(
    ctx sdk.Context,
    portID,
    channelID string,
) error {
  //... do custom finalization logic

  //Use above arguments to determine if we want to abort handshake
    err := checkArguments(args)
    return err
}
```

#### チャネルハンドシェイクバージョンネゴシエーション

アプリケーションモジュールは、チャネルハンドシェイク中に使用されたバージョン管理を検証する必要があります。

* `ChanOpenInit`コールバックは、` MsgChanOpenInit.Version`が有効かどうかを確認する必要があります
* `ChanOpenTry`コールバックは、` MsgChanOpenTry.Version`が有効であり、 `MsgChanOpenTry.CounterpartyVersion`が有効であることを確認する必要があります。
* `ChanOpenAck`コールバックは、` MsgChanOpenAck.CounterpartyVersion`が有効でサポートされていることを確認する必要があります。

バージョンは文字列である必要がありますが、任意のバージョン管理構造を実装できます。 アプリケーションが計画している場合
線形バージョンがあり、セマンティックバージョン管理が推奨されます。 アプリケーションのリリースが予定されている場合
メジャーバージョン間でさまざまな機能がある場合は、同じバージョン管理スキームを使用することをお勧めします
IBCとして。 このバージョン管理スキームは、バージョン識別子と互換性のある機能セットを指定します
その識別子。 有効なバージョンの選択には、互換性のあるバージョン識別子の選択が含まれます
アプリケーションがこのバージョンでサポートする機能のサブセット。 この構造はこれに使用されます
解決策は[03-connection/types]にあります。

バージョンタイプは文字列であるため、アプリケーションは簡単なバージョン検証を行うことができます
文字列照合を介して、またはすでに実装されているバージョン管理システムを使用してプロトタイプを渡すことができます
必要に応じて、エンコードバージョンを各ハンドシェイク呼び出しにエンコードします。

ICS20は現在、サポートされている単一のバージョンを使用して、基本的な文字列照合を実装しています。

### バインディングポート

現在、アプリケーションの初期化時にポートをバインドする必要があります。 モジュールは `InitGenesis`のポートにバインドできます
このような:

```go
func InitGenesis(ctx sdk.Context, keeper keeper.Keeper, state types.GenesisState) {
  //... other initialization logic

  //Only try to bind to port if it is not already bound, since we may already own
  //port capability from capability InitGenesis
    if !isBound(ctx, state.PortID) {
      //module binds to desired ports on InitChain
      //and claims returned capabilities
        cap1 := keeper.IBCPortKeeper.BindPort(ctx, port1)
        cap2 := keeper.IBCPortKeeper.BindPort(ctx, port2)
        cap3 := keeper.IBCPortKeeper.BindPort(ctx, port3)

      //NOTE: The module's scoped capability keeper must be private
        keeper.scopedKeeper.ClaimCapability(cap1)
        keeper.scopedKeeper.ClaimCapability(cap2)
        keeper.scopedKeeper.ClaimCapability(cap3)
    }

  //... more initialization logic
}
```

### カスタムデータパッケージ

チャネルを介して接続されたモジュールは、チャネルを介して送信するアプリケーションデータに同意する必要があります。
チャネル、およびそれらがどのようにそれをエンコード/デコードするか。 IBCは、このプロセスが開始されたため、このプロセスを指定しませんでした
各アプリケーションモジュールに移動して、このプロトコルの実装方法を決定します。 ただし、ほとんどの場合
アプリケーションこれは、チャネルハンドシェイク中のバージョンネゴシエーションとして発生します。 もっと
複雑なバージョンネゴシエーションは、チャネルオープニングハンドシェイク内で実現できます。
シンプルなバージョンネゴシエーションは[ibc-transfermodule](https://github.com/cosmos/ibc-go/tree/main/modules/apps/transfer/module.go)に実装されています。

したがって、モジュールは、カスタムデータパケットデータ構造と、明確に定義されたメソッドを定義する必要があります
`[] byte`としてエンコードおよびデコードします。 

```go
//Custom packet data defined in application module
type CustomPacketData struct {
  //Custom fields ...
}

EncodePacketData(packetData CustomPacketData) []byte {
  //encode packetData to bytes
}

DecodePacketData(encoded []byte) (CustomPacketData) {
  //decode from bytes to packet data
}
```

次に、モジュールはIBCを介して送信する前にパケットデータをエンコードする必要があります。 

```go
//Sending custom application packet data
data := EncodePacketData(customPacketData)
packet.Data = data
IBCChannelKeeper.SendPacket(ctx, packet)
```

パケットを受信するモジュールは、[PacketData]を期待する構造にデコードして、次のことができるようにする必要があります。
行動を起こす。 

```go
//Receiving custom application packet data (in OnRecvPacket)
packetData := DecodePacketData(packet.Data)
//handle received custom packet data
```

#### パケットフロー処理

IBCがモジュールがチャネルハンドシェイクコールバックを実装することを期待しているように、IBCもモジュールを期待しています
チャネルを介したパケットのフローを処理するためのコールバックを実装します。

モジュールAとモジュールBが相互に接続されると、リピーターはデータパケットの中継を開始できます。
チャネルで前後に確認します。

![IBCデータパケットのフローチャート](https://media.githubusercontent.com/media/cosmos/ics/master/spec/ics-004-channel-and-packet-semantics/packet-state-machine.png)

つまり、成功したパケットフローは次のように機能します。

1. モジュールAは、IBCモジュールを介してデータパケットを送信します
2. データパケットはモジュールBによって受信されます
3. モジュールBがデータパケットの確認応答を書き込む場合、モジュールAは処理します
    確認
4. タイムアウト前にデータパケットが正常に受信されなかった場合、モジュールAは処理します
    パケットがタイムアウトしました。

##### データパケットを送信する

モジュールは送信アクションを開始するため、モジュールはコールバックを介してデータパケットを送信しません
パケットは、IBCのパケットフローの他の部分にメッセージを送信する代わりに、IBCモジュールに送信されます。
モジュールは、コールバックを使用してポートバインディングモジュールの実行をトリガーする必要があります。 したがって、送信するには
モジュールは、[IBCChannelKeeper]で[SendPacket]を呼び出すだけで済みます。 

```go
//retrieve the dynamic capability for this channel
channelCap := scopedKeeper.GetCapability(ctx, channelCapName)
//Sending custom application packet data
data := EncodePacketData(customPacketData)
packet.Data = data
//Send packet to IBC, authenticating with channelCap
IBCChannelKeeper.SendPacket(ctx, channelCap, packet)
```

::: 警告
モジュールが所有していないチャネルでパケットを送信しないようにするために、IBCは
このモジュールは、データパケットの送信元チャネルに適切なチャネル機能を提供します。
:::

##### パケットを受信する

受信したパケットを処理するには、モジュールは `OnRecvPacket`コールバックを実装する必要があります。 これは
IBCがデータパケットが有効で正しく処理されていることを証明した後、IBCモジュールによって呼び出されます。
ゴールキーパー。 したがって、 `OnRecvPacket`コールバックは、適切な状態にすることだけを気にする必要があります
パケットが有効かどうかを気にせずに、特定のパケットのデータを変更します。

モジュールは確認をバイト文字列として返し、IBCハンドラーに返すことができます。
次に、IBCハンドラーはパケットの確認応答を送信して、リピーターが中継できるようにします。
確認はトランスミッタモジュールに返されます。

```go
OnRecvPacket(
    ctx sdk.Context,
    packet channeltypes.Packet,
) (res *sdk.Result, ack []byte, abort error) {
  //Decode the packet data
    packetData := DecodePacketData(packet.Data)

  //do application state changes based on packet data
  //and return result, acknowledgement and abortErr
  //Note: abortErr is only not nil if we need to abort the entire receive packet, and allow a replay of the receive.
  //If the application state change failed but we do not want to replay the packet,
  //simply encode this failure with relevant information in ack and return nil error
    res, ack, abortErr := processPacket(ctx, packet, packetData)

  //if we need to abort the entire receive packet, return error
    if abortErr != nil {
        return nil, nil, abortErr
    }

  //Encode the ack since IBC expects acknowledgement bytes
    ackBytes := EncodeAcknowledgement(ack)

    return res, ackBytes, nil
}
```

::: 警告
受信したパケット全体を実行したい場合、 `OnRecvPacket`は**のみ**エラーを返す必要があります
(IBC処理を含む)復元されます。 これにより、この状況でパケットを再生できるようになります
リレーのエラーにより、パケット処理が失敗しました。

パケットデータの処理中にアプリケーションレベルのエラーが発生した場合、ほとんどの場合、
パケット処理を再開したくない。 代わりに、この失敗を次のようにエンコードすることをお勧めします
データパケットの処理を確認して完了します。 これにより、パケットを再生できなくなります。
また、メッセージが受信されたときに、送信者モジュールがこの状況を潜在的に修正できるようにします
認める。 この手法の例は、 `ibc-transfer`モジュールにあります
[`OnRecvPacket`](https://github.com/cosmos/ibc-go/tree/main/modules/apps/transfer/module.go)。
:::

### Acknowledgements

同期パケット処理の場合、モジュールはパケットの受信と処理時に確認応答をコミットする場合があります。
パケットが受信された後のある時点でパケットが処理される場合(非同期実行)、確認応答
パケットがアプリケーションによって処理されると書き込まれます。これは、パケットの受信後の可能性があります。

注:ほとんどのブロックチェーンモジュールは、モジュールが確認応答を処理して書き込む同期実行モデルを使用する必要があります
IBCモジュールからパケットを受信するとすぐにパケットの場合。

その後、この確認応答を元の送信者チェーンに中継して、アクションを実行できます。
謝辞の内容によって異なります。

パケットデータがIBCに対して不透明であったように、確認応答も同様に不透明です。モジュールは合格する必要があり、
IBCモジュールで確認応答をバイト文字列として受信します。

したがって、モジュールは確認応答をエンコード/デコードする方法について合意する必要があります。を作成するプロセス
確認応答構造体とそのエンコードおよびデコードは、パケットデータと非常によく似ています。
上記の例。 [ICS 04](https://github.com/cosmos/ics/tree/master/spec/ics-004-channel-and-packet-semantics#acknowledgement-envelope)
確認応答の推奨形式を指定します。この確認応答タイプは、からインポートできます。
[チャネルタイプ](https://github.com/cosmos/ibc-go/tree/main/modules/core/04-channel/types)。

モジュールは任意の確認応答構造体を選択できますが、デフォルトの確認応答タイプはIBC [ここ](https://github.com/cosmos/ibc-go/blob/main/proto/ibc/core/channel/v1/channel)によって提供されます。プロト):

```proto
//Acknowledgement is the recommended acknowledgement format to be used by
//app-specific protocols.
//NOTE: The field numbers 21 and 22 were explicitly chosen to avoid accidental
//conflicts with other protobuf message formats used for acknowledgements.
//The first byte of any message with this format will be the non-ASCII values
//`0xaa` (result) or `0xb2` (error). Implemented as defined by ICS:
//https://github.com/cosmos/ics/tree/master/spec/ics-004-channel-and-packet-semantics#acknowledgement-envelope
message Acknowledgement {
//response contains either a result or an error and must be non-empty
  oneof response {
    bytes  result = 21;
    string error  = 22;
  }
}
```

#### パケットの確認

モジュールが確認を書き込んだ後、リピーターは確認をトランスミッタモジュールに中継して戻すことができます。 送信モジュールは
次に、 `OnAcknowledgementPacket`コールバックを使用して確認を処理します。 コンテンツ
確認応答は、チャネル上のモジュールに完全に依存します(パケットデータと同様)。ただし、
多くの場合、パケットが正常に処理されたかどうかに関する情報が含まれている可能性があります
パケット処理が失敗した場合、いくつかの追加データが修復に役立つ可能性があります。

モジュールは、パケットデータのエンコード/デコード標準について合意に達する責任があるため、
確認、IBCは確認を `[] byte`としてこのコールバックに渡します。 折り返し電話
確認のデコードと処理を担当します。 

```go
OnAcknowledgementPacket(
    ctx sdk.Context,
    packet channeltypes.Packet,
    acknowledgement []byte,
) (*sdk.Result, error) {
  //Decode acknowledgement
    ack := DecodeAcknowledgement(acknowledgement)

  //process ack
    res, err := processAck(ack)
    return res, err
}
```

#### タイムアウトパケット

パケットが正常に受信される前にパケットタイムアウト期間に達した場合、または
チャネルのもう一方の端は、データパケットが正常に受信される前に閉じられ、その後受信されます
チェーンはそれを処理できなくなります。 したがって、送信チェーンは使用する必要があります
この状況を処理するための `OnTimeoutPacket`。 IBCモジュールは、タイムアウトが
それは確かに効果的であるため、私たちのモジュールは、一度行うことのステートマシンロジックを実装するだけで済みます
タイムアウト期間が終了し、データパケットを受信できなくなりました。

```go
OnTimeoutPacket(
    ctx sdk.Context,
    packet channeltypes.Packet,
) (*sdk.Result, error) {
  //do custom timeout logic
}
```

### Routing

上記のように、モジュールはIBCモジュールインターフェイス(チャネルを含む)を実装する必要があります
ハンドシェイクコールバックとパケット処理コールバック)。 このインターフェースの具体的な実現
モジュール名は、IBC`Router`にルートとして登録するために使用する必要があります。

```go
//app.go
func NewApp(...args) *App {
//...

//Create static IBC router, add module routes, then set and seal it
ibcRouter := port.NewRouter()

ibcRouter.AddRoute(ibctransfertypes.ModuleName, transferModule)
//Note: moduleCallbacks must implement IBCModule interface
ibcRouter.AddRoute(moduleName, moduleCallbacks)

//Setting Router will finalize all routes by sealing router
//No more routes can be added
app.IBCKeeper.SetRouter(ibcRouter)
```

## 実例

IBCアプリケーションの実際の動作例については、[ibc-transfer]モジュールを確認できます。
上記のすべてを実装します。

モジュールの便利な部分は次のとおりです。

[バインディング転送
ポート)(https://github.com/cosmos/ibc-go/blob/main/modules/apps/transfer/types/genesis.go)

[転送を送信
データパッケージ](https://github.com/cosmos/ibc-go/blob/main/modules/apps/transfer/keeper/relay.go)

[IBCを実装する
コールバック](https://github.com/cosmos/ibc-go/blob/main/modules/apps/transfer/module.go)

## 次へ{hide}

[モジュールの構築](https://github.com/cosmos/cosmos-sdk/blob/master/docs/building-modules/intro.md)を理解する{hide}
