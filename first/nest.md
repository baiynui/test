# NEST2.0协议
[TOC]

## 映射合约
0xE87F169797503b5612aE9B29d14199468a37fc66
### 属性
属性 | 类型 | 功能 | 描述
---|---|---|---
ContractAddress | mapping(string => address) | 保存业务合约地址 | 存储字段对应合约地址
owners | mapping (address => bool) | 保存投票合约地址 | 存储地址是否具有修改权限

### 方法
1、初始化方法
```
constructor () public
```
描述：设置部署者为默认修改权限

2.添加或修改协议合约地址
输入参数 | 类型 | 描述
---|---|---
name | string | 协议合约地址对应查询字段
contractAddress | address | 协议合约地址
```
function addContractAddress(string memory name, address contractAddress) public
```
描述：添加或修改协议合约地址

3.增加管理权限地址
输入参数 | 类型 | 描述
---|---|---
superMan | address | 管理权限地址

```
function addSuperMan(address superMan) public
```
描述：增加后，对应地址可以发出操作协议内部所有有修改权限的交易

4.删除管理权限地址
输入参数 | 类型 | 描述
---|---|---
superMan | address | 管理权限地址
```
function deleteSuperMan(address superMan) public
```
描述：删除后，该地址不再具有修改权限

---

5.查询合约地址
输入参数 | 类型 | 描述
---|---|---
name | string | 协议合约地址对应查询字段

```
function checkAddress(string memory name) public view returns (address contractAddress)
```
描述：根据字符串查询对应合约地址

2.查看是否有修改权限
输入参数 | 类型 | 描述
---|---|---
man | address | 查询地址
```
function checkOwners(address man) public view returns (bool)
```
描述：查看输入地址是否具有修改权限

## 投票体系
### 流程
```
sequenceDiagram
    投票发起者->>投票工厂: 创建投票
    投票工厂->>投票合约: 生成投票合约
    loop NEST投票
        用户->>投票工厂: NEST投票
        投票工厂->>投票合约:计票
        投票合约->>NEST锁仓合约:获取投票用户锁仓NEST数量
        NEST锁仓合约-->>投票合约:投票用户锁仓NEST数量
    end
    loop 守护者节点投票
        守护者节点->>投票合约: 授权票数
        守护者节点->>投票合约: 投票，按比例计票
    end
    用户-->>投票合约:取消NEST投票，可选
    守护者节点-->>投票合约:取消守护者节点投票，可选
    投票发起者->>投票合约:投票过期未生效，取回抵押NEST
    用户->>投票合约:投票通过，执行
    投票合约->>销毁地址:销毁抵押NEST
    投票合约->>可执行地址:执行修改内容
```
### 投票工厂合约
#### 属性
属性 | 类型 | 功能 | 描述
---|---|---|---
mappingContract | NEST_2_Mapping | 保存映射合约地址 | 
myVote | mapping(address => address) | 我最近一次投票合约的地址 | 当操作新的投票合约后，会覆盖之前的地址
limitTime | uint256 | 投票时间单位，1小时
circulationProportion | uint256 | 通过投票比例，51%
NNAmount | uint256 | 守护者节点1票 = 100000
ContractAddress | event | 创建投票合约日志，返回创建的投票合约地址

#### 方法
1.初始化方法
输入参数 | 类型 | 描述
---|---|---
map | address | 映射合约地址
```
constructor (address map) public 
```

2.重置映射合约
输入参数 | 类型 | 描述
---|---|---
map | address | 映射合约地址

```
function changeMapping(address map) public onlyOwner
```

3.创建投票合约
输入参数 | 类型 | 描述
---|---|---
contractAddress | address | 可执行合约地址
time | uint256 | 投票持续时间，单位小时

```
function createVote(address contractAddress, uint256 time) public
```

4.NEST投票
输入参数 | 类型 | 描述
---|---|---
contractAddress | address | 要参与的投票合约地址

```
function nestVote(address contractAddress) public
```

---

5.修改投票时间单位(秒)
输入参数 | 类型 | 描述
---|---|---
num | uint256 | 投票时间单位，秒

```
function changeLimitTime(uint256 num) public onlyOwner
```

6.修改通过投票比例
输入参数 | 类型 | 描述
---|---|---
num | uint256 | 通过投票比例，百分制

```
function changeCirculationProportion(uint256 num) public onlyOwner
```

7.修改守护者节点票数
输入参数 | 类型 | 描述
---|---|---
num | uint256 | 守护者节点对应票数

```
function changeNNAmount(uint256 num) public onlyOwner
```

---

8.查看投票时间单位(秒)
返回参数 | 类型 | 描述
---|---|---
--- | uint256 | 投票时间单位，秒
```
function checkLimitTime() public view returns(uint256)
```

9.查看通过投票比例
返回参数 | 类型 | 描述
---|---|---
--- | uint256 | 通过投票比例，百分制
```
function checkCirculationProportion() public view returns(uint256)
```
10.查看NN token对应票数比例
返回参数 | 类型 | 描述
---|---|---
--- | uint256 | 守护者节点对应票数
```
function checkNNAmount() public view returns(uint256)
```

11.查看我的投票
输入参数 | 类型 | 描述
---|---|---
user | address | 用户地址

返回参数 | 类型 | 描述
---|---|---
--- | address | 最近参与的投票合约地址
 
```
function checkMyVote(address user) public view returns (address)
```

12.查看是否有正在参与的投票 
输入参数 | 类型 | 描述
---|---|---
user | address | 用户地址

返回参数 | 类型 | 描述
---|---|---
--- | bool | 是否有正在参与的投票
 
```
function checkVoteNow(address user) public view returns(bool)
```


