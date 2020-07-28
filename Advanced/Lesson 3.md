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
## Calling other contract
前面有提到Caller smart contract，它的功能就是知道現在1ETH值多少USD，而要讓他與oracle互動，我們需要提供以下兩點資訊:
* The address of the oracle smart contract
* The signature of the function you want to call
為了方便日後維護，我們要寫一個function來把oracle smart contract的地址儲存在一個變數裡，然後我們就可以產生oracle smart contract的實例，好讓我們可以使用裡面的function。
## Calling the oracle contract
為了要讓caller contract能夠與oracle互動，我們需要建立`interface`:
```
pragma solidity 0.5.0;
contract EthPriceOracleInterface {
  function getLatestEthPrice() public returns (uint256);
}
```
* import interface
```
import "./EthPriceOracleInterface.sol";
```
* 新增一變數`oracleInstace`，並令他為private型態
```
EthPriceOracleInterface private oracleInstance;
```
* instantiate EthPriceOracleInterface
```
oracleInstance = EthPriceOracleInterface(oracleAddress);
```
最後結果:
```
pragma solidity 0.5.0;
//1. Import from the "./EthPriceOracleInterface.sol" file
import "./EthPriceOracleInterface.sol";
contract CallerContract {
  // 2. Declare `EthPriceOracleInterface`
  EthPriceOracleInterface private oracleInstance;
  address private oracleAddress;
  function setOracleInstanceAddress (address _oracleInstanceAddress) public {
    oracleAddress = _oracleInstanceAddress;
    //3. Instantiate `EthPriceOracleInterface`
    oracleInstance = EthPriceOracleInterface(oracleAddress);
  }
}
```
