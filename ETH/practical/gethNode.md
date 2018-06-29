

# Go Ethereum

<br/>

## 1. 简单启动

- 主网络节点： geth --fast --cache=512 console
- 测试节点：  geth --testnet --fast --cache=512 console


## 2. 本地geth节点

  - 本地geth节点： 
  -     geth --rpc --rpccorsdomain "http://localhost:3000" 
  - 本地连接测试节点：  
  -      geth attach ipc:testNet/geth.ipc
  - 本地rpc：
  
     geth --testnet --syncmode "fast" --rpc --rpcapi db,eth,net,web3,personal --cache=1024 --rpcport 8545 --rpcaddr 127.0.0.1 --rpccorsdomain "*" --bootnodes XXX
  - 本地目录节点
  -     geth --datadir /Users/wyf.wang/testNet --dev console
  -     geth --datadir chain0/data --networkid 1108 --rpc console
  
  - 私链节点
    
    geth --rpc --port 30304 --rpcport 8599 --targetgaslimit 9000000000000 --gasprice 18000000000 --rpcapi db,eth,web3,net,admin,miner,personal --cache 1024 --maxpeers 25 --datadir /home/testnet/.ethereum --keystore /home/testnet/.ethereum/keystore --nousb --fast --testnet --mine --minerthreads 1 --unlock 0xC1320531dF2612B85e6FEDFd4d6358ed711bbe66 --password /home/testnet/pass

> 本地节点私钥地址： ~/Library/Ethereum/testnet/keystore  


## 3. 同步Ropsten测试节点

