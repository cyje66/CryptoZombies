# Lesson 6
## web3.js
以太坊網路是由多個節點(node)組成，每個節點都有區塊鏈的副本(copy)，當你要使用一個智能合約上的function時，你需要對其中一個節點發出請求並告訴他們:  
* The address of the smart contract
* The function you want to call, and
* The variables you want to pass to that function.  
而以太坊網路溝通的語言是JSON-RPC，人閱讀起來會有困難，例如:  
```
{"jsonrpc":"2.0","method":"eth_sendTransaction","params":[{"from":"0xb60e8dd61c5d32be8058bb8eb970870f07233155","to":"0xd46e8dd67c5d32be8058bb8eb970870f07244567","gas":"0x76c0","gasPrice":"0x9184e72a000","value":"0x9184e72a","data":"0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675"}],"id":1}
```
web3.js提供一個在中間溝通的媒介，所以我們只需要跟簡易閱讀的JavaScript介面互動就好，例如:
```
CryptoZombies.methods.createRandomZombie("Vitalik Nakamoto 🤔")
  .send({ from: "0xb60e8dd61c5d32be8058bb8eb970870f07233155", gas: "3000000" })
```
## call v.s send
`call` 是用於`view`和`pure` function上，他只在本地節點執行，不會在區塊鏈上新增一筆交易。例如:
```
myContract.methods.myMethod(123).call()
```
`send`則會在區塊鏈上新增一筆交易，並且改變data值。例如:
```
myContract.methods.myMethod(123).send()
```
## 陣列取值
```
Zombie[] public zombies;
```
在Solidity中，當你宣告一個變數是public的話，它會以該變數的名稱自動生成一個`getter`的方法，例如你想找id=15的zombies你可以打:
```
zombies(15)
```
