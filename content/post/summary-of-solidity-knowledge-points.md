---
title: "solidity知识点总结"
date: 2023-02-17T11:37:36+08:00
draft: false
tags: ["solidity", "eth", "evm", "contract"]
categories: ["solidity", "eth"]
author: "Jobin Leung"
contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'
---

### 1. 类型

#### 1.1 值类型

值类型的变量将始终按值来传递，即值拷贝。

##### 1.1.1 布尔类型

`bool`: `true`或者`false`

运算符：

- `!` （逻辑非）
- `&&` （逻辑与， “and” ）
- `||` （逻辑或， “or” ）
- `==` （等于）
- `!=` （不等于）

运算符 `||` 和 `&&` 都遵循同样的短路（ short-circuiting ）规则。就是说在表达式 `f(x) || g(y)` 中， 如果 `f(x)` 的值为 `true` ，那么 `g(y)` 就不会被执行

##### 1.1.2 整形

`int`：有符号，从`int8`到`int256`，以 `8` 位为步长递增

`uint`：无符号，从`uint8`到`uint256`，以 `8` 位为步长递增

`uint` 和 `int` 分别是 `uint256` 和 `int256` 的别名

运算符：

- 比较运算符： `<=` ， `<` ， `==` ， `!=` ， `>=` ， `>` （返回布尔值）
- 位运算符： `&` ， `|` ， `^` （异或）， `~` （位取反）
- 算数运算符： `+` ， `-` ， 一元运算 `-` ， 一元运算 `+` ， `*` ， `/` ， `%` （取余） ， `**` （幂）， `<<` （左移位） ， `>>` （右移位）

##### 1.1.3 定长浮点型

Solidity 还没有完全支持定长浮点型，因此不建议使用。在 solidity 中一般喜欢使用“单位”的把戏来处理小数，例如 0.01 元钱，可以表示为 1 分钱，从而避免小数的产生

`fixed` / `ufixed`：表示各种大小的有符号和无符号的定长浮点型。 在关键字 `ufixedMxN` 和 `fixedMxN` 中，`M` 表示该类型占用的位数，`N` 表示可用的小数位数。 `M` 必须能整除 8，即 8 到 256 位。 `N` 则可以是从 0 到 80 之间的任意数。 `ufixed` 和 `fixed` 分别是 `ufixed128x19` 和 `fixed128x19` 的别名。

##### 1.1.4 地址类型

`address`：地址类型存储一个 20 字节的值（以太坊地址的大小）。 地址类型也有成员变量，并作为所有合约的基础。

运算符：

- `<=`， `<`， `==`， `!=`， `>=` 和 `>`

###### 1.1.4.1 地址类型的成员变量

`address.balance :`

以 Wei 为单位的地址类型的余额。

- `address.transfer(uint256 amount)`:

向地址类型发送数量为 amount 的 Wei，失败时抛出异常，发送 2300 gas 的矿工费，不可调节。

- `address.send(uint256 amount) returns (bool)`:

向地址类型发送数量为 amount 的 Wei，失败时返回 `false`，发送 2300 gas 的矿工费用，不可调节。

`send` 是 `transfer` 的低级版本。如果执行失败，当前的合约不会因为异常而终止，但 `send` 会返回 `false`。

- `address.call(...) returns (bool)`:

发出低级函数 `CALL`，失败时返回 `false`，发送所有可用 gas，可调节。

    namReg.call.gas(1000000)("register", "MyName");
    nameReg.call.gas(1000000).value(1 ether)("register", "MyName");

- `address.callcode(...) returns (bool)`：

发出低级函数 `CALLCODE`，失败时返回 `false`，发送所有可用 gas，可调节。

- `address.delegatecall(...) returns (bool)`:

发出低级函数 `DELEGATECALL`，失败时返回 `false`，发送所有可用 gas，可调节。

##### 1.1.5 定长字节数组

`bytes1`， `bytes2`， `bytes3`， …， `bytes32`。`byte` 是 `bytes1` 的别名。

运算符：

- 比较运算符：`<=`， `<`， `==`， `!=`， `>=`， `>` （返回布尔型）
- 位运算符： `&`， `|`， `^` （按位异或）， `~` （按位取反）， `<<` （左移位）， `>>` （右移位）
- 索引访问：如果 `x` 是 `bytesI` 类型，那么 `x[k]` （其中 `0 <= k < I`）返回第 `k` 个字节（只读）。

成员变量：

- `.length` 表示这个字节数组的长度（只读）.

##### 1.1.6 变长字节数组

是引用类型，见后文