同步ropsten的区块：(https://github.com/ethereum/ropsten)

官网提供：

> geth --testnet --fast --bootnodes "enode://20c9ad97c081d63397d7b685a412227a40e23c8bdc6688c6f37e97cfbc22d2b4d1db1510d8f61e6a8866ad7f0e17c02b14182d37ea7c3c8b9c2683aeb6b733a1@52.169.14.227:30303,enode://6ce05930c72abc632c58e2e4324f7c7ea478cec0ed4fa2528982cf34483094e9cbc9216e7aa349691242576d552a2a56aaeae426c5303ded677ce455ba1acd9d@13.84.180.240:30303"

    
- add rpc and localhost:

geth --fast --cache=1048 --testnet --rpc --rpcapi="db,eth,net,web3,personal,web3" --rpccorsdomain '*' --rpcaddr localhost --rpcport 8545 --bootnodes  "enode://..."
    
    
- add unlock (address):

geth --fast --cache=1048 --testnet --unlock "0xAf588a130396276F496bA0EBE24e85711B43D2aA" --rpc --rpcapi="db,eth,net,web3,personal,web3" --rpccorsdomain '*' --rpcaddr localhost --rpcport 8545 --bootnodes  "enode://..."

example:

> geth --testnet --syncmode "fast" --rpc --rpcapi db,eth,net,web3,personal --cache=1024 --rpcport 8545 --rpcaddr 127.0.0.1 --rpccorsdomain "*" --bootnodes "enode://20c9ad97c081d63397d7b685a412227a40e23c8bdc6688c6f37e97cfbc22d2b4d1db1510d8f61e6a8866ad7f0e17c02b14182d37ea7c3c8b9c2683aeb6b733a1@52.169.14.227:30303,enode://6ce05930c72abc632c58e2e4324f7c7ea478cec0ed4fa2528982cf34483094e9cbc9216e7aa349691242576d552a2a56aaeae426c5303ded677ce455ba1acd9d@13.84.180.240:30303"


### 编程接口Geth节点 API

> Geth建立了基于JSON-RPC的API（标准API和 Geth特定API）。

> 可以通过HTTP，WebSockets和IPC（基于unix的平台上的unix套接字，Windows上的命名管道）进行暴露。
IPC接口默认启用，并公开Geth支持的所有API，而HTTP和WS接口需要手动启用，并且仅由于安全原因而暴露了一部分API。这些可以打开/关闭并按照预期进行配置。

 基于HTTP的JSON-RPC API选项：
-   --rpc 启用HTTP-RPC服务器
-   --rpcaddr HTTP-RPC服务器侦听界面（默认：“localhost”）
-   --rpcport HTTP-RPC服务器侦听端口（默认值：8545）
-   --rpcapi API通过HTTP-RPC接口提供（默认：“eth，net，web3” 其他: personal,db等）
-   --rpccorsdomain 可接受跨源请求（浏览器强制执行）的域名逗号分隔列表
-   --ws 启用WS-RPC服务器
-   --wsaddr WS-RPC服务器侦听界面（默认：“localhost”）
-   --wsport WS-RPC服务器侦听端口（默认值：8546）
-   --wsapi API通过WS-RPC接口提供（默认值：“eth，net，web3”）
-   --wsorigins 来源于接受websockets请求
-   --ipcdisable 禁用IPC-RPC服务器
-   --ipcapi API通过IPC-RPC接口提供（默认为“admin，debug，eth，miner，net，personal，shh，txpool，web3”）
-   --ipcpath 在datadir中的IPC套接字/管道的文件名（显式路径转义）
-   
- --port geth节点p2p的端口
- --datadir 规定本地数据库，区块链存储，以及私钥等存放的地址，默认地址 ～/.ethereum
- --dev 加上这个参数则会启动私有链

> 您将需要使用自己的编程环境的功能（库，工具等）通过HTTP，WS或IPC连接到配置了上述标志的Geth节点，您需要 在所有传输上使用JSON-RPC。您可以为多个请求重复使用相同的连接！

> 注意：在这样做之前，请了解打开基于HTTP / WS的传输的安全隐患！互联网上的黑客正在积极地尝试用暴露的API来颠覆艾薇儿节点！此外，所有浏览器标签都可以访问本地运行的Web服务器，因此恶意网页可能会尝试颠覆本地可用的API！



## Network  ID （节点网络ID）

- 0: Olympic, Ethereum public pre-release testnet
- 1: Frontier, Homestead, Metropolis, the Ethereum public main network
- 1: Classic, the (un)forked public Ethereum Classic main network, chain ID 61
- 1: Expanse, an alternative Ethereum implementation, chain ID 2
- 2: Morden, the public Ethereum testnet, now Ethereum Classic testnet
- 3: **++Ropsten++**, the public cross-client Ethereum testnet
- 4: Rinkeby, the public Geth PoA testnet
- 8: Ubiq, the public Gubiq main network with flux difficulty chain ID 8
- 42: Kovan, the public Parity PoA testnet
- 77: Sokol, the public POA Network testnet
- 99: Core, the public POA Network main network
- 7762959: Musicoin, the music blockchain
- [Other]: Could indicate that your connected to a local development test network.


## COMMANDS:

    account     Manage accounts
    attach      Start an interactive JavaScript environment (connect to node)
    bug         opens a window to report a bug on the geth repo
    console     Start an interactive JavaScript environment
    copydb      Create a local chain from a target chaindata folder
    dump        Dump a specific block from storage
    dumpconfig  Show configuration values
    export      Export blockchain into file
    import      Import a blockchain file
    init        Bootstrap and initialize a new genesis block
    js          Execute the specified JavaScript files
    license     Display license information
    makecache   Generate ethash verification cache (for testing)
    makedag     Generate ethash mining DAG (for testing)
    monitor     Monitor and visualize node metrics
    removedb    Remove blockchain and state databases
    version     Print version numbers
    wallet      Manage Ethereum presale wallets
    help, h     Shows a list of commands or help for one command
    
    
---

![image](http://7xku28.com1.z0.glb.clouddn.com/gethApi.png)

---


## 常用Web3查询API：

  - 查询账户： 
  -     web3.eth.accounts
  - 创建账号： 
  -     web3.personal.newAccount('这里输入你的密码')
  - 解锁账号：
  -     web3.personal.unlockAccount('0xAf588a130396276F496bA0EBE24e85711B43D2aA','yunfeng123456',15000)
  - 查询余额： 
  -     web3.eth.getBalance('0xAf588a130396276F496bA0EBE24e85711B43D2aA').toString()


## node调用 JCON-RPC:

    var express = require('express');
    var router = express.Router();
    var request = require("request");
    
    /* GET home page. */
    router.get('/', function (req, res, next) {
    　　var headers = {
       　　'User-Agent': 'Super Agent/0.0.1',
       　　'Content-Type': 'application/x-www-form-urlencoded',
       　　'Accept': 'application/json-rpc',
       　　'keyname':'xx'
       };
    　　var options = {
    　　　　url: "http://192.168.20.131:8088",
    　　　　method: 'POST',
    　　　　headers: headers,
    　　　　form: {jsonrpc:'{"method": "ses/user/stringme", "params": ["XXXXX"], "id": 4}'}
    　　};
        request(options, function (error, response, body) {
            res.render("index", {title:res.statusCode.toString() + " " + body});
        });
    });
    module.exports = router;


---


参考：
- [geth中文介绍](http://www.blockcode.org/archives/201)
- https://learnblockchain.cn/2017/12/01/geth_cmd_short/
- https://blog.csdn.net/chenyufeng1991/article/details/53458175













