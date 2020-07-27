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
## Migrations 
經過test之後，下一步就是migrations了。但下了migration指令，也就代表smart contract被部署了，所以我們要先deploy 在 Rinkeby上，因為一旦deploy在main net上，就意味著不能再更改程式碼了，而且需要真的錢來支付交易和手續費。 *(Note: truffle migration就是truffle deploy的別名，兩個其實是一樣的)*
```
truffle migrate --network rinkeby
```
## Loom Basechain
在Loom上，使用者可以使用更快的速度和免費的交易手續費，要把Truffle deploy到Loom上，需要`loom-truffle-provider`的幫助，`loom-truffle-provider`的功能就是當Web3與Loom之間的橋樑，讓他們彼此相容，使使用者不用知道他們之間運作的原理。
## Deploy to Loom testnet
首先，我們要先創造我們的Loom private key: 
```
$./loom genkey -a public_key -k private_key
local address: 0x42F401139048AB106c9e25DCae0Cf4b1Df985c39
local address base64: QvQBE5BIqxBsniXcrgz0sd+YXDk=
$cat private_key
/i0Qi8e/E+kVEIJLRPV5HJgn0sQBVi88EQw/Mq4ePFD1JGV1Nm14dA446BsPe3ajte3t/tpj7HaHDL84+Ce4Dg==
```
## Updating truffle.js
* 我們要先初始化`loom-truffle-provider`:
```
const LoomTruffleProvider = require('loom-truffle-provider');
```
* 接下來，我們要讓Truffle知道怎麼deploy到Loom testnet上，所以我們在truffle.js裡加上一個object: 
```
loom_testnet: {
  provider: function() {
    const privateKey = 'YOUR_PRIVATE_KEY'
    const chainId = 'extdev-plasma-us1';
    const writeUrl = 'http://extdev-plasma-us1.dappchains.com:80/rpc';
    const readUrl = 'http://extdev-plasma-us1.dappchains.com:80/query';
    return new LoomTruffleProvider(chainId, writeUrl, readUrl, privateKey);
    },
  network_id: '9545242630824'
}
```
* 最後就是deploy了:
```
truffle migrate --network loom_testnet
```
## Deploy to Basechain
有幾個步驟:  
* Create a new private key
* Securely pass the private key to Truffle
* Tell Truffle how to deploy to Basechain by adding a new object to the truffle.js configuration file
* Whitelist your deployment keys so you can deploy to Basechain
* Lastly, we wrap everything up by actually deploying the smart contract
## Create a new private key
```
./loom genkey -a mainnet_public_key -k mainnet_private_key
local address: 0x07419790A773Cc6a2840f1c092240922B61eC778
local address base64: B0GXkKdzzGooQPHAkiQJIrYex3g=
```
## Securely pass the private key to Truffle
為了防止私鑰檔被發佈在Github上，所以要新增`.gitignore`檔:
```
touch .gitignore
```
告訴Github要忽略這個檔案:
```
echo mainnet_private_key >> .gitignore
```
完成後，我們要告訴Truffle讀取私鑰檔，所以在truffle.js中要先import一些東西: 
```
const { readFileSync } = require('fs')
const path = require('path')
const { join } = require('path')
```
接著，我們要定義一個function讓他讀取private key，並初始化一個新的`LoomTruffleProvider`:
```
function getLoomProviderWithPrivateKey (privateKeyPath, chainId, writeUrl, readUrl) {
  const privateKey = readFileSync(privateKeyPath, 'utf-8');
  return new LoomTruffleProvider(chainId, writeUrl, readUrl, privateKey);
}
```
## Tell Truffle how to deploy to Basechain
```
basechain: {
  provider: function() {
    const chainId = 'default';
    const writeUrl = 'http://basechain.dappchains.com/rpc';
    const readUrl = 'http://basechain.dappchains.com/query';
    return new LoomTruffleProvider(chainId, writeUrl, readUrl, privateKey);
    const privateKeyPath = path.join(__dirname, 'mainnet_private_key');
    const loomTruffleProvider = getLoomProviderWithPrivateKey(privateKeyPath, chainId, writeUrl, readUrl);
    return loomTruffleProvider;
    },
  network_id: '*'
}
```
最後的truffle.js會變成:
```
// Initialize HDWalletProvider
const HDWalletProvider = require("truffle-hdwallet-provider");

const { readFileSync } = require('fs')
const path = require('path')
const { join } = require('path')


// Set your own mnemonic here
const mnemonic = "YOUR_MNEMONIC";

function getLoomProviderWithPrivateKey (privateKeyPath, chainId, writeUrl, readUrl) {
  const privateKey = readFileSync(privateKeyPath, 'utf-8');
  return new LoomTruffleProvider(chainId, writeUrl, readUrl, privateKey);
}

// Module exports to make this configuration available to Truffle itself
module.exports = {
  // Object with configuration for each network
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
    },

    basechain: {
      provider: function() {
        const chainId = 'default';
        const writeUrl = 'http://basechain.dappchains.com/rpc';
        const readUrl = 'http://basechain.dappchains.com/query';
        return new LoomTruffleProvider(chainId, writeUrl, readUrl, privateKey);
        const privateKeyPath = path.join(__dirname, 'mainnet_private_key');
        const loomTruffleProvider = getLoomProviderWithPrivateKey(privateKeyPath, chainId, writeUrl, readUrl);
        return loomTruffleProvider;
        },
      network_id: '*'
    }
  }
};
```
## Whitelist your deployment keys
在deploy到Basechain上之前，我們要先把私鑰加入白名單，參考這個教學: [Deploy to Mainnet](https://loomx.io/developers/en/deploy-loom-mainnet.html)
最後執行:
```
truffle migrate --network basechain
```
