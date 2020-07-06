# Lesson 5
## SafeMath 
是OpenZeppelin提供的加減乘除library，有4個functions — add, sub, mul, and div，他可以防止數值資料在計算時，出現overflow和underflow的情況。
```
using SafeMath for uint256;

uint256 a = 5;
uint256 b = a.add(3); // 5 + 3 = 8
uint256 c = a.mul(2); // 5 * 2 = 10
```
## library 
跟contract很像，但有幾點不一樣，例如上面的SafeMath:
```
library SafeMath {

  function mul(uint256 a, uint256 b) internal pure returns (uint256) {
    if (a == 0) {
      return 0;
    }
    uint256 c = a * b;
    assert(c / a == b);
    return c;
  }

  function div(uint256 a, uint256 b) internal pure returns (uint256) {
    // assert(b > 0); // Solidity automatically throws when dividing by 0
    uint256 c = a / b;
    // assert(a == b * c + a % b); // There is no case in which this doesn't hold
    return c;
  }

  function sub(uint256 a, uint256 b) internal pure returns (uint256) {
    assert(b <= a);
    return a - b;
  }

  function add(uint256 a, uint256 b) internal pure returns (uint256) {
    uint256 c = a + b;
    assert(c >= a);
    return c;
  }
}
```
`library`讓我們可以使用`using`這個保留字，其可以把`library`裡的function自動加在*某個資料型態上*，例如:
```
using SafeMath for uint;
// now we can use these methods on any uint
uint test = 2;
test = test.mul(3); // test now equals 6
test = test.add(5); // test now equals 11
```
## assert v.s. require
兩個很像，都會在有error時，回傳false。不一樣的是，require會在function fail時退回user gas，而assert不會。所以assert通常用於有重大錯誤時，例如uint overflow，以減少使用者的惡意執行。
## 
