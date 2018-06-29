
# 深入Solidity

### 1. 代币

> 一个 代币 在以太坊基本上就是一个遵循一些共同规则的智能合约——即它实现了所有其他代币合约共享的一组标准函数，例如 transfer(address _to, uint256 _value) 和 balanceOf(address _owner)

所有 ERC20 代币共享具有相同名称的同一组函数，它们都可以以相同的方式进行交互.

这意味着如果你构建的应用程序能够与一个 ERC20 代币进行交互，那么它就也能够与任何 ERC20 代币进行交互。 这样一来，将来你就可以轻松地将更多的代币添加到你的应用中，而无需进行自定义编码。

其中一个例子就是交易所。 当交易所添加一个新的 ERC20 代币时，实际上它只需要添加与之对话的另一个智能合约。 用户可以让那个合约将代币发送到交易所的钱包地址，然后交易所可以让合约在用户要求取款时将代币发送回给他们

> 交易所只需要实现这种转移逻辑一次，然后当它想要添加一个新的 ERC20 代币时，只需将新的合约地址添加到它的数据库即可。

ERC721 代币是不能互换的，因为每个代币都被认为是唯一且不可分割的。 你只能以整个单位交易它们，并且每个单位都有唯一的 ID。 

    contract ERC721 {
      event Transfer(address indexed _from, address indexed _to, uint256 _tokenId);
      event Approval(address indexed _owner, address indexed _approved, uint256 _tokenId);
    
      function balanceOf(address _owner) public view returns (uint256 _balance);
      function ownerOf(uint256 _tokenId) public view returns (address _owner);
      function transfer(address _to, uint256 _tokenId) public;
      function approve(address _to, uint256 _tokenId) public;
      function takeOwnership(uint256 _tokenId) public;
    }

> 注意： ERC721目前是一个 草稿，还没有正式商定的实现。

实现 ERC721:

    contract ZombieOwnership is ZombieAttack, ERC721 {
    
      mapping (uint => address) zombieApprovals;
    
      function balanceOf(address _owner) public view returns (uint256 _balance) {
        return ownerZombieCount[_owner];
      }
    
      function ownerOf(uint256 _tokenId) public view returns (address _owner) {
        return zombieToOwner[_tokenId];
      }
    
      function _transfer(address _from, address _to, uint256 _tokenId) private {
        ownerZombieCount[_to]++;
        ownerZombieCount[_from]--;
        zombieToOwner[_tokenId] = _to;
        Transfer(_from, _to, _tokenId);
      }
    
      function transfer(address _to, uint256 _tokenId) public onlyOwnerOf(_tokenId) {
        _transfer(msg.sender, _to, _tokenId);
      }
    
      function approve(address _to, uint256 _tokenId) public onlyOwnerOf(_tokenId) {
        zombieApprovals[_tokenId] = _to;
        Approval(msg.sender, _to, _tokenId);
      }
    
      function takeOwnership(uint256 _tokenId) public {
        require(zombieApprovals[_tokenId] == msg.sender);
        address owner = ownerOf(_tokenId);
        _transfer(owner, msg.sender, _tokenId);
      }
    }

### 2. 预防溢出

需要些额外的检查，来确保用户不会不小心把他们的僵尸转移给0 地址（这被称作 “烧币”, 基本上就是把代币转移到一个谁也没有私钥的地址，让这个代币永远也无法恢复）

合约安全增强: 溢出(overflow)和下溢(underflow)

溢出：

    uint8 number = 255;
    number++;  // 0

下溢：
    
    uint8 number = 0;
    number--;  // 255

使用 SafeMath

> OpenZeppelin 建立了一个叫做 SafeMath 的 库(library)，默认情况下可以防止这些问题。

一个库 是 Solidity 中一种特殊的合约。其中一个有用的功能是给原始数据类型增加一些方法。

SafeMath 库有四个方法 — add， sub， mul， div

    using SafeMath for uint256;
    
    uint256 a = 5;
    uint256 b = a.add(3); // 5 + 3 = 8
    uint256 c = a.mul(2); // 5 * 2 = 10


