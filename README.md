# 1，基于Chainlink CCIP的rETH和rMATIC汇率跨链

[StaFi](https://www.stafi.io)是第一个多链流动性质押协议，目前已发行了rETH, rMATIC, rATOM, rBNB, rSOL, rSEI等多个LST。其中rETH和rMATIC均是发行在Ethereum链上，由于业务扩展需要，rETH rate需要跨到Arbitrum上参与Balancer的pool，rMATIC rate需要跨到Polygon上参与QuickSwap的pool。因此我们采用CCIP实现从Layer1到Layer2的汇率跨链。

## 合约简介

代码仓库：[StaFi CCIP Rate Contracts](https://github.com/stafiprotocol/evm-ext-contracts/tree/main/contracts/ccip)

该项目主要包含三个合约：

- **RateSender:** 部署在Ethereum链上，定时检查汇率更新，并发送汇率消息。
- **RateReceiver:** 部署在Arbitrum链上和Polygon链上，分别接收rETH和rMATIC的汇率。
- **CCIPRateProvider:** 部署在Arbitrum链上和Polygon链上，分别提供rETH和rMATIC的汇率给第三方使用。

另外，我们还使用了[Chainlink Automation](https://docs.chain.link/chainlink-automation)服务，用于实现自动触发汇率检查更新的功能。

## 具体流程

- 原链上（Ethereum链）部署RateSender合约（指定router、LINK token等合约地址），获取Sender合约地址，router合约地址请查看[supported-networks](https://docs.chain.link/ccip/supported-networks)
- 目标链上部署RateReceiver合约（指定router、和sender合约地址），获取Receiver合约地址
- 目标链上部署CCIPRateProvider合约（指定Receiver合约地址），获取RateProvider合约地址
- Sender合约中，分别调用addRETHRateInfo和addMATICRateInfo，注册汇率来源和接收目标合约等信息
- 通过[Automation](https://automation.chain.link)服务，注册new Upkeep，即Sender合约
- 转适量的LINK token至Sender合约和Automation服务

## 具体case - rETH(Mainnet)

- RateSender: https://etherscan.io/address/0x655603c5c034f89d8e0c25c7bb22cae091219665
- RateReceiver: https://arbiscan.io/address/0x1a5474e63519bf47860856f03f414445382dc3f1
- CCIPRateProvider: https://arbiscan.io/address/0x4fd35afa32310eaa1354768be6ad2c5c6a62d572
- Automation: https://automation.chain.link/mainnet/72594672919459294368139451197298642592810528545624872573362647938899669012818

# 2，基于Chainlink CCIP的StaFi LSD Stack Modules

[StaFi LSD Stack](https://lsaas-docs.stafi.io)，主要为想要创建LST（流动性质押代币）的开发者打造。使用StaFi LSD Stack，只需根据步骤，输入一些参数然后一键部署即可构建一个新的LST，同时获得LST的可组合性和可编程性。
目前我们正在集成各种Modules到stack中，包括Validator Selection AI Module、Point Module等等，考虑到跨链的需求，现在我们把基于CCIP的token和rate跨链作为Modules集成至我们的Stack中。

## 合约简介

代码仓库：[StaFi CCIP Modules Contracts](https://github.com/stafiprotocol/ccip-contracts)

[Rate跨链](https://github.com/stafiprotocol/ccip-contracts/tree/main/contracts/ccip/RateMsg)主要包含四个合约：

- **RateSender:** 部署在原链上，定时检查汇率更新，并发送汇率消息，支持Proxy代理部署，提升可扩展性。
- **RateReceiver:** 部署在目标链上，接收LST的汇率。
- **CCIPRateProvider:** 部署在目标链上，提供LST的汇率给第三方使用。
- **RegisterUpkeep:** 集成Automation服务，部署即可实现自动触发功能。


[Token跨链](https://github.com/stafiprotocol/ccip-contracts/tree/main/contracts/ccip/TokenTransfer)主要包含一个合约：

- **TokenTransferor:** Proxy模式部署，支持升级和权限控制等。

备注：Token跨链需要申请白名单