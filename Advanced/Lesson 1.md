# Lesson 1
## Build Artifact
每次編譯智能合約時，Solidity會自動產生一個json檔(又稱為build artifacts)，它包含了那個合約的二進位碼，而且被儲存於`build/contracts`的資料夾中。
然後，當你跑migration時，Truffle會更新關於該網路的資訊。
當我們想測試時，需要先載入我們想要互動的智能合約的build artifacts，例如:
```
// 合約名稱是 myAwesomeContract
const myAwesomeContract = artifacts.require(“myAwesomeContract”);
```
## contract() function
載入好智能合約後，接下來就是測試了，會用到兩個function:  
* contract(): 兩個參數，第一個是string，指的是我們現在要做甚麼測試；第二個是callback，是我們要寫測試的地方  
* it(): 也是兩個，string and callback。
```
 contract("MyAwesomeContract", (accounts) => {
   it("should be able to receive Ethers", () => {
   })
 })
 ```
