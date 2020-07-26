# Lesson 2 How to deploy smart contracts with truffle
## truffle init
初始化後，會發現目錄長這樣:
```
├── contracts
    ├── Migrations.sol
├── migrations
    ├── 1_initial_migration.js
└── test
truffle-config.js
truffle.js
```
先了解一下各資料夾的意義: 
* contracts: this is the place where Truffle expects to find all our smart contracts. *Note: truffle init should automatically create a contract called Migrations.sol and the corresponding migration file. Migrations.sol keeps track of the changes you're making to your code. The way it works is that the history of changes is saved onchain. Thus, there's no way you will ever deploy the same code twice.*
* migrations: a migration is a JavaScript file that tells Truffle how to deploy a smart contract.
* test: here we are expected to put the unit tests which will be JavaScript or Solidity files. Remember, once a contract is deployed it can't be changed, making it essential that we test our smart contracts before we deploy them.
* truffle.js and truffle-config.js: config files used to store the network settings for deployment. Truffle needs two config files because on Windows having both truffle.js and truffle.exe in the same folder might generate conflicts. Long story short - if you are running Windows, it is advised to delete truffle.js and use truffle-config.js as the default config file.
## truffle-hdwallet-provider
我們會用*Infura*把我們的程式碼部署到Ethereum上，這樣我們就不需要建立自己的Ethereum節點和錢包也可以執行應用程式了。但*Infura*不能管理我們的private key，就代表他不能代替我們簽署交易(sign transaction)，由於部署智能合約需要Truffle簽署交易，所以我們需要`truffle-hdwallet-provider`來處理transaction signing的問題。
## Creating a new migration
`truffle init`中，他已經幫我們建好一個檔案了--`./contracts/1_initial_migration.js`，看一下裡面的程式碼:
```
var Migrations = artifacts.require("./Migrations.sol");
module.exports = function(deployer) {
  deployer.deploy(Migrations);
};
```
首先，這個script告訴truffle我們想要跟Migrations互動。接著，他匯出一個以`deployer`為參數的function，這個物件扮演著在developer和Truffle's deployment engine之間的interface。
## Ethereum Test Networks
我們還差一步就可以deploy了，那就是我們要先告訴Truffle我們要deploy到哪個network。  
許多的公共Ethereum test networks讓我們可以免費的測試，這些test networks用相較於main net不一樣的共識演算法(通常是PoA)，而在測試的過程中不用花費Ether。在這個課程中，我們會用Rinkeby，一個由The Ethereum Foundation創造的public test network: 
```
networks: {
  // Configuration for mainnet
  mainnet: {
    provider: function () {
      // Setting the provider with the Infura Rinkeby address and Token
      return new HDWalletProvider(mnemonic, "https://mainnet.infura.io/v3/YOUR_TOKEN")
    },
    network_id: "1"
  },
  // Configuration for rinkeby network
  rinkeby: {
    // Special function to setup the provider
    provider: function () {
      // Setting the provider with the Infura Rinkeby address and Token
      return new HDWalletProvider(mnemonic, "https://rinkeby.infura.io/v3/YOUR_TOKEN")
    },
    // Network id is 4 for Rinkeby
    network_id: 4
  }
```