### 投票合约
#### 属性
输入参数 | 类型 | 描述
---|---|---
miningSave | NEST_2_MiningSave | 矿池合约
mappingContract | NEST_2_Mapping | 映射合约
implementContract | NEST_2_Implement | 可执行合约
nestSave | NESTSave | 锁仓合约
voteFactory | NEST_2_VoteFactory | 投票工厂合约
nestToken | ERC20 | nestToken
NNToken | ERC20 | 守护者节点
implementAddress | address | 执行地址
creator | address | 创建者地址
createTime | uint256 | 创建时间
endTime | uint256 | 结束时间
totalAmount | uint256 | 总投票数
circulation | uint256 | 通过票数
destroyedNest | uint256 | 已销毁NEST数量
effective | bool | 是否生效
personalAmount | mapping(address => uint256) | 个人投票数
personalNNAmount | mapping(address => uint256) | 个人NN投票数

#### 方法
1.初始化方法
输入参数 | 类型 | 描述
---|---|---
contractAddress | address | 执行合约地址
time | uint256 | 投票持续时间（小时）
map | address | 映射合约地址
```
constructor (address contractAddress, uint256 time, address map) public onlyFactory
```
描述：设置可执行合约地址，开始时间，结束时间，投票通过数量。其他相关业务合约实例化

2.NEST投票
```
function nestVote() public onlyFactory
```
描述：仅限工厂合约调用，通过锁仓合约NEST数量计票，如果投票数大于通过投票数，投票合约生效

3.守护者节点投票
输入参数 | 类型 | 描述
---|---|---
amount | uint256 | 投票数量
```
function nestNodeVote(uint256 amount) public
```
描述：将守护者节点NN转入投票合约，按比例计票

4.NEST取消投票

```
function nestVoteCancel() public 
```
描述：在截止时间前并且投票没有生效，可以取消NEST投票

5.守护者节点取消投票

```
function nestNodeVoteCancel() public
```
描述：守护者节点取回NN币，NN所产生的NEST销毁。

6.执行投票合约

```
function startChange() public
```
描述：投票通过，任何人可以触发执行方法，触发后执行修改。

7.取回NEST

```
function turnOutNest() public onlyCreator
```
描述：当合约没有生效并且已经超过结束时间，创建者可以取回合约中的NEST。

---

8.查看投票合约是否结束

```
function checkContractEffective() public view returns(bool)
```
9.查看执行合约地址 

```
function checkImplementAddress() public view returns(address)
```
10.查看合约创建者

```
function checkCreator() public view returns(address)
```
11.查看投票开始时间

```
function checkCreateTime() public view returns(uint256)
```
12.查看投票结束时间

```
function checkEndTime() public view returns(uint256)
```
13.查看当前总投票数

```
function checkTotalAmount() public view returns(uint256)
```
14.查看通过投票数

```
function checkCirculation() public view returns(uint256)
```
15.查看个人投票数

```
function checkPersonalAmount(address user) public view returns(uint256)
```
16.查看个人 NN 投票数

```
function checkPersonalNNAmount(address user) public view returns(uint256)
```
17.查看合约中准备销毁的nest

```
function checkNestBalance() public view returns(uint256)
```
18.查看已经销毁NEST

```
function checkDestroyedNest() public view returns(uint256)
```
19.查看合约是否生效

```
function checkEffective() public view returns(bool)
```

## 报价体系
### 流程
```
sequenceDiagram
    loop 生成个人资产合约
        用户->>报价工厂: 生成个人资产合约
        报价工场->>个人资产合约: 生成
    end
    loop 个人资产合约操作
        用户->>报价工厂: 授权erc20
        用户->>报价工厂: 存入erc20
        报价工厂->>个人资产合约: 授权转账erc20
        用户->>报价工厂: 存入eth
        报价工厂->>个人资产合约: 存入eth
        用户->>个人资产合约: 取回erc20
        个人资产合约->>用户:转账erc20
        用户->>个人资产合约: 取回eth
        个人资产合约->>用户:转账eth
    end
    loop 报价
        用户->>个人资产合约: 报价
        个人资产合约->>报价工厂:报价
        报价工厂->>矿池逻辑:挖矿
        报价工厂->>价格合约:记录价格
        报价工厂->>报价合约:生成报价合约
    end
    loop 吃单
        用户->>个人资产合约: 吃单
        个人资产合约->>报价工厂:报价乘2
        个人资产合约->>报价工厂:吃单
        报价工厂->>个人资产合约:资产结算
        报价工厂->>报价合约:修改可交易金额
        报价工厂->>价格合约:修改价格
    end

```
### 报价数据合约
#### 属性
属性 | 类型 | 功能 | 描述
---|---|---|---
addressMapping | mapping (address => bool) | 保存已部署的业务合约地址 | 地址对应的bool值为true，表示该地址为NEST产出的合约
mappingContract | NEST_2_Mapping | 映射合约对象 | 合约内部逻辑中使用映射合约进行查询操作
#### 方法
1.初始化方法

输入参数 | 类型 | 描述
---|---|---
map | address | 映射合约地址

```
constructor(address map) public
```
部署合约，并根据输入地址生成映射合约

2.修改映射合约地址

输入参数 | 类型 | 描述
---|---|---
map | address | 映射合约地址

```
function changeMapping(address map) public onlyOwner
```
修改映射合约地址，重新生成 mappingContract 对象，合约内部逻辑使用新的映射合约

3.添加地址数据

输入参数 | 类型 | 描述
---|---|---
contractAddress | address | 已部署的借贷合约地址

```
function addContractAddress(address contractAddress) public
```

添加已部署的借贷合约地址，将 addressMapping 中对应地址的 value 设置为 true

---

4.验证合约

输入参数 | 类型 | 描述
---|---|---
contractAddress | address | 需要验证的合约地址

输出参数 | 类型 | 描述
---|---|---
--- | bool | 是、否

```
function checkContract(address contractAddress) public view returns (bool)
```
检查合约地址是否在数据合约中，查询输入地址在 addressMapping 对应的 value，true为NEST产出的合约

### 价格合约
#### 属性
属性 | 类型 | 功能 | 描述
---|---|---|---
mappingContract | NEST_2_Mapping | 映射合约对象 | 合约内部逻辑中使用映射合约进行查询操作
offerFactory | NEST_2_OfferFactory | 报价工厂合约 | 实例化报价工厂
addressAllow | mapping(address => bool) | 允许查看价格地址 | ---
Price | struct | 价格结构体 | 存储价格 | ---
addressPrice | struct | token价格信息数据 | ---
tokenInfo | mapping(address => addressPrice) | token地址对应token价格信息 | ---
#### 方法
1.初始化方法
输入参数 | 类型 | 描述
---|---|---
map | address | 映射合约地址

