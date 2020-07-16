# Lesson 2
## Address
每一個帳號都有一個address，可以把他想成是一組匯款的帳號，透過這個address，你可以匯款給那個address的擁有者。
## Mapping(類似dictionary)
  以key-value為基準的儲存或查找data的方法，處理方式就和array一樣，例如:
  ```
  //儲存一個uint，其代表一個address的帳戶餘額
  mapping (address => uint) public accountBalance;
  ```
  此時輸入的key 是 address , 回傳的value 是 uint，另一個例子:
  ```
  //根據一個人的userId來查詢/儲存帳戶擁有者的名稱
  mapping (uint => string) userIdToName;
  ```
  此時輸入的key 是 uint , 回傳的value 是 string
## msg.sender
回傳正在呼叫此function的那個人或智能合約之**地址(address)**，例如:
```
mapping (address => uint) favoriteNumber;

function setMyNumber(uint _myNumber) public {
  //新增一筆 _myNumber的值在msg.sender底下
  favoriteNumber[msg.sender] = _myNumber;
}
function whatIsMyNumber() public view returns (uint) {
  //拿取儲存在msg.sender底下的值
  return favoriteNumber[msg.sender];
}

```
## Require
如果條件不符合就丟出error，並終止執行，例如
```
function sayHiToVitalik(string memory _name) public returns (string memory) {
  // Compares if _name equals "Vitalik". Throws an error and exits if not true.
  // Solidity中沒有原生的字串比較function，所以這邊採取比較keccak256的hash值來看這兩個字串是否相等
  require(keccak256(abi.encodePacked(_name)) == keccak256(abi.encodePacked("Vitalik")));
  // If it's true, proceed with the function:
  return "Hi!";
}
```
## Storage vs Memory(Data 儲存的位置)
1. Storage
Storage 是永久儲存在整個blockchain中的變數
2. Memory
Memory 裡的變數則是暫時的，其會因為contract呼叫不同的function而被消除
## internal & external(Solidity特有)
1. internal:  
跟private很像，但差別在宣告為internal的function可以被*繼承的類別*呼叫
2. external:  
很像public，但被宣告為此類型的function**只能**從contract外面被呼叫
3. external vs public(之後補充) 
