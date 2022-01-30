# ADR 032:入力されたイベント

## 変更ログ

-2020年9月28日:最初のドラフト

## 著者

-アニルクマール(@anilcse)
-ジャックザンポリン(@jackzampolin)
-アダムボザンシ(@boz)

## 状態

提案

## 概要

現在、Cosmos SDKでは、イベントは各メッセージのハンドラーと[BeginBlock]および[EndBlock]で定義されています。各モジュールは各イベントのタイプを定義せず、 `map [string] string`として実装されます。最も重要なことは、これにより、多くの生の文字列の照合と解析が必要になるため、これらのイベントの使用が困難になることです。提案の焦点は、各モジュールで定義された**型付きイベント**を使用するようにイベントを更新し、イベントの発行とサブスクライブを容易にすることです。このワークフローは、Akashネットワークチームの経験に基づいています。

## 環境

現在、Cosmos SDKでは、イベントは各メッセージのハンドラーで定義されています。つまり、各モジュールには、各イベントの正規タイプのセットがありません。最も重要なことは、これにより、多くの生の文字列の照合と解析が必要になるため、これらのイベントの使用が困難になることです。提案の焦点は、各モジュールで定義された**型付きイベント**を使用するようにイベントを更新し、イベントの発行とサブスクライブを容易にすることです。このワークフローは、Akashネットワークチームの経験に基づいています。