SafeMath 的部分代码:

    library SafeMath {
    
      function mul(uint256 a, uint256 b) internal pure returns (uint256) {
        if (a == 0) {
          return 0;
        }
        uint256 c = a * b;
        assert(c / a == b);
        return c;
      }
    
      function div(uint256 a, uint256 b) internal pure returns (uint256) {
        // assert(b > 0); // Solidity automatically throws when dividing by 0
        uint256 c = a / b;
        // assert(a == b * c + a % b); // There is no case in which this doesn't hold
        return c;
      }
    
      function sub(uint256 a, uint256 b) internal pure returns (uint256) {
        assert(b <= a);
        return a - b;
      }
    
      function add(uint256 a, uint256 b) internal pure returns (uint256) {
        uint256 c = a + b;
        assert(c >= a);
        return c;
      }
    }
    
### library 关键字 — 库

和 合约很相似，但是又有一些不同。 就我们的目的而言，库允许我们使用 using 关键字，它可以自动把库的所有方法添加给一个数据类型：

    using SafeMath for uint;
    // 这下我们可以为任何 uint 调用这些方法了
    uint test = 2;
    test = test.mul(3); // test 等于 6 了
    test = test.add(5); // test 等于 11 了
    
> assert 和 require 相似，若结果为否它就会抛出错误。 assert 和 require 区别在于，require 若失败则会返还给用户剩下的 gas， assert 则不会。所以大部分情况下，你写代码的时候会比较喜欢 require，assert 只在代码可能出现严重错误的时候使用，比如 uint 溢出。


### 3. 注释

Solidity 社区所使用的一个标准是使用一种被称作 natspec 的格式

    /// @title 一个简单的基础运算合约
    /// @author H4XF13LD MORRIS
    /// @notice 现在，这个合约只添加一个乘法
    contract Math {
      /// @notice 两个数相乘
      /// @param x 第一个 unit
      /// @param y  第二个 uint
      /// @return z  (x * y) 的结果
      /// @dev 现在这个方法不检查溢出
      function multiply(uint x, uint y) returns (uint z) {
        // 这只是个普通的注释，不会被 natspec 解释
        z = x * y;
      }
    }

> @title（标题） 和 @author （作者）很直接了.
> 
> @notice （须知）向 用户 解释这个方法或者合约是做什么的。 @dev （开发者） 是向开发者解释更多的细节。
> 
> @param （参数）和 @return （返回） 用来描述这个方法需要传入什么参数以及返回什么值。


<br/>
<br/>


# ERC20实现 简单实例