##### 1.1.7 地址常量

`address public a = 0xdCad3a6d3569DF655070DEd06cb7A1b2Ccd1D3AF;`

##### 1.1.8 整形常量

`uint256 public x = 2e10;`

##### 1.1.9 字符串常量

`string public abc = 'abc';`

可以隐式地转换成 `bytes1`，……，`bytes32`，如果合适的话，还可以转换成 `bytes`

##### 1.1.10 十六进制字面常数

`string public h = hex"010A31";`

##### 1.1.11 枚举

```
    contract  EgEnum{
        enum Season{Spring, Summer, Autumn, Winter}    function printSeason(Season s) public returns(Season) {
            return s;
        }    function test1() public returns(Season){
            return printSeason(Season.Spring);
        }    function test2() public returns(uint){
            uint s = uint(Season.Spring);
            return s;
        }    function test3() public returns(Season){
            //Season s = Season(5);//越界
            Season s = Season(3);
            return s;
        }
    }
```

`enum`的实际类型是无符号整数，根据其数字大小在`uint8`到`uint256`之间

##### 1.1.12 函数类型

函数类型是一种表示函数的类型。可以将一个函数赋值给另一个函数类型的变量，也可以将一个函数作为参数进行传递，还能在函数调用中返回函数类型变量。 函数类型有两类：- _内部（internal）_ 函数和 _外部（external）_ 函数：

内部函数只能在当前合约内被调用（更具体来说，在当前代码块内，包括内部库函数和继承的函数中），因为它们不能在当前合约上下文的外部被执行。 调用一个内部函数是通过跳转到它的入口标签来实现的，就像在当前合约的内部调用一个函数。

外部函数由一个地址和一个函数签名组成，可以通过外部函数调用传递或者返回。

函数类型表示成如下的形式:

```
    function (<parameter types>) {internal|external} [pure|constant|view|payable] [returns (<return types>)]
```

例如：

`function (int256) pure returns(int256) f;`

##### 1.1.13 常量

状态变量可以被声明为 `constant`。在这种情况下，只能使用那些在编译时有确定值的表达式来给它们赋值。

```
        uint constant x = 32**22 + 8;
        string constant text = "abc";
        bytes32 constant myHash = keccak256("abc");
```

#### 1.2 引用类型

数据存储：即数据是保存在 memory 还是在 storage 中，大多数时候数据都有默认的存储位置，也可以通过在类型名后加关键字进行修改。

函数参数的默认位置是 memory

局部变量的默认位置是 storage

状态变量的数据位置强制为 storage

```
    pragma solidity ^0.4.0;

    contract C {
        uint[] x; // x 的数据存储位置是 storage

        // memoryArray 的数据存储位置是 memory
        function f(uint[] memoryArray) public {
            x = memoryArray; // 将整个数组拷贝到 storage 中，可行
            var y = x;  // 分配一个指针（其中 y 的数据存储位置是 storage），可行
            y[7]; // 返回第 8 个元素，可行
            y.length = 2; // 通过 y 修改 x，可行
            delete x; // 清除数组，同时修改 y，可行
            // 下面的就不可行了；需要在 storage 中创建新的未命名的临时数组， /
            // 但 storage 是“静态”分配的：
            // y = memoryArray;
            // 下面这一行也不可行，因为这会“重置”指针，
            // 但并没有可以让它指向的合适的存储位置。
            // delete y;

            g(x); // 调用 g 函数，同时移交对 x 的引用
            h(x); // 调用 h 函数，同时在 memory 中创建一个独立的临时拷贝
        }

        function g(uint[] storage storageArray) internal {}
        function h(uint[] memoryArray) public {}
    }
```

##### 1.2.1 数组

数组可以在声明时指定长度，也可以动态调整大小。 对于 存储(storage)的数组来说，元素类型可以是任意的（即元素也可以是数组类型，映射类型或者结构体）。 对于 内存(memory)的数组来说，元素类型不能是映射类型，如果作为 public 函数的参数，它只能是 ABI 类型。

一个元素类型为 `T`，固定长度为 `k` 的数组可以声明为 `T[k]`，而动态数组声明为 `T[]`。

**length**:

数组有 `length` 成员变量表示当前数组的长度。 动态数组可以在 存储 storage （而不是 内存 memory ）中通过改变成员变量 `.length` 改变数组大小。 并不能通过访问超出当前数组长度的方式实现自动扩展数组的长度。 一经创建，内存 memory 数组的大小就是固定的（但却是动态的，也就是说，它依赖于运行时的参数）。

**push**:

变长的 存储 storage 数组以及 `bytes` 类型（而不是 `string` 类型）都有一个叫做 `push` 的成员函数，它用来附加新的元素到数组末尾。 这个函数将返回新的数组长度。

```
    pragma solidity ^0.4.16;

    contract ArrayContract {
        uint[2**20] m_aLotOfIntegers;
        // 注意下面的代码并不是一对动态数组，
        // 而是一个数组元素为一对变量的动态数组（也就是数组元素为长度为 2 的定长数组的动态数组）。
        bool[2][] m_pairsOfFlags;
        // newPairs 存储在 memory 中 —— 函数参数默认的存储位置

        function setAllFlagPairs(bool[2][] newPairs) public {
            // 向一个 storage 的数组赋值会替代整个数组
            m_pairsOfFlags = newPairs;
        }

        function setFlagPair(uint index, bool flagA, bool flagB) public {
            // 访问一个不存在的数组下标会引发一个异常
            m_pairsOfFlags[index][0] = flagA;
            m_pairsOfFlags[index][1] = flagB;
        }

        function changeFlagArraySize(uint newSize) public {
            // 如果 newSize 更小，那么超出的元素会被清除
            m_pairsOfFlags.length = newSize;
        }

        function clear() public {
            // 这些代码会将数组全部清空
            delete m_pairsOfFlags;
            delete m_aLotOfIntegers;
            // 这里也是实现同样的功能
            m_pairsOfFlags.length = 0;
        }

        bytes m_byteData;

        function byteArrays(bytes data) public {
            // 字节的数组（语言意义中的 byte 的复数 ``bytes``）不一样，因为它们不是填充式存储的，
            // 但可以当作和 "uint8[]" 一样对待
            m_byteData = data;
            m_byteData.length += 7;
            m_byteData[3] = byte(8);
            delete m_byteData[2];
        }

        function addFlag(bool[2] flag) public returns (uint) {
            return m_pairsOfFlags.push(flag);
        }

        function createMemoryArray(uint size) public pure returns (bytes) {
            // 使用 `new` 创建动态 memory 数组：
            uint[2][] memory arrayOfPairs = new uint[2][](size);
            // 创建一个动态字节数组：
            bytes memory b = new bytes(200);
            for (uint i = 0; i < b.length; i++)
                b[i] = byte(i);
            return b;
        }
    }
```

##### 1.2.2 结构体

Solidity 支持通过构造结构体的形式定义新的类型，以下是一个结构体使用的示例：

```
    pragma solidity ^0.4.11;

    contract CrowdFunding {
        // 定义的新类型包含两个属性。
        struct Funder {
            address addr;
            uint amount;
        }

        struct Campaign {
            address beneficiary;
            uint fundingGoal;
            uint numFunders;
            uint amount;
            mapping (uint => Funder) funders;
        }

        uint numCampaigns;
        mapping (uint => Campaign) campaigns;

        function newCampaign(address beneficiary, uint goal) public returns (uint campaignID) {
            campaignID = numCampaigns++; // campaignID 作为一个变量返回
            // 创建新的结构体示例，存储在 storage 中。我们先不关注映射类型。
            campaigns[campaignID] = Campaign(beneficiary, goal, 0, 0);
        }

        function contribute(uint campaignID) public payable {
            Campaign storage c = campaigns[campaignID];
            // 以给定的值初始化，创建一个新的临时 memory 结构体，
            // 并将其拷贝到 storage 中。
            // 注意你也可以使用 Funder(msg.sender, msg.value) 来初始化。
            c.funders[c.numFunders++] = Funder({addr: msg.sender, amount: msg.value});
            c.amount += msg.value;
        }

        function checkGoalReached(uint campaignID) public returns (bool reached) {
            Campaign storage c = campaigns[campaignID];
            if (c.amount < c.fundingGoal)
                return false;
            uint amount = c.amount;
            c.amount = 0;
            c.beneficiary.transfer(amount);
            return true;
        }
    }
```

#### 1.3 映射 mapping

映射类型在声明时的形式为 `mapping(_KeyType => _ValueType)`。 其中 `_KeyType` 可以是除了映射、变长数组、合约、枚举以及结构体以外的几乎所有类型。 `_ValueType` 可以是包括映射类型在内的任何类型。

```
    pragma solidity ^0.4.0;

    contract MappingExample {
        mapping(address => uint) public balances;

        function update(uint newBalance) public {
            balances[msg.sender] = newBalance;
        }
    }

    contract MappingUser {
        function f() public returns (uint) {
            MappingExample m = new MappingExample();
            m.update(100);
            return m.balances(this);
        }
    }
```

