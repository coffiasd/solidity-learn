# Solidity之地址(address)

# 概览

1.  什么是ETH地址？
1.  可支付地址
1.  地址对应的方法(solidity)
1.  检查地址是否存在
1.  零地址
1.  地址相关的web3.js

# 介绍

ETH中的地址都是唯一的，因为他们都是来自一个公钥或者合约。

在ETH交易中的支付环节，预期的收款人都是一个地址。类似于银行中的转账账号。

ETH中的地址(address)分为两种:

1.  Externally Owned Accounts (EOA)(外部拥有账户):这些账户通过私钥来控制。通过私钥可以控制账户中的TOKEN，在和智能合约进行交互的时候，私钥还可以用来提供身份认证。
1.  Contracts Accounts (Smart Contracts)(合约地址):这些账号是通过代码(solidity)控制的。和EOAs账号不同的是，智能合约账号是没有关联一个秘钥的。因为智能合约不受私钥控制，所以我们可以说智能合约是独立的。每个合约的地址都来自于合约创建的交易。ETH的合约地址可以作为交易中的接收者，发送token到合约或者调用别的合约中的方法。

# 什么是ETH地址(技术上)?

hash函数是把ETH公钥(或者合约)转化为地址的关键。ETH使用hash函数（keccak-256)来生成地址。在ETH或者solidity中，一个地址由20byte长度的值组成，这个地址对应于公钥的hash函数(keccak-256)的最后20bytes的值。一个地址总固定是以ox开头,因为它是以16进制的格式表示的。

下面就来详细的介绍下如何创建地址：

1.  首先我们拿到一个公钥

    ```
    pubKey = 6e145ccef1033dea239875dd00dfb4fee6e3348b84985c92f103444683b......
    ```

1.  对这个公钥进行hash(keccak-256)运算,最后输出一个64个字符/32bytes长度的字符串。

    ```
    Keccak256(pubKey) = 2a5bc342ed616b5ba5732269001d3f1ef827552ae1114027bd3ecf1f086ba0f9
    ```

1.  截取hash结果字符串末尾的40个字符/20bytes.这个40个字符/20bytes就是我们的地址

    ```
    Address = 0x001d3f1ef827552ae1114027bd3ecf1f086ba0f9
    ```

当我们加上0x的固定开头之后，这个地址就变成42个字符。ETH的地址以原始的16进制表示，而且需要重点注意的是它不区分大小写。所有的钱包都支持接收大写或者小写的地址，解析起来没有任何的区别。

```
0x001d3f1ef827552ae1114027bd3ecf1f086ba0f9 or
0X001D3F1EF827552AE1114027BD3ECF1F086BA0F9
```

# 如何来定义一个地址变量?

在ETH中只需要简单的在变量的前面加上address关键词就可以定义一个变量为一个地址。

下面的例子展示了solidity中我们获取当前与合约进行交互的地址。

```
address user = msg.sender
```

# 地址字面量(address literal)

TODO

# 地址 VS 可支付地址

地址(address)和可支付地址(payable address)的区别在solidity0.5版本中有过介绍。可支付地址可以接受ETH TOKEN，而普通的地址则不行。但给地址分配了一个payable关键字之后,这个地址可以接受和发送ETH TOKEN。结果就是别的一些方法（transfer, send, call, delegatecall and staticcall），对这个地址也是可行的。

# 地址和可支付地址之间的类型转换

-   从**可支付地址**到**地址**的隐式转换（Implicit conversions）是被允许的
-   从**地址**到**可支付地址**的隐式转换（Implicit conversions）是不被允许的
-   从**地址**或者到**地址**的显式转换只允许：integers、integers literal、bytes20、contract type
-   显式转换方式payable(x),x可以是integers, integer literal, bytes 或者contract type

# 合约地址

在0.5.0之前的版本中，合约直接来源于地址类型（因为地址和可支付地址之间没有任何区别）。