[私たちのプラットフォーム](http://github.com/ovrclk/akash)は、プロバイダー(データセンター-新しい注文に入札し、作成されたリースを監視する)とユーザー(アプリケーション開発者-アプリケーションリストをプロバイダー)終了。さらに、Akashチームは現在IBC [`relayer`](https://github.com/ovrclk/relayer)を維持しています。これは、もう1つの非常にイベント駆動型のプロセスです。これらのインフラストラクチャのコア部分を処理し、Kubernetes開発から学んだ教訓を統合する際に、私たちのチームはCosmosSDKモジュールで型付きイベントを定義して使用するための標準的な方法を開発しました。このタイプのイベント駆動型アプリケーションを構築するときに非常に便利です。

Cosmos SDKは、[ペギー]、他のフック領域、IBC、DeFiなどのアプリケーションでより広く使用されるため、ユーザーが必要とする新機能をサポートするために、イベント駆動型アプリケーションの需要が爆発的に増加します。すべてのCosmosSDKアプリケーションがイベント駆動型アプリケーションを迅速かつ簡単に構築してコアアプリケーションを支援できるように、調査結果をCosmosSDKにアップロードすることをお勧めします。ウォレット、エクスチェンジ、ブラウザ、defiプロトコルはすべてこの作業の恩恵を受けます。

この提案が受け入れられると、ユーザーはgoでイベント駆動型のCosmos SDKアプリケーションを構築し、特定のイベントタイプのEventHandlerを記述して、CosmosSDKで定義されたEventEmittersに渡すことができます。

このリファクタリング後にイベントを消費する方法を説明するために、この提案の最後に詳細な例が含まれています。

この提案は、モジュール間の通信ではなく、ブロックチェーンのクライアントとしてこれらのイベントをどのように使用するかについて具体的に説明しています。

## 決定

__Step-1__: `types`パッケージに追加の関数を実装します:` EmitTypedEvent`および `ParseTypedEvent`関数 

```go
/.types/events.go

/.EmitTypedEvent takes typed event and emits converting it into sdk.Event
func (em *EventManager) EmitTypedEvent(event proto.Message) error {
	evtType := proto.MessageName(event)
	evtJSON, err := codec.ProtoMarshalJSON(event)
	if err != nil {
		return err
	}

	var attrMap map[string]json.RawMessage
	err = json.Unmarshal(evtJSON, &attrMap)
	if err != nil {
		return err
	}

	var attrs []abci.EventAttribute
	for k, v := range attrMap {
		attrs = append(attrs, abci.EventAttribute{
			Key:   []byte(k),
			Value: v,
		})
	}

	em.EmitEvent(Event{
		Type:       evtType,
		Attributes: attrs,
	})

	return nil
}

/.ParseTypedEvent converts abci.Event back to typed event
func ParseTypedEvent(event abci.Event) (proto.Message, error) {
	concreteGoType := proto.MessageType(event.Type)
	if concreteGoType == nil {
		return nil, fmt.Errorf("failed to retrieve the message of type %q", event.Type)
	}

	var value reflect.Value
	if concreteGoType.Kind() == reflect.Ptr {
		value = reflect.New(concreteGoType.Elem())
	} else {
		value = reflect.Zero(concreteGoType)
    }

	protoMsg, ok := value.Interface().(proto.Message)
	if !ok {
		return nil, fmt.Errorf("%q does not implement proto.Message", event.Type)
	}

	attrMap := make(map[string]json.RawMessage)
	for _, attr := range event.Attributes {
		attrMap[string(attr.Key)] = attr.Value
	}

	attrBytes, err := json.Marshal(attrMap)
	if err != nil {
		return nil, err
	}

	err = jsonpb.Unmarshal(strings.NewReader(string(attrBytes)), protoMsg)
	if err != nil {
		return nil, err
	}

	return protoMsg, nil
}
```

ここで、 `EmitTypedEvent`は、型付きイベントを入力として受け取り、それにjsonシリアル化を適用する` EventManager`のメソッドです。 次に、JSONキーと値のペアを `event.Attributes`にマップし、` sdk.Event`の形式で出力します。 `Event.Type`は、プロトメッセージのタイプURLになります。

テンダーミントWebSocketで発行されるイベントをサブスクライブすると、それらは `abci.Event`の形式で発行されます。 `ParseTypedEvent`は、イベントを解析して元のプロトメッセージに戻します。

__Step-2__:各モジュールのmsgsの型付きイベントのプロトタイプ定義を追加します。

たとえば、 `gov`モジュールの` MsgSubmitProposal`を使用して、このイベントタイプを実装しましょう。  

```protobuf
/.proto/cosmos/gov/v1beta1/gov.proto
/.Add typed event definition

package cosmos.gov.v1beta1;

message EventSubmitProposal {
    string from_address   = 1;
    uint64 proposal_id    = 2;
    TextProposal proposal = 3;
}
```

__Step-3__: `sdk.EmitTypedEvent`を使用して作成および発行された型付きイベントを使用するように、イベント発行をリファクタリングします。

```go
/.x/gov/handler.go
func handleMsgSubmitProposal(ctx sdk.Context, keeper keeper.Keeper, msg types.MsgSubmitProposalI) (*sdk.Result, error) {
    ...
    types.Context.EventManager().EmitTypedEvent(
        &EventSubmitProposal{
            FromAddress: fromAddress,
            ProposalId: id,
            Proposal: proposal,
        },
    )
    ...
}
```

#### `クライアント`でこれらのタイプのイベントをサブスクライブする方法

>注:以下の完全なコード例

ユーザーは `client.Context.Client.Subscribe`を使用して、` EventHandler`を使用して発行されたイベントをサブスクライブおよび使用できるようになります。

Akash Networkは、単純な[`pubsub`](https://github.com/ovrclk/akash/blob/90d258caeb933b611d575355b8df281208a214f8/pubsub/bus.go#L20)を構築しました。これを使用して、タイプイベントとして `abci.Events`および[publish](https://github.com/ovrclk/akash/blob/90d258caeb933b611d575355b8df281208a214f8/events/publish.go#L21)をサブスクライブできます。

クライアントを見つけるためのこのプロセスの詳細については、次のコード例を参照してください。

## 結果

### ポジティブ

*現在のCosmosSDKでのイベント実装の一貫性が向上しました
*イベントを処理し、イベント駆動型アプリケーションの作成を容易にする、より人間工学的な方法を提供します
*この実装は、 `EventHandler`のミドルウェアエコシステムをサポートします

### ネガティブ

## 公開イベントの詳細なコード例

ADRは、これらのイベントを発行して使用するための可用性を追加することも推奨しています。したがって、開発者は書くだけで済みます
`EventHandler`は、実行するアクションを定義します。  

```go
/.EventEmitter is a type that describes event emitter functions
/.This should be defined in `types/events.go`
type EventEmitter func(context.Context, client.Context, ...EventHandler) error

/.EventHandler is a type of function that handles events coming out of the event bus
/.This should be defined in `types/events.go`
type EventHandler func(proto.Message) error

/.Sample use of the functions below
func main() {
    ctx, cancel := context.WithCancel(context.Background())

    if err := TxEmitter(ctx, client.Context{}.WithNodeURI("tcp://localhost:26657"), SubmitProposalEventHandler); err != nil {
        cancel()
        panic(err)
    }

    return
}

/.SubmitProposalEventHandler is an example of an event handler that prints proposal details
/.when any EventSubmitProposal is emitted.
func SubmitProposalEventHandler(ev proto.Message) (err error) {
    switch event := ev.(type) {
   ..Handle governance proposal events creation events
    case govtypes.EventSubmitProposal:
       ..Users define business logic here e.g.
        fmt.Println(ev.FromAddress, ev.ProposalId, ev.Proposal)
        return nil
    default:
        return nil
    }
}

/.TxEmitter is an example of an event emitter that emits just transaction events. This can and
/.should be implemented somewhere in the Cosmos SDK. The Cosmos SDK can include an EventEmitters for tm.event='Tx'
/.and/or tm.event='NewBlock' (the new block events may contain typed events)
func TxEmitter(ctx context.Context, cliCtx client.Context, ehs ...EventHandler) (err error) {
   ..Instantiate and start tendermint RPC client
    client, err := cliCtx.GetNode()
    if err != nil {
        return err
    }

    if err = client.Start(); err != nil {
        return err
    }

   ..Start the pubsub bus
    bus := pubsub.NewBus()
    defer bus.Close()

   ..Initialize a new error group
    eg, ctx := errgroup.WithContext(ctx)

   ..Publish chain events to the pubsub bus
    eg.Go(func() error {
        return PublishChainTxEvents(ctx, client, bus, simapp.ModuleBasics)
    })

   ..Subscribe to the bus events
    subscriber, err := bus.Subscribe()
    if err != nil {
        return err
    }

	/.Handle all the events coming out of the bus
	eg.Go(func() error {
        var err error
        for {
            select {
            case <-ctx.Done():
                return nil
            case <-subscriber.Done():
                return nil
            case ev := <-subscriber.Events():
                for _, eh := range ehs {
                    if err = eh(ev); err != nil {
                        break
                    }
                }
            }
        }
        return nil
	})

	return group.Wait()
}

/.PublishChainTxEvents events using tmclient. Waits on context shutdown signals to exit.
func PublishChainTxEvents(ctx context.Context, client tmclient.EventsClient, bus pubsub.Bus, mb module.BasicManager) (err error) {
   ..Subscribe to transaction events
    txch, err := client.Subscribe(ctx, "txevents", "tm.event='Tx'", 100)
    if err != nil {
        return err
    }

   ..Unsubscribe from transaction events on function exit
    defer func() {
        err = client.UnsubscribeAll(ctx, "txevents")
    }()

   ..Use errgroup to manage concurrency
    g, ctx := errgroup.WithContext(ctx)

   ..Publish transaction events in a goroutine
    g.Go(func() error {
        var err error
        for {
            select {
            case <-ctx.Done():
                break
            case ed := <-ch:
                switch evt := ed.Data.(type) {
                case tmtypes.EventDataTx:
                    if !evt.Result.IsOK() {
                        continue
                    }
                   ..range over events, parse them using the basic manager and
                   ..send them to the pubsub bus
                    for _, abciEv := range events {
                        typedEvent, err := sdk.ParseTypedEvent(abciEv)
                        if err != nil {
                            return er
                        }
                        if err := bus.Publish(typedEvent); err != nil {
                            bus.Close()
                            return
                        }
                        continue
                    }
                }
            }
        }
        return err
	})

   ..Exit on error or context cancelation
    return g.Wait()
}
```

## 参照

-[バス経由でカスタムイベントを公開する](https://github.com/ovrclk/akash/blob/90d258caeb933b611d575355b8df281208a214f8/events/publish.go#L19-L58)
-[`クライアント`でイベントを使用](https://github.com/ovrclk/deploy/blob/bf6c633ab6c68f3026df59efd9982d6ca1bf0561/cmd/event-handlers.go#L57) 