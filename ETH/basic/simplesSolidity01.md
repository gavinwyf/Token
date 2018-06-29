

# Solidity 基础（一）

所有的 Solidity 源码都必须冠以 "version pragma" — 标明 Solidity 编译器的版本. 以避免将来新的编译器可能破坏你的代码。

    pragma solidity ^0.4.19;
    
    contract MyContract {
    
    }

> Solidity中的 Contract 类似于面向对象语言的类。每个Contract都可以声明 状态变量、函数、函数修饰符、事件、结构类型和枚举类型 (State Variables, Functions, Function Modifiers, Events, Struct Types and Enum Types)。

### 1. 状态变量和整数

uint 无符号数据类型， 指其值不能是负数，对于有符号的整数存在名为 int 的数据类型

> 注: Solidity中， uint 实际上是 uint256代名词， 一个256位的无符号整数。你也可以定义位数少的uints — uint8， uint16， uint32， 等…… 但一般来讲你愿意使用简单的 uint， 除非在某些特殊情况下，这我们后面会讲


### 2. 结构体

结构体允许你生成一个更复杂的数据类型，它有多个属性。

    struct Person {
      uint age;
      string name;
    }

> string 字符串用于保存任意长度的 UTF-8 编码数据

    Person[] public people;
    // 创建一个新的Person:
    Person satoshi = Person(172, "Satoshi");
    // 将新创建的satoshi添加进people数组:
    people.push(satoshi);

### 3. 数组

Solidity 支持两种数组: 静态 数组和动态 数组:

    // 固定长度为2的静态数组:
    uint[2] fixedArray;
    // 固定长度为5的string类型的静态数组:
    string[5] stringArray;
    // 动态数组，长度不固定，可以动态添加元素:
    uint[] dynamicArray;

结构体类型的数组：

    Person[] people; // dynamic Array, we can keep adding to it

> 状态变量被永久保存在区块链中。所以在你的合约中创建动态数组来保存成结构的数据是非常有意义的。

定义 public 数组, Solidity 会自动创建 getter 方法

    Person[] public people;
    
> 其它的合约可以从这个数组读取数据（但不能写入数据），所以这在合约中是一个有用的保存公共数据的模式。

### 4. 函数

    function eatHamburgers(string _name, uint _amount) {
    
    }

> 习惯上函数里的变量都是以(_)开头 (但不是硬性规定) 以区别全局变量

    function (<parameter types>) {internal|external} [pure|constant|view|payable] [returns (<return types>)]

Solidity 定义的函数的属性默认为public。 这就意味着任何一方 (或其它合约) 都可以调用你合约里的函数。

#### 定义私有函数

    function _addToArray(uint _number) private {
      numbers.push(_number);
    }

> 私有函数的名字用( _ )起始

#### 返回值

    function sayHello() public returns (string) {
      return ‘哈哈哈’;
    }


#### 函数修饰符

可以把函数定义为 view, 意味着它只能读取数据不能更改数据

    function sayHello() public view returns (string) {}

Solidity 还支持 pure 函数（纯函数）, 表明这个函数甚至都不读取应用里的数据（状态）

    function _multiply(uint a, uint b) private pure returns (uint) {
      return a * b;
    }
    
由于结构体的存储指针可以以参数的方式传递给一个 private 或 internal 的函数，因此结构体可以在多个函数之间相互传递

    function _doStuff(Dog storage _dog) private {
      // do stuff with _dog
    }

### 5. Keccak256

Ethereum 内部有一个散列函数keccak256，它用了SHA3版本。一个散列函数基本上就是把一个字符串转换为一个256位的16进制数字。字符串的一个微小变化会引起散列数据极大变化。

    //6e91ec6b618bb462a4a6ee5aa2cb0e9cf30f7a052bb467b0ba58b8748c00d2e5
    keccak256("aaaab");
    //b1f078126895a1424524de5321b339ab00408010b7cf0e6ed451514981e58aa9
    keccak256("aaaac");

>注: 在区块链中安全地产生一个随机数是一个很难的问题， 本例方法不安全.

