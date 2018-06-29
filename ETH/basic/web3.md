

# Web3.js （一）

以太坊网络是由节点组成的，每一个节点都包含了区块链的一份拷贝。当你想要调用一份智能合约的一个方法，你需要从其中一个节点中查找并告诉它:

> 1. 智能合约的地址
> 1. 你想调用的方法
> 1. 你想传入那个方法的参数

以太坊节点只能识别一种叫做 JSON-RPC 的语言。当你你想调用一个合约的方法的时候，需要发送的查询语句类似：

    {"jsonrpc":"2.0","method":"eth_sendTransaction","params":[{"from":"0xb60e8dd61c5d32be8058bb8eb970870f07233155","to":"0xd46e8dd67c5d32be8058bb8eb970870f07244567","gas":"0x76c0","gasPrice":"0x9184e72a000","value":"0x9184e72a","data":"0xd46e8dd67c5d32be8d46e8dd67c5d32be8058bb8eb970870f072445675058bb8eb970870f072445675"}],"id":1}

Web3.js 把这些令人讨厌的查询语句都隐藏起来了


### 1. Web3 Provider

以太坊是由共享同一份数据的相同拷贝的 节点 构成的。 在 Web3.js 里设置 Web3 的 Provider 告诉我们的代码应该和 哪个节点 交互。

可以运行你自己的以太坊节点来作为 provider。 不过，有一个第三方的服务，可以让你的生活变得轻松点，让你不必为了给你的用户提供DApp而维护一个以太坊节点— Infura.

### Infura