### 2.内置的变量和函数

#### 2.1 以太币单位

以太币 Ether 单位之间的换算就是在数字后边加上 `wei`、 `finney`、 `szabo` 或 `ether` 来实现的，如果后面没有单位，**缺省为 Wei**。例如 `2 ether == 2000 finney`

#### 2.2 时间单位

**秒是缺省时间单位**，在时间单位之间，数字后面带有 `seconds`、 `minutes`、 `hours`、 `days`、 `weeks` 和 `years` 的可以进行换算，基本换算关系如下：

`1 == 1 seconds`

`1 minutes == 60 seconds`

`1 hours == 60 minutes`

`1 days == 24 hours`

`1 weeks == 7 days`

#### 2.3 函数

##### 2.3.1 区块和交易属性

- `block.blockhash(uint blockNumber) returns (bytes32)`：指定区块的区块哈希——仅可用于最新的 256 个区块且不包括当前区块；而 blocks 从 0.4.22 版本开始已经不推荐使用，由 `blockhash(uint blockNumber)` 代替
- `block.coinbase returns(address)`: 挖出当前区块的矿工地址
- `block.difficulty returns(uint)`: 当前区块难度
- `block.gaslimit returns(uint)`: 当前区块 gas 限额
- `block.number returns(uint)`: 当前区块号
- `block.timestamp returns(uint)`: 自 unix epoch 起始当前区块以秒计的时间戳
- `gasleft() returns (uint256)`：剩余的 gas
- `msg.data returns(bytes)`: 完整的 calldata
- `msg.gas returns(uint)`: 剩余 gas - 自 0.4.21 版本开始已经不推荐使用，由 `gesleft()` 代替
- `msg.sender returns(address)`: 消息发送者（当前调用）
- `msg.sig returns(bytes4)`: calldata 的前 4 字节（也就是函数标识符）
- `msg.value returns(uint)`: 随消息发送的 wei 的数量
- `now returns(uint)`: 目前区块时间戳（`block.timestamp`）
- `tx.gasprice returns(uint)`: 交易的 gas 价格
- `tx.origin returns(address)`: 交易发起者（完全的调用链）

##### 2.3.2 ABI 编码函数

- `abi.encode(...) returns (bytes)`： ABI- 对给定参数进行编码
- `abi.encodePacked(...) returns (bytes)`：对给定参数执行紧打包编码
- `abi.encodeWithSelector(bytes4 selector, ...) returns (bytes)`： ABI- 对给定参数进行编码，并以给定的函数选择器作为起始的 4 字节数据一起返回
- `abi.encodeWithSignature(string signature, ...) returns (bytes)`：等价于 `abi.encodeWithSelector(bytes4(keccak256(signature), ...)`

##### 2.3.3 错误处理

- `assert(bool condition)`:

  如果条件不满足，则使当前交易没有效果 — 用于检查内部错误。

- `require(bool condition)`:

  如果条件不满足则撤销状态更改 - 用于检查由输入或者外部组件引起的错误。

- `require(bool condition, string message)`:

  如果条件不满足则撤销状态更改 - 用于检查由输入或者外部组件引起的错误，可以同时提供一个错误消息。

- `revert()`:

  终止运行并撤销状态更改。

- `revert(string reason)`:

  终止运行并撤销状态更改，可以同时提供一个解释性的字符串。

##### 2.3.4 数学和密码学函数

- `addmod(uint x, uint y, uint k) returns (uint)`:

  计算 `(x + y) % k`，加法会在任意精度下执行，并且加法的结果即使超过 `2**256` 也不会被截取。从 0.5.0 版本的编译器开始会加入对 `k != 0` 的校验（assert）。

- `mulmod(uint x, uint y, uint k) returns (uint)`:

  计算 `(x * y) % k`，乘法会在任意精度下执行，并且乘法的结果即使超过 `2**256` 也不会被截取。从 0.5.0 版本的编译器开始会加入对 `k != 0` 的校验（assert）。

- `keccak256(...) returns (bytes32)`:

  Ethereum-SHA-3 （Keccak-256）哈希。

- `sha256(...) returns (bytes32)`:

  计算 SHA-256 哈希。

- `sha3(...) returns (bytes32)`:

  等价于 keccak256。

- `ripemd160(...) returns (bytes20)`:

  计算 RIPEMD-160 哈希。

- `ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address)` ：

  利用椭圆曲线签名恢复与公钥相关的地址，错误返回零值。

##### 2.3.5 地址相关

- `address.balance (uint256)`:  
  以 Wei 为单位的 地址类型 的余额。
