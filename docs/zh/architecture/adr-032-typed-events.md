# ADR 032:类型化事件

## 变更日志

- 2020 年 9 月 28 日:初稿

##作者

- 阿尼尔·库马尔 (@anilcse)
- 杰克赞普林 (@jackzampolin)
- 亚当·博赞奇 (@boz)

## 地位

建议的

## 摘要

目前在 Cosmos SDK 中，事件在每条消息的处理程序中以及“BeginBlock”和“EndBlock”中定义。每个模块都没有为每个事件定义类型，它们被实现为`map[string]string`。最重要的是，这使得这些事件难以使用，因为它需要大量的原始字符串匹配和解析。该提案的重点是更新事件以使用每个模块中定义的 **typed events**，从而使事件的发出和订阅变得更加容易。此工作流程来自 Akash Network 团队的经验。

## 语境

目前在 Cosmos SDK 中，事件在每个消息的处理程序中定义，这意味着每个模块没有每个事件的规范类型集。最重要的是，这使得这些事件难以使用，因为它需要大量的原始字符串匹配和解析。该提案的重点是更新事件以使用每个模块中定义的 **typed events**，从而使事件的发出和订阅变得更加容易。此工作流程来自 Akash Network 团队的经验。

[我们的平台](http://github.com/ovrclk/akash) 需要在提供者(数据中心 - 竞标新订单并监听创建的租约)和用户(应用程序开发人员 - 到将应用清单发送到提供者)端。此外，Akash 团队现在正在维护 IBC [`relayer`](https://github.com/ovrclk/relayer)，这是另一个非常事件驱动的过程。在处理这些基础设施的核心部分，并整合从 Kubernetes 开发中吸取的经验教训时，我们的团队开发了一种标准方法来定义和使用 Cosmos SDK 模块中的类型化事件。我们发现它在构建这种类型的事件驱动应用程序时非常有用。

随着 Cosmos SDK 被更广泛地用于“peggy”、其他挂钩区域、IBC、DeFi 等应用程序，对事件驱动应用程序的需求将呈爆炸式增长，以支持用户所需的新功能。我们建议将我们的发现上传到 Cosmos SDK 中，以使所有 Cosmos SDK 应用程序能够快速轻松地构建事件驱动的应用程序，以帮助其核心应用程序。钱包、交易所、浏览器和 defi 协议都将从这项工作中受益。

如果这个提议被接受，用户将能够在 go 中构建事件驱动的 Cosmos SDK 应用程序，只需为他们的特定事件类型编写 `EventHandler` 并将它们传递给 Cosmos SDK 中定义的 `EventEmitters`。

本提案的末尾包含了一个详细的示例，说明在此重构后如何消费事件。

这个提案具体是关于如何将这些事件作为区块链的客户端来消费，而不是用于模块间的通信。

## 决定

__Step-1__:在 `types` 包中实现附加功能:`EmitTypedEvent` 和 `ParseTypedEvent` 函数 

```go
//types/events.go

//EmitTypedEvent takes typed event and emits converting it into sdk.Event
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

//ParseTypedEvent converts abci.Event back to typed event
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

在这里，`EmitTypedEvent` 是`EventManager` 上的一个方法，它将类型化事件作为输入并对其应用 json 序列化。 然后它将 JSON 键/值对映射到 `event.Attributes` 并以 `sdk.Event` 的形式发出它。 `Event.Type` 将是 proto 消息的类型 URL。

当我们在tendermint websocket上订阅发出的事件时，它们以`abci.Event`的形式发出。 `ParseTypedEvent` 将事件解析回它的原始 proto 消息。

__Step-2__:在每个模块中为 msgs 的类型化事件添加原型定义:

例如，让我们以 `gov` 模块的 `MsgSubmitProposal` 来实现这个事件的类型。 

```protobuf
//proto/cosmos/gov/v1beta1/gov.proto
//Add typed event definition

package cosmos.gov.v1beta1;

message EventSubmitProposal {
    string from_address   = 1;
    uint64 proposal_id    = 2;
    TextProposal proposal = 3;
}
```

__Step-3__:重构事件发射以使用使用`sdk.EmitTypedEvent`创建和发射的类型化事件:

```go
//x/gov/handler.go
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

#### 如何在`Client` 中订阅这些类型的事件

> 注意:下面的完整代码示例

用户将能够使用 `client.Context.Client.Subscribe` 订阅并使用使用 `EventHandler` 发出的事件。

Akash Network 构建了一个简单的 [`pubsub`](https://github.com/ovrclk/akash/blob/90d258caeb933b611d575355b8df281208a214f8/pubsub/bus.go#L20)。 这可用于订阅 `abci.Events` 和 [publish](https://github.com/ovrclk/akash/blob/90d258caeb933b611d575355b8df281208a214f8/events/publish.go#L21) 作为类型事件。

有关此流程查找客户端的更多详细信息，请参阅以下代码示例。 

## Consequences

### Positive

* 提高了当前 Cosmos SDK 中事件实现的一致性 
* 提供一种更符合人体工程学的方式来处理事件并促进编写事件驱动的应用程序
* 此实现将支持`EventHandler`s 的中间件生态系统 

### Negative

## Detailed code example of publishing events

该 ADR 还建议添加可供性以发出和使用这些事件。 这样开发人员只需要编写
`EventHandler`s 定义了他们想要采取的行动。 

```go
//EventEmitter is a type that describes event emitter functions
//This should be defined in `types/events.go`
type EventEmitter func(context.Context, client.Context, ...EventHandler) error

//EventHandler is a type of function that handles events coming out of the event bus
//This should be defined in `types/events.go`
type EventHandler func(proto.Message) error

//Sample use of the functions below
func main() {
    ctx, cancel := context.WithCancel(context.Background())

    if err := TxEmitter(ctx, client.Context{}.WithNodeURI("tcp://localhost:26657"), SubmitProposalEventHandler); err != nil {
        cancel()
        panic(err)
    }

    return
}

//SubmitProposalEventHandler is an example of an event handler that prints proposal details
//when any EventSubmitProposal is emitted.
func SubmitProposalEventHandler(ev proto.Message) (err error) {
    switch event := ev.(type) {
   //Handle governance proposal events creation events
    case govtypes.EventSubmitProposal:
       //Users define business logic here e.g.
        fmt.Println(ev.FromAddress, ev.ProposalId, ev.Proposal)
        return nil
    default:
        return nil
    }
}

//TxEmitter is an example of an event emitter that emits just transaction events. This can and
//should be implemented somewhere in the Cosmos SDK. The Cosmos SDK can include an EventEmitters for tm.event='Tx'
//and/or tm.event='NewBlock' (the new block events may contain typed events)
func TxEmitter(ctx context.Context, cliCtx client.Context, ehs ...EventHandler) (err error) {
   //Instantiate and start tendermint RPC client
    client, err := cliCtx.GetNode()
    if err != nil {
        return err
    }

    if err = client.Start(); err != nil {
        return err
    }

   //Start the pubsub bus
    bus := pubsub.NewBus()
    defer bus.Close()

   //Initialize a new error group
    eg, ctx := errgroup.WithContext(ctx)

   //Publish chain events to the pubsub bus
    eg.Go(func() error {
        return PublishChainTxEvents(ctx, client, bus, simapp.ModuleBasics)
    })

   //Subscribe to the bus events
    subscriber, err := bus.Subscribe()
    if err != nil {
        return err
    }

	//Handle all the events coming out of the bus
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

//PublishChainTxEvents events using tmclient. Waits on context shutdown signals to exit.
func PublishChainTxEvents(ctx context.Context, client tmclient.EventsClient, bus pubsub.Bus, mb module.BasicManager) (err error) {
   //Subscribe to transaction events
    txch, err := client.Subscribe(ctx, "txevents", "tm.event='Tx'", 100)
    if err != nil {
        return err
    }

   //Unsubscribe from transaction events on function exit
    defer func() {
        err = client.UnsubscribeAll(ctx, "txevents")
    }()

   //Use errgroup to manage concurrency
    g, ctx := errgroup.WithContext(ctx)

   //Publish transaction events in a goroutine
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
                   //range over events, parse them using the basic manager and
                   //send them to the pubsub bus
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

   //Exit on error or context cancelation
    return g.Wait()
}
```

## References

- [通过总线发布自定义事件](https://github.com/ovrclk/akash/blob/90d258caeb933b611d575355b8df281208a214f8/events/publish.go#L19-L58)
- [使用`Client`中的事件](https://github.com/ovrclk/deploy/blob/bf6c633ab6c68f3026df59efd9982d6ca1bf0561/cmd/event-handlers.go#L57) 
