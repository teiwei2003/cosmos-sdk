# ADR 013:可観測性

## 変更ログ

-20-01-2020:最初のドラフト

## 状態

提案

## 環境

テレメトリは、アプリケーションが実行していることと、アプリケーションがどのように進行しているかをデバッグおよび理解するために不可欠です。
パフォーマンス。私たちの目標は、CosmosSDKのモジュールおよびその他のコア部分からのインジケーターを公開することです。

さらに、私たちの目標は、オペレーターが選択できる複数の構成可能な受信機をサポートすることです。
デフォルトでは、テレメトリが有効になっている場合、アプリはメトリクスを追跡して公開する必要があります
メモリに保存されます。オペレーターは追加の受信機を有効にすることを選択できます。サポートするのは
[Prometheus](https://prometheus.io/)これで、実際の戦闘でテストされたため、セットアップとオープンソースが簡単になりました。
そして、豊富なエコシステムツールがあります。

また、インジケーターをCosmos SDKに最もシームレスな方法で統合して、
インジケーターは、あまり摩擦することなく、自由に追加または削除できます。このために使用します
[go-metrics](https://github.com/armon/go-metrics)ライブラリ。

最後に、オペレーターはテレメトリと特定の構成オプションを有効にできます。有効になっている場合、インジケーター
`/metrics？format = {text | prometheus}`を介してAPIサーバーを介して公開されます。

## 決定

テレメトリ設定を定義するために、 `app.toml`に追加の構成ブロックを追加します。 

```toml
###############################################################################
###                         Telemetry Configuration                         ###
###############################################################################

[telemetry]

# Prefixed with keys to separate services
service-name = {{ .Telemetry.ServiceName }}

# Enabled enables the application telemetry functionality. When enabled,
# an in-memory sink is also enabled by default. Operators may also enabled
# other sinks such as Prometheus.
enabled = {{ .Telemetry.Enabled }}

# Enable prefixing gauge values with hostname
enable-hostname = {{ .Telemetry.EnableHostname }}

# Enable adding hostname to labels
enable-hostname-label = {{ .Telemetry.EnableHostnameLabel }}

# Enable adding service to labels
enable-service-label = {{ .Telemetry.EnableServiceLabel }}

# PrometheusRetentionTime, when positive, enables a Prometheus metrics sink.
prometheus-retention-time = {{ .Telemetry.PrometheusRetentionTime }}
```

指定された構成では、メモリ内とPrometheusの2つのシンクが可能です。 `メトリクス`を作成します
オペレーターのすべてのブートストラップを実行するタイプであるため、メトリックのキャプチャーはシームレスになります。 

```go
//Metrics defines a wrapper around application telemetry functionality. It allows
//metrics to be gathered at any point in time. When creating a Metrics object,
//internally, a global metrics is registered with a set of sinks as configured
//by the operator. In addition to the sinks, when a process gets a SIGUSR1, a
//dump of formatted recent metrics will be sent to STDERR.
type Metrics struct {
  memSink           *metrics.InmemSink
  prometheusEnabled bool
}

//Gather collects all registered metrics and returns a GatherResponse where the
//metrics are encoded depending on the type. Metrics are either encoded via
//Prometheus or JSON if in-memory.
func (m *Metrics) Gather(format string) (GatherResponse, error) {
  switch format {
  case FormatPrometheus:
    return m.gatherPrometheus()

  case FormatText:
    return m.gatherGeneric()

  case FormatDefault:
    return m.gatherGeneric()

  default:
    return GatherResponse{}, fmt.Errorf("unsupported metrics format: %s", format)
  }
}
```

さらに、[メトリクス]を使用すると、任意の時点で現在のメトリクスのセットを収集できます。 一
オペレーターは、信号SIGUSR1を送信して、フォーマットされた標識をSTDERRにダンプおよび印刷することもできます。

アプリケーションの起動およびビルドフェーズで、[Telemetry.Enabled]が[true]の場合、
APIサーバーは、 `Metrics`オブジェクトへの参照インスタンスを作成し、メトリックを登録します
対応する処理手順。 

```go
func (s *Server) Start(cfg config.Config) error {
./...

  if cfg.Telemetry.Enabled {
    m, err := telemetry.New(cfg.Telemetry)
    if err != nil {
      return err
    }

    s.metrics = m
    s.registerMetrics()
  }

./...
}

func (s *Server) registerMetrics() {
  metricsHandler := func(w http.ResponseWriter, r *http.Request) {
    format := strings.TrimSpace(r.FormValue("format"))

    gr, err := s.metrics.Gather(format)
    if err != nil {
      rest.WriteErrorResponse(w, http.StatusBadRequest, fmt.Sprintf("failed to gather metrics: %s", err))
      return
    }

    w.Header().Set("Content-Type", gr.ContentType)
    _, _ = w.Write(gr.Metrics)
  }

  s.Router.HandleFunc("/metrics", metricsHandler).Methods("GET")
}
```

アプリケーション開発者は、カウンター、ゲージ、要約、およびキー/値メトリックを追跡できます。
プロファイリングメトリックを活用するためにモジュールに必要な追加のリフティング。これを行うには、次のように簡単です。 

```go
func (k BaseKeeper) MintCoins(ctx sdk.Context, moduleName string, amt sdk.Coins) error {
  defer metrics.MeasureSince(time.Now(), "MintCoins")
./...
}
```

## 結果

### ポジティブ

-アプリケーションのパフォーマンスと動作を理解する

### ネガティブ

### ニュートラル

## 参照 
