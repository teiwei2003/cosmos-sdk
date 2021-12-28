# cosmovisor

`cosmovisor`は、Cosmos SDKアプリケーションバイナリの小さなプロセスマネージャーであり、着信チェーンアップグレードの提案についてガバナンスモジュールを監視します。 承認されたプロポーザルを検出すると、`cosmovisor`は自動的に新しいバイナリをダウンロードし、現在のバイナリを停止し、古いバイナリから新しいバイナリに切り替え、最後に新しいバイナリでノードを再起動します。

#### 設計

Cosmovisorは、`CosmosSDK`アプリのラッパーとして使用するように設計されています。

* 関連するアプリ(`DAEMON_NAME`環境変数で構成)に引数を渡します。`cosmovisor run arg1 arg2 ....`を実行すると、`app arg1 arg2 ...`が実行されます。
* 必要に応じて再起動してアップグレードすることでアプリを管理します。
* 位置引数ではなく、環境変数を使用して構成されます。

*注:アプリケーションの新しいバージョンがインプレースストア移行を実行するように設定されていない場合は、新しいバイナリで`cosmovisor`を再起動する前に、移行を手動で実行する必要があります。 このため、アプリケーションでインプレースストア移行を採用することをお勧めします。*

*注:バリデーターが自動ダウンロードオプション([お勧めしません](＃auto-download))を有効にし、現在Cosmos SDK`v0.42`を使用してアプリケーションを実行している場合は、 Cosmovisor [`v0.1`](https://github.com/cosmos/cosmos-sdk/releases/tag/cosmovisor%2Fv0.1.0)を使用するには。 自動ダウンロードオプションが有効になっている場合、Cosmovisorの新しいバージョンはCosmos SDK`v0.44.3`以前をサポートしていません。*

## 贡献

CosmovisorはCosmosSDKモノレポの一部ですが、独自のリリーススケジュールを持つ別個のモジュールです。

リリースブランチの形式は`release/cosmovisor/vA.B.x`です。ここで、AとBは数字です(例:`release/cosmovisor/v0.1.x`)。 リリースには、`cosmovisor/vA.B.C`の形式を使用してタグが付けられます。

## 設定

### インストール

最新バージョンの`cosmovisor`をインストールするには、次のコマンドを実行します。

```
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@latest
```

以前のバージョンをインストールするには、バージョンを指定できます。重要:Cosmos-SDK v0.44.3以前(v0.44.2など)を使用し、自動ダウンロード機能を使用するチェーンは、Cosmovisorv0.1.0を使用する必要があります

```
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@v0.1.0
```

Cosmovisor v1.0.0を使用している場合は、cosmovisorのバージョンを確認できますが、`v0.1.0`では確認できません。

cosmos-sdkリポジトリをプルし、正しいバージョンに切り替えて次のようにビルドすることにより、ソースからインストールすることもできます。 

```
git clone git@github.com:cosmos/cosmos-sdk
cd cosmos-sdk
git checkout cosmovisor/vx.x.x
cd cosmovisor
make
```

これにより、現在のディレクトリにcosmovisorが構築されます。 その後、次のようにマシンのPATHに配置することができます。

```
cp cosmovisor ~/go/bin/cosmovisor
```

*注:go`v1.15`以前を使用している場合は、`go get`を使用する必要があり、プロジェクトディレクトリの外部でコマンドを実行することをお勧めします。*

### コマンドライン引数と環境変数

`cosmovisor`に渡される最初の引数は、`cosmovisor`が実行するアクションです。 オプションは次のとおりです。

*`help`、`--help`、または`-h`-`cosmovisor`ヘルプ情報を出力し、`cosmovisor`構成を確認します。
*`run`-提供された残りの引数を使用して、構成されたバイナリを実行します。
*`version`、または`--version`-`cosmovisor`バージョンを出力し、`version`引数を指定してバイナリを実行します。

`cosmovisor run`に渡されるすべての引数は、(サブプロセスとして)アプリケーションバイナリに渡されます。`cosmovisor`は、サブプロセスの`/dev/stdout`と`/dev/stderr`を独自のものとして返します。 このため、`cosmovisor run`は、アプリケーションバイナリで使用可能なコマンドライン引数以外のコマンドライン引数を受け入れることができません。

*注:アクション引数の1つを指定せずに`cosmovisor`を使用することは非推奨です。 後方互換性のために、最初の引数がアクション引数でない場合、`run`が想定されます。 ただし、このフォールバックは将来のバージョンで削除される可能性があるため、常に`run`を指定することをお勧めします。*

`cosmovisor`は、環境変数から構成を読み取ります。

*`DAEMON_HOME`は、ジェネシスバイナリ、アップグレードバイナリ、および各バイナリに関連付けられた追加の補助ファイル(`$ HOME/.gaiad`、`$ HOME/など)を含む`cosmovisor/`ディレクトリが保持される場所です。 regend`、`$ HOME/.simd`など)。
*`DAEMON_NAME`は、バイナリ自体の名前です(たとえば、`gaiad`、`regend`、`simd`など)。
*`DAEMON_ALLOW_DOWNLOAD_BINARIES`(*オプション*)を`true`に設定すると、新しいバイナリの自動ダウンロードが有効になります(セキュリティ上の理由から、これはバリデーターではなくフルノードを対象としています)。デフォルトでは、`cosmovisor`は新しいバイナリを自動ダウンロードしません。
*`DAEMON_RESTART_AFTER_UPGRADE`(*オプション*、デフォルト=`true`)、`true`の場合、アップグレードが成功した後、同じコマンドライン引数とフラグを使用して(ただし新しいバイナリを使用して)サブプロセスを再起動します。それ以外の場合(`false`)、`cosmovisor`はアップグレード後に実行を停止し、システム管理者が手動で再起動する必要があります。再起動はアップグレード後にのみ行われ、エラーが発生した後にサブプロセスを自動再起動しないことに注意してください。
*`DAEMON_POLL_INTERVAL`は、アップグレード計画ファイルをポーリングするための間隔の長さです。値は、数値(ミリ秒単位)または期間(「1s」など)のいずれかです。デフォルト:300ミリ秒。
*`UNSAFE_SKIP_BACKUP`(デフォルトは`false`)は、`true`に設定されている場合、バックアップを実行せずに直接アップグレードします。それ以外の場合(`false`、デフォルト)は、アップグレードを試行する前にデータをバックアップします。デフォルト値のfalseは、障害が発生した場合やバックアップをロールバックする必要がある場合に役立ち、推奨されます。デフォルトのバックアップオプション`UNSAFE_SKIP_BACKUP = false`を使用することをお勧めします。
*`DAEMON_PREUPGRADE_MAX_RETRIES`(デフォルトは`0`)。終了ステータスが`31`になった後、アプリケーションで`pre-upgrade`を呼び出す最大回数。最大再試行回数の後、cosmovisorはアップグレードに失敗します。

### フォルダレイアウト

`$ DAEMON_HOME/cosmovisor`は、完全に`cosmovisor`とそれによって制御されるサブプロセスに属することが期待されます。 フォルダの内容は次のように構成されています。

```
.
├── current -> genesis or upgrades/<name>
├── genesis
│   └── bin
│       └── $DAEMON_NAME
└── upgrades
    └── <name>
        ├── bin
        │   └── $DAEMON_NAME
        └── upgrade-info.json
```

`cosmovisor/`ディレクトリには、アプリケーションの各バージョンのサブディレクトリが含まれます(つまり、`genesis`または`upgrades/<name>`)。 各サブディレクトリ内には、アプリケーションバイナリ(つまり、`bin/$ DAEMON_NAME`)と、各バイナリに関連付けられた追加の補助ファイルがあります。`current`は、現在アクティブなディレクトリ(つまり、`genesis`または`upgrades/<name>`)へのシンボリックリンクです。`upgrades/<name>`の`name`変数は、アップグレードモジュールプランで指定されているアップグレードのURIエンコードされた名前です。

`$ DAEMON_HOME/cosmovisor`は*アプリケーションバイナリ*のみを保存することに注意してください。`cosmovisor`バイナリ自体は、任意の一般的な場所(たとえば、`/usr/local/bin`)に格納できます。 アプリケーションは引き続き、デフォルトのデータディレクトリ(例:`$ HOME/.gaiad`)または`--home`フラグで指定されたデータディレクトリにデータを保存します。`$ DAEMON_HOME`はデータディレクトリから独立しており、任意の場所に設定できます。`$ DAEMON_HOME`をデータディレクトリと同じディレクトリに設定すると、次のような構成になります。

```
.gaiad
├── config
├── data
└── cosmovisor
```

## 使用法

システム管理者の責任は次のとおりです。

-`cosmovisor`バイナリのインストール
-ホストのinitシステムの構成(例:`systemd`、`launchd`など)
-環境変数を適切に設定する
-`genesis`フォルダを手動でインストールする
-`upgrades/<name>`フォルダを手動でインストールします

`cosmovisor`は、最初の開始時(つまり、`current`リンクが存在しない場合)に`current`リンクが`genesis`を指すように設定し、システム管理者が数日前に準備できるように、正しい時点でバイナリの切り替えを処理します。 アップグレード時にリラックスしてください。

ダウンロード可能なバイナリをサポートするには、アップグレードバイナリごとにtarballをパッケージ化して、正規URLから利用できるようにする必要があります。 さらに、ジェネシスバイナリと利用可能なすべてのアップグレードバイナリを含むtarballをパッケージ化して利用できるようにすることで、最初からフルノードを同期するために必要なすべてのバイナリを簡単にダウンロードできます。

`DAEMON`固有のコードと操作(例:tendermint構成、アプリケーションデータベース、同期ブロックなど)はすべて期待どおりに機能します。 コマンドラインフラグや環境変数などのアプリケーションバイナリのディレクティブも期待どおりに機能します。

### アップグレードの検出

`cosmovisor`は`$ DAEMON_HOME/data/upgrade-info.json`ファイルをポーリングして新しいアップグレード手順を探します。 ファイルは、アップグレードが検出され、ブロックチェーンがアップグレードの高さに達したときに、`BeginBlocker`のx/upgradeモジュールによって作成されます。
次のヒューリスティックは、アップグレードを検出するために適用されます。

+ 起動時、`cosmovisor`は、`current/bin/`であるバイナリを除いて、現在実行中のアップグレードについてあまり知りません。`current/update-info.json`ファイルを読み取って、現在のアップグレード名に関する情報を取得しようとします。
+`cosmovisor/current/upgrade-info.json`も`data/upgrade-info.json`も存在しない場合、`cosmovisor`は`data/upgrade-info.json`ファイルがアップグレードをトリガーするのを待ちます。
+`cosmovisor/current/upgrade-info.json`が存在しないが、`data/upgrade-info.json`が存在する場合、`cosmovisor`は`data/upgrade-info.json`にあるものはすべて有効であると見なします アップグレードリクエスト。 この場合、`cosmovisor`は`data/upgrade-info.json`の`name`属性に従ってすぐにアップグレードを試みます。
+ それ以外の場合、`cosmovisor`は`upgrade-info.json`の変更を待ちます。 新しいアップグレード名がファイルに記録されるとすぐに、`cosmovisor`はアップグレードメカニズムをトリガーします。

アップグレードメカニズムがトリガーされると、`cosmovisor`は次のようになります。

1.`DAEMON_ALLOW_DOWNLOAD_BINARIES`が有効になっている場合は、新しいバイナリを`cosmovisor/<name>/bin`に自動ダウンロードすることから始めます(ここで、`<name>`は`upgrade-info.json:name`属性です)。
2.`current`シンボリックリンクを更新して新しいディレクトリをポイントし、`data/upgrade-info.json`を`cosmovisor/current/upgrade-info.json`に保存します。

### 自動ダウンロード

通常、`cosmovisor`では、アップグレードを実行する前に、システム管理者が関連するすべてのバイナリをディスクに配置する必要があります。 ただし、そのような制御を必要とせず、自動セットアップが必要な場合(おそらく、検証されていないフルノードを同期していて、メンテナンスをほとんど行わない場合)、別のオプションがあります。

**注:バイナリが使用可能かどうかを事前に確認しないため、自動ダウンロード**の使用はお勧めしません。 バイナリのダウンロードに問題がある場合、cosmovisorは停止し、アプリを再起動しません(チェーンの停止につながる可能性があります)。

`DAEMON_ALLOW_DOWNLOAD_BINARIES`が`true`に設定されていて、アップグレードがトリガーされたときにローカルバイナリが見つからない場合、`cosmovisor`は`dataの`info`属性の指示に基づいてバイナリ自体をダウンロードしてインストールしようとします/upgrade-info.json`ファイル。 ファイルはx/upgradeモジュールによって構築され、アップグレード`Plan`オブジェクトからのデータが含まれています。`Plan`には、ダウンロードを指定するために次の2つの有効な形式のいずれかを持つことが期待される情報フィールドがあります。

1. os/architecture->バイナリURIマップをアップグレードプラン情報フィールドの`" binaries "`キーの下にJSONとして保存します。 例えば:

```json
{
  "binaries": {
    "linux/amd64":"https://example.com/gaia.zip?checksum=sha256:aec070645fe53ee3b3763059376134f058cc337247c978add178b6ccdfb0019f"
  }
}
```

一度に複数のバイナリを含めて、複数の環境が正しいバイナリを受け取るようにすることができます。

```json
{
  "binaries": {
    "linux/amd64":"https://example.com/gaia.zip?checksum=sha256:aec070645fe53ee3b3763059376134f058cc337247c978add178b6ccdfb0019f",
    "linux/arm64":"https://example.com/gaia.zip?checksum=sha256:aec070645fe53ee3b3763059376134f058cc337247c978add178b6ccdfb0019f",
    "darwin/amd64":"https://example.com/gaia.zip?checksum=sha256:aec070645fe53ee3b3763059376134f058cc337247c978add178b6ccdfb0019f"
    }
}
```

これを提案として提出するときは、スペースがないことを確認してください。`gaiad`を使用したコマンドの例は次のようになります。

```
> gaiad tx gov submit-proposal software-upgrade Vega \
--title Vega \
--deposit 100uatom \
--upgrade-height 7368420 \
--upgrade-info '{"binaries":{"linux/amd64":"https://github.com/cosmos/gaia/releases/download/v6.0.0-rc1/gaiad-v6.0.0-rc1-linux-amd64","linux/arm64":"https://github.com/cosmos/gaia/releases/download/v6.0.0-rc1/gaiad-v6.0.0-rc1-linux-arm64","darwin/amd64":"https://github.com/cosmos/gaia/releases/download/v6.0.0-rc1/gaiad-v6.0.0-rc1-darwin-amd64"}}' \
--description "upgrade to Vega" \
--gas 400000 \
--from user \
--chain-id test \
--home test/val2 \
--node tcp://localhost:36657 \
--yes
```

2. 上記の形式のすべての情報を含むファイルへのリンクを保存します(たとえば、ブロックチェーンをいっぱいにせずに多くのバイナリ、変更ログ情報などを指定する場合)。 例えば:

```
https://example.com/testnet-1001-info.json?checksum=sha256:deaaa99fda9407c4dbe1d04bd49bab0cc3c1dd76fa392cd55a9425be074af01e
```

`cosmovisor`がトリガーされて新しいバイナリがダウンロードされると、`cosmovisor`は`" binarys "`フィールドを解析し、[go-getter](https://github.com/hashicorp/go-getter)を使用して新しいバイナリをダウンロードします。 、新しいバイナリを`upgrades/<name>`フォルダに解凍して、手動でインストールしたかのように実行できるようにします。

このメカニズムで強力なセキュリティ保証を提供するには、すべてのURLにSHA256/512チェックサムを含める必要があることに注意してください。 これにより、誰かがサーバーをハッキングしたりDNSを乗っ取ったりした場合でも、誤ったバイナリが実行されないことが保証されます。`go-getter`は、ダウンロードされたファイルが提供されている場合、チェックサムと一致することを常に確認します。`go-getter`は、ディレクトリへのアーカイブの解凍も処理します(この場合、ダウンロードリンクは`bin`ディレクトリ内のすべてのデータの`zip`ファイルを指している必要があります)。。

Linuxでsha256チェックサムを適切に作成するには、`sha256sum`ユーティリティを使用できます。 例えば: 

```
sha256sum ./testdata/repo/zip_directory/autod.zip
```

結果は次のようになります。`29139e1381b8177aec909fab9a75d11381cab5adf7d3af0c05ff1c9c117743a7`。

より長いハッシュを使用したい場合は`sha512sum`を使用し、壊れたハッシュを使用したい場合は`md5sum`を使用することもできます。 どちらを選択する場合でも、URLのチェックサム引数でハッシュアルゴリズムを適切に設定してください。

## 例:SimAppのアップグレード

次の手順は、Cosmos SDKのソースコードに付属のシミュレーションアプリケーション(`simapp`)を使用した`cosmovisor`のデモンストレーションを提供します。 次のコマンドは、`cosmos-sdk`リポジトリ内から実行されます。

まず、最新の`v0.42`リリースを確認してください。

```
git checkout v0.42.7
```

`simd`バイナリをコンパイルします。

```
make build
```

`〜/.simapp`をリセットします(実稼働環境では絶対に行わないでください)。

```
./build/simd unsafe-reset-all
```

テスト用に`simd`バイナリを設定します。

```
./build/simd config chain-id test
./build/simd config keyring-backend test
./build/simd config broadcast-mode block
```

ノードを初期化し、以前のジェネシスファイルを上書きします(本番環境では絶対に行わないでください)。

<!-- TODO: init does not read chain-id from config -->

```
./build/simd init test --chain-id test --overwrite
```

`〜/.simapp/config/app.toml`で最低ガス価格を`0stake`に設定します。

```
minimum-gas-prices = "0stake"
```

バリデーターの新しいキーを作成してから、ジェネシスアカウントとトランザクションを追加します。

<!-- TODO: add-genesis-account does not read keyring-backend from config -->
<!-- TODO: gentx does not read chain-id from config -->

```
./build/simd keys add validator
./build/simd add-genesis-account validator 1000000000stake --keyring-backend test
./build/simd gentx validator 1000000stake --chain-id test
./build/simd collect-gentxs
```

必要な環境変数を設定します。

```
export DAEMON_NAME=simd
export DAEMON_HOME=$HOME/.simapp
```

オプションの環境変数を設定して、自動再起動をトリガーします。

```
export DAEMON_RESTART_AFTER_UPGRADE=true
```

ジェネシスバイナリ用のフォルダを作成し、`simd`バイナリをコピーします。

```
mkdir -p $DAEMON_HOME/cosmovisor/genesis/bin
cp ./build/simd $DAEMON_HOME/cosmovisor/genesis/bin
```

このデモンストレーションのために、`genesis.json`の`voting_period`を20秒(`20s`)の短縮された時間に修正します。

```
cat <<< $(jq '.app_state.gov.voting_params.voting_period = "20s"' $HOME/.simapp/config/genesis.json) > $HOME/.simapp/config/genesis.json
```

次に、コードの変更をシミュレートするために、`simapp`で変更をハードコーディングします。`simapp/app.go`で、`UpgradeKeeper`初期化を含む行を見つけます。 次のようになります。

```go
app.UpgradeKeeper = upgradekeeper.NewKeeper(skipUpgradeHeights, keys[upgradetypes.StoreKey], appCodec, homePath)
```

その行の後に、以下を追加します。

```go
app.UpgradeKeeper.SetUpgradeHandler("test1", func(ctx sdk.Context, plan upgradetypes.Plan) {
	//Add some coins to a random account
	addr, err := sdk.AccAddressFromBech32("cosmos18cgkqduwuh253twzmhedesw3l7v3fm37sppt58")
	if err != nil {
		panic(err)
	}
	err = app.BankKeeper.AddCoins(ctx, addr, sdk.Coins{sdk.Coin{Denom: "stake", Amount: sdk.NewInt(345600000)}})
	if err != nil {
		panic(err)
	}
})
```

次に、アップグレードハンドラーを追加して`simd`バイナリを再コンパイルします。

```
make build
```

アップグレードバイナリ用のフォルダを作成し、`simd`バイナリをコピーします。

```
mkdir -p $DAEMON_HOME/cosmovisor/upgrades/test1/bin
cp ./build/simd $DAEMON_HOME/cosmovisor/upgrades/test1/bin
```

`cosmosvisor`を開始します。

```
cosmovisor run start
```

新しいターミナルウィンドウを開き、デポジットと投票とともにアップグレード提案を送信します(これらのコマンドは互いに20秒以内に実行する必要があります)。

```
./build/simd tx gov submit-proposal software-upgrade test1 --title upgrade --description upgrade --upgrade-height 20 --from validator --yes
./build/simd tx gov deposit 1 10000000stake --from validator --yes
./build/simd tx gov vote 1 yes --from validator --yes
```

アップグレードは高さ20で自動的に行われます。
