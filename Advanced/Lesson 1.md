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
* 若要在同一個contract上多加幾個test，就再加it()就好了。
```
 contract("MyAwesomeContract", (accounts) => {
   it("should be able to receive Ethers", () => {
   // first test
   })
   it("should be able to do sth", () => {
   // second test
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
// make sure alice calls
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
  contractInstance = await CryptoZombies.new();
});
```
## selfdestruct()
雖然`contract.new`使用起來很方便，但如果大家都可以創建無數個contract，區塊鏈就會變得很肥大，所以我們必須要把用完的contractInstance刪掉。
* 首先，我們要在smart contract中加入kill():
```
function kill() public onlyOwner {
   // selfdestruct()可以把某個address從區塊鏈中移除
   selfdestruct(owner());
}
```
* 接著，就像`beforeEach`一樣，我們也需要`afterEach`:
```
afterEach(async () => {
   await contractInstance.kill();
});
```
* 最後，Truffle會確保每一個test結束後，都會執行此function。
## The context function
是個讓我們可以團體測試(group test)的function。由於我們的cryptozombie是*ERC721*協定，所以我們有兩種transfer token的方式:  
(1) 比較直觀的方法
```
function transferFrom(address _from, address _to, uint256 _tokenId) external payable;
```
(2) 先approve，再transfer
```
function approve(address _approved, uint256 _tokenId) external payable;
```
```
function transferFrom(address _from, address _to, uint256 _tokenId) external payable;
```
第二種的流程是: Alice先呼叫`approve`，並帶入Bob的`address`和`zombieId`，合約上就會先紀錄Bob可以拿zombie。之後，Alice或Bob呼叫`transferFrom`，然後合約會檢查`msg.sender`是不是等於
Alice或Bob，如果是的話，zombie就會給Bob。  
在這個情況下，我們就會有兩種transfer zombie的「情況」(scenario)，這時候就需要用到context，方便記錄我們的test情形。
```
context("with the single-step transfer scenario", async () => {
    it("should transfer a zombie", async () => {
      // TODO: Test the single-step transfer scenario.
    })
})

context("with the two-step transfer scenario", async () => {
    it("should approve and then transfer a zombie when the approved address calls transferForm", async () => {
      // TODO: Test the two-step scenario.  The approved address calls transferFrom
    })
    it("should approve and then transfer a zombie when the owner calls transferForm", async () => {
        // TODO: Test the two-step scenario.  The owner calls transferFrom
     })
})
```
在truffle上看起來就會是這樣:
```
Contract: CryptoZombies
    ✓ should be able to create a new zombie (100ms)
    ✓ should not allow two zombies (251ms)
    with the single-step transfer scenario
      ✓ should transfer a zombie
    with the two-step transfer scenario
      ✓ should approve and then transfer a zombie when the owner calls transferForm
      ✓ should approve and then transfer a zombie when the approved address calls transferForm


  5 passing (2s)
```
但很明顯的，我們`it()`裡根本沒放東西，所以他不應該是打勾的狀態。很簡單，只要在`context`前面加上`x`就可以了(it前也可以加)，變成`xcontext`。記得在補上程式碼後，把`x`刪除。
```
Contract: CryptoZombies
    ✓ should be able to create a new zombie (199ms)
    ✓ should not allow two zombies (175ms)
    with the single-step transfer scenario
      - should transfer a zombie
    with the two-step transfer scenario
      - should approve and then transfer a zombie when the owner calls transferForm
      - should approve and then transfer a zombie when the approved address calls transferForm


  2 passing (827ms)
  3 pending
```
## Time traveling
在測試攻擊殭屍的功能時，我們發現了一個問題:
```
it("zombies should be able to attack another zombie", async () => {
        let result;
        result = await contractInstance.createRandomZombie(zombieNames[0], {from: alice});
        const firstZombieId = result.logs[0].args.zombieId.toNumber();
        result = await contractInstance.createRandomZombie(zombieNames[1], {from: bob});
        const secondZombieId = result.logs[0].args.zombieId.toNumber();
        //TODO: increase the time
        await contractInstance.attack(firstZombieId, secondZombieId, {from: alice});
        assert.equal(result.receipt.status, true);
    })
```
結果回傳...
```
Contract: CryptoZombies
    ✓ should be able to create a new zombie (102ms)
    ✓ should not allow two zombies (321ms)
    ✓ should return the correct owner (333ms)
    1) zombies should be able to attack another zombie
    with the single-step transfer scenario
      ✓ should transfer a zombie (307ms)
    with the two-step transfer scenario
      ✓ should approve and then transfer a zombie when the approved address calls transferFrom (357ms)


  5 passing (7s)
  1 failing

  1) Contract: CryptoZombies
       zombies should be able to attack another zombie:
     Error: Returned error: VM Exception while processing transaction: revert
```
為什麼出錯了呢?我們來看一下`createRandomZombie()`:
```
function createRandomZombie(string _name) public {
  require(ownerZombieCount[msg.sender] == 0);
  uint randDna = _generateRandomDna(_name);
  randDna = randDna - randDna % 100;
  _createZombie(_name, randDna);
}
```
再看看`_createZombie()`:
```
function _createZombie(string _name, uint _dna) internal {
  uint id = zombies.push(Zombie(_name, _dna, 1, uint32(now + cooldownTime), 0, 0)) - 1;
  zombieToOwner[id] = msg.sender;
  ownerZombieCount[msg.sender] = ownerZombieCount[msg.sender].add(1);
  emit NewZombie(id, _name, _dna);
}
```
原來是因為我們在設計時，讓使用者要等一天才能再次attack或feed的機制，所以出了error。因此我們使用Ganache提供的兩個function來解決問題:
* evm_increaseTime: increases the time for the next block.
* evm_mine: mines a new block.
解釋一下他的運作原理:
* 每當有一個新的區塊被驗證(mined)，礦工就會在那個block貼上timestamp。假設新增殭屍的transaction在第五個block被驗證(mined)。
* 接著，我們呼叫`evm_increasetime`，由於區塊鏈是無法被修改的，所以contract會檢查時間，但不會進行更改。
* 如果我們執行`evm_mine`，第六個block就會被驗證(還有timestamped)，代表說當我們讓殭屍進行攻擊，smart contract會「認為」已經過了一天了。
## More expressive assertions with Chai
我們原本是用內建的module來進行一些判斷(assert)，但這樣會造成閱讀上的困難，所以我們改用另一個強大的工具----Chai。  
用以下指令就能安裝:`npm -g install chai`，其import語法:`var expect = require('chai').expect;`  
三個Chai的assertion style:
* expect: lets you chain natural language assertions as follows:
```
let lessonTitle = "Testing Smart Contracts with Truffle";
expect(lessonTitle).to.be.a("string");
```
* should: allows for similar assertions as expect interface, but the chain starts with a should property:
```
let lessonTitle = "Testing Smart Contracts with Truffle";
lessonTitle.should.be.a("string");
```
* assert: provides a notation similar to that packaged with node.js and includes several additional tests and it's browser compatible:
```
let lessonTitle = "Testing Smart Contracts with Truffle";
assert.typeOf(lessonTitle, "string");
```
## Loom
在Loom上，使用者可以取得比Ethereum快的速度而且gas-free的transactions。
### Configure Truffle for Testing on Loom
```
loom_testnet: {
      provider: function() {
        const privateKey = 'YOUR_PRIVATE_KEY';
        const chainId = 'extdev-plasma-us1';
        const writeUrl = 'wss://extdev-basechain-us1.dappchains.com/websocket';
        const readUrl = 'wss://extdev-basechain-us1.dappchains.com/queryws';
        return new LoomTruffleProvider(chainId, writeUrl, readUrl, privateKey);
      },
      network_id: 'extdev'
    }
```
### The accounts array
```
const loomTruffleProvider = new LoomTruffleProvider(chainId, writeUrl, readUrl, privateKey);
// Using the mnemonic code to create 10 accounts
loomTruffleProvider.createExtraAccountsFromMnemonic(mnemonic, 10);
return loomTruffleProvider;
```