[Infura](https://infura.io/) 是一个服务，它维护了很多以太坊节点并提供了一个缓存层来实现高速读取。你可以用他们的 API 来免费访问这个服务。 用 Infura 作为节点提供者，你可以不用自己运营节点就能很可靠地向以太坊发送、接收信息。

可以这样来把 Infura 作为你的 Web3 节点提供者：

    var web3 = new Web3(new Web3.providers.WebsocketProvider("wss://mainnet.infura.io/ws"));

因为我们的 DApp 将被很多人使用，这些用户不单会从区块链读取信息，还会向区块链 写 入信息，我们需要用一个方法让用户可以用他们的私钥给事务签名。

> 注意: 以太坊 (以及通常意义上的 blockchains )使用一个公钥/私钥对来对给事务做数字签名。把它想成一个数字签名的异常安全的密码。这样当我修改区块链上的数据的时候，我可以用我的公钥来 证明 我就是签名的那个。但是因为没人知道我的私钥，所以没人能伪造我的事务。

加密学非常复杂，所以除非你是个专家并且的确知道自己在做什么，你最好不要在你应用的前端中管理你用户的私钥。

### Metamask

[Metamask](https://metamask.io/) 是 Chrome 和 Firefox 的浏览器扩展， 它能让用户安全地维护他们的以太坊账户和私钥， 并用他们的账户和网站互动来使用 Web3.js。（如果你还没用过它，你肯定会想去安装的——这样你的浏览器就能使用 Web3.js 了，然后你就可以和任何与以太坊区块链通信的网站交互了）

> 注意: Metamask 默认使用 Infura 的服务器做为 web3 提供者。 就像我们上面做的那样。不过它还为用户提供了选择他们自己 Web3 提供者的选项。所以使用 Metamask 的 web3 提供者，你就给了用户选择权，而自己无需操心这一块。


### 使用 Metamask 的 web3 提供者

Metamask 把它的 web3 提供者注入到浏览器的全局 JavaScript对象web3中。所以你的应用可以检查 web3 是否存在。若存在就使用 web3.currentProvider 作为它的提供者。

    window.addEventListener('load', function() {
    
      // 检查web3是否已经注入到(Mist/MetaMask)
      if (typeof web3 !== 'undefined') {
        // 使用 Mist/MetaMask 的提供者
        web3js = new Web3(web3.currentProvider);
      } else {
        // 处理用户没安装的情况， 比如显示一个消息
        // 告诉他们要安装 MetaMask 来使用我们的应用
      }
    
      // 现在你可以启动你的应用并自由访问 Web3.js:
      startApp()
    
    })

> 注意: 除了MetaMask，你的用户也可能在使用其他他的私钥管理应用，比如 Mist 浏览器。不过，它们都实现了相同的模式来注入 web3 变量。所以我这里描述的方法对两者是通用的


### 2.与合约对话

Web3.js 需要两个东西来和你的合约对话: 它的 地址 和它的 ABI。

> 部署智能合约到以太坊后，将获得一个以太坊上的永久地址。

> ABI 意为应用二进制接口（Application Binary Interface）。 基本上，它是以 JSON 格式表示合约的方法，告诉 Web3.js 如何以合同理解的方式格式化函数调用。

实例化 Web3.js

    var myContract = new web3js.eth.Contract(myABI, myContractAddress);

Web3.js 有两个方法来调用我们合约的函数: call 和 send

### Call

call 用来调用 view 和 pure 函数。它只运行在本地节点，不会在区块链上创建事务

> view 和 pure 函数是只读的并不会改变区块链的状态。它们也不会消耗任何gas。用户也不会被要求用MetaMask对事务签名。

### Send

send 将创建一个事务并改变区块链上的数据。你需要用 send 来调用任何非 view 或者 pure 的函数。

> 注意: send 一个事务将要求用户支付gas，并会要求弹出对话框请求用户使用 Metamask 对事务签名。在我们使用 Metamask 作为我们的 web3 提供者的时候，所有这一切都会在我们调用 send() 的时候自动发生。

    myContract.methods.myMethod(123).call()
    myContract.methods.myMethod(123).send()

> 在 Solidity 里，当你定义一个 public变量的时候， 它将自动定义一个公开的 "getter" 同名方法


### 3. Metaask 和账户

MetaMask 允许用户在扩展中管理多个账户。

    var userAccount = web3.eth.accounts[0]

因为用户可以随时在 MetaMask 中切换账户，我们的应用需要监控这个变量，一旦改变就要相应更新界面

    var accountInterval = setInterval(function() {
      // 检查账户是否切换
      if (web3.eth.accounts[0] !== userAccount) {
        userAccount = web3.eth.accounts[0];
        // 调用一些方法来更新界面
        updateInterface();
      }
    }, 100);


### 4. 发送事务

> 1. send 一个事务需要一个 from 地址来表明谁在调用这个函数（也就是你 Solidity 代码里的 msg.sender )。 我们需要这是我们 DApp 的用户，这样一来 MetaMask 才会弹出提示让他们对事务签名。
>
> 1. send 一个事务将花费 gas
>
> 1. 在用户 send 一个事务 到 该事务对区块链产生实际影响之间有个延迟。这是因为我们必须等待事务被包含进一个区块里，以太坊上一个区块的时间平均下来是15秒左右。如果当前在以太坊上有大量挂起事务或者用户发送了过低的 gas 价格，我们的事务可能需要等待数个区块才能被包含进去，往往可能花费数分钟。所以在我们的代码中我们需要编写逻辑来处理这部分异步特性。

send 一个事务到我们的 Web3 提供者，然后链式添加一些事件监听:

- receipt 将在合约被包含进以太坊区块上以后被触发，这意味着僵尸被创建并保存进我们的合约了。
- error 将在事务未被成功包含进区块后触发，比如用户未支付足够的 gas。我们需要在界面中通知用户事务失败以便他们可以再次尝试。

> 注意:你可以在调用 send 时选择指定 gas 和 gasPrice， 例如： .send({ from: userAccount, gas: 3000000 })。如果你不指定，MetaMask 将让用户自己选择数值。


### 5. payable 函数

调用 Payable 函数需要指定发送多少 wei，而不是以太。

> 一个 wei 是以太的最小单位 — 1 ether 等于 10^18 wei

    // 把 1 ETH 转换成 Wei
    web3js.utils.toWei("1", "ether");

    // 合约内
    function levelUp(uint _zombieId) external payable {
      require(msg.value == levelUpFee);
      zombies[_zombieId].level++;
    }
    
    // web3调用
    CryptoZombies.methods.levelUp(zombieId).send({ from: userAccount, value: web3js.utils.toWei("0.001","ether") })

### 6. 订阅事件

订阅合约事件

    event NewZombie(uint zombieId, string name, uint dna);

在 Web3.js里， 你可以 订阅 一个事件，这样你的 Web3 提供者可以在每次事件发生后触发你的一些代码逻辑：

    cryptoZombies.events.NewZombie()
    .on("data", function(event) {
      let zombie = event.returnValues;
      console.log("一个新僵尸诞生了！", zombie.zombieId, zombie.name, zombie.dna);
    }).on('error', console.error);

### 使用 indexed

为了筛选仅和当前用户相关的事件，我们的 Solidity 合约将必须使用 indexed 关键字，就像我们在 ERC721 实现中的Transfer 事件中那样：

    event Transfer(address indexed _from, address indexed _to, uint256 _tokenId);

在这种情况下， 因为_from 和 _to 都是 indexed，这就意味着我们可以在前端事件监听中过滤事件

    cryptoZombies.events.Transfer({ filter: { _to: userAccount } })
    .on("data", function(event) {
      let data = event.returnValues;
      // 当前用户更新了一个僵尸！更新界面来显示
    }).on('error', console.error);

### 查询过去的事件

可以用 getPastEvents 查询过去的事件，并用过滤器 fromBlock 和 toBlock 给 Solidity 一个事件日志的时间范围("block" 在这里代表以太坊区块编号）：

    cryptoZombies.getPastEvents("NewZombie", { fromBlock: 0, toBlock: 'latest' })
    .then(function(events) {
      // events 是可以用来遍历的 `event` 对象 
      // 这段代码将返回给我们从开始以来创建的僵尸列表
    });

可以用这个方法来查询从最开始起的事件日志，这就有了一个非常有趣的用例： 用事件来作为一种更便宜的存储。

> 短板是，事件不能从智能合约本身读取。但是，如果你有一些数据需要永久性地记录在区块链中以便可以在应用的前端中读取，这将是一个很好的用例。这些数据不会影响智能合约向前的状态。

### Web3.js 事件 和 MetaMask

MetaMask 尚且不支持最新的事件 API (尽管如此，他们已经在实现这部分功能了，  [查看进度](https://github.com/MetaMask/metamask-extension/issues/3642))

可以使用单独 Web3 提供者，它针对事件提供了WebSockets支持。 我们可以用 Infura 来像实例化第二份拷贝：

    Infura = new Web3(new Web3.providers.WebsocketProvider("wss://mainnet.infura.io/ws"));
    var czEvents = new web3Infura.eth.Contract(cryptoZombiesABI, cryptoZombiesAddress);

将使用 czEvents.events.Transfer 来监听事件，而不再使用 cryptoZombies.events.Transfer