```
constructor(address map) public
```
部署合约，并根据输入地址生成映射合约

2.修改映射合约地址

输入参数 | 类型 | 描述
---|---|---
map | address | 映射合约地址

```
function changeMapping(address map) public onlyOwner
```
修改映射合约地址，重新生成 mappingContract 对象，合约内部逻辑使用新的映射合约，实例化报价工厂合约

3.增加价格
输入参数 | 类型 | 描述
---|---|---
_ethAmount | uint256 | 报价ETH数量
_tokenAmount | uint256 | 报价ERC20数量
_tokenAddress | address | 报价ERC20地址

```
function addPrice(uint256 _ethAmount, uint256 _tokenAmount, address _tokenAddress) public onlyFactory
```
新增价格数据，并更新当前实时价格

4.修改价格（吃单）
输入参数 | 类型 | 描述
---|---|---
_ethAmount | uint256 | 吃单ETH数量
_tokenAmount | uint256 | 吃单ERC20数量
_tokenAddress | address | 吃单ERC20地址
blockNum | uint256 | 价格所在区块
```
function changePrice(uint256 _ethAmount, uint256 _tokenAmount, address _tokenAddress, uint256 blockNum) public onlyFactory
```

5.修改允许查看地址

输入参数 | 类型 | 描述
---|---|---
add | address | 申请查看地址
allow | bool | 是否允许
```
function changeAddressAllow(address add, bool allow) public onlyOwner
```
---

6.查看允许查看地址
输入参数 | 类型 | 描述
---|---|---
token | address | 申请查看地址

返回参数 | 类型 | 描述
---|---|---
allow | bool | 是否允许
```
function checkAddressAllow(address token) public view returns(bool)
```

7.查找历史区块价格
输入参数 | 类型 | 描述
---|---|---
tokenAddress | address | token地址
blockNum | uint256 | 价格所在区块

返回参数 | 类型 | 描述
---|---|---
ethAmount | uint256 | 报价ETH数量
erc20Amount | uint256 | 报价ERC20数量

```
function checkPriceForBlock(address tokenAddress, uint256 blockNum) public view returns (uint256 ethAmount, uint256 erc20Amount)
```

8.查看实时价格
输入参数 | 类型 | 描述
---|---|---
tokenAddress | address | token地址

返回参数 | 类型 | 描述
---|---|---
ethAmount | uint256 | 报价ETH数量
erc20Amount | uint256 | 报价ERC20数量

```
function checkPriceNow(address tokenAddress) public view returns (uint256 ethAmount, uint256 erc20Amount)
```

9.按区块查找历史加权价格 
输入参数 | 类型 | 描述
---|---|---
tokenAddress | address | token地址
endBlock | uint256 | 结束区块
blockTimes | uint256 | 统计区块数量

返回参数 | 类型 | 描述
---|---|---
ethAmount | uint256 | 报价ETH数量
erc20Amount | uint256 | 报价ERC20数量

```
function checkPriceFrontForBlock(address tokenAddress, uint256 endBlock, uint256 blockTimes) public view returns(uint256 ethAmount, uint256 erc20Amount)
```


### 报价工厂合约
#### 属性
属性 | 类型 | 功能 | 描述
---|---|---|---
tokenAllow | mapping(address => bool) | 可报价挖矿token | token地址=>bool
userOfferContract | mapping(address => address) | 个人资产合约 | 用户地址=>个人资产合约地址
dataContract | NEST_2_OfferData | 报价数据合约 | ---
offerPrice | NEST_2_OfferPrice | 价格合约 | ---
orePoolLogic | NEST_2_OrePoolLogic | 挖矿合约 | ---
accountBook | NEST_2_AccountBook | 延时账本 | ---
miningETH | uint256 | 报价挖矿手续费挖矿比例 | 千分10
tranEth | uint256 | 吃单手续费比例 | 千分2
offerLimit | uint256 | 个人报价上限  | 默认10
blockLimit | uint256 | 区块间隔上限 | 默认25
#### 方法
1.初始化方法
输入参数 | 类型 | 描述
---|---|---
map | address | 映射合约地址
```
constructor (address map) public
```

2.重置映射合约
输入参数 | 类型 | 描述
---|---|---
map | address | 映射合约地址
```
function changeMapping(address map) public onlyOwner
```

3.生成个人资产存储合约

```
function createOfferToken() public
```

4.生成报价
输入参数 | 类型 | 描述
---|---|---
ethMining | uint256 | 挖矿ETH数量
ethAmount | uint256 | 报价eth数量
erc20Amount | uint256 | 报价erc20数量
erc20Address | address | 报价erc20地址

返回参数 | 类型 | 描述
---|---|---
--- | address | 报价合约地址
```
function offer(uint256 ethMining,uint256 ethAmount, uint256 erc20Amount, address erc20Address) public payable returns(address)
```

5.领取 nest（不挖矿，领取延时发放）

```
function getNest() public
```

6.吃单转ETH
输入参数 | 类型 | 描述
---|---|---
contractAddress | address | 报价方资产合约地址
```
function tranShiftEth(address contractAddress) public payable
```

7.转入eth
输入参数 | 类型 | 描述
---|---|---
contractAddress | address | 交易发起者的个人资产合约地址
```
function shiftToETH(address contractAddress) public payable
```

