# ブロックチェーンアーキテクチャ

## ステートマシン

ブロックチェーンのコアは[複製された決定論的ステートマシン]です。(https://en.wikipedia.org/wiki/State_machine_replication)。

ステートマシンは、マシンが複数の状態を持つことができるコンピュータサイエンスの概念ですが、一度に1つの状態しか持つことができません。 システムの現在の状態を表す[状態]と、状態遷移をトリガーする[トランザクション]があります。

状態SとトランザクションTが与えられると、ステートマシンは新しい状態S 'を返します。 

```
+--------+                 +--------+
|        |                 |        |
|   S    +---------------->+   S'   |
|        |    apply(T)     |        |
+--------+                 +--------+
```

実際には、トランザクションはプロセス効率を向上させるためにブロックにバンドルされています。 状態SとトランザクションブロックBが与えられると、ステートマシンは新しい状態S 'を返します。 

```
+--------+                              +--------+
|        |                              |        |
|   S    +----------------------------> |   S'   |
|        |   For each T in B: apply(T)  |        |
+--------+                              +--------+
```

ブロックチェーンのコンテキストでは、ステートマシンは決定論的です。 これは、ノードが特定の状態で同じ一連のトランザクションを開始および再生した場合、常に同じ最終状態で終了することを意味します。

Cosmos SDKは、開発者に、アプリケーションの状態、トランザクションタイプ、および状態遷移関数を定義するための最大の柔軟性を提供します。 Cosmos SDKを使用してステートマシンを構築するプロセスについては、次のセクションで詳しく説明します。 ただし、最初に、** Tendermint **を使用してステートマシンを複製する方法を見てみましょう。

## Tendermint

Cosmos SDKのおかげで、開発者はステートマシンを定義するだけで、[* Tendermint *](https://tendermint.com/docs/introduction/what-is-tendermint.html)がネットワークを介したレプリケーションを処理します。

```
                ^  +-------------------------------+  ^
                |  |                               |  |   Built with Cosmos SDK
                |  |  State-machine = Application  |  |
                |  |                               |  v
                |  +-------------------------------+
                |  |                               |  ^
Blockchain node |  |           Consensus           |  |
                |  |                               |  |
                |  +-------------------------------+  |   Tendermint Core
                |  |                               |  |
                |  |           Networking          |  |
                |  |                               |  |
                v  +-------------------------------+  v
```

[Tendermint](https://docs.tendermint.com/v0.34/introduction/what-is-tendermint.html)は、ブロックチェーンの処理を担当するアプリケーションに依存しないエンジンです。実際には、これはTendermintがトランザクションバイトの拡散とソートを担当することを意味します。 Tendermint Coreは、同じ名前のビザンチンフォールトトレランス(BFT)アルゴリズムに依存して、トランザクションの順序についてコンセンサスを得ています。

Tendermint [コンセンサスアルゴリズム](https://docs.tendermint.com/v0.34/introduction/what-is-tendermint.html#consensus-overview)は、*バリデーター*と呼ばれる一連の特別なノードで動作します。バリデーターは、トランザクションブロックをブロックチェーンに追加する責任があります。任意のブロックに、バリデーターセットVがあります。 Vのバリデーターは、次のブロックの提案者としてアルゴリズムによって選択されます。 Vの3分の2以上が署名されている場合* [prevote](https://docs.tendermint.com/v0.34/spec/consensus/consensus.html#prevote-step-height-h-round -r)*また* [precommit](https://docs.tendermint.com/v0.34/spec/consensus/consensus.html#precommit-step-height-h-round-r)*、およびすべてのトランザクションが含まれている場合はすべて有効です。バリデーターセットは、ステートマシンに書き込まれたルールによって変更できます。

## ABCI

Tendermintは、[ABCI](https://docs.tendermint.com/v0.34/spec/abci/)という名前のインターフェイスを介してトランザクションをアプリケーションに渡します。このインターフェイスは、アプリケーションで実装する必要があります。 

```
              +---------------------+
              |                     |
              |     Application     |
              |                     |
              +--------+---+--------+
                       ^   |
                       |   | ABCI
                       |   v
              +--------+---+--------+
              |                     |
              |                     |
              |     Tendermint      |
              |                     |
              |                     |
              +---------------------+
```

**Tendermintはトランザクションバイトのみを処理する**ことに注意してください。 これらのバイトの意味はわかりません。 Tendermintが行うことは、これらのトランザクションバイトを決定論的にソートすることです。 Tendermintは、ABCIを介してバイトをアプリケーションに渡し、トランザクションに含まれるメッセージが正常に処理されたかどうかを通知するリターンコードを期待します。

ABCIからの最も重要な情報は次のとおりです。

- `CheckTx`:Tendermint Coreはトランザクションを受信すると、それをアプリケーションに渡して、いくつかの基本的な要件を満たしているかどうかを確認します。 `CheckTx`は、ノード全体のメモリプールをスパムトランザクションから保護するために使用されます。 [`AnteHandler`](../basics/gas-fees.md#antehandler)という名前の特別なハンドラーを使用して、十分な料金があるかどうかの確認や署名の確認など、一連の確認手順を実行します。チェックが有効な場合、トランザクションは[mempool](https://docs.tendermint.com/v0.34/tendermint-core/mempool.html#mempool)に追加され、ピアノードに中継されます。 [CheckTx]はまだブロックに含まれていないため、トランザクションを処理しない(つまり、状態の変更は発生しない)ことに注意してください。
- `DeliverTx`:Tendermint Coreが[有効なブロック](https://docs.tendermint.com/v0.34/spec/blockchain/blockchain.html#validation)を受信すると、ブロック内のすべてのトランザクションが処理用の[DeliverTx]アプリケーション。状態遷移が発生するのはこの段階です。 `AnteHandler`は、トランザクション内の各メッセージの実際の[` Msg`サービス](../building-modules/msg-services.md)RPCとともに再度実行されます。
- `BeginBlock`/`EndBlock`:これらのメッセージは、ブロックにトランザクションが含まれているかどうかに関係なく、各ブロックの最初と最後に実行されます。トリガーロジックの自動実行は便利です。ただし、計算コストの高いループはブロックチェーンの速度を低下させたり、ループが無限大の場合はブロックチェーンをフリーズさせたりする可能性があるため、注意して進めてください。

ABCIメソッドの詳細については、[Tendermint Documentation](https://docs.tendermint.com/v0.34/spec/abci/abci.html#overview)を参照してください。

Tendermintで構築されたアプリケーションは、基盤となるローカルTendermintエンジンと通信するために、ABCIインターフェースを実装する必要があります。幸い、ABCIインターフェースを実装する必要はありません。 Cosmos SDKは、サンプル実装を[baseapp](./sdk-design.md#baseapp)の形式で提供します。

## 次へ{hide}

[Cosmos SDKの高度な設計原則](./sdk-design.md){hide}をお読みください