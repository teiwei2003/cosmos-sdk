# ADR 016:ベリファイアコンセンサスキーローテーション

## 変更ログ

-2019年10月23日:最初のドラフト
-2019年11月28日:キーローテーション料金を引き上げます

## 環境

より安全なベリファイアキー管理戦略(https://github.com/tendermint/tendermint/issues/1136など)のために、ベリファイアコンセンサスキーローテーション機能が長い間議論され、要求されてきました。したがって、CosmosSDKでバリデーターコンセンサスキーローテーションの最も単純な実装の1つを使用することをお勧めします。

Tendermintにはコンセンサスキーとベリファイアオペレータキーの間のマッピング情報がないため、Tendermintのコンセンサスロジックを更新する必要はありません。つまり、Tendermintの観点からは、ベリファイアのコンセンサスキーローテーションは交換別のコンセンサスキー。

さらに、このADRには、最も単純な形式のコンセンサスキーローテーションのみが含まれ、複数のコンセンサスキーの概念は考慮されていないことに注意してください。複数のコンセンサスキーのこの概念は、TendermintおよびCosmosSDKの長期的な目標であり続けます。

## 決定

###コンセンサスキーローテーションの疑似プロセス

-新しいランダムコンセンサスキーを作成します。
-「MsgRotateConsPubKey」を使用して、新しいコンセンサスキーがバリデーターオペレーターと結合され、バリデーターオペレーターキーからの署名があることを宣言するトランザクションを作成してブロードキャストします。
-チェーンのキーマッピングステータスが更新された直後は、古いコンセンサスキーをコンセンサスに参加させることはできません。
-検証のために新しいコンセンサスキーの使用を開始します。
-`MsgRotateConsPubKey`がブロックチェーンに送信されると、HSMとKMSを使用するバリデーターは、 `h`の高さの後に新しいローテーションキーを使用するようにHSMのコンセンサスキーを更新する必要があります。

###予防

-コンセンサスキーマッピング情報管理戦略
    -各キーマッピングの変更の履歴をkvstoreに保存します。
    -ステートマシンは、最新のバインド解除期間中に、特定のバリデーターオペレーターとペアになっている任意の高さの対応するコンセンサスキーを検索できます。
    -ステートマシンは、バインド解除期間を超えて履歴マッピング情報を必要としません。
-LCDおよびIBCに関連するキーローテーションコスト
    -LCDとIBCは、電源が頻繁に変更されると、フロー/計算の負担が発生します
    -現在のテンダーミントの設計では、コンセンサスキーの回転は、LCDまたはIBCの観点からは電力の変化と見なされます
    -したがって、不必要な頻繁なキーローテーション動作を最小限に抑えるために、最新のバインド解除期間中の最大ローテーション数を制限し、指数関数的に増加したローテーション料金も適用しました
-制限
    -バリデーターは、スパムを防ぐために、バインド解除期間中、コンセンサスキーを `MaxConsPubKeyRotations`より長くローテーションすることはできません。
    -パラメータはガバナンスによって決定され、ジェネシスファイルに保存されます。
-キーローテーション料金
    -検証者は、コンセンサスキーをローテーションするために `KeyRotationFee`を支払う必要があります。これは、次のように計算されます。
    -`KeyRotationFee` =(max( `VotingPowerPercentage` * 100、1)*` InitialKeyRotationFee`)* 2 ^(最新のバインド解除中の `ConsPubKeyRotationHistory`の回転数)
-証​​拠モジュール
    -証​​拠モジュールは、スラッシュキーパーから任意の高さの対応するコンセンサスキーを検索できるため、特定の高さに使用するコンセンサスキーを決定できます。
-abci.ValidatorUpdate
    -Tendermintは、ABCI通信( `ValidatorUpdate`)を介してコンセンサスキーを変更できるようになりました。
    -ベリファイアのコンセンサスキーの更新は、パワーをゼロに変更して新しいものを作成し、古いものを削除することで実行できます。
    -したがって、この機能を実現するために、テンダーミントのコードベースを変更する必要さえないことを願っています。
-`stakeing`モジュールの新しい作成パラメータ
    -`MaxConsPubKeyRotations`:バリデーターが最新のバインド解除期間中に実行できる回転の最大数。デフォルト値の10を使用することをお勧めします(11番目のキーローテーションは拒否されます)
    -`InitialKeyRotationFee`:直近のバインド解除期間中にキーローテーションが発生しなかった場合の初期キーローテーション料金。デフォルト値の1atomを使用することをお勧めします(最近のバインド解除期間中の最初のキーローテーションのコストは1atomです)

### ワークフロー

1.ベリファイアは、新しいコンセンサスキーペアを生成します。
2.バリデーターは、オペレーターキーと新しいConsPubKeyを使用して、 `MsgRotateConsPubKey`txを生成して署名します。 

    ```go
    type MsgRotateConsPubKey struct {
        ValidatorAddress  sdk.ValAddress
        NewPubKey         crypto.PubKey
    }
    ```

3.`handleMsgRotateConsPubKey`は `MsgRotateConsPubKey`を取得し、emitsイベントで` RotateConsPubKey`を呼び出します
4.`RotateConsPubKey`
      -`NewPubKey`が `ValidatorsByConsAddr`に複製されていないかどうかを確認します
      -`ConsPubKeyRotationHistory`を繰り返して、バリデーターがパラメーター `MaxConsPubKeyRotations`を超えていないかどうかを確認します。
      -署名アカウントに `KeyRotationFee`を支払うのに十分な残高があるかどうかを確認します
      -コミュニティ基金に `KeyRotationFee`を支払う
      -`validator.ConsPubKey`の `NewPubKey`をオーバーライドします
      -古い `ValidatorByConsAddr`を削除します
      -`NewPubKey`の `SetValidatorByConsAddr`
      -回転を追跡するために `ConsPubKeyRotationHistory`を追加しました 

    ```go
    type ConsPubKeyRotationHistory struct {
        OperatorAddress         sdk.ValAddress
        OldConsPubKey           crypto.PubKey
        NewConsPubKey           crypto.PubKey
        RotatedHeight           int64
    }
    ```

5. `ApplyAndReturnValidatorSetUpdates`は、` ConsPubKeyRotationHistory`と `ConsPubKeyRotationHistory.RotatedHeight == ctx.BlockHeight()`があるかどうかをチェックします。 

    ```go
    abci.ValidatorUpdate{
        PubKey: tmtypes.TM2PB.PubKey(OldConsPubKey),
        Power:  0,
    }

    abci.ValidatorUpdate{
        PubKey: tmtypes.TM2PB.PubKey(NewConsPubKey),
        Power:  v.ConsensusPower(),
    }
    ```

6.`AllocateTokens`の `previousVotes`の反復ロジックでは、` previousVote`は `OldConsPubKey`を使用して` ConsPubKeyRotationHistory`と一致し、トークン配布のバリデーターを置き換えます
7.`ValidatorSigningInfo`と `ValidatorMissedBlockBitArray`を` OldConsPubKey`から `NewConsPubKey`に移行します

-注:上記のすべての関数は、 `stakeing`モジュールに実装する必要があります。

## 状態

提案

## 結果

### ポジティブ

-検証者は、セキュリティポリシーを向上させるために、コンセンサスキーを即座にまたは定期的にローテーションできます
-バリデーターが古いコンセンサスキーを破棄するため、リモート攻撃に対するセキュリティが向上しました(https://nearprotocol.com/blog/long-range-attacks-and-a-new-fork-choice-rule)

### ネガティブ

-スラッシュモジュールは、高さごとに対応するバリデーターコンセンサスキーを見つける必要があるため、より多くの計算が必要です
-キーを頻繁に回転させると、軽いクライアントの二分法の効率が低下します
### ニュートラル

## 参照

-テンダーミントリポジトリ:https://github.com/tendermint/tendermint/issues/1136
-cosmos-sdkリポジトリ:https://github.com/cosmos/cosmos-sdk/issues/5231
-複数のコンセンサスキーについて:https://github.com/tendermint/tendermint/issues/1758#issuecomment-545291698 