```
pragma solidity ^0.4.4;

contract Token {

    /// @return total amount of tokens
    function totalSupply() constant returns (uint256 supply) {}

    /// @param _owner The address from which the balance will be retrieved
    /// @return The balance
    function balanceOf(address _owner) constant returns (uint256 balance) {}

    /// @notice send `_value` token to `_to` from `msg.sender`
    /// @param _to The address of the recipient
    /// @param _value The amount of token to be transferred
    /// @return Whether the transfer was successful or not
    function transfer(address _to, uint256 _value) returns (bool success) {}

    /// @notice send `_value` token to `_to` from `_from` on the condition it is approved by `_from`
    /// @param _from The address of the sender
    /// @param _to The address of the recipient
    /// @param _value The amount of token to be transferred
    /// @return Whether the transfer was successful or not
    function transferFrom(address _from, address _to, uint256 _value) returns (bool success) {}

    /// @notice `msg.sender` approves `_addr` to spend `_value` tokens
    /// @param _spender The address of the account able to transfer the tokens
    /// @param _value The amount of wei to be approved for transfer
    /// @return Whether the approval was successful or not
    function approve(address _spender, uint256 _value) returns (bool success) {}

    /// @param _owner The address of the account owning tokens
    /// @param _spender The address of the account able to transfer the tokens
    /// @return Amount of remaining tokens allowed to spent
    function allowance(address _owner, address _spender) constant returns (uint256 remaining) {}

    event Transfer(address indexed _from, address indexed _to, uint256 _value);
    event Approval(address indexed _owner, address indexed _spender, uint256 _value);
    
}



contract StandardToken is Token {

    function transfer(address _to, uint256 _value) returns (bool success) {
        //Default assumes totalSupply can't be over max (2^256 - 1).
        //If your token leaves out totalSupply and can issue more tokens as time goes on, you need to check if it doesn't wrap.
        //Replace the if with this one instead.
        //if (balances[msg.sender] >= _value && balances[_to] + _value > balances[_to]) {
        if (balances[msg.sender] >= _value && _value > 0) {
            balances[msg.sender] -= _value;
            balances[_to] += _value;
            Transfer(msg.sender, _to, _value);
            return true;
        } else { return false; }
    }

    function transferFrom(address _from, address _to, uint256 _value) returns (bool success) {
        //same as above. Replace this line with the following if you want to protect against wrapping uints.
        //if (balances[_from] >= _value && allowed[_from][msg.sender] >= _value && balances[_to] + _value > balances[_to]) {
        if (balances[_from] >= _value && allowed[_from][msg.sender] >= _value && _value > 0) {
            balances[_to] += _value;
            balances[_from] -= _value;
            allowed[_from][msg.sender] -= _value;
            Transfer(_from, _to, _value);
            return true;
        } else { return false; }
    }

    function balanceOf(address _owner) constant returns (uint256 balance) {
        return balances[_owner];
    }

    function approve(address _spender, uint256 _value) returns (bool success) {
        allowed[msg.sender][_spender] = _value;
        Approval(msg.sender, _spender, _value);
        return true;
    }

    function allowance(address _owner, address _spender) constant returns (uint256 remaining) {
      return allowed[_owner][_spender];
    }

    mapping (address => uint256) balances;
    mapping (address => mapping (address => uint256)) allowed;
    uint256 public totalSupply;
}


//name this contract whatever you'd like
contract ERC20Token is StandardToken {

    function () {
        //if ether is sent to this address, send it back.
        throw;
    }

    /* Public variables of the token */

    /*
    NOTE:
    The following variables are OPTIONAL vanities. One does not have to include them.
    They allow one to customise the token contract & in no way influences the core functionality.
    Some wallets/interfaces might not even bother to look at this information.
    */
    string public name;                   //fancy name: eg Simon Bucks
    uint8 public decimals;                //How many decimals to show. ie. There could 1000 base units with 3 decimals. Meaning 0.980 SBX = 980 base units. It's like comparing 1 wei to 1 ether.
    string public symbol;                 //An identifier: eg SBX
    string public version = 'H1.0';       //human 0.1 standard. Just an arbitrary versioning scheme.

//
// CHANGE THESE VALUES FOR YOUR TOKEN
//

//make sure this function name matches the contract name above. So if you're token is called TutorialToken, make sure the //contract name above is also TutorialToken instead of ERC20Token

    function ERC20Token() {
        balances[msg.sender] = 1*10**28;               // Give the creator all initial tokens 
        totalSupply = 1*10**28;   // Update total supply 
        name = "SuperToken";        // Set the name for display purposes
        decimals = 18;           // Amount of decimals for display purposes
        symbol = "ST";         // Set the symbol for display purposes
    }

    /* Approves and then calls the receiving contract */
    function approveAndCall(address _spender, uint256 _value, bytes _extraData) returns (bool success) {
        allowed[msg.sender][_spender] = _value;
        Approval(msg.sender, _spender, _value);

        //call the receiveApproval function on the contract you want to be notified. This crafts the function signature manually so one doesn't have to include a contract in here just for this.
        //receiveApproval(address _from, uint256 _value, address _tokenContract, bytes _extraData)
        //it is assumed that when does this that the call *should* succeed, otherwise one would use vanilla approve instead.
        if(!_spender.call(bytes4(bytes32(sha3("receiveApproval(address,uint256,address,bytes)"))), msg.sender, _value, this, _extraData)) { throw; }
        return true;
    }
}

```

官方新版ERC20模板：

    pragma solidity ^0.4.16;
    
    interface tokenRecipient { function receiveApproval(address _from, uint256 _value, address _token, bytes _extraData) external; }
    
    contract TokenERC20 {
        // Public variables of the token
        string public name;
        string public symbol;
        uint8 public decimals = 18;
        // 18 decimals is the strongly suggested default, avoid changing it
        uint256 public totalSupply;
    
        // This creates an array with all balances
        mapping (address => uint256) public balanceOf;
        mapping (address => mapping (address => uint256)) public allowance;
    
        // This generates a public event on the blockchain that will notify clients
        event Transfer(address indexed from, address indexed to, uint256 value);
    
        // This notifies clients about the amount burnt
        event Burn(address indexed from, uint256 value);
    
        /**
         * Constructor function
         *
         * Initializes contract with initial supply tokens to the creator of the contract
         */
        function TokenERC20(
            uint256 initialSupply,
            string tokenName,
            string tokenSymbol
        ) public {
            totalSupply = initialSupply * 10 ** uint256(decimals);  // Update total supply with the decimal amount
            balanceOf[msg.sender] = totalSupply;                // Give the creator all initial tokens
            name = tokenName;                                   // Set the name for display purposes
            symbol = tokenSymbol;                               // Set the symbol for display purposes
        }
    
        /**
         * Internal transfer, only can be called by this contract
         */
        function _transfer(address _from, address _to, uint _value) internal {
            // Prevent transfer to 0x0 address. Use burn() instead
            require(_to != 0x0);
            // Check if the sender has enough
            require(balanceOf[_from] >= _value);
            // Check for overflows
            require(balanceOf[_to] + _value >= balanceOf[_to]);
            // Save this for an assertion in the future
            uint previousBalances = balanceOf[_from] + balanceOf[_to];
            // Subtract from the sender
            balanceOf[_from] -= _value;
            // Add the same to the recipient
            balanceOf[_to] += _value;
            emit Transfer(_from, _to, _value);
            // Asserts are used to use static analysis to find bugs in your code. They should never fail
            assert(balanceOf[_from] + balanceOf[_to] == previousBalances);
        }
    
        /**
         * Transfer tokens
         *
         * Send `_value` tokens to `_to` from your account
         *
         * @param _to The address of the recipient
         * @param _value the amount to send
         */
        function transfer(address _to, uint256 _value) public {
            _transfer(msg.sender, _to, _value);
        }
    
        /**
         * Transfer tokens from other address
         *
         * Send `_value` tokens to `_to` on behalf of `_from`
         *
         * @param _from The address of the sender
         * @param _to The address of the recipient
         * @param _value the amount to send
         */
        function transferFrom(address _from, address _to, uint256 _value) public returns (bool success) {
            require(_value <= allowance[_from][msg.sender]);     // Check allowance
            allowance[_from][msg.sender] -= _value;
            _transfer(_from, _to, _value);
            return true;
        }
    
        /**
         * Set allowance for other address
         *
         * Allows `_spender` to spend no more than `_value` tokens on your behalf
         *
         * @param _spender The address authorized to spend
         * @param _value the max amount they can spend
         */
        function approve(address _spender, uint256 _value) public
            returns (bool success) {
            allowance[msg.sender][_spender] = _value;
            return true;
        }
    
        /**
         * Set allowance for other address and notify
         *
         * Allows `_spender` to spend no more than `_value` tokens on your behalf, and then ping the contract about it
         *
         * @param _spender The address authorized to spend
         * @param _value the max amount they can spend
         * @param _extraData some extra information to send to the approved contract
         */
        function approveAndCall(address _spender, uint256 _value, bytes _extraData)
            public
            returns (bool success) {
            tokenRecipient spender = tokenRecipient(_spender);
            if (approve(_spender, _value)) {
                spender.receiveApproval(msg.sender, _value, this, _extraData);
                return true;
            }
        }
    
        /**
         * Destroy tokens
         *
         * Remove `_value` tokens from the system irreversibly
         *
         * @param _value the amount of money to burn
         */
        function burn(uint256 _value) public returns (bool success) {
            require(balanceOf[msg.sender] >= _value);   // Check if the sender has enough
            balanceOf[msg.sender] -= _value;            // Subtract from the sender
            totalSupply -= _value;                      // Updates totalSupply
            emit Burn(msg.sender, _value);
            return true;
        }
    
        /**
         * Destroy tokens from other account
         *
         * Remove `_value` tokens from the system irreversibly on behalf of `_from`.
         *
         * @param _from the address of the sender
         * @param _value the amount of money to burn
         */
        function burnFrom(address _from, uint256 _value) public returns (bool success) {
            require(balanceOf[_from] >= _value);                // Check if the targeted balance is enough
            require(_value <= allowance[_from][msg.sender]);    // Check allowance
            balanceOf[_from] -= _value;                         // Subtract from the targeted balance
            allowance[_from][msg.sender] -= _value;             // Subtract from the sender's allowance
            totalSupply -= _value;                              // Update totalSupply
            emit Burn(_from, _value);
            return true;
        }
    }


<br/>
<br/>
<br/>