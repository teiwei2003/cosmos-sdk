# 升级前处理

Cosmovisor 支持自定义升级前处理。 当您需要在执行升级之前实施较新版本中所需的应用程序配置更改时，请使用升级前处理。

使用 Cosmovisor 升级前处理是可选的。 如果未实施升级前处理，升级将继续。

例如，在升级前处理期间对 `app.toml` 设置进行所需的新版本更改。 升级前处理过程意味着升级后无需手动更新文件。

在应用程序二进制文件升级之前，Cosmovisor 调用了一个可由应用程序实现的“pre-upgrade”命令。

`pre-upgrade` 命令不接受任何命令行参数，预计会以以下退出代码终止: 

| Exit status code | How it is handled in Cosmosvisor                                                                                    |
|------------------|---------------------------------------------------------------------------------------------------------------------|
| `0`              | Assumes `pre-upgrade` command executed successfully and continues the upgrade.                                      |
| `1`              | Default exit code when `pre-upgrade` command has not been implemented.                                              |
| `30`             | `pre-upgrade` command was executed but failed. This fails the entire upgrade.                                       |
| `31`             | `pre-upgrade` command was executed but failed. But the command is retried until exit code `1` or `30` are returned. |

## Sample

以下是“pre-upgrade”命令的示例结构: 

```go
func preUpgradeCommand() *cobra.Command {
	cmd := &cobra.Command{
		Use:   "pre-upgrade",
		Short: "Pre-upgrade command",
        Long: "Pre-upgrade command to implement custom pre-upgrade handling",
		Run: func(cmd *cobra.Command, args []string) {

			err := HandlePreUpgrade()

			if err != nil {
				os.Exit(30)
			}

			os.Exit(0)

		},
	}

	return cmd
}
```

确保已在应用程序中注册了 pre-upgrade 命令:

```go
rootCmd.AddCommand(
		// ..
		preUpgradeCommand(),
		// ..
	)
```
