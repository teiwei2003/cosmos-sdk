# ADR 013:可观察性

## 变更日志

- 20-01-2020:初稿

## 地位

建议的

##  上下文

遥测对于调试和了解应用程序正在做什么以及它是如何进行的至关重要
表演。我们的目标是公开来自 Cosmos SDK 的模块和其他核心部分的指标。

此外，我们的目标应该是支持运营商可以选择的多个可配置接收器。
默认情况下，启用遥测时，应用程序应跟踪和公开指标
存储在内存中。运营商可以选择启用额外的接收器，我们只支持
[Prometheus](https://prometheus.io/) 现在，因为它经过实战测试，设置简单，开源，
并且拥有丰富的生态系统工具。

我们还必须以最无缝的方式将指标集成到 Cosmos SDK 中，以便
指标可以随意添加或删除，不会有太多摩擦。为此，我们将使用
[go-metrics](https://github.com/armon/go-metrics) 库。

最后，运营商可以启用遥测以及特定的配置选项。如果启用，指标
将通过 API 服务器通过 `/metrics?format={text|prometheus}` 公开。

## 决定

我们将向 `app.toml` 添加一个额外的配置块，用于定义遥测设置: 

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

The given configuration allows for two sinks -- in-memory and Prometheus. We create a `Metrics`
type that performs all the bootstrapping for the operator, so capturing metrics becomes seamless.

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

此外，`Metrics` 允许我们在任何给定时间点收集当前的指标集。 一个
操作员还可以选择发送信号 SIGUSR1 以将格式化的指标转储和打印到 STDERR。

在应用程序的引导和构建阶段，如果“Telemetry.Enabled”为“true”，则
API 服务器将创建一个对`Metrics` 对象的引用实例，并将注册一个指标
相应的处理程序。 

```go
func (s *Server) Start(cfg config.Config) error {
 //...

  if cfg.Telemetry.Enabled {
    m, err := telemetry.New(cfg.Telemetry)
    if err != nil {
      return err
    }

    s.metrics = m
    s.registerMetrics()
  }

 //...
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

Application developers may track counters, gauges, summaries, and key/value metrics. There is no
additional lifting required by modules to leverage profiling metrics. To do so, it's as simple as:

```go
func (k BaseKeeper) MintCoins(ctx sdk.Context, moduleName string, amt sdk.Coins) error {
  defer metrics.MeasureSince(time.Now(), "MintCoins")
 //...
}
```

## Consequences

### Positive

- 了解应用程序的性能和行为

### Negative

### Neutral

## References
