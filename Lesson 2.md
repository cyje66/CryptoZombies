# Lesson 2
## Address
每一個帳號都有一個address，可以把他想成是一組匯款的帳號，透過這個address，你可以匯款給那個address的擁有者。
## Mapping(類似dictionary)
  以key-value為基準的儲存或查找data的方法，處理方式就和array一樣，例如:
  ```
  //儲存一個uint，其代表一個address的帳戶餘額
  mapping (address => uint) public accountBalance;
  ```
  此時的key : address , value : uint，另一個例子:
  ```
  //根據一個人的userId來查詢/儲存帳戶擁有者的名稱
  mapping (uint => string) userIdToName;
  ```
  此時的key : uint , value : string
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
  // (Side note: Solidity doesn't have native string comparison, so we
  // compare their keccak256 hashes to see if the strings are equal)
  require(keccak256(abi.encodePacked(_name)) == keccak256(abi.encodePacked("Vitalik")));
  // If it's true, proceed with the function:
  return "Hi!";
}
```