8.转入erc20
输入参数 | 类型 | 描述
---|---|---
tokenAddress | address | token地址
tokenAmount | uint256 | 转入token数量
contractAddress | address | 交易发起者的个人资产合约地址
```
function shiftToERC20(address tokenAddress, uint256 tokenAmount, address contractAddress) public
```
9.吃单eth
输入参数 | 类型 | 描述
---|---|---
contractAddress | address | 被吃报价合约地址
tranEthAmount | uint256 | 被吃eth数量
tranTokenAmount | uint256 | 被吃erc20数量
tokenAddress | address | 被吃erc20地址
```
function ethTran(address contractAddress, uint256 tranEthAmount, uint256 tranTokenAmount, address tokenAddress) public payable
```
10.吃erc20
输入参数 | 类型 | 描述
---|---|---
contractAddress | address | 被吃报价合约地址
tranEthAmount | uint256 | 被吃eth数量
tranTokenAmount | uint256 | 被吃erc20数量
tokenAddress | address | 被吃erc20地址
```
function ercTran(address contractAddress, uint256 tranEthAmount, uint256 tranTokenAmount, address tokenAddress) public
```
11.修改报价挖矿手续费挖矿比例
输入参数 | 类型 | 描述
---|---|---
num | uint256 | 报价挖矿手续费挖矿比例，千分制
```
function changeMiningETH(uint256 num) public onlyOwner
```

12.修改吃单手续费比例
输入参数 | 类型 | 描述
---|---|---
num | uint256 | 吃单手续费比例，千分制
```
function changeTranEth(uint256 num) public onlyOwner
```
13.修改个人报价上限
输入参数 | 类型 | 描述
---|---|---
num | uint256 | 个人报价上限，10
```
function changeOfferLimit(uint256 num) public onlyOwner
```
14.修改区块间隔上限
输入参数 | 类型 | 描述
---|---|---
num | uint256 | 区块间隔上限，25
```
function changeBlockLimit(uint256 num) public onlyOwner 
```
15.修改token 是否允许挖矿
输入参数 | 类型 | 描述
---|---|---
token | address | token地址
allow | bool | 是否允许
```
function changeTokenAllow(address token, bool allow) public onlyOwner
```
---

16.查看个人资产合约地址
输入参数 | 类型 | 描述
---|---|---
user | address | 用户地址

返回参数 | 类型 | 描述
---|---|---
--- | address | 个人资产合约地址
```
function checkUserOfferContract(address user) public view returns(address)
```
17.查看报价单上限
返回参数 | 类型 | 描述
---|---|---
--- | uint256 | 报价单上限
```
function checkOfferLimit() public view returns(uint256)
```
18.查看区块间隔上限
返回参数 | 类型 | 描述
---|---|---
--- | uint256 | 区块间隔上限
```
function checkBlockLimit() public view returns(uint256)
```
19.查看报价手续费
返回参数 | 类型 | 描述
---|---|---
--- | uint256 | 报价手续费比例
```
function checkMiningETH() public view returns (uint256)
```
20.查看交易手续费
返回参数 | 类型 | 描述
---|---|---
--- | uint256 | 交易手续费比例
```
function checkTranEth() public view returns (uint256)
```
21.查看 token 是否允许挖矿 
输入参数 | 类型 | 描述
---|---|---
token | address | token地址

返回参数 | 类型 | 描述
---|---|---
--- | bool | 是否允许挖矿 
```
function checkTokenAllow(address token) public view returns(bool)
```

### 个人资产存储合约
#### 属性
属性 | 类型 | 功能 | 描述
---|---|---|---
owner | address | 拥有者 | 
ethAmount | uint256 | token数量 | 用户地址
mappingContract | NEST_2_Mapping | 映射合约 | ---
offerFactory | NEST_2_OfferFactory | 报价工厂 | ---
dataContract | NEST_2_OfferData | 报价数据合约 | ---
offerList | mapping(uint256 => offerClass) | 报价列表 | ---
latestOfferNum | uint256 | 最近报价编号 | ---
offerClass | struct | uint256 blockNum 报价区块号；address contractAddress 报价合约地址； | 价格信息

#### 方法
1.初始化方法
输入参数 | 类型 | 描述
---|---|---
map | address | 映射合约地址

```
constructor (address map) public
```
2.转入资产ETH

```
function shiftToETH() public payable onlyFactory
```

3.报价-验证可报价资产
输入参数 | 类型 | 描述
---|---|---
ethAmount | uint256 | 报价eth数量
erc20Amount | uint256 | 报价erc20数量
erc20Address | address | 报价erc20地址
mining | bool | 是否挖矿
```
function offer(uint256 ethAmount, uint256 erc20Amount, address erc20Address, bool mining) public
```

4.吃单eth

输入参数 | 类型 | 描述
---|---|---
ethAmount | uint256 | 报价eth数量
erc20Amount | uint256 | 报价erc20数量
contractAddress | address | 吃单报价合约地址
tranEthAmount | uint256 | 交易的eth规模
```
function tranETH(uint256 ethAmount, uint256 erc20Amount, address contractAddress, uint256 tranEthAmount) public
```
5.吃单erc20

输入参数 | 类型 | 描述
---|---|---
ethAmount | uint256 | 报价eth数量
erc20Amount | uint256 | 报价erc20数量
contractAddress | address | 吃单报价合约地址
tranErc20Amount | uint256 | 交易的erc规模
```
function tranERC(uint256 ethAmount, uint256 erc20Amount, address contractAddress, uint256 tranErc20Amount) public
```
6.被吃eth，erc20转账
输入参数 | 类型 | 描述
---|---|---
tokenAmount | uint256 | 转账erc20数量
tokenAddress | address | 转账erc20地址
target | address | 转账目标地址
```
function settlementEth(uint256 tokenAmount, address tokenAddress, address target) public onlyFactory
```
7.被吃erc20，eth转账
输入参数 | 类型 | 描述
---|---|---
ethAmount | uint256 | 转账eth数量
target | address | 转账eth目标地址
```
function settlementErc(uint256 ethAmount, address target) public onlyFactory
```
8.取出eth
输入参数 | 类型 | 描述
---|---|---
amount | uint256 | 取出eth数量
```
function turnOutETH(uint256 amount) public
```
9.取出erc20
输入参数 | 类型 | 描述
---|---|---
amount | uint256 | 取出erc20数量
tokenAddress | address | 取出erc20地址
```
function turnOutERC20(uint256 amount, address tokenAddress) public
```
---
10.查看不可用资产
输入参数 | 类型 | 描述
---|---|---
token | address | 查询erc20地址

