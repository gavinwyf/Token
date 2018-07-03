
[官方安装](https://github.com/ethereum/go-ethereum/wiki/Installing-Geth)

### 环境配置

> 至少 2CPU 4G 150G SSD

### 安装所需基础工具：

    yum update -y && yum install git wget bzip2 vim gcc-c++ ntp epel-release nodejs cmake -y

### 安装Go

    wget https://dl.google.com/go/go1.10.linux-amd64.tar.gz
    tar -C /usr/local -xzf go1.10.linux-amd64.tar.gz
    echo 'export GOROOT=/usr/local/go' >> /etc/profile  
    echo 'export PATH=$PATH:$GOROOT/bin' >> /etc/profile  
    echo 'export GOPATH=/root/go' >> /etc/profile
    echo 'export PATH=$PATH:$GOPATH/bin' >> /etc/profile
    source /etc/profile

### 验证

    $ go version
    go version go1.10 linux/amd64

### 克隆编译项目go-ethereum

    git clone https://github.com/ethereum/go-ethereum.git  
    cd go-ethereum  
    make all

### 配置环境变量
    
    /var/www/go-ethereum/build/bin/geth
    echo "export PATH=$PATH:/var/www/go-ethereum/build/bin" >> /etc/profile

### 编写启动脚本

```bash
    #!/bin/bash

    case "$1" in
        "main" )
                echo "================ lanch main node ============="
                nohup geth --cache=1024 --rpc --rpcaddr 127.0.0.1 --rpcport 8545 --rpccorsdomain "http://localhost:3000" > mainOutput 2>&1 &
                exit
        ;;
        "test" )
                echo "================ lauch Ropsten test node =============== "
                nohup geth --testnet --syncmode "fast" --rpc  --cache=1024 --rpcport 9545 --rpcaddr 127.0.0.1 --rpccorsdomain "http://localhost:3000" --bootnodes "enode://20c9ad97c081d63397d7b685a412227a40e23c8bdc6688c6f37e97cfbc22d2b4d1db1510d8f61e6a8866ad7f0e17c02b14182d37ea7c3c8b9c2683aeb6b733a1@52.169.14.227:30303,enode://6ce05930c72abc632c58e2e4324f7c7ea478cec0ed4fa2528982cf34483094e9cbc9216e7aa349691242576d552a2a56aaeae426c5303ded677ce455ba1acd9d@13.84.180.240:30303" &
                exit
        ;;
        "easy" )
                echo "================ lanch easy main node ============="
                nohup geth --fast --cache=1024 --rpc --rpccorsdomain "http://localhost:3000"  &
                exit
        ;;
        * ) echo "Bad option, please choose again! switch  1. main => 主节点  2. test => ropsten 测试节点  3. easy => 简单主节点"
    esac

```

### 常用命令

- 查看账号 personal.listAccounts

- 新建账号，密码123456

        personal.newAccount(“123456”)
    
- 查看cionbase

        eth.coinbase

- 查看区块数量
        
        eth.blockNumber

- 查看账号余额

        eth.getBalance("0x7d84a2638c9b25a758026446e67c822f34247bd8")

- 换另一个用户挖矿
    
        miner.setEtherbase("0x3e5b31e581546f2900c0f3289153c788c92a2b41")

- 解锁挖矿用户

        personal.unlockAccount(eth.coinbase)


</br>
</br>
</br>