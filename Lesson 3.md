# Lesson 3
## Ownable
若有一個function的型態是external的話，由於大家都可以呼叫他，因此產生一個漏洞。創建一個Ownable Contract就可以避免這個問題，其意義是擁有者(你自己)有特殊的權利。
#### 如何使用?
在OpenZeppelin 上已經有寫好的程式碼了，只要import ownable.sol，接著將該contract繼承Ownable就行了。(OpenZeppelin 是一個有許多安全且經過審查的智能合約的library)
## Gas
在Solidity中，使用者(users)要在DApp上執行一個function的時候，需要一個叫做gas的貨幣
