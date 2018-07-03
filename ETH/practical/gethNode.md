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
> geth --testnet removedb
>
> geth --testnet --fast --bootnodes "enode://20c9ad97c081d63397d7b685a412227a40e23c8bdc6688c6f37e97cfbc22d2b4d1db1510d8f61e6a8866ad7f0e17c02b14182d37ea7c3c8b9c2683aeb6b733a1@52.169.14.227:30303,enode://6ce05930c72abc632c58e2e4324f7c7ea478cec0ed4fa2528982cf34483094e9cbc9216e7aa349691242576d552a2a56aaeae426c5303ded677ce455ba1acd9d@13.84.180.240:30303"

    
- add rpc and localhost:

geth --fast --cache=1048 --testnet --rpc --rpcapi="db,eth,net,web3,personal,web3" --rpccorsdomain '*' --rpcaddr localhost --rpcport 8545 --bootnodes  "enode://..."
    
    
- add unlock (address):

geth --fast --cache=1048 --testnet --unlock "0xAf588a130396276F496bA0EBE24e85711B43D2aA" --rpc --rpcapi="db,eth,net,web3,personal,web3" --rpccorsdomain '*' --rpcaddr localhost --rpcport 8545 --bootnodes  "enode://..."


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


```
    ETHEREUM OPTIONS:
      --config value                                TOML configuration file
      --datadir "/Users/wyf.wang/Library/Ethereum"  Data directory for the databases and keystore
      --keystore                                    Directory for the keystore (default = inside the datadir)
      --nousb                                       Disables monitoring for and managing USB hardware wallets
      --networkid value                             Network identifier (integer, 1=Frontier, 2=Morden (disused), 3=Ropsten, 4=Rinkeby) (default: 1)
      --testnet                                     Ropsten network: pre-configured proof-of-work test network
      --rinkeby                                     Rinkeby network: pre-configured proof-of-authority test network
      --syncmode "fast"                             Blockchain sync mode ("fast", "full", or "light")
      --gcmode value                                Blockchain garbage collection mode ("full", "archive") (default: "full")
      --ethstats value                              Reporting URL of a ethstats service (nodename:secret@host:port)
      --identity value                              Custom node name
      --lightserv value                             Maximum percentage of time allowed for serving LES requests (0-90) (default: 0)
      --lightpeers value                            Maximum number of LES client peers (default: 100)
      --lightkdf                                    Reduce key-derivation RAM & CPU usage at some expense of KDF strength
    
    DEVELOPER CHAIN OPTIONS:
      --dev               Ephemeral proof-of-authority network with a pre-funded developer account, mining enabled
      --dev.period value  Block period to use in developer mode (0 = mine only if transaction pending) (default: 0)
    
    ETHASH OPTIONS:
      --ethash.cachedir                          Directory to store the ethash verification caches (default = inside the datadir)
      --ethash.cachesinmem value                 Number of recent ethash caches to keep in memory (16MB each) (default: 2)
      --ethash.cachesondisk value                Number of recent ethash caches to keep on disk (16MB each) (default: 3)
      --ethash.dagdir "/Users/wyf.wang/.ethash"  Directory to store the ethash mining DAGs (default = inside home folder)
      --ethash.dagsinmem value                   Number of recent ethash mining DAGs to keep in memory (1+GB each) (default: 1)
      --ethash.dagsondisk value                  Number of recent ethash mining DAGs to keep on disk (1+GB each) (default: 2)
    
    TRANSACTION POOL OPTIONS:
      --txpool.nolocals            Disables price exemptions for locally submitted transactions
      --txpool.journal value       Disk journal for local transaction to survive node restarts (default: "transactions.rlp")
      --txpool.rejournal value     Time interval to regenerate the local transaction journal (default: 1h0m0s)
      --txpool.pricelimit value    Minimum gas price limit to enforce for acceptance into the pool (default: 1)
      --txpool.pricebump value     Price bump percentage to replace an already existing transaction (default: 10)
      --txpool.accountslots value  Minimum number of executable transaction slots guaranteed per account (default: 16)
      --txpool.globalslots value   Maximum number of executable transaction slots for all accounts (default: 4096)
      --txpool.accountqueue value  Maximum number of non-executable transaction slots permitted per account (default: 64)
      --txpool.globalqueue value   Maximum number of non-executable transaction slots for all accounts (default: 1024)
      --txpool.lifetime value      Maximum amount of time non-executable transaction are queued (default: 3h0m0s)
  
    PERFORMANCE TUNING OPTIONS:
      --cache value            Megabytes of memory allocated to internal caching (default: 1024)
      --cache.database value   Percentage of cache memory allowance to use for database io (default: 75)
      --cache.gc value         Percentage of cache memory allowance to use for trie pruning (default: 25)
      --trie-cache-gens value  Number of trie node generations to keep in memory (default: 120)
    
    
    ACCOUNT OPTIONS:
      --unlock value    Comma separated list of accounts to unlock
      --password value  Password file to use for non-interactive password input
    
    API AND CONSOLE OPTIONS:
      --rpc                  Enable the HTTP-RPC server
      --rpcaddr value        HTTP-RPC server listening interface (default: "localhost")
      --rpcport value        HTTP-RPC server listening port (default: 8545)
      --rpcapi value         API's offered over the HTTP-RPC interface
      --ws                   Enable the WS-RPC server
      --wsaddr value         WS-RPC server listening interface (default: "localhost")
      --wsport value         WS-RPC server listening port (default: 8546)
      --wsapi value          API's offered over the WS-RPC interface
      --wsorigins value      Origins from which to accept websockets requests
      --ipcdisable           Disable the IPC-RPC server
      --ipcpath              Filename for IPC socket/pipe within the datadir (explicit paths escape it)
      --rpccorsdomain value  Comma separated list of domains from which to accept cross origin requests (browser enforced)
      --rpcvhosts value      Comma separated list of virtual hostnames from which to accept requests (server enforced). Accepts '*' wildcard. (default: "localhost")
      --jspath loadScript    JavaScript root path for loadScript (default: ".")
      --exec value           Execute JavaScript statement
      --preload value        Comma separated list of JavaScript files to preload into the console
    
    NETWORKING OPTIONS:
      --bootnodes value     Comma separated enode URLs for P2P discovery bootstrap (set v4+v5 instead for light servers)
      --bootnodesv4 value   Comma separated enode URLs for P2P v4 discovery bootstrap (light server, full nodes)
      --bootnodesv5 value   Comma separated enode URLs for P2P v5 discovery bootstrap (light server, light nodes)
      --port value          Network listening port (default: 30303)
      --maxpeers value      Maximum number of network peers (network disabled if set to 0) (default: 25)
      --maxpendpeers value  Maximum number of pending connection attempts (defaults used if set to 0) (default: 0)
      --nat value           NAT port mapping mechanism (any|none|upnp|pmp|extip:<IP>) (default: "any")
      --nodiscover          Disables the peer discovery mechanism (manual peer addition)
      --v5disc              Enables the experimental RLPx V5 (Topic Discovery) mechanism
      --netrestrict value   Restricts network communication to the given IP networks (CIDR masks)
      --nodekey value       P2P node key file
      --nodekeyhex value    P2P node key as hex (for testing)
    
    MINER OPTIONS:
      --mine                    Enable mining
      --minerthreads value      Number of CPU threads to use for mining (default: 8)
      --etherbase value         Public address for block mining rewards (default = first account created) (default: "0")
      --targetgaslimit value    Target gas limit sets the artificial target gas floor for the blocks to mine (default: 4712388)
      --gasprice "18000000000"  Minimal gas price to accept for mining a transactions
      --extradata value         Block extra data set by the miner (default = client version)
    
    GAS PRICE ORACLE OPTIONS:
      --gpoblocks value      Number of recent blocks to check for gas prices (default: 20)
      --gpopercentile value  Suggested gas price is the given percentile of a set of recent transaction gas prices (default: 60)
    
    VIRTUAL MACHINE OPTIONS:
      --vmdebug  Record information useful for VM and contract debugging
    
    LOGGING AND DEBUGGING OPTIONS:
      --metrics                 Enable metrics collection and reporting
      --fakepow                 Disables proof-of-work verification
      --nocompaction            Disables db compaction after import
      --verbosity value         Logging verbosity: 0=silent, 1=error, 2=warn, 3=info, 4=debug, 5=detail (default: 3)
      --vmodule value           Per-module verbosity: comma-separated list of <pattern>=<level> (e.g. eth/*=5,p2p=4)
      --backtrace value         Request a stack trace at a specific logging statement (e.g. "block.go:271")
      --debug                   Prepends log messages with call-site location (file and line number)
      --pprof                   Enable the pprof HTTP server
      --pprofaddr value         pprof HTTP server listening interface (default: "127.0.0.1")
      --pprofport value         pprof HTTP server listening port (default: 6060)
      --memprofilerate value    Turn on memory profiling with the given rate (default: 524288)
      --blockprofilerate value  Turn on block profiling with the given rate (default: 0)
      --cpuprofile value        Write CPU profile to the given file
      --trace value             Write execution trace to the given file
    
    WHISPER (EXPERIMENTAL) OPTIONS:
      --shh                       Enable Whisper
      --shh.maxmessagesize value  Max message size accepted (default: 1048576)
      --shh.pow value             Minimum POW accepted (default: 0.2)
    
    DEPRECATED OPTIONS:
      --fast   Enable fast syncing through state downloads
      --light  Enable light client mode
    
    MISC OPTIONS:
      --help, -h  show help
```

参考：
- [geth中文介绍](http://www.blockcode.org/archives/201)
- https://learnblockchain.cn/2017/12/01/geth_cmd_short/
- https://blog.csdn.net/chenyufeng1991/article/details/53458175