- `address.transfer(uint256 amount)`:  
  向 地址类型 发送数量为 amount 的 Wei，失败时抛出异常，发送 2300 gas 的矿工费，不可调节。
- `address.send(uint256 amount) returns (bool)`:  
  向 地址类型 发送数量为 amount 的 Wei，失败时返回 false，发送 2300 gas 的矿工费用，不可调节。

  如果调用栈深度已经达到 1024，转账会失败

- `address.call(...) returns (bool)`:  
  发出低级函数 CALL，失败时返回 false，发送所有可用 gas，可调节。
- `address.callcode(...) returns (bool)`：  
  发出低级函数 CALLCODE，失败时返回 false，发送所有可用 gas，可调节。(不鼓励使用)
- `address.delegatecall(...) returns (bool)`:  
  发出低级函数 DELEGATECALL，失败时返回 false，发送所有可用 gas，可调节。

##### 2.3.6 合约相关

- `this (current contract's type)`:  
  当前合约，可以明确转换为 地址类型。
- `selfdestruct(address recipient)`:  
  销毁合约，并把余额发送到指定 地址类型。
- `suicide(address recipient)`:  
  与 selfdestruct 等价，但已不推荐使用。  
  此外，当前合约内的所有函数都可以被直接调用，包括当前函数。

### 3 方法/函数

#### 3.1 方法的多返回值

```
    pragma solidity >0.4.23 <0.5.0;

    contract C {
        uint[] data;

        function f() public pure returns (uint, bool, uint) {
            return (7, true, 2);
        }

        function g() public {
            //基于返回的元组来声明变量并赋值
            (uint x, bool b, uint y) = f();
            //交换两个值的通用窍门——但不适用于非值类型的存储 (storage) 变量。
            (x, y) = (y, x);
            //元组的末尾元素可以省略（也适用于变量声明）。
            (data.length,,) = f(); // 将长度设置为 7
            //省略元组中末尾元素的写法，仅可以在赋值操作的左侧使用，除了这个例外：
            (x,) = (1,);
            //(1,) 是指定单元素元组的唯一方法，因为 (1)相当于 1。
        }
    }
```

#### 3.2 错误处理

Solidity 提供了很多错误检查和错误处理的方法。通常，检查是为了防止未经授权的代码访问，当发生错误时，状态会恢复到初始状态。

下面是错误处理中，使用的一些重要方法：

- `assert(bool condition)`

  如果不满足条件，此方法调用将导致一个无效的操作码，对状态所做的任何更改将被还原。这个方法是用来处理内部错误的。

- `require(bool condition)`

  如果不满足条件，此方法调用将恢复到原始状态。此方法用于检查输入或外部组件的错误。

- `require(bool condition, string memory message)`

  如果不满足条件，此方法调用将恢复到原始状态。此方法用于检查输入或外部组件的错误。它提供了一个提供自定义消息的选项。

- `revert()`

  此方法将中止执行并将所做的更改还原为执行前状态。

- `revert(string memory reason)`

  此方法将中止执行并将所做的更改还原为执行前状态。它提供了一个提供自定义消息的选项。

#### 3.3 函数修饰符 modifier

使用 修饰器 modifier 可以轻松改变函数的行为。 例如，它们可以在执行函数之前自动检查某个条件。 修饰器 modifier 是合约的可继承属性， 并可能被派生合约覆盖。

```
    pragma solidity ^0.4.11;

    contract owned {
        function owned() public { owner = msg.sender; }
        address owner;

        // 这个合约只定义一个修饰器，但并未使用： 它将会在派生合约中用到。
        // 修饰器所修饰的函数体会被插入到特殊符号 _; 的位置。
        // 这意味着如果是 owner 调用这个函数，则函数会被执行，否则会抛出异常。
        modifier onlyOwner {
            require(msg.sender == owner);
            _;
        }
    }

    contract mortal is owned {
        // 这个合约从 `owned` 继承了 `onlyOwner` 修饰符，并将其应用于 `close` 函数，
        // 只有在合约里保存的 owner 调用 `close` 函数，才会生效。
        function close() public onlyOwner {
        // 销毁合约
            selfdestruct(owner);
        }
    }

    contract priced {
        // 修改器可以接收参数：
        modifier costs(uint price) {
            if (msg.value >= price) {
                _;
            }
        }
    }

    contract Register is priced, owned {
        mapping (address => bool) registeredAddresses;
        uint price;

        function Register(uint initialPrice) public { price = initialPrice; }

        // 在这里也使用关键字 `payable` 非常重要，否则函数会自动拒绝所有发送给它的以太币。
        function register() public payable costs(price) {
            registeredAddresses[msg.sender] = true;
        }

        function changePrice(uint _price) public onlyOwner {
            price = _price;
        }
    }

    contract Mutex {
        bool locked;
        modifier noReentrancy() {
            require(!locked);
            locked = true;
            _;
            locked = false;
        }

        // 这个函数受互斥量保护，这意味着 `msg.sender.call` 中的重入调用不能再次调用  `f`。
        // `return 7` 语句指定返回值为 7，但修改器中的语句 `locked = false` 仍会执行。
        function f() public noReentrancy returns (uint) {
            require(msg.sender.call());
            return 7;
        }
    }
```

