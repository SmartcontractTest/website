---
title: 第一笔链上交易
weight: 7
---


这篇文章指导你如何在 starcoin 区块链上执行自己的第一笔交易。
在这之前，建议你先阅读 tech 相关的文章，对 starcoin 的基本概念有一定了解。

<!--more-->

## 前提

假设你已经在本地跑起来一个 starcoin dev 节点。


## 提交交易的几个步骤

- 启动 cli console，并连接到 starcoin 节点。
- 创建两个账户，Alice,Bob。
- 给 Alice 账户挖钱。
- 提交转账交易：Alice 给 Bob 打钱。

### 启动 cli console

假设你的节点目录是 `dev_node`（如果你的节点目录不一样，后续命令中的 `dev_node` 需要替换成你自己的节点目录）。

执行以下命令，进入 starcoin console。

- 通过本地的 IPC 进行连接：

``` shell
> starcoin -d dev_node console
```

- 通过 websocket 连接：
查看你 节点 config 文件中的 websocket 端口，config文件在 `dev_node/dev/config.yml`

以下是我本机的相关配置。

``` toml
[rpc]
ws_address = "127.0.0.1:56982"
```

然后执行以下命令进入 console。（请将`56982`成你的端口）

```shell
starcoin --connect ws://127.0.0.1:56982 console
```


console 的基本展示如下：

``` shell
2020-05-28T17:42:07.577269+08:00 INFO starcoin::cmd/starcoin/src/main.rs::27 - Starcoin opts: StarcoinOpt { connect: Some(WebSocket("ws://127.0.0.1:56982")), data_dir: None, net: None, seed: None, dev_period: 0, node_key: None, node_key_file: None, sync_mode: FAST, disable_std_log: false, disable_file_log: false, enable_mine: None, miner_thread: None, disable_seed: false }
2020-05-28T17:42:07.577565+08:00 INFO starcoin::cmd/starcoin/src/main.rs::57 - Try to connect node by websocket: "ws://127.0.0.1:56982"
2020-05-28T17:42:07.583089+08:00 INFO starcoin::cmd/starcoin/src/main.rs::76 - Start console, disable stderr output.
Logger is disabled.
starcoin%

```

### 创建账户

连接到节点后，我们先来创建两个账户。


第一个账户，是节点的默认账户，我们称之为 Alice。
```bash
starcoin% account create -p my-pass
+------------+------------------------------------------------------------------+
| address    | 1d8133a0c1a07366de459fb08d28d2a6                                 |
+------------+------------------------------------------------------------------+
| is_default | false                                                            |
+------------+------------------------------------------------------------------+
| public_key | 7add08c841d0f99f1f90ea2632c72aee483fab882e0d8d6d6defed2f1987345d |
+------------+------------------------------------------------------------------+

starcoin% account show 1d8133a0c1a07366de459fb08d28d2a6
+--------------------+------------------------------------------------------------------+
| account.address    | 1d8133a0c1a07366de459fb08d28d2a6                                 |
+--------------------+------------------------------------------------------------------+
| account.is_default | false                                                            |
+--------------------+------------------------------------------------------------------+
| account.public_key | 7add08c841d0f99f1f90ea2632c72aee483fab882e0d8d6d6defed2f1987345d |
+--------------------+------------------------------------------------------------------+
| auth_key_prefix    | 7bc6066656bb248755686d2ab78aef14                                 |
+--------------------+------------------------------------------------------------------+
+--------------------+------------------------------------------------------------------+
| sequence_number    |                                                                  |
+--------------------+------------------------------------------------------------------+
```

- address 是账户地址。
- public_key 是账户地址对应的公钥。

> 注意，创建账户只是在 starcoin node里创建一对密钥，并不会更新链的状态。所以 balance 和  sequence_number 此时还是空的。

然后再创建另外一个，称之为 Bob。

``` bash
starcoin% account create -p my-pass
+------------+------------------------------------------------------------------+
| address    | bfbed907d7ba364e1445b971f9182949                                 |
+------------+------------------------------------------------------------------+
| is_default | false                                                            |
+------------+------------------------------------------------------------------+
| public_key | d80234b11619e62a62fac048b2b79a9eec1727b476155e1f8fe19c89c7443076 |
+------------+------------------------------------------------------------------+

starcoin% account show bfbed907d7ba364e1445b971f9182949
+--------------------+------------------------------------------------------------------+
| account.address    | bfbed907d7ba364e1445b971f9182949                                 |
+--------------------+------------------------------------------------------------------+
| account.is_default | false                                                            |
+--------------------+------------------------------------------------------------------+
| account.public_key | d80234b11619e62a62fac048b2b79a9eec1727b476155e1f8fe19c89c7443076 |
+--------------------+------------------------------------------------------------------+
| auth_key_prefix    | 7c87272c7fc2f5586a0770d1d718f14f                                 |
+--------------------+------------------------------------------------------------------+
| sequence_number    |                                                                  |
+--------------------+------------------------------------------------------------------+
```

console 还提供了查看当前所有账户的命令：

