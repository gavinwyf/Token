
# HD钱包

钱包的核心功能是私钥的创建、存储和使用。HD 是 Hierarchical Deterministic（分层确定性）分层确定性的概念早在 BIP32 提案提出。根据比特币核心开发者 Gregory Maxwell 的原始描述和讨论，Pieter Wuille 在2012 年 02月 11日整理完善提交 BIP32 。直到 2016年 6月 15 日 才被合并到 Bitcoin Core，目前几乎所有的钱包服务商都整合了该协议。

# BIPs

> BIP全名是Bitcoin Improvement Proposals，是提出Bitcoin的新功能或改进措施的文件。可由任何人提出，经过审核后公布在 [bitcoin/bips](https://github.com/bitcoin/bips) 上。BIP和Bitcoin的关系，就像是RFC之于Internet。

而其中的BIP32, BIP39, BIP44共同定义了目前被广泛使用的HD Wallet，包含其设计动机和理念、实作方式、实例等。

## [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)

> 定义Hierarchical Deterministic wallet (简称“HD Wallet”)，是一个系统可以从单一个seed 产生一树状结构储存多组keypairs（私钥和公钥）。好处是可以方便的备份、转移到其他相容装置（因为都只需要seed），以及分层的权限控制等。

## [BIP39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)

> 将seed用方便记忆和书写的单字表示。一般由12个单字组成，称为mnemonic code(phrase)，中文称为助记词或助记码，例如：

```
    rose rocket invest real refuse margin festival danger anger border idle brown
```

## [BIP44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki)

> 基于BIP32的系统，赋予树状结构中的各层特殊的意义。让同一个seed可以支援多币种、多帐户等。各层定义如下
```
    m / purpose' / coin_type' / account' / change(公共派生) / address_index(公共派生)
```

purporse固定是44，代表使用BIP44。coin_type用来表示不同币种，例如Bitcoin就是0，Ethereum是60。change,常量0用于外部链，常量1用于内部链. address_index在BIP32中用作子索引，可进行公共派生。


# Ethereum HD Wallet

Ethereum的钱包目前均采用以上Bitcoin HD Wallet的架构，并订coin_type'为60'，可以在ethereum/EIPs/issues中看到相关的讨论。举例来说，在一个Ethereum HD Wallet中，第一个帐户（这里的帐户指BIP44中定义的account'）的第一组keypair，其路径会是m/44'/60'/0'/0/0。



# Bitcoin HD Wallet

⼀笔⽐特币交易是⼀个含有输⼊值和输出值的数据结构，该数据结构植⼊了将⼀笔资⾦从初始点（输⼊值）转移⾄⽬标地址（输出值）的代码信息

UTXO（Unspent Transaction Output），它是⽐特币交易的基本单位，是⼀个未经使⽤的交易输出。⼀个⽤⼾的⽐特币会被当作UTXO分散到数百个交易和数百个区块中。实际上，并不存在储存⽐特币地址或账⼾余额的地点，只有被所有者锁住的、分散的UTXO

UTXO可以是任意值，但只要它被创造出来了，就像不能被切成两半的硬币⼀样不可再分.如果⼀个UTXO⽐⼀笔交易所需量⼤，它仍会被当作⼀个整体⽽消耗掉，但同时会在交易中⽣成零头。

中本聪在设计比特币时，运用的找零机制是创建了一个新的地址作为每笔交易的找零地址。这样设计的好处，是为了保护交易用户的隐私和避免一些安全隐患。