如果同一个函数有多个 修饰器 modifier，它们之间以空格隔开，修饰器 modifier 会依次检查执行。

#### 3.4 函数的 view 和 pure 修饰符及 fallback 函数

##### 3.4.1 `view`

View(视图)函数不会修改状态。如果函数中存在以下语句，则被视为修改状态，编译器将抛出警告。

- 修改状态变量。
- 触发事件。
- 创建合约。
- 使用`selfdestruct`。
- 发送以太。
- 调用任何不是视图函数或纯函数的函数
- 使用底层调用
- 使用包含某些操作码的内联程序集。

Getter 方法是默认的视图函数。声明视图函数，可以在函数声明里，添加`view`关键字。

**`constant` 是 `view` 的别名。**

**编译器没有强制 `view` 方法不能修改状态。**

##### 3.4.2 `pure`

Pure(纯)函数**不读取**或修改状态。如果函数中存在以下语句，则被视为读取状态，编译器将抛出警告。

- 读取状态变量。
- 访问 `address(this).balance` 或 `<address>.balance`
- 访问任何区块、交易、msg 等特殊变量(msg.sig 与 msg.data 允许读取)。
- 调用任何不是纯函数的函数。
- 使用包含特定操作码的内联程序集。

如果发生错误，纯函数可以使用`revert()`和`require()`函数来还原潜在的状态更改。

声明纯函数，可以在函数声明里，添加`pure`关键字。

##### 3.4.3 payable

`payable` 修饰函数时：允许该方法接收以太币 Ether

##### 3.4.4 fallback

fallback(回退) 函数是合约中的特殊函数。它有以下特点

- 当合约中不存在的函数被调用时，将调用 fallback 函数。
- 被标记为外部函数。
- 它没有名字。
- 它没有参数。
- 它不能返回任何东西。
- 每个合约定义一个 fallback 函数。
- 如果没有被标记为`payable`，则当合约收到无数据的以太币转账时，将抛出异常。

**语法**

    // 没有名字，没有参数，不返回，标记为external，可以标记为payable
    function() external {
        // statements
    }

复制

下面的示例展示了合约中的回退函数概念。

```
    pragma solidity ^0.5.0;

    contract Test {
       uint public x ;
       function() external { x = 1; }
    }
    contract Sink {
       function() external payable { }
    }
    contract Caller {

       function callTest(Test test) public returns (bool) {
          (bool success,) = address(test).call(abi.encodeWithSignature("nonExistingFunction()"));
          require(success);
          // test.x 是 1

          address payable testPayable = address(uint160(address(test)));

          // 发送以太测试合同,
          // 转账将失败，这里返回false。
          return (testPayable.send(2 ether));
       }

       function callSink(Sink sink) public returns (bool) {
          address payable sinkPayable = address(sink);
          return (sinkPayable.send(2 ether));
       }
    }
```

#### 3.5 函数重载

同一个作用域内，相同函数名可以定义多个函数。这些函数的参数(参数类型或参数数量)必须不一样。仅仅是返回值不一样不被允许。

下面的例子展示了 Solidity 中的函数重载。

```
    pragma solidity ^0.5.0;

    contract Test {
       function getSum(uint a, uint b) public pure returns(uint){
          return a + b;
       }
       function getSum(uint a, uint b, uint c) public pure returns(uint){
          return a + b + c;
       }
    }
```

#### 3.6 数学函数

Solidity 也提供了内置的数学函数。下面是常用的数学函数：

- `addmod(uint x, uint y, uint k) returns (uint)` 计算(x + y) % k，计算中，以任意精度执行加法，且不限于 2^256 大小。
- `mulmod(uint x, uint y, uint k) returns (uint)` 计算(x \* y) % k，计算中，以任意精度执行乘法，且不限于 2^256 大小。

下面的例子说明了数学函数的用法。

