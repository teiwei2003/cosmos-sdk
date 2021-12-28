# 開始-ブロック

鋳造パラメータが再計算され、インフレが発生します
各ブロックの開始時に支払われます。

## NextInflationRate

目標年間インフレ率は、ブロックごとに再計算されます。
インフレも金利変動の影響を受けます（プラスまたはマイナス）
希望の比率（67％）からの距離に応じて。 可能な最大レート変更は年間13％と定義されていますが、年間インフレ率は7％から20％に制限されています。 

```
NextInflationRate(params Params, bondedRatio sdk.Dec) (inflation sdk.Dec) {
	inflationRateChangePerYear = (1 - bondedRatio/params.GoalBonded) * params.InflationRateChange
	inflationRateChange = inflationRateChangePerYear/blocksPerYr

	// increase the new annual inflation for this next cycle
	inflation += inflationRateChange
	if inflation > params.InflationMax {
		inflation = params.InflationMax
	}
	if inflation < params.InflationMin {
		inflation = params.InflationMin
	}

	return inflation
}
```

## NextAnnualProvisions

現在の総供給とインフレ率に基づいて年間引当金を計算します。 このパラメータは、ブロックごとに1回計算されます。

```
NextAnnualProvisions(params Params, totalSupply sdk.Dec) (provisions sdk.Dec) {
	return Inflation * totalSupply
```

## BlockProvision

現在の年間引当金に基づいて、各ブロックに対して生成された引当金を計算します。 次に、プロビジョニングは `mint`モジュールの` ModuleMinterAccount`によって作成され、 `auth`の`FeeCollector``ModuleAccount`に転送されます。

```
BlockProvision(params Params) sdk.Coin {
	provisionAmt = AnnualProvisions/ params.BlocksPerYear
	return sdk.NewCoin(params.MintDenom, provisionAmt.Truncate())
```