返回参数 | 类型 | 描述
---|---|---
ethAmount | uint256 | eth不可用数量
erc20Amount | uint256 | erc20不可用数量
```
function checkFreeToken(address token) public view returns(uint256 ethAmount, uint256 erc20Amount)
```
11.查看可报价次数
返回参数 | 类型 | 描述
---|---|---
--- | uint256 | 可报价次数
```
function checkOfferTimes() public view returns(uint256)
```
12.查看拥有者
返回参数 | 类型 | 描述
---|---|---
--- | address | 合约拥有者地址
```
function checkOwner() public view returns(address)
```



### 报价合约
#### 属性
属性 | 类型 | 功能 | 描述
---|---|---|---
owner | address | 拥有者 | 个人资产合约地址
ethAmount | uint256 | token数量 | ---
tokenAmount | address | token地址 | ---
dealEthAmount | uint256 | 成交eth数量 | ---
dealTokenAmount | uint256 | 成交token数量 | ---
blockNum | uint256 | 报价区块 | ---
mappingContract | NEST_2_Mapping | 映射合约 | ---
offerFactory | NEST_2_OfferFactory | 报价工厂 | ---
#### 方法
1.初始化方法
输入参数 | 类型 | 描述
---|---|---
_ethAmount | uint256 | 报价ETH数量
_tokenAmount | uint256 | 报价erc20数量
_tokenAddress | address | 报价erc20地址
ownerAddress | address | 个人资产合约地址
map | address | 映射合约地址

```
constructor (uint256 _ethAmount, uint256 _tokenAmount, address _tokenAddress, address ownerAddress,address map) public
```

2.吃单
输入参数 | 类型 | 描述
---|---|---
_ethAmount | uint256 | 吃单ETH数量
_tokenAmount | uint256 | 吃单erc20数量
_tokenAddress | address | 吃单erc20地址
```
function changeOffer(uint256 _ethAmount, uint256 _tokenAmount, address _tokenAddress) public onlyFactory
```

3.查看合约状态
返回参数 | 类型 | 描述
---|---|---
--- | uint256 | 0：可交易，未生效，1：不可交易，已生效
```
function checkContractState() public view returns (uint256)
```
4.查看剩余可成交数量
返回参数 | 类型 | 描述
---|---|---
leftEth | uint256 | eth数量
leftErc20 | uint256 | erc20数量
erc20Address | address | erc20地址
```
function checkDealAmount() public view returns(uint256 leftEth, uint256 leftErc20, address erc20Address)
```
5.查看价格
返回参数 | 类型 | 描述
---|---|---
_ethAmount | uint256 | eth数量
_tokenAmount | uint256 | erc20数量
_tokenAddress | address | erc20地址
```
function checkPrice() public view returns(uint256 _ethAmount, uint256 _tokenAmount, address _tokenAddress)
```

6.查看发布报价的资产合约地址
返回参数 | 类型 | 描述
---|---|---
--- | address | 个人资产合约地址
```
function checkOwner() public view returns(address)
```

7.查看价格区块
返回参数 | 类型 | 描述
---|---|---
--- | uint256 | 区块编号
```
function checkBlockNum() public view returns (uint256)
```



## 挖矿体系
### 流程
### 挖矿逻辑合约
### 矿池合约
### 延时账本合约
#### 属性
属性 | 类型 | 功能 | 描述
---|---|---|---
nestToken | ERC20 | NEST Token合约 | NEST Token合约
mappingContract | NEST_2_Mapping | 映射合约 | 映射合约对象
NNcontract | NEST_NodeAssignment | 守护者节点合约 | 守护者节点合约
offerFactory | NEST_2_OfferFactory | 报价工厂合约 | 报价工厂合约
coderAmount | uint256 | 开发者分配比例 | 百分制

#### 方法

## 分红体系
### 流程
```
sequenceDiagram
    矿池逻辑->>分红存储合约:手续费ETH
    其他业务->>分红存储合约:手续费ETH
    loop 存入NEST
        用户->>NEST锁仓合约:授权
        用户->>分红逻辑合约:存入NEST
        分红逻辑合约->>NEST锁仓合约:授权转账，账本记账
    end
    loop 取出NEST
        用户->>分红逻辑合约:取出NEST
        分红逻辑合约->>NEST锁仓合约:向目标转账
        NEST锁仓合约->>用户:转账NEST
    end
    loop 取出NEST
        用户->>NEST锁仓合约:取出NEST
        NEST锁仓合约->>用户:转账NEST
    end
    loop 本期第一次领取，平准
        用户->>分红逻辑合约:领取
        分红逻辑合约->>分红存储合约:扣除10%,仅限本期第一次领取
        分红存储合约->>平准合约:平准资金转账
        分红逻辑合约->>平准合约:补齐分红，仅限本期分红低于本期预期分红
        平准合约->>分红存储合约:转账平准资金
    end
    loop 领取
        用户->>分红逻辑合约:领取
        分红逻辑合约->>分红存储合约:按锁仓比例计算分红数量
        分红存储合约->>用户:转账分红
    end
    
```
### 分红逻辑合约
#### 属性
属性 | 类型 | 功能 | 描述
---|---|---|---
mappingContract | IBMapping | 映射合约 | 映射合约对象
nestContract | IBNEST | NEST Token合约 | NEST Token合约
baseMapping | NESTSave | NEST 存储合约 | NEST 存储合约
abonusContract | Abonus | 分红池合约 | 分红池合约
nestLeveling | NESTLeveling | 平准合约 | 平准合约
voteFactory | NEST_2_VoteFactory | 投票工厂合约 | 投票工厂合约
timeLimit | uint256 | 分红周期 | 两次分红之间的间隔时间
nextTime | uint256 | 下次分红时间 | 时间戳
getAbonusTimeLimit | uint256 | 领取分红时间 | 开始领取分红至结束领取分红时间间隔
ethNum | uint256 | 分红ETH数量 | 分红池快照
nestAllValue | uint256 | nest 总流通量 | 总流通量快照
times | uint256 | 每次分红账本 | 分红次数账本
expectedIncrement | uint256 | 预期分红增量比例 | 百分制
expectedMinimum | uint256 | 预期最低分红 | 初始值100ether
levelingProportion | uint256 | 扣除分红比例 | 百分制
getMapping | mapping(uint256 => mapping(address => bool)) | 领取分红记录账本 | 地址对应领取分红权限
#### 方法
1.初始化方法