```
    pragma solidity ^0.5.0;

    contract Test {
       function callAddMod() public pure returns(uint){
          return addmod(4, 5, 3);
       }
       function callMulMod() public pure returns(uint){
          return mulmod(4, 5, 3);
       }
    }
```

**输出**

    0: uint256: 0
    0: uint256: 2

#### 3.7 加密函数

Solidity 提供了常用的加密函数。以下是一些重要函数：

- `keccak256(bytes memory) returns (bytes32)` 计算输入的 Keccak-256 散列。
- `sha256(bytes memory) returns (bytes32)` 计算输入的 SHA-256 散列。
- `ripemd160(bytes memory) returns (bytes20)` 计算输入的 RIPEMD-160 散列。
- `ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address)` 从椭圆曲线签名中恢复与公钥相关的地址，或在出错时返回零。函数参数对应于签名的 ECDSA 值: r – 签名的前 32 字节; s: 签名的第二个 32 字节; v: 签名的最后一个字节。这个方法返回一个地址。

下面的例子说明了加密函数的用法。

```
    pragma solidity ^0.5.0;

    contract Test {
       function callKeccak256() public pure returns(bytes32 result){
          return keccak256("ABC");
       }
    }
```

**输出**

    0: bytes32: result 0xe1629b9dda060bb30c7908346f6af189c16773fa148d3366701fbaa35d54f3c8

#### 3.8 事件 event

事件是智能合约发出的信号。智能合约的前端 UI，例如，DApps、web.js，或者任何与 Ethereum JSON-RPC API 连接的东西，都可以侦听这些事件。事件可以被索引，以便以后可以搜索事件记录。

Solidity 中，要定义事件，可以使用`event`关键字(在用法上类似于`function`关键字)。然后可以在函数中使用`emit`关键字触发事件。

```
    pragma solidity ^0.5.0;

    contract Counter {
        uint256 public count = 0;

        event Increment(address who);   // 声明事件

        function increment() public {
            emit Increment(msg.sender); // 触发事件
            count += 1;
        }
    }
```

使用 JavaScript API 调用事件的用法如下：

```
    var abi = /* abi 由编译器产生 */;
    var ClientReceipt = web3.eth.contract(abi);
    var clientReceipt = ClientReceipt.at("0x1234...ab67" /* 地址 */);

    var event = clientReceipt.Increment();

    // 监视变化
    event.watch(function(error, result){
        // 结果包括对 `Increment` 的调用参数在内的各种信息。
        if (!error)
            console.log(result);
    });

    // 或者通过回调立即开始观察
    var event = clientReceipt.Increment(function(error, result) {
        if (!error)
            console.log(result);
    });
```

索引(indexed)参数

一个事件最多有 3 个参数可以标记为索引。可以使用索引参数有效地过滤事件。下面的代码增强了前面的示例，来跟踪多个计数器，每个计数器由一个数字 ID 标识:

```
    pragma solidity ^0.4.21;

    contract Multicounter {
        mapping (uint256 => uint256) public counts;

        event Increment(uint256 indexed which, address who);

        function increment(uint256 which) public {
            emit Increment(which, msg.sender);
            counts[which] += 1;
        }
    }
```

复制

- `counts`替换`count`，`counts`是一个 map。
- `event Increment(uint256 indexed which, address who)` 添加一个索引参数，该参数表示哪个计数器。
- `emit Increment(which, msg.sender)` 用 2 个参数记录事件。

在 Javascript 中，可以使用索引访问计数器：

```

    counter.Increment({ which: counterId }, function (err, result) {
      if (err) {
        return error(err);
      }

      log("Counter " + result.args.which + " was incremented by address: "
          + result.args.who);
      getCount();
    });

```

### 4 可见性，作用域

#### 4.1 函数的可见性

有 4 种可见类型，`external` ，`public` ，`internal` ，`private`,默认情况下是`public`

- `external`

  即外部函数，作为合约接口的一部分，意味着我们可以从其他合约和交易中调用。 一个外部函数 `f()` 不能从内部调用（即 `f()` 不起作用，但 `this.f()` 可以）。

- `public`

  public 函数是合约接口的一部分，可以在内部或通过消息调用。

- `internal`

  这些函数和状态变量只能是内部访问（即从当前合约内部或从它派生的合约访问），不使用 `this` 调用。

- `private`

  private 函数和状态变量仅在当前定义它们的合约中使用，并且不能被派生合约使用。

#### [](#4-2-变量作用域 "4.2    变量作用域")4.2 变量作用域

局部变量的作用域仅限于定义它们的函数，但是状态变量可以有三种作用域类型：`public` ，`internal` ，`private`。

