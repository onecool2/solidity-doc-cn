.. index:: ! event

.. _events:

******
事件
******

事件允许我们方便地使用 EVM 的日志基础设施。
我们可以在 dapp 的用户界面中监听事件，EVM 的日志机制可以反过来“调用”用来监听事件的 Javascript 回调函数。

事件在合约中可被继承。当他们被调用时，会使参数被存储到交易的日志中 —— 一种区块链中的特殊数据结构。
这些日志与地址相关联，被并入区块链中，只要区块可以访问就一直存在（在 Frontier 和 Homestead 版本中会被永久保存，在 Serenity 版本中可能会改动)。
日志和事件在合约内不可直接被访问（甚至是创建日志的合约也不能访问）。

对日志的 SPV（Simplified Payment Verification）证明是可能的，如果一个外部实体提供了一个带有这种证明的合约，它可以检查日志是否真实存在于区块链中。
但需要留意的是，由于合约中仅能访问最近的 256 个区块哈希，所以还需要提供区块头信息。

最多三个参数可以接收 ``indexed`` 属性，从而使它们可以被搜索：在用户界面上可以使用 indexed 参数的特定值来进行过滤。

如果数组（包括 ``string`` 和 ``bytes``）类型被标记为索引项，则它们的 keccak-256 哈希值会被作为 topic 保存。

除非你用 ``anonymous`` 说明符声明事件，否则事件签名的哈希值是 topic 之一。
同时也意味着对于匿名事件无法通过名字来过滤。

所有非索引参数都将存储在日志的数据部分中。

.. note::
    索引参数本身不会被保存。你只能搜索它们的值（来确定相应的日志数据是否存在），而不能获取它们的值本身。

::

    pragma solidity  >=0.4.21 <0.7.0;

    contract ClientReceipt {
        event Deposit(
            address indexed _from,
            bytes32 indexed _id,
            uint _value
        );

        function deposit(bytes32 _id) public payable {
            // 我们可以过滤对 `Deposit` 的调用，从而用 Javascript API 来查明对这个函数的任何调用（甚至是深度嵌套调用）。
            Deposit(msg.sender, _id, msg.value);
        }
    }

使用 JavaScript API 调用事件的用法如下：

::

    var abi = /* abi 由编译器产生 */;
    var ClientReceipt = web3.eth.contract(abi);
    var clientReceipt = ClientReceipt.at("0x1234...ab67" /* 地址 */);

    var event = clientReceipt.Deposit();

    // 监视变化
    event.watch(function(error, result){
        // 结果包括对 `Deposit` 的调用参数在内的各种信息。
        if (!error)
            console.log(result);
    });

    // 或者通过回调立即开始观察
    var event = clientReceipt.Deposit(function(error, result) {
        if (!error)
            console.log(result);
    });

.. index:: ! log

日志的底层接口
===========================

通过函数 ``log0``，``log1``， ``log2``， ``log3`` 和 ``log4`` 可以访问日志机制的底层接口。
``logi``  接受 ``i + 1`` 个 ``bytes32`` 类型的参数。其中第一个参数会被用来做为日志的数据部分，
其它的会做为 topic。上面的事件调用可以以相同的方式执行。

::

    pragma solidity ^0.4.10;

    contract C {
        function f() public payable {
            bytes32 _id = 0x420042;
            log3(
                bytes32(msg.value),
                bytes32(0x50cb9fe53daa9737b786ab3646f04d0150dc50ef4e75f59509d83667ad5adb20),
                bytes32(msg.sender),
                _id
            );
        }
    }

其中的长十六进制数的计算方法是 ``keccak256("Deposit(address,hash256,uint256)")``，即事件的签名。

其它学习事件机制的资源
==============================================

- `Javascript 文档 <https://github.com/ethereum/wiki/wiki/JavaScript-API#contract-events>`_
- `Web3.js 0.2x 中文文档 <https://learnblockchain.cn/docs/web3js-0.2x/web3.eth.html#contract-events>`_
- `事件使用例程 <https://github.com/debris/smart-exchange/blob/master/lib/contracts/SmartExchange.sol>`_
- `如何在 js 中访问它们 <https://github.com/debris/smart-exchange/blob/master/lib/exchange_transactions.js>`_
