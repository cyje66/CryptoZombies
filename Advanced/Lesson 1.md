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
## Ganache
是個可以讓我們在local端測試Ethereum network的工具，初始時，他會創造一`accounts`array，裡面包含10個test accounts並給他們100 Ether。
但用`account[0]`和`account[1]`會增加我們閱讀上的難度，所以我們先給他們名字(alice and bob):
```
let [alice, bob] = accounts
```
## 測試三階段: set up、act、assert
到這邊我們該有的雛形都已經出來了，接下來就是測試...
1. Set up  
在上面的步驟中，我們有做*contract abstraction*，也就是這行:
```
// 合約名稱是 myAwesomeContract
const myAwesomeContract = artifacts.require(“myAwesomeContract”);
```
但要真正與智能合約互動，我們需要建立一個功能和合約實例(contract instance)一樣的JavaScript物件。沿用上面的例子`myAwesomeContract`，我們可以用*contract abstraction*來初始化實例(instance)，例如:
```
const contractInstance = await myAwesomeContract.new();
```
有了instance後，我們就可以呼叫contract內的function了，例如: 
```
const zombieNames = ["Zombie #1", "Zombie #2"];
contractInstance.createRandomZombie(zombieNames[0]);
```
2. Act  
有了instance之後，接下來就要用`createRandomZombie`來為Alice創造一個zombie，但我們要怎麼確定Alice(不是Bob)就是那個zombie的擁有者?
下面這個例子可以讓我們確保msg.sender就是Alice:
```
const result = await contractInstance.createRandomZombie(zombieNames[0], {from: alice});
```
那result裡面究竟放些什麼東西呢?
* Logs and Events  
當我們使用`artifacts.require`指出我們想要測試的contract後，Truffle自動提供由smart contract生成的*logs*，藉由這點，我們可以取出Alice新創造的zombie的名字:
```
result.logs[0].args.name
```
相同的，我們也可以取出`id`或`_dna`。  
除了這些，`result`還可以取出有關交易的有用資料:
* result.tx: the transaction hash
* result.receipt: an object containing the transaction receipt. If `result.receipt.status` is equal to `true` it means that the transaction was successful. Otherwise, it means that the transaction failed.
3. Assert  
在這章中，我們會先使用內建的assertion模組，例如`equal()`和`deepEqual()`。簡單來說，這些function會檢查條件，如果結果不如預期，則會丟出error。
我們可以使用以下的例子來確認test是否通過:
```
assert.equal(result.receipt.status, true);
```
## Hooks
當我們要run一次test時，就要一直重寫這段:
```
const contractInstance = await CryptoZombies.new();
```
如果我們不要一直重複寫`contract.new()`，那可以用`beforeEach()`:
```
beforeEach(async () => {
  // let's put here the code that creates a new contract instance
});
```