输入参数 | 类型 | 描述
---|---|---
map | address | 映射合约地址

```
constructor (address map) public
```

2.修改映射合约

输入参数 | 类型 | 描述
---|---|---
map | address | 映射合约地址

```
function changeMapping(address map) public onlyOwner
```
3.存入

输入参数 | 类型 | 描述
---|---|---
amount | uint256 | 存入数量

```
function depositIn(uint256 amount) public
```
需要提前对 nest 存储合约进行授权

4.取出

输入参数 | 类型 | 描述
---|---|---
amount | uint256 | 取出数量

```
function takeOut(uint256 amount) public
```

5.领取

```
function getETH() public
```
领取 ETH 分红

6.更新分红周期
输入参数 | 类型 | 描述
---|---|---
hour | uint256 | 单位小时

```
function changeTimeLimit(uint256 hour) public onlyOwner
```
7.更新领取时间
输入参数 | 类型 | 描述
---|---|---
hour | uint256 | 单位小时

```
function changeGetAbonusTimeLimit(uint256 hour) public onlyOwner
```
8.更新预期分红增量比例
输入参数 | 类型 | 描述
---|---|---
num | uint256 | wei

```
function changeExpectedIncrement(uint256 num) public onlyOwner
```
9.更新预期最低分红
输入参数 | 类型 | 描述
---|---|---
num | uint256 | wei

```
function changeExpectedMinimum(uint256 num) public onlyOwner
```
10.更新扣除分红比例
输入参数 | 类型 | 描述
---|---|---
num | uint256 | 百分制

```
function changeLevelingProportion(uint256 num) public onlyOwner
```

---

11.获取nest流通量额度

返回参数 | 类型 | 描述
---|---|---
--- | uint256 | nest 流通量

```
function allValue() public view returns (uint256)
```

12.下次分红时间

返回参数 | 类型 | 描述
---|---|---
--- | uint256 | 下次分红时间

```
function getNextTime() public view returns (uint256)
```

13.获取分红信息

返回参数 | 类型 | 描述
---|---|---
_nextTime | uint256 | 下次分红时间
_getAbonusTime | uint256 | 本次分红截止时间
_ethNum | uint256 | 分红ETH总数
_nestValue | uint256 | nest 总流通量
_myJoinNest | uint256 | 我存入的nest数量
_getEth | uint256 | 预期收益
_allowNum | uint256 | nest授权金额
_leftNum | uint256 | nest余额
allowAbonus | bool | 目前是否可以领取

```
function getInfo() public view returns (uint256 _nextTime, uint256 _getAbonusTime, uint256 _ethNum, uint256 _nestValue, uint256 _myJoinNest, uint256 _getEth, uint256 _allowNum, uint256 _leftNum, bool allowAbonus)
```
14.查看分红周期

返回参数 | 类型 | 描述
---|---|---
--- | uint256 | 下次分红时间

```
function getNextTime() public view returns (uint256)
```
15.查看领取时间

返回参数 | 类型 | 描述
---|---|---
--- | uint256 | 领取时间

```
function checkGetAbonusTimeLimit() public view returns(uint256)
```


### 分红存储合约
#### 属性
属性 | 类型 | 功能 | 描述
---|---|---|---
mappingContract | IBMapping | 映射合约 | 映射合约对象

#### 方法
1.初始化方法

输入参数 | 类型 | 描述
---|---|---
map | address | 映射合约地址

```
constructor (address map) public
```
初始化分红池合约，实例化映射合约

2.修改映射合约

输入参数 | 类型 | 描述
---|---|---
map | address | 映射合约地址

```
function changeMapping(address map) public onlyOwner
```
重新初始化，初始化映射合约

3.领取分红

输入参数 | 类型 | 描述
---|---|---
num | uint256 | 转账数量
target | address | 转账目标地址

```
function getETH(uint256 num, address target) public onlyContract
```
转移对应数量的ETH到发起人地址

4.查询分红池ETH余额

返回参数 | 类型 | 描述
---|---|---
--- | uint256 | 分红池ETH余额

```
function getETHNum() public view returns (uint256)
```

### NEST锁仓合约
#### 属性
属性 | 类型 | 功能 | 描述
---|---|---|---
mappingContract | IBMapping | 映射合约 | 映射合约对象
nestContract | IBNEST | NEST Token合约
baseMapping | mapping (address => uint256) | 地址对应存储NEST账本
#### 方法
1.初始化方法

输入参数 | 类型 | 描述
---|---|---
map | address | 映射合约地址

```
constructor (address map) public
```
初始化分红池合约，实例化映射合约，实例化NEST代币合约

2.修改映射合约

输入参数 | 类型 | 描述
---|---|---
map | address | 映射合约地址

```
function changeMapping(address map) public onlyOwner
```

重新初始化，初始化映射合约，实例化NEST代币合约

3.取出

输入参数 | 类型 | 描述
---|---|---
num | uint256 | 转账数量

```
function takeOut(uint256 num) public onlyContract
```
检测 账本中对应地址的NEST 数量大于 传入的数量，转移对应数量的NEST到发起人地址

4.存入

输入参数 | 类型 | 描述
---|---|---
num | uint256 | 转账数量

```
function depositIn(uint256 num) public onlyContract
```
检测 余额与授权金额大于传入的数量，转移对应数量的NEST到nest 存储合约 并记录数量

5.查询nest额度

输入参数 | 类型 | 描述
---|---|---
sender | address | 被查询地址

返回参数 | 类型 | 描述
---|---|---
--- | uint256 | nest 存储中 nest 额度

```
function checkAmount(address sender) public view returns(uint256)
```

6.取出nest

