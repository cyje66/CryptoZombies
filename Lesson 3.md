# Lesson 3
## Ownable
若有一個function的型態是external的話，由於大家都可以呼叫他，因此產生一個漏洞。創建一個Ownable Contract就可以避免這個問題，其意義是擁有者(你自己)有特殊的權利。
#### 如何使用?
在OpenZeppelin 上已經有寫好的程式碼了，只要import ownable.sol，接著將該contract繼承Ownable就行了。(OpenZeppelin 是一個有許多安全且經過審查的智能合約的library)
## Modifier
modifier看起來就跟function沒兩樣，但它不能像function一樣直接被呼叫；反而，我們需要在一個function後面加上modifier，來改變此function的行為。例如:
```
// A mapping to store a user's age:
mapping (uint => uint) public age;

// Modifier that requires this user to be older than a certain age:
modifier olderThan(uint _age, uint _userId) {
  require(age[_userId] >= _age);
  _;   //讀到這行時，會跳出modifier，並回到function中執行裡面的內容
}

// Must be older than 16 to drive a car (in the US, at least).
// We can call the `olderThan` modifier with arguments like so:
function driveCar(uint _userId) public olderThan(16, _userId) {
  // Some function logic
}
```
## Gas — 以太坊上DApp運行的燃料(手續費)
在Solidity中，使用者(users)要在DApp上執行一個function的時候，需要一個叫做gas的貨幣(以Ether來計價)。  
為什麼要收這個手續費呢?  
原因是怕不良的使用者惡意癱瘓區塊鏈網路(例如無限迴圈)，使其一直處於運行的狀態。
## struct
一般來說，使用較短的整數在Solidity中，並不會幫你節省gas，例如uint8, uint16, uint32等等，因為uint的size不管多少，Solidity都會保留256bits的儲存空間。  
但是用struct結構，Solidity就會把資料型態一樣的變數一起打包，例如:
```
struct NormalStruct {
  uint a;
  uint b;
  uint c;
}

struct MiniMe {
  uint32 a;
  uint32 b;
  uint c;
}

// `mini` will cost less gas than `normal` because of struct packing
NormalStruct normal = NormalStruct(10, 20, 30);
MiniMe mini = MiniMe(10, 20, 30); 
```
## Time units
1 minutes is 60, 1 hours is 3600, 1 days is 86400 
## 
