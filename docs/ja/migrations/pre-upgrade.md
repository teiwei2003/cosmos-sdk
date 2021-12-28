# アップグレード前の処理

Cosmovisorは、カスタムのアップグレード前処理をサポートしています。 アップグレードを実行する前に、新しいバージョンで必要なアプリケーション構成の変更を実装する必要がある場合は、アップグレード前の処理を使用してください。

Cosmovisorを使用したアップグレード前の処理はオプションです。 アップグレード前の処理が実装されていない場合、アップグレードは続行されます。

たとえば、アップグレード前の処理中に、必要な新しいバージョンの変更を `app.toml`設定に加えます。 アップグレード前のプロセスは、アップグレード後にファイルを手動で更新する必要がないことを意味します。

アプリケーションのバイナリファイルをアップグレードする前に、Cosmovisorはアプリケーションで実装できる「pre-upgrade」コマンドを呼び出します。

`pre-upgrade`コマンドはコマンドラインパラメータを受け入れず、次の終了コードで終了することが期待されています。

| Exit status code | How it is handled in Cosmosvisor                                                                                    |
|------------------|---------------------------------------------------------------------------------------------------------------------|
| `0`              | Assumes `pre-upgrade` command executed successfully and continues the upgrade.                                      |
| `1`              | Default exit code when `pre-upgrade` command has not been implemented.                                              |
| `30`             | `pre-upgrade` command was executed but failed. This fails the entire upgrade.                                       |
| `31`             | `pre-upgrade` command was executed but failed. But the command is retried until exit code `1` or `30` are returned. |

## Sample

次に、「pre-upgrade」コマンドの構造例を示します。

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