### 6. 类型转换

    uint8 a = 5;
    uint b = 6;
    // 将会抛出错误，因为 a * b 返回 uint, 而不是 uint8:
    uint8 c = a * b;
    // 我们需要将 b 转换为 uint8:
    uint8 c = a * uint8(b);

### 7. 事件

事件 是合约和区块链通讯的一种机制。你的前端应用“监听”某些事件，并做出反应。

    // 这里建立事件
    event IntegersAdded(uint x, uint y, uint result);
    
    function add(uint _x, uint _y) public {
      uint result = _x + _y;
      //触发事件，通知app
      IntegersAdded(_x, _y, result);
      return result;
    }
    
你的 app 前端可以监听这个事件。JavaScript 实现如下:

    YourContract.IntegersAdded(function(error, result) { 
      // 干些事
    }

### 8. 枚举类型

    enum State { Created, Locked, Inactive } // Enum

例子：

    pragma solidity ^0.4.19;
    
    contract DogFactory {
    
        // 建立事件
        event NewEvent(uint dogId, string name, uint dna);
        
        uint dnaDigits = 16;
        uint dnaModulus = 10 ** dnaDigits;
    
        struct Dog {
            string name;
            uint dna;
        }
    
        Dog[] public dogs;
    
        function _createDog(string _name, uint _dna) private {
            uint id = dogs.push(Dog(_name, _dna)) - 1;
            NewEvent(id, _name, _dna);
        }
    
        function _generateRandomDna(string _str) private view returns (uint) {
            uint rand = uint(keccak256(_str));
            return rand % dnaModulus;
        }
    
        function createRandomDog(string _name) public {
            uint randDna = _generateRandomDna(_name);
            _createDog(_name, randDna);
        }
    
    }


# Solidity基础（二）


### 1. Address 地址

以太坊区块链由 account (账户)组成，你可以把它想象成银行账户。一个帐户的余额是 以太 （在以太坊区块链上使用的币种），你可以和其他帐户之间支付和接受以太币，就像你的银行帐户可以电汇资金到其他银行帐户一样。

> 每个帐户都有一个“地址”，你可以把它想象成银行账号。这是账户唯一的标识符. 地址属于特定用户（或智能合约）

### 2. Mapping 映射

    //对于金融应用程序，将用户的余额保存在一个 uint类型的变量中：
    mapping (address => uint) public accountBalance;
    //或者可以用来通过userId 存储/查找的用户名
    mapping (uint => string) userIdToName;

映射本质上是存储和查找数据所用的键-值对

    mapping(_KeyType => _ValueType) 
    
> _KeyType 支持几乎任何类型（mapping, dynamically sized array, contract, enum, struct）
>
> _ValueType 任何类型

### 3. msg.sender

在 Solidity 中，有一些全局变量可以被所有函数调用。 其中一个就是 msg.sender，它指的是当前调用者（或智能合约）的 address

> 注意：在 Solidity 中，功能执行始终需要从外部调用者开始。 一个合约只会在区块链上什么也不做，除非有人调用其中的函数。所以 msg.sender总是存在的.

使用 msg.sender 来更新 mapping:

    mapping (address => uint) favoriteNumber;
    
    function setMyNumber(uint _myNumber) public {
      // 更新我们的 `favoriteNumber` 映射来将 `_myNumber`存储在 `msg.sender`名下
      favoriteNumber[msg.sender] = _myNumber;
      // 存储数据至映射的方法和将数据存储在数组相似
    }
    
    function whatIsMyNumber() public view returns (uint) {
      // 拿到存储在调用者地址名下的值
      // 若调用者还没调用 setMyNumber， 则值为 `0`
      return favoriteNumber[msg.sender];
    }

> 使用 msg.sender 很安全，因为它具有以太坊区块链的安全保障 —— 除非窃取与以太坊地址相关联的私钥，否则是没有办法修改其他人的数据的

[全局变量](http://solidity.readthedocs.io/en/v0.4.21/units-and-global-variables.html)：
> - block.blockhash(uint blockNumber) returns (bytes32): hash of the given block - only works for 256 most recent blocks excluding current
> - block.coinbase (address): current block miner’s address
> - block.difficulty (uint): current block difficulty
> - block.gaslimit (uint): current block gaslimit
> - block.number (uint): current block number
> - block.timestamp (uint): current block timestamp as seconds since unix epoch
> - gasleft() returns (uint256): remaining gas
> - msg.data (bytes): complete calldata
> - msg.gas (uint): remaining gas - deprecated in version 0.4.21 and to be replaced by gasleft()
> - msg.sender (address): sender of the message (current call)
> - msg.sig (bytes4): first four bytes of the calldata (i.e. function identifier)
> - msg.value (uint): number of wei sent with the message
> - now (uint): current block timestamp (alias for block.timestamp)
> - tx.gasprice (uint): gas price of the transaction
> - tx.origin (address): sender of the transaction (full call chain)


### 4. Require

require使得函数在执行过程中，当不满足某些条件时抛出错误，并停止执行。

    function sayHiToVitalik(string _name) public returns (string) {
      // 比较 _name 是否等于 "Vitalik". 如果不成立，抛出异常并终止程序
      require(keccak256(_name) == keccak256("Vitalik"));
      // 如果返回 true, 运行如下语句
      return "Hi!";
    }

> 敲黑板: Solidity 并不支持原生的字符串比较, 我们只能通过比较两字符串的 keccak256 哈希值来进行判断

### [错误处理](http://solidity.readthedocs.io/en/v0.4.21/control-structures.html#error-handling-assert-require-revert-and-exceptions) (Assert, Require, Revert)

- assert函数只能用于测试内部错误，并检查不变量
- require函数应用于确保有效条件（如输入或合同状态变量），或者验证从外部合同调用返回值
- revert函数 可用于标记错误并恢复当前的呼叫
- throw关键字也可以用作替代revert() 

> 从版本0.4.13开始，该throw关键字将被弃用，并将在未来逐步淘汰。

### 5. Inheritance 继承

合约 inheritance (继承)

    contract Doge {
      function catchphrase() public returns (string) {
        return "So Wow CryptoDoge";
      }
    }
    
    // BabyDoge可以访问 catchphrase和 anotherCatchphrase和其他在 Doge 中定义的公共函数
    contract BabyDoge is Doge {
      function anotherCatchphrase() public returns (string) {
        return "Such Moon BabyDoge";
      }
    }

> 可以用于逻辑继承（比如表达子类的时候，Cat 是一种 Animal）。 但也可以简单地将类似的逻辑组合到不同的合约中以组织代码

### 6. Import 引入

    import "./someothercontract.sol";
    
    contract newContract is SomeOtherContract {
    
    }

> 类似ES6 :
> - import "filename";
> - import * as symbolName from "filename";
> - import {symbol1 as alias, symbol2} from "filename";

> 不属于ES6的语法：
> import "filename" as symbolName; 等价于 import * as symbolName from "filename";


### 7. Storage 与 Memory

> - Storage 变量是指永久存储在区块链中的变量. 
> - Memory 变量则是临时的.

当外部函数对某合约调用完成时，内存型变量即被移除。 你可以把它想象成存储在你电脑的硬盘或是RAM中数据的关系。

默认情况下 Solidity 会自动处理它们。 状态变量（在函数之外声明的变量）默认为“存储”形式，并永久写入区块链；而在函数内部声明的变量是“内存”型的，它们函数调用结束后消失。

有一些情况下，你需要手动声明存储类型，主要用于处理函数内的 结构体 和 数组 时：

    contract SandwichFactory {
      struct Sandwich {
        string name;
        string status;
      }
    
      Sandwich[] sandwiches;
    
      function eatSandwich(uint _index) public {
        // Sandwich mySandwich = sandwiches[_index];
    
        // ^ 看上去很直接，不过 Solidity 将会给出警告
        // 告诉你应该明确在这里定义 `storage` 或者 `memory`。
    
        // 所以你应该明确定义 `storage`:
        Sandwich storage mySandwich = sandwiches[_index];
        // 这样 `mySandwich` 是指向 `sandwiches[_index]`的指针
        mySandwich.status = "Eaten!";
        // 这将永久把 `sandwiches[_index]` 变为区块链上的存储
    
        // 如果你只想要一个副本，可以使用`memory`:
        Sandwich memory anotherSandwich = sandwiches[_index + 1];
        // 这样 `anotherSandwich` 就仅仅是一个内存里的副本了
        anotherSandwich.status = "Eaten!";
        // 将仅仅修改临时变量，对 `sandwiches[_index + 1]` 没有任何影响
        // 不过你可以这样做:
        sandwiches[_index + 1] = anotherSandwich;
        // ...如果你想把副本的改动保存回区块链存储
      }
    }

### 8. 函数可见性

除 public 和 private 属性之外，Solidity 还使用了另外两个描述函数可见性的修饰词：internal（内部） 和 external（外部）

internal 和 private 类似，不过， 如果某个合约继承自其父合约，这个合约即可以访问父合约中定义的“内部”函数

external 与public 类似，只不过这些函数只能在合约之外调用 - 它们不能被合约内的其他函数调用

    contract Sandwich {
      uint private sandwichesEaten = 0;
    
      function eat() internal {
        sandwichesEaten++;
      }
    }
    
    contract BLT is Sandwich {
      uint private baconSandwichesEaten = 0;
    
      function eatWithBacon() public returns (string) {
        baconSandwichesEaten++;
        // 因为eat() 是internal 的，所以我们能在这里调用
        eat();
      }
    }

### 9. 和其他合约交互

如果我们的合约需要和区块链上的其他的合约会话，则需先定义一个 interface (接口)。

    contract LuckyNumber {
      mapping(address => uint) numbers;
    
      function setNum(uint _num) public {
        numbers[msg.sender] = _num;
      }
    
      function getNum(address _myAddress) public view returns (uint) {
        return numbers[_myAddress];
      }
    }
    // 定义 LuckyNumber 合约的 interface
    contract NumberInterface {
      function getNum(address _myAddress) public view returns (uint);
    }
    
    contract MyContract {
      address NumberInterfaceAddress = 0xab38...;
      // 这是FavoriteNumber合约在以太坊上的地址
      NumberInterface numberContract = NumberInterface(NumberInterfaceAddress);
      // 现在变量 `numberContract` 指向另一个合约对象
    
      function someFunction() public {
        // 可以调用在那个合约中声明的 `getNum`函数:
        uint num = numberContract.getNum(msg.sender);
        
      }
    }
        
> - 只声明了要与之交互的函数
> - 没有使用大括号 {  } 定义函数体，我们单单用分号（;）结束了函数声明。这使它看起来像一个合约框架

在 Solidity中，您可以让一个函数返回多个值

    function multipleReturns() internal returns(uint a, uint b, uint c) {
      return (1, 2, 3);
    }
    // 只想返回其中一个变量:
    function getLastReturnValue() external {
      uint c;
      // 可以对其他字段留空:
      (,,c) = multipleReturns();
      //(a, b, c) = multipleReturns();
    }

JS和Web3.js使用：

    var abi = /* abi generated by the compiler */
    var ZombieFeedingContract = web3.eth.contract(abi)
    var contractAddress = /* our contract address on Ethereum after deploying */
    var ZombieFeeding = ZombieFeedingContract.at(contractAddress)
    
    // 假设我们有我们的僵尸ID和要攻击的猫咪ID
    let zombieId = 1;
    let kittyId = 1;
    
    // 要拿到猫咪的DNA，我们需要调用它的API。这些数据保存在它们的服务器上而不是区块链上。
    // 如果一切都在区块链上，我们就不用担心它们的服务器挂了，或者它们修改了API，
    // 或者因为不喜欢我们的僵尸游戏而封杀了我们
    let apiUrl = "https://api.cryptokitties.co/kitties/" + kittyId
    $.get(apiUrl, function(data) {
      let imgUrl = data.image_url
      // 一些显示图片的代码
    })
    
    // 当用户点击一只猫咪的时候:
    $(".kittyImage").click(function(e) {
      // 调用我们合约的 `feedOnKitty` 函数
      ZombieFeeding.feedOnKitty(zombieId, kittyId)
    })
    
    // 侦听来自我们合约的新僵尸事件好来处理
    ZombieFactory.NewZombie(function(error, result) {
      if (error) return
      // 这个函数用来显示僵尸:
      generateZombie(result.zombieId, result.name, result.dna)
    })