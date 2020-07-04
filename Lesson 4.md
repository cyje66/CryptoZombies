# Lesson 4
## payable modifier
在Solidity中，可以支付以太幣(ether)的function
## msg.value
可以看在合約中，有多少以太幣被送出
## withdraws
我們把錢送進了合約上的以太坊帳戶，當然要把它提款出來才能獲得以太幣，看一個例子:
```
contract GetPaid is Ownable {
  function withdraw() external onlyOwner {
    address payable _owner = address(uint160(owner()));
    _owner.transfer(address(this).balance);
  }
}
```
import Ownable後，我們就可以使用`_owner`和`onlyOwner`了，要注意的是`_owner`必須要是`address payable`的型態，才可以做送出或轉帳以太幣的指令。但是`owner()`並不是`address payable` type，
所以要先做轉型。我們可以透過把任何integer型態的變數(像uint160)，轉成`address`型態，就可以產生`address payable` type。
接著，我們用transfer function來轉帳，address(this).balance會回傳整個合約上儲存的總損益(total balance)，如果有100個users付給合約1個以太幣，那address(this).balance就會等於100個以太幣。