输入参数 | 类型 | 描述
---|---|---
num | uint256 | 转账数量

```
function takeOutPrivate(uint256 num) public
```
备用取出接口，可直接调用，检测 账本中对应地址的NEST 数量大于 传入的数量，转移对应数量的NEST到发起人地址

### 平准合约
#### 属性
属性 | 类型 | 功能 | 描述
---|---|---|---
mappingContract | IBMapping | 映射合约 | 映射合约对象

#### 方法
1.初始化方法

输入参数 | 类型 | 描述
---|---|---
map | address | 映射合约地址

```
constructor (address map) public
```
2.修改映射合约

输入参数 | 类型 | 描述
---|---|---
map | address | 映射合约地址

```
function changeMapping(address map) public onlyOwner
```
3.平准转账

输入参数 | 类型 | 描述
---|---|---
amount | uint256 | 转账数量
target | address | 转账目标地址

```
function tranEth(uint256 amount, address target) public
```

## 守护者节点
### 流程
```
sequenceDiagram
    延时账本->>分配合约:授权15%NEST
    延时账本->>分配合约:触发授权转账
    分配合约->>NEST存储合约:转账NEST
    分配合约->>分配数据合约:记录数据
    loop 转账
        用户->>NNToken:转账
        NNToken->>分配合约:强制结算，领取NEST
        分配合约->>NEST存储合约:转账NEST
        NEST存储合约->>用户:转账NEST
    end
    loop 领取
        用户->>分配合约:领取NEST
        分配合约->>NEST存储合约:转账NEST
        NEST存储合约->>用户:转账NEST
    end
    
```
### NEST存储合约
#### 属性
属性 | 类型 | 功能 | 描述
---|---|---|---
mappingContract | IBMapping | 映射合约对象 | 合约内部逻辑中使用映射合约
nestContract | IBNEST | NEST Token 合约对象 | 合约内部逻辑使用 NEST Token 合约
#### 方法
1.初始化方法

输入参数 | 类型 | 描述
---|---|---
map | address | 映射合约地址

```
constructor(address map) public
```
部署合约，根据输入地址生成映射合约，查询映射合约返回NEST Token 并生成合约对象

2.重置映射合约

输入参数 | 类型 | 描述
---|---|---
map | address | 映射合约地址

```
function changeMapping(address map) public onlyOwner
```
重置映射合约，根据输入地址生成映射合约，查询映射合约返回NEST Token 并生成合约对象

3.转出NEST

输入参数 | 类型 | 描述
---|---|---
amount | uint256 | 转出NEST数量
to | address | 转出地址

返回参数 | 类型 | 描述
---|---|---
--- | uint256 | 转出NEST数量

```
function turnOut(uint256 amount, address to) public onlyMiningCalculation returns(uint256)
```

1. 该方法仅限节点分配合约调用
2. 查询合约内 NEST 余额
3. 余额 大于或等于 amount 直接转账
4. 否则 返回 0

### 分配数据合约
#### 属性
属性 | 类型 | 功能 | 描述
---|---|---|---
mappingContract | IBMapping | 映射合约对象 | 合约内部逻辑中使用映射合约
nodeAllAmount | uint256 | 累加NEST数量 | 用于计算每个节点可领取NEST数量
nodeLatestAmount | mapping(address => uint256) | 记录每个节点上次领取时累加NEST的总数 | 用于计算每个节点可领取NEST数量

#### 方法
1.初始化方法

输入参数 | 类型 | 描述
---|---|---
map | address | 映射合约地址

```
constructor(address map) public
```
部署合约，根据输入地址生成映射合约

2.重置映射合约

输入参数 | 类型 | 描述
---|---|---
map | address | 映射合约地址

```
function changeMapping(address map) public onlyOwner
```
重置映射合约，根据输入地址生成映射合约

3.增加NEST，记录数量

输入参数 | 类型 | 描述
---|---|---
amount | uint256 | 增加 NEST 数量

```
function addNest(uint256 amount) public onlyNodeAssignment
```
nodeAllAmount 累加 amount

4.记录节点上次领取数量

输入参数 | 类型 | 描述
---|---|---
add | address | 领取地址
amount | uint256 | 领取时 nodeAllAmount 值

```
function addNodeLatestAmount(address add ,uint256 amount) public onlyNodeAssignment
```
将 nodeLatestAmount 中 add 对应的值 覆盖为 amount

---

5.查看累计总数

返回参数 | 类型 | 描述
---|---|---
--- | uint256 | 返回累计分配NEST总数

```
function checkNodeAllAmount() public view returns (uint256)
```

6.查看上次领取时累计总数

输入参数 | 类型 | 描述
---|---|---
add | address | 查询地址

返回参数 | 类型 | 描述
---|---|---
--- | uint256 | 返回累计分配NEST总数

```
function checkNodeLatestAmount(address add) public view returns (uint256)
```

### 分配合约
#### 属性
属性 | 类型 | 功能 | 描述
---|---|---|---
mappingContract | IBMapping | 映射合约对象 | 合约内部逻辑中使用映射合约
nestContract | IBNEST | NEST Token 合约对象 | 合约内部逻辑使用 NEST Token 合约
supermanContract | SuperMan | NestNode Token 合约对象 | 合约内部逻辑使用 NEST Token 合约
nodeSave | NEST_NodeSave | 节点分配NEST 存储合约对象 | 合约内部使用 节点分配NEST存储合约
nodeAssignmentData | NEST_NodeAssignmentData | 节点分配领取账本合约 | 合约内部逻辑使用节点分配领取账本合约

#### 方法
1.初始化方法

输入参数 | 类型 | 描述
---|---|---
map | address | 映射合约地址

```
constructor(address map) public
```
部署合约，根据输入地址生成映射合约，查询映射合约返回NEST Token 合约地址、NestNode合约地址、节点分配NEST 存储合约地址、节点分配领取账本合约地址，并分别生成合约对象

2.重置映射合约

输入参数 | 类型 | 描述
---|---|---
map | address | 映射合约地址

```
function changeMapping(address map) public onlyOwner
```

