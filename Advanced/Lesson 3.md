# Lesson 3
## 前言
假設我們現在要建立一個DeFi dapp (decentralized finance)，而且我們想要使用者可以取出價值等同於多少USD的ETH，為了達成這個目的，我們這智能合約需要知道1ETH價值多少USD。雖然JavaScript可以很容易地由Binance public API取得這種資訊，但smart contract不能直接存取外界的資料，他需要靠oracle才可以取得資料，如下圖: 
![image](https://cryptozombies.io/course/static/image/lesson-14/EthPriceOracleOverview.png)
## 初始化
* 執行`npm init -y`指令來初始化一個新的project，結果:
```
Wrote to /Users/CZ/Documents/EthPriceOracle/package.json:
{
  "name": "test",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```
* 安裝以下檔案: `truffle`, `openzeppelin-solidity`, `loom-js`, `loom-truffle-provider`, `bn.js`, and `axios`
```
// install multiple packages...
//npm i <package-a> <package-b> <package-c>
npm i truffle openzeppelin-solidity loom-js loom-truffle-provider bn.js axios
```