- `public`

  公共状态变量可以在内部访问，也可以通过消息访问。对于公共状态变量，将**生成一个自动 getter 函数**，例如：

```
      pragma solidity ^0.4.0;

      contract C {
          uint public data = 42;
      }

      contract Caller {
          C c = new C();
          function f() public {
              uint local = c.data();
          }
      }
```

- `internal`

  内部状态变量只能从当前合约或其派生合约内访问。

- `private`

  私有状态变量只能从当前合约内部访问，派生合约内不能访问。

可见性标识符的定义位置，对于状态变量来说是在类型后面，对于函数是在参数列表和返回关键字中间。

```
    pragma solidity ^0.4.16;

    contract C {
        function f(uint a) private pure returns (uint b) { return a + 1; }
        function setData(uint a) internal { data = a; }
        uint public data;
    }
```

### 5 合约

#### 5.1 抽象合约

约函数可以缺少实现，如下例所示（请注意函数声明头由 `;` 结尾）:

```
    pragma solidity ^0.4.0;

    contract Feline {
        function utterance() public returns (bytes32);
    }
```

这些合约无法成功编译（即使它们除了未实现的函数还包含其他已经实现了的函数），但他们可以用作基类合约:

```
    pragma solidity ^0.4.0;

    contract Feline {
        function utterance() public returns (bytes32);
    }

    contract Cat is Feline {
        function utterance() public returns (bytes32) { return "miaow"; }
    }
```

如果合约继承自抽象合约，并且没有通过重写来实现所有未实现的函数，那么它本身就是抽象的。

#### 5.2 接口

接口类似于抽象合约，但是它们不能实现任何函数。还有进一步的限制：

1.  无法继承其他合约或接口。
2.  无法定义构造函数。
3.  无法定义变量。
4.  无法定义结构体。
5.  无法定义枚举。

将来可能会解除这里的某些限制。

接口基本上仅限于合约 ABI 可以表示的内容，并且 ABI 和接口之间的转换应该不会丢失任何信息。

接口由它们自己的关键字表示：

```
    pragma solidity ^0.4.11;

    interface Token {
        function transfer(address recipient, uint amount) public;
    }
```

就像继承其他合约一样，合约可以继承接口。

#### 5.3 库

库类似于合约，但主要作用是代码重用。库中包含了可以被合约调用的函数。

Solidity 中，对库的使用有一定的限制。以下是库的主要特征。

- 如果库函数不修改状态，则可以直接调用它们。这意味着纯函数或视图函数只能从库外部调用。
- 库不能被销毁，因为它被认为是无状态的。
- 库不能有状态变量。
- 库不能继承任何其他元素。
- 库不能被继承。

尝试下面的代码，来理解库是如何工作的。

```
    pragma solidity ^0.5.0;

    library Search {
       function indexOf(uint[] storage self, uint value) public view returns (uint) {
          for (uint i = 0; i < self.length; i++) if (self[i] == value) return i;
          return uint(-1);
       }
    }
    contract Test {
       uint[] data;
       constructor() public {
          data.push(1);
          data.push(2);
          data.push(3);
          data.push(4);
          data.push(5);
       }
       function isValuePresent() external view returns(uint){
          uint value = 4;

          // 使用库函数搜索数组中是否存在值
          uint index = Search.indexOf(data, value);
          return index;
       }
    }
```

**输出**

    0: uint256: 3

**Using For**

`using A for B`指令，可用于将库 A 的函数附加到给定类型 B。这些函数将把调用者类型作为第一个参数(使用`self`标识)。

尝试下面的代码，来理解`Using For`是怎么工作的。

```
    pragma solidity ^0.5.0;

    library Search {
       function indexOf(uint[] storage self, uint value) public view returns (uint) {
          for (uint i = 0; i < self.length; i++)if (self[i] == value) return i;
          return uint(-1);
       }
    }
    contract Test {
       using Search for uint[];
       uint[] data;
       constructor() public {
          data.push(1);
          data.push(2);
          data.push(3);
          data.push(4);
          data.push(5);
       }
       function isValuePresent() external view returns(uint){
          uint value = 4;

          // data 表示库
          uint index = data.indexOf(value);
          return index;
       }
    }
```

**输出**

    0: uint256: 3

新增部分：

- 状态变量可以声明为 `constant` 或 `immutable` . 在这两种情况下，在构建合约之后，变量都不能被修改。为 `constant` 变量时，该值必须在编译时固定，而对于 `immutable` ，仍可以在发布时分配。

---

转载请注明来源