修改映射合约，根据输入地址生成映射合约，查询映射合约返回NEST Token 合约地址、NestNode合约地址、节点分配NEST 存储合约地址、节点分配领取账本合约地址，并分别生成合约对象

3.存入NEST 

输入参数 | 类型 | 描述
---|---|---
amount | uint256 | 存入NEST数量

```
NEST:function bookKeeping(uint256 amount) public
```
向超级节点体系存入NEST

1. amount必须大于0
2. 消息发起地址 nest 余额 大于或等于 amount
3. 消息发起地址对本合约的 nest 授权 大于或等于 amount
4. 从 消息发起地址 转移 amount数量的NEST 至 节点分配NEST 存储合约
5. 节点分配领取账本合约记录分配nest总量

4.领取NEST

```
function nodeGet() public
```
超级节点凭借持有 NestNode比例领取 NEST

1. 禁止合约调用
2. 消息（交易）发起者 NestNode 持有数量 大于0
3. 按持有 NestNode 比例计算可领取NEST数量
4. 通知 节点分配 NEST 存储合约 向消息（交易）发起地址转移 NEST

5.转让结算

输入参数 | 类型 | 描述
---|---|---
fromAdd | address | 转出方地址
toAdd | address | 转入方地址

```
function nodeCount(address fromAdd, address toAdd) public
```
NestNode 发生转移时，会强制按双方此时持有NestNode比例领取NEST，完成后再转移 NestNode。

1. 消息发起地址必须为 NestNode地址
2. 转出方地址持有 NestNode 数量必须大于0
3. 分别计算转出地址及转入地址可领取NEST数量
4. 通知 节点分配 NEST 存储合约 向转出地址及转入地址转移NEST

---

6.查询超级节点可领取金额

返回参数 | 类型 | 描述
---|---|---
--- | uint256 | 可领取NEST数量

```
function checkNodeNum() public view returns (uint256)
```
## 映射合约查询字段
合约 | 合约名 | 查询字段 | 地址
---|---|---|---
NEST Token | IBNEST | nest | 0x9519aa54c0dB2787810513015418eE9f2636C47B
NESTNODE Token | SuperMan | nestNode | 0xbaf05333c24C58904C0225e48929A7d837BD114e
挖矿-延时账本 | NEST_2_AccountBook | accountBook | 0xAF36B23EAFDC9CCaEcDF7A4449422BBacC983806
挖矿-矿池 | NEST_2_MiningSave | miningSave | 0xfaAA9E3c48fE5a963bBC4b15ae52531755e834A2
挖矿-逻辑 | NEST_2_OrePoolLogic | miningCalculation | 0x24C211df07Edc30887985b54599dcF8017bB8A85
投票-工厂 | NEST_2_VoteContract | voteFactory | 0x4c124213453844d575333295D513eA29c526E348
分红-分红池 | Abonus | abonus | 0x688f016CeDD62AD1d8dFA4aBcf3762ab29294489
分红-NEST锁仓 | NESTSave | nestSave | 0x0D485B3f15C2Ff5751c42c9C1bEd8a3B0D7c0656
分红-逻辑 | NESTAbonus | nestAbonus | 0x40016dD9B7A0f1f7305F6818fC57658E7abb4B1d
守护节点-分配 | NEST_NodeAssignment | nodeAssignment | 0xDd479A1147e3a4595d6F1042babaCe0F64344F37
守护节点-存储 | NEST_NodeSave | nestNodeSave | 0x79dB8a5D74E558C7624f4B7203EE90864F9E9E74
守护节点-账本存储 | NEST_NodeAssignmentData | nodeAssignmentData | 0xf20aC04eC5493aE5175f53b5E03eB23578f31E02
报价工厂 | NEST_2_OfferFactory | offerFactory | 0x52B8D5dEDffB07487a604CaD26204fF6517569cc
报价数据 | NEST_2_OfferData | offerData | 0x9D38cdBB7fFa033b4e273dA1D5AC872B6034D8FC
价格合约 | NEST_2_OfferPrice | offerPrice | 0xbf0A7F5c3FB04F3f594D65D5B4406E0fE76bD43F

## 合约架构图
```
graph TB
    用户-. 领取分红 .->分红计算
    用户-. 存入NEST .->分红计算
    用户-. 取出NEST .->分红计算
    用户-. 报价 .-> 资产合约
    用户-. 吃单 .->资产合约
    用户-. 领取剩余NEST .->报价工厂
    用户-. 守护者节点领取NEST .->节点结算逻辑
    用户-. 发起投票 .->投票工厂
    用户-. NEST投票 .->投票工厂
    用户-. NN投票 .->投票合约
    用户-. 取消投票 .->投票合约
    用户-. 执行投票 .->投票合约
    矿池逻辑-. 转账手续费 .->分红池
    延时账本-. 节点转账15% .-> 节点存储
    报价工厂-. 吃单手续费 .->分红池
    报价工厂-. 报价挖矿 .-> 矿池逻辑
    subgraph 报价体系
    资产合约-. 报价 .-> 报价工厂
    资产合约-. 吃单 .-> 报价工厂
    报价工厂-. 记录报价合约地址 .-> 报价数据合约
    报价工厂-. 生成报价 .-> 报价合约
    报价工厂-. 生成价格 .-> 价格合约
    报价工厂-. 分配 .-> 延时账本
    end
    subgraph 矿池体系
    矿池逻辑-. 通知转账 .->矿池
    矿池-. 转账 .-> 延时账本
    延时账本-. 开发者转账5% .-> 开发者
    end
    subgraph 守护者节点
    节点结算逻辑-->节点存储
    节点结算逻辑-->节点数据记录
    end
    subgraph 用户
    用户
    end
    subgraph 分红
    分红计算 -. 领取分红 .-> 分红池
    分红计算 -. NEST存取 .-> NEST存储
    平准合约 -. 转入 .-> 分红池
    分红池 -. 扣除 .-> 平准合约
    end
    subgraph 投票体系
    投票工厂 -. 生成投票 .-> 投票合约
    投票合约 -. 执行 .-> 执行合约
    end
```