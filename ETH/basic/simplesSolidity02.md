

# Solidity 进阶（一）


### 1. 智能合约特性

- 永固性 - 永久的，不可更改的，存在以太网上



### 2. 外部依赖

不能硬编码，而要采用“函数”，以便于 DApp 的关键部分可以以参数形式修改

> 运行时再设定对应的真实地址，说白了就是不要写死地址，实现个set函数即可。

### 3. Ownable Contracts

指定合约的“所有权” - 就是说，给它指定一个主人，只有主人对它享有特权

[OpenZeppelin](https://github.com/OpenZeppelin/zeppelin-solidity) 库的 Ownable 合约

    /**
     * @title Ownable
     * @dev The Ownable contract has an owner address, and provides basic authorization control
     * functions, this simplifies the implementation of "user permissions".
     */
    contract Ownable {
      address public owner;
      event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);
    
      /**
       * @dev The Ownable constructor sets the original `owner` of the contract to the sender
       * account.
       */
      function Ownable() public {
        owner = msg.sender;
      }
    
      /**
       * @dev Throws if called by any account other than the owner.
       */
      modifier onlyOwner() {
        require(msg.sender == owner);
        _;
      }
    
      /**
       * @dev Allows the current owner to transfer control of the contract to a newOwner.
       * @param newOwner The address to transfer ownership to.
       */
      function transferOwnership(address newOwner) public onlyOwner {
        require(newOwner != address(0));
        OwnershipTransferred(owner, newOwner);
        owner = newOwner;
      }
    }


### 4. onlyOwner 函数修饰符

函数修饰符看起来跟函数没什么不同，不过关键字modifier 告诉编译器，这是个modifier(修饰符)，而不是个function(函数)。它不能像函数那样被直接调用，只能被添加到函数定义的末尾，用以改变函数的行为。

    /**
     * @dev 调用者不是‘主人’，就会抛出异常
     */
    modifier onlyOwner() {
      require(msg.sender == owner);
      _;
    }

onlyOwner 函数修饰符使用：

    contract MyContract is Ownable {
      event LaughManiacally(string laughter);
    
      function likeABoss() external onlyOwner {
        LaughManiacally("Muahahahaha");
      }
    }

调用 likeABoss 时，首先执行 onlyOwner 中的代码， 执行到 onlyOwner 中的 _ ; 语句时，程序再返回并执行 likeABoss 中的代码。

> 场景： 尽管函数修饰符也可以应用到各种场合，但最常见的还是放在函数执行之前添加快速的 require检查。

> 注意：主人对合约享有的特权当然是正当的，不过也可能被恶意使用。比如，万一，主人添加了个后门，允许他偷走别人的僵尸呢？
>
> 所以非常重要的是，部署在以太坊上的 DApp，并不能保证它真正做到去中心，你需要阅读并理解它的源代码，才能防止其中没有被部署者恶意植入后门；作为开发人员，如何做到既要给自己留下修复 bug 的余地，又要尽量地放权给使用者，以便让他们放心你，从而愿意把数据放在你的 DApp 中，这确实需要个微妙的平衡


### 带参数的函数修饰符

    // 存储用户年龄的映射
    mapping (uint => uint) public age;
    
    // 限定用户年龄的修饰符
    modifier olderThan(uint _age, uint _userId) {
      require(age[_userId] >= _age);
      _;
    }
    
    // 调用`olderThan` 修饰符:
    function driveCar(uint _userId) public olderThan(16, _userId) {
      
    }

olderThan 修饰符可以像函数一样接收参数，是“宿主”函数 driveCar 把参数传递给它的修饰符的。


### 5. Gas

在 Solidity 中，你的用户想要每次执行你的 DApp 都需要支付一定的 gas，gas 可以用以太币购买，因此，用户每次跑 DApp 都得花费以太币。

一个 DApp 收取多少 gas 取决于功能逻辑的复杂程度。每个操作背后，都在计算完成这个操作所需要的计算资源，（比如，存储数据就比做个加法运算贵得多）， 一次操作所需要花费的 gas 等于这个操作背后的所有运算花销的总和。

为什么要用 gas 来驱动？

> 以太坊就像一个巨大、缓慢、但非常安全的电脑。当你运行一个程序的时候，网络上的每一个节点都在进行相同的运算，以验证它的输出 —— 这就是所谓的”去中心化“ 由于数以千计的节点同时在验证着每个功能的运行，这可以确保它的数据不会被被监控，或者被刻意修改。
> 
> 可能会有用户用无限循环堵塞网络，抑或用密集运算来占用大量的网络资源，为了防止这种事情的发生，以太坊的创建者为以太坊上的资源制定了价格，想要在以太坊上运算或者存储，你需要先付费。

注意：如果你使用侧链，倒是不一定需要付费。可以找个算法理念不同的侧链来玩它。

### 省 gas 的招数：结构封装 （Struct packing）

通常情况下我们不会考虑使用 unit 变种，因为无论如何定义 uint的大小，Solidity 为它保留256位的存储空间。

除非，把 unit 绑定到 struct 里面。

struct 中有多个 uint，则尽可能使用较小的 uint, Solidity 会将这些 uint 打包在一起，从而占用较少的存储空间

> 当 uint 定义在一个 struct 中的时候，尽量使用最小的整数子类型以节约空间。 并且把同样类型的变量放一起（即在 struct 中将把变量按照类型依次放置），这样 Solidity 可以将存储空间最小化


### 6.时间单位

Solidity 使用自己的本地时间单位。

变量 now 将返回当前的unix时间戳（自1970年1月1日以来经过的秒数）

> 注意：Unix时间传统用一个32位的整数进行存储。这会导致“2038年”问题，当这个32位的unix时间戳不够用，产生溢出，使用这个时间的遗留系统就麻烦了。所以，如果我们想让我们的 DApp 跑够20年，我们可以使用64位整数表示时间，但为此我们的用户又得支付更多的 gas。真是个两难的设计啊！

Solidity 还包含秒(seconds)，分钟(minutes)，小时(hours)，天(days)，周(weeks) 和 年(years) 等时间单位。它们都会转换成对应的秒数放入 uint 中。

    uint lastUpdated;
    
    // 将‘上次更新时间’ 设置为 ‘现在’
    function updateTimestamp() public {
      lastUpdated = now;
    }
    
    // 如果到上次`updateTimestamp` 超过5分钟，返回 'true'
    // 不到5分钟返回 'false'
    function fiveMinutesHavePassed() public view returns (bool) {
      return (now >= (lastUpdated + 5 minutes));
    }


### 7. 利用 'View' 函数节省 Gas

> “view” 函数不花 “gas”

在所能只读的函数上标记上表示“只读”的“external view 声明，就能为你的玩家减少在 DApp 中 gas 用量

> 注意：如果一个 view 函数在另一个函数的内部被调用，而调用函数与 view 函数的不属于同一个合约，也会产生调用成本。这是因为如果主调函数在以太坊创建了一个事务，它仍然需要逐个节点去验证。所以标记为 view 的函数只有在外部调用时才是免费的

Solidity 使用storage(存储)是相当昂贵的，”写入“操作尤其贵。

> 在数组后面加上 memory关键字， 表明这个数组是仅仅在内存中创建，不需要写入外部存储，并且在函数调用结束时它就解散了

    function getArray() external pure returns(uint[]) {
      // 初始化一个长度为3的内存数组
      uint[] memory values = new uint[](3);
      // 赋值
      values.push(1);
      values.push(2);
      values.push(3);
      // 返回数组
      return values;
    }

> 注意：内存数组 必须 用长度参数（在本例中为3）创建。目前不支持 array.push()之类的方法调整数组大小，在未来的版本可能会支持长度修改。

### 8. for循环
语法在 Solidity 和 JavaScript 中类似

    function getEvens() pure external returns(uint[]) {
      uint[] memory evens = new uint[](5);
      // 在新数组中记录序列号
      uint counter = 0;
      // 在循环从1迭代到10：
      for (uint i = 1; i <= 10; i++) {
        // 如果 `i` 是偶数...
        if (i % 2 == 0) {
          // 把它加入偶数数组
          evens[counter] = i;
          //索引加一， 指向下一个空的‘even’
          counter++;
        }
      }
      return evens;
    }


# Solidity 进阶(二)


### 1. 可支付


函数修饰符：

1. 我们有决定函数何时和被谁调用的可见性修饰符: private 意味着它只能被合约内部调用； internal 就像 private 但是也能被继承的合约调用； external 只能从合约外部调用；最后 public 可以在任何地方调用，不管是内部还是外部。

2. 我们也有状态修饰符， 告诉我们函数如何和区块链交互: view 告诉我们运行这个函数不会更改和保存任何数据； pure 告诉我们这个函数不但不会往区块链写数据，它甚至不从区块链读取数据。这两种在被从合约外部调用的时候都不花费任何gas（但是它们在被内部其他函数调用的时候将会耗费gas）。

3. 然后我们有了自定义的 modifiers，例如 onlyOwner 和 aboveLevel。 对于这些修饰符我们可以自定义其对函数的约束逻辑。

    
    function test() external view onlyOwner anotherModifier { /* ... */ }
    
### payable 修饰符

payable方法： 一种可以接收以太的特殊函数

> 在以太坊中， 因为钱 (以太), 数据 (事务负载)， 以及合约代码本身都存在于以太坊。你可以在同时调用函数 并付钱给另外一个合约

    contract OnlineStore {
      function buySomething() external payable {
        // 检查以确定0.001以太发送出去来运行函数:
        require(msg.value == 0.001 ether);
        // 如果为真，一些用来向函数调用者发送数字内容的逻辑
        transferThing(msg.sender);
      }
    }

msg.value 是一种可以查看向合约发送了多少以太的方法，另外 ether 是一个內建单元

    // 假设 `OnlineStore` 在以太坊上指向你的合约:
    OnlineStore.buySomething().send(from: web3.eth.defaultAccount, value: web3.utils.toWei(0.001))

> 注意： 如果一个函数没标记为payable， 而你尝试利用上面的方法发送以太，函数将拒绝你的事务

### 2. 提现

写一个函数来从合约中提现以太

    contract GetPaid is Ownable {
      function withdraw() external onlyOwner {
        owner.transfer(this.balance);
      }
    }
    
> 注意我们使用 Ownable 合约中的 owner 和 onlyOwner. 可以通过 transfer 函数向一个地址发送以太， 然后 this.balance 将返回当前合约存储了多少以太



可以通过 transfer 向任何以太坊地址付钱。 比如，你可以有一个函数在 msg.sender 超额付款的时候给他们退钱：


### 3. 随机数

用 keccak256 来制造随机数

    // 生成一个0到100的随机数:
    uint randNonce = 0;
    uint random = uint(keccak256(now, msg.sender, randNonce)) % 100;
    randNonce++;
    uint random2 = uint(keccak256(now, msg.sender, randNonce)) % 100;

这个方法很容易被不诚实的节点攻击

> 在以太坊上, 当你在和一个合约上调用函数的时候, 你会把它广播给一个节点或者在网络上的 transaction 节点们。 网络上的节点将收集很多事务, 试着成为第一个解决计算密集型数学问题的人，作为“工作证明”，然后将“工作证明”(Proof of Work, PoW)和事务一起作为一个 block 发布在网络上。

一旦一个节点解决了一个PoW, 其他节点就会停止尝试解决这个 PoW, 并验证其他节点的事务列表是有效的，然后接受这个节点转而尝试解决下一个节点。

这就让我们的随机数函数变得可利用了

> 假设我们有一个硬币翻转合约——正面你赢双倍钱，反面你输掉所有的钱。假如它使用上面的方法来决定是正面还是反面 (random >= 50 算正面, random < 50 算反面)。

如果我正运行一个节点，我可以 只对我自己的节点 发布一个事务，且不分享它。 我可以运行硬币翻转方法来偷窥我的输赢 — 如果我输了，我就不把这个事务包含进我要解决的下一个区块中去。我可以一直运行这个方法，直到我赢得了硬币翻转并解决了下一个区块，然后获利。

> [如何在以太坊上安全地生成随机数呢?](https://ethereum.stackexchange.com/questions/191/how-can-i-securely-generate-a-random-number-in-my-smart-contract)

尽管这个方法在以太坊上不安全，在实际中，除非我们的随机函数有一大笔钱在上面，你游戏的用户一般是没有足够的资源去攻击的


###  优化合约

- require 的预检查部分，可以放在 modifier（函数修饰符里）
- 节省gas：  Struct  view/pure  减少写入存储 
- 安全  owner 

