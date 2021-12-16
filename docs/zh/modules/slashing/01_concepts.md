# 概念

## 状态

在任何给定时间，都有任意数量的验证器在该状态中注册
机器。每个区块，顶部的 `MaxValidators`(由 `x/staking` 定义)验证器
没有入狱的人成为 _bonded_，这意味着他们可以提议并投票
块。 _bonded_ 的验证者是 _at stock_，这意味着部分或全部
如果他们犯了协议错误，他们的利益和他们的委托人的利益就会面临风险。

对于这些验证器中的每一个，我们都保留了一个 `ValidatorSigningInfo` 记录，其中包含
与验证者的活跃度和其他违规相关的信息
属性。

## 墓碑帽

为了减轻最初可能的非恶意类别的影响
协议错误，Cosmos Hub 为每个验证器实现
_tombstone_ cap，它只允许验证器被削减一次以获得双倍
签错。例如，如果您错误地配置了 HSM 并对一堆
旧块，您只会因第一个双重标志而受到惩罚(然后立即被墓葬)。这仍然会非常昂贵并且需要避免，但是墓碑帽
在某种程度上减弱了无意配置错误的经济影响。

活性错误没有上限，因为它们不能相互叠加。一旦违规发生，活性错误就会被“检测到”，并且验证器会立即被关进监狱，因此他们不可能在没有释放的情况下犯下多个活性错误。

## 违规时间表

为了说明 `x/slashing` 模块如何通过以下方式处理提交的证据
Tendermint 共识，请考虑以下示例: 
**Definitions**:

_[_ : timeline start  
_]_ : timeline end  
_C<sub>n</sub>_ : infraction `n` committed  
_D<sub>n</sub>_ : infraction `n` discovered  
_V<sub>b</sub>_ : validator bonded  
_V<sub>u</sub>_ : validator unbonded

### Single Double Sign Infraction

<----------------->
[----------C<sub>1</sub>----D<sub>1</sub>,V<sub>u</sub>-----]

犯了一个单一的违规行为，然后被发现，此时
验证器被解除绑定并因违规而被全额削减。 

### Multiple Double Sign Infractions

<--------------------------->
[----------C<sub>1</sub>--C<sub>2</sub>---C<sub>3</sub>---D<sub>1</sub>,D<sub>2</sub>,D<sub>3</sub>V<sub>u</sub>-----]

犯下多次违规行为，然后被发现，此时
验证者仅因一次违规而被判入狱并受到惩罚。 因为验证器
也是墓碑，他们不能重新加入验证器集。 