```
starcoin% account list
+----------------------------------+------------+------------------------------------------------------------------+
| address                          | is_default | public_key                                                       |
+----------------------------------+------------+------------------------------------------------------------------+
| bfbed907d7ba364e1445b971f9182949 | false      | d80234b11619e62a62fac048b2b79a9eec1727b476155e1f8fe19c89c7443076 |
+----------------------------------+------------+------------------------------------------------------------------+
| 1d8133a0c1a07366de459fb08d28d2a6 | true       | 7add08c841d0f99f1f90ea2632c72aee483fab882e0d8d6d6defed2f1987345d |
+----------------------------------+------------+------------------------------------------------------------------+
```

### 使用 Faucet 给账户充钱

 Dev 环境下，可以利用 faucet 给账户充钱。faucet 只在 dev 和 test net 中存在，以便于开发者做开发和测试。
 下面我们就通过 console 给 Alice 充钱。

``` bash
starcoin% dev get_coin -v 5000000
+-----------------+------------------------------------------------------------------+
| gas_unit_price  | 1                                                                |
+-----------------+------------------------------------------------------------------+
| id              | 65610116a010de657c9faeca94c2b91b9e4fd36f62bc8d7ccbdbb6fdd2e64769 |
+-----------------+------------------------------------------------------------------+
| max_gas_amount  | 2000000                                                          |
+-----------------+------------------------------------------------------------------+
| sender          | 0000000000000000000000000a550c18                                 |
+-----------------+------------------------------------------------------------------+
| sequence_number | 1                                                                |
+-----------------+------------------------------------------------------------------+
```


`dev get_coin` 会往默认账户中充钱，如果链上不存在这个账户，它会先创建这个账户，然后再往该账户转入 `-v` 指定数量的 coin。
 命令输出的是以 faucet 账户（地址是 `0000000000000000000000000a550c18` ）发出的交易信息。

等待几秒钟，然后再查看账户信息。

```
starcoin% account show 1d8133a0c1a07366de459fb08d28d2a6
+--------------------+------------------------------------------------------------------+
| account.address    | 1d8133a0c1a07366de459fb08d28d2a6                                 |
+--------------------+------------------------------------------------------------------+
| account.is_default | false                                                            |
+--------------------+------------------------------------------------------------------+
| account.public_key | 7add08c841d0f99f1f90ea2632c72aee483fab882e0d8d6d6defed2f1987345d |
+--------------------+------------------------------------------------------------------+
| auth_key_prefix    | 7bc6066656bb248755686d2ab78aef14                                 |
+--------------------+------------------------------------------------------------------+
| balances.STC       | 5000000                                                          |
+--------------------+------------------------------------------------------------------+
| sequence_number    | 0                                                                |
+--------------------+------------------------------------------------------------------+
```

可以看到 `balances` 和 `sequence_number` 有数据了。

### 提交交易 - 转账


首先需要解锁 Alice 的账户，授权 node 使用 alice 的私钥对交易签名。

``` bash
account unlock -p my-pass 1d8133a0c1a07366de459fb08d28d2a6
```
其中 `-p my-pass` 是创建账户时传入的密码。

账户解锁后，执行以下命令：

```
starcoin% txn transfer -f 1d8133a0c1a07366de459fb08d28d2a6 -t bfbed907d7ba364e1445b971f9182949 -k 7add08c841d0f99f1f90ea2632c72aee483fab882e0d8d6d6defed2f1987345d -v 10000
+-----------------+------------------------------------------------------------------+
| gas_unit_price  | 1                                                                |
+-----------------+------------------------------------------------------------------+
| id              | 44433463c38aacd31731fba1a38d3a38f9a14c0281a264634e470c8f25bd557d |
+-----------------+------------------------------------------------------------------+
| max_gas_amount  | 1000000                                                          |
+-----------------+------------------------------------------------------------------+
| sender          | 1d8133a0c1a07366de459fb08d28d2a6                                 |
+-----------------+------------------------------------------------------------------+
| sequence_number | 0                                                                |
+-----------------+------------------------------------------------------------------+
```

其中：

- `-f 1d8133a0c1a07366de459fb08d28d2a6`: 是 Alice 的账户地址。
- `-t bfbed907d7ba364e1445b971f9182949`：是 Bob 的账户地址。
- `-k 7add08c841d0f99f1f90ea2632c72aee483fab882e0d8d6d6defed2f1987345d`：是 Bob 的公钥。

> 如果，Bob 的账户在链上还不存在，需要提供他的公钥，转账交易会自动在链上创建 Bob 的账户。


此时，交易已经被提交到链上。还需要等待几秒（dev 环境出块时间比较短），让交易上链。然后再查看 Bob 的账户信息：


``` bash
starcoin% account show bfbed907d7ba364e1445b971f9182949
+--------------------+------------------------------------------------------------------+
| account.address    | bfbed907d7ba364e1445b971f9182949                                 |
+--------------------+------------------------------------------------------------------+
| account.is_default | false                                                            |
+--------------------+------------------------------------------------------------------+
| account.public_key | d80234b11619e62a62fac048b2b79a9eec1727b476155e1f8fe19c89c7443076 |
+--------------------+------------------------------------------------------------------+
| auth_key_prefix    | 7c87272c7fc2f5586a0770d1d718f14f                                 |
+--------------------+------------------------------------------------------------------+
| balances.STC       | 10000                                                            |
+--------------------+------------------------------------------------------------------+
| sequence_number    | 0                                                                |
+--------------------+------------------------------------------------------------------+
```

可以发现：Bob 账户已经有钱了。


至此，你已经成功完成了一笔链上转账！