从0.5.0版本开始，合约不再来源于地址类型。然而如果合约有一个payable fallback方法，它们仍然可以转化为地址和可支付地址。

```
contract NotPayable {

    // No payable function here

}

contract Payable {

    // Payable function
    function() payable {

        // do something with payable function

    }

}

contract HelloWorld {

    address x = address(NotPayable);
    address y = address(Payable);

    function hello_world() public pure returns (string memory) {
        return "hello world";
    }

}
```

让我们看下上面这个HelloWorld 合约：

-   NotPayable是一个合约不包含payable fallback方法。变量X分配了address(NotPayable)将会是一个地址类型。
-   Payable是一个合约包含了payable fallback方法。变量Y分配了一个address(Payable)将会是一个可支付地址。
-   外部函数签名地址用于地址类型和可支付地址类型。

# 地址类型比较

-   ≤
-   <
-   ==
-   ≠
-   ≥
-   `>

# 检查一个地址是否存在?

```
modifier addressExist(address _address) {
    require(_address != address(0));
    _;
}
```

# 可以用于地址的交易方法

你可以想象到，地址可以像别的地址发送资金(ethers)。这块内容主要介绍不同的可用的方法。

> 1 Ether = 1¹⁸ Wei = 1,000,000,000,000,000,000 wei

**1.address.transfer() (not recommend)**

-   转账一定量的ether(以wei为单位)到指定的账户.
-   失败后还原并且抛出一个错误异常.
-   花费2300gas，并且不可调节

**2.address.send() (not recommend)**

-   和address.transfer()类似
-   失败之后返回false
-   花费2300gas，并且不可调节

**3.address.balance**

-   返回账号对应的金额以Wei为单位

有两种方法可以查看eth地址/合约地址的余额

```
address public _account = msg.sender;

// look-up balance from sender (Method 1)
uint public sender_balance = _account.balance;

// look-up balance from _account (Method 2)
uint public sender_balance = address(_account).balance;

// look-up balance from the current contract (Method 2)
uint public contract_balance = address(this).balance;
```

**4.address.call(bytes memory) returns (bool, bytes memory)**

-   创建一个任意的消息调用，通过传递一个memory类型的参数

-   返回一个tuple类型包括:

    -   一个调用结果的bool值（成功的时候是true，失败的时候是false）
    -   data以bytes格式

-   转发所有可用gas，可调节

**5.address.delegatecall(bytes memory) returns (bool, bytes memory)**

-   底层的DELEGATECALL,通过传递一个memory类型的参数

-   返回一个tuple类型包括:

    -   一个调用结果的bool值（成功的时候是true，失败的时候是false）
    -   data以bytes格式

-   转发所有可用gas，可调节

**6.address.staticcall(bytes memory) returns (bool, bytes memory)**

**7.address.callcode(payload) (deprecated)**

# 相关的返回一个地址的方法

-   msg.sender()
-   tx.origin
-   block.coinbase()
-   block.coinbase(_address payable)
-   ecrecover()

# 零地址

一旦一个只能合约被编译之后，它就会使用一个特殊的合约创建交易来部署到eth平台上。然后，一笔交易可以会得到一个具体的地址。所以创建合约的时候应该拿到哪个具体的地址呢？

当在接受者的字段上填上了一个特殊的地址的时候，EVM就会明白这笔交易打算创建一个新的合约。这个特殊的地址就被称作为零地址(zero-address)。

一个零地址在ETH中也是20bytes的长度，但是只包含了空的bytes。这就解释了为什么叫做零地址,因为它只包含0x0。

# 参考

-   `<address>.balance(uint256)`
-   `<address payable>.transfer(uint256 _amount)`
-   `<address payable>.send(uint256 _amount) returns (bool)`
-   `<address payable>.call(bytes memory) returns (bool, bytes memory)`
-   `<address payable>.delegatecall(bytes memory) returns (bool, bytes memory)`
-   `<address payable>.staticcall(bytes memory) returns (bool, bytes memory)`