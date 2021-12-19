# 开始块

重新计算铸造参数和通货膨胀
在每个区块的开始支付。

## NextInflationRate

每个区块都会重新计算目标年通胀率。
通货膨胀也受利率变化(正或负)的影响
取决于与所需比率 (67%) 的距离。 最大变化率
可能定义为每年 13%，但年度通货膨胀率有上限
在 7% 到 20% 之间。 

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

根据当前总供应量和通货膨胀计算年度准备金
速度。 该参数每块计算一次。 

```
NextAnnualProvisions(params Params, totalSupply sdk.Dec) (provisions sdk.Dec) {
	return Inflation * totalSupply
```

## BlockProvision

根据当前年度准备金计算为每个区块生成的准备金。 然后这些规定由`mint` 模块的`ModuleMinterAccount` 铸造，然后转移到`auth` 的`FeeCollector` `ModuleAccount`。 

```
BlockProvision(params Params) sdk.Coin {
	provisionAmt = AnnualProvisions/ params.BlocksPerYear
	return sdk.NewCoin(params.MintDenom, provisionAmt.Truncate())
```
