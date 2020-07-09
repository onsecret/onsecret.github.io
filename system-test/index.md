# 区块链实验9-系统测试


万事俱备，只待测试，本篇进行系统测试。

<!--more-->

测试分为三部分

1. 隐私合约及交易测试
2. 访问控制系统测试
3. 信誉系统测试

各部分都不太一样，下面分节进行介绍

## 1. 隐私合约及交易测试

隐私合约及交易是 Quorum 自带的功能，因此测试起来比较简单。我们令Node2 代表农场，Node3 代表超市，假设农场部署了一个私有存储合约，状态只能被超市查看。测试用的合约如下

```js
pragma solidity ^0.5.4;
contract SimpleStorage {

    uint storedData;

    event Change(string message, uint newVal);

    constructor(uint initVal) public {
        emit Change("initialized", initVal);
        storedData = initVal;
    }

    function set(uint x) public {
        emit Change("set", x);
        storedData = x;
    }

    function get() view public returns (uint retVal) {
        return storedData;
    }

}
```

农场部署合约传入的初始值为 50，理论上，Node2 和 Node3 可以通过调用 get() 函数获取到该数据，其它的节点无法查看该数据。

合约的部署与交互使用了 Quorum for Remix 插件

分别进入 Node3（超市） 和 Node4 的 Geth console 进行验证，如下图所示，左侧是 Node3，可以查看合约状态并获取数据，右边是 Node4，无法查看合约状态，也无法获得数据。

![私有合约状态测试](/images/区块链实验9-系统测试/私有合约状态测试.png)

从 Cakeshop 区块链浏览器可以更清楚地看到两种情况

![节点3可以查看私有合约](/images/区块链实验9-系统测试/节点3可以查看私有合约.png)

![节点4无法查看私有合约](/images/区块链实验9-系统测试/节点4无法查看私有合约.png)

## 2. 访问控制测试

访问控制测试主要分为三部分

1. 未加入信誉系统时的访问控制时间；
2. 加入信誉系统后的访问控制时间；
3. 传统 ABAC 架构利用智能合约实现的访问控制时间，用来对比我们实现的系统的时间性能。

### 2.1 访问控制时间测量

我们首先将信誉系统删除，只剩下 MC 和 ACC，然后进行测试。修改后的两个合约代码位于仓库中。

此时 MC 部署的 Gas 消耗为 1958457，ACC 部署消耗为 4128313。然后定义属性和策略，合法与非法的访问控制一共进行了30次。此时关于访问控制时间的定义为发起访问到得到交易 Hash，合法访问测试30次的平均时间为12.3 ms，最坏时间为 61ms；非法访问测试30次的平均时间为12.8ms，最坏时间为47ms。

但最终对以上测量心存疑虑，考虑到访问控制的目的是取得允许还是拒绝这个结果，决定将时间的定义调整为发起访问到获得 Emit 的事件。结果如下，30 次合法访问的平均时间为 4410ms，30 次非法访问的平均时间为 2434ms。

```bash
Note: Every 2 minutes, send an access request

The 1-th request
First Account Return Message: Access authorized
Begin-End: 1592878045667-1592878049043
Time Cost: 3376ms
Second Account Return Message: Policy check failed
Begin-End: 1592878051561-1592878054042
Time Cost: 2481ms

The 2-th request
First Account Return Message: Access authorized
Begin-End: 1592878174596-1592878179034
Time Cost: 4438ms
Second Account Return Message: Policy check failed
Begin-End: 1592878181583-1592878184035
Time Cost: 2452ms

The 3-th request
First Account Return Message: Access authorized
Begin-End: 1592878304541-1592878309073
Time Cost: 4532ms
Second Account Return Message: Policy check failed
Begin-End: 1592878311574-1592878314086
Time Cost: 2512ms

The 4-th request
First Account Return Message: Access authorized
Begin-End: 1592878434737-1592878439036
Time Cost: 4299ms
Second Account Return Message: Policy check failed
Begin-End: 1592878441866-1592878444045
Time Cost: 2179ms

The 5-th request
First Account Return Message: Access authorized
Begin-End: 1592878564583-1592878569075
Time Cost: 4492ms
Second Account Return Message: Policy check failed
Begin-End: 1592878571602-1592878574074
Time Cost: 2472ms

The 6-th request
First Account Return Message: Access authorized
Begin-End: 1592878694584-1592878699063
Time Cost: 4479ms
Second Account Return Message: Policy check failed
Begin-End: 1592878701544-1592878704072
Time Cost: 2528ms

The 7-th request
First Account Return Message: Access authorized
Begin-End: 1592878824587-1592878829031
Time Cost: 4444ms
Second Account Return Message: Policy check failed
Begin-End: 1592878831549-1592878834044
Time Cost: 2495ms

The 8-th request
First Account Return Message: Access authorized
Begin-End: 1592878954804-1592878959072
Time Cost: 4268ms
Second Account Return Message: Policy check failed
Begin-End: 1592878961811-1592878964034
Time Cost: 2223ms

The 9-th request
First Account Return Message: Access authorized
Begin-End: 1592879084542-1592879089051
Time Cost: 4509ms
Second Account Return Message: Policy check failed
Begin-End: 1592879091605-1592879094060
Time Cost: 2455ms

The 10-th request
First Account Return Message: Access authorized
Begin-End: 1592879214557-1592879219065
Time Cost: 4508ms
Second Account Return Message: Policy check failed
Begin-End: 1592879221555-1592879224068
Time Cost: 2513ms

The 11-th request
First Account Return Message: Access authorized
Begin-End: 1592879344622-1592879349060
Time Cost: 4438ms
Second Account Return Message: Policy check failed
Begin-End: 1592879351583-1592879354077
Time Cost: 2494ms

The 12-th request
First Account Return Message: Access authorized
Begin-End: 1592879474566-1592879479063
Time Cost: 4497ms
Second Account Return Message: Policy check failed
Begin-End: 1592879481669-1592879484057
Time Cost: 2388ms

The 13-th request
First Account Return Message: Access authorized
Begin-End: 1592879604568-1592879609074
Time Cost: 4506ms
Second Account Return Message: Policy check failed
Begin-End: 1592879611637-1592879614036
Time Cost: 2399ms

The 14-th request
First Account Return Message: Access authorized
Begin-End: 1592879734562-1592879739067
Time Cost: 4505ms
Second Account Return Message: Policy check failed
Begin-End: 1592879741572-1592879744031
Time Cost: 2459ms

The 15-th request
First Account Return Message: Access authorized
Begin-End: 1592879864581-1592879869060
Time Cost: 4479ms
Second Account Return Message: Policy check failed
Begin-End: 1592879871632-1592879874061
Time Cost: 2429ms

The 16-th request
First Account Return Message: Access authorized
Begin-End: 1592879994607-1592879999031
Time Cost: 4424ms
Second Account Return Message: Policy check failed
Begin-End: 1592880001566-1592880004037
Time Cost: 2471ms

The 17-th request
First Account Return Message: Access authorized
Begin-End: 1592880124518-1592880129085
Time Cost: 4567ms
Second Account Return Message: Policy check failed
Begin-End: 1592880131601-1592880134061
Time Cost: 2460ms

The 18-th request
First Account Return Message: Access authorized
Begin-End: 1592880254598-1592880259041
Time Cost: 4443ms
Second Account Return Message: Policy check failed
Begin-End: 1592880261527-1592880264030
Time Cost: 2503ms

The 19-th request
First Account Return Message: Access authorized
Begin-End: 1592880384726-1592880389048
Time Cost: 4322ms
Second Account Return Message: Policy check failed
Begin-End: 1592880391578-1592880394049
Time Cost: 2471ms

The 20-th request
First Account Return Message: Access authorized
Begin-End: 1592880514551-1592880519036
Time Cost: 4485ms
Second Account Return Message: Policy check failed
Begin-End: 1592880521586-1592880524036
Time Cost: 2450ms

The 21-th request
First Account Return Message: Access authorized
Begin-End: 1592880644511-1592880649057
Time Cost: 4546ms
Second Account Return Message: Policy check failed
Begin-End: 1592880651617-1592880654044
Time Cost: 2427ms

The 22-th request
First Account Return Message: Access authorized
Begin-End: 1592880774539-1592880779032
Time Cost: 4493ms
Second Account Return Message: Policy check failed
Begin-End: 1592880781586-1592880784058
Time Cost: 2472ms

The 23-th request
First Account Return Message: Access authorized
Begin-End: 1592880904609-1592880909068
Time Cost: 4459ms
Second Account Return Message: Policy check failed
Begin-End: 1592880911741-1592880914039
Time Cost: 2298ms

The 24-th request
First Account Return Message: Access authorized
Begin-End: 1592881034517-1592881039100
Time Cost: 4583ms
Second Account Return Message: Policy check failed
Begin-End: 1592881041630-1592881044167
Time Cost: 2537ms

The 25-th request
First Account Return Message: Access authorized
Begin-End: 1592881164740-1592881169095
Time Cost: 4355ms
Second Account Return Message: Policy check failed
Begin-End: 1592881171671-1592881174033
Time Cost: 2362ms

The 26-th request
First Account Return Message: Access authorized
Begin-End: 1592881294545-1592881299043
Time Cost: 4498ms
Second Account Return Message: Policy check failed
Begin-End: 1592881301570-1592881304079
Time Cost: 2509ms

The 27-th request
First Account Return Message: Access authorized
Begin-End: 1592881424598-1592881429042
Time Cost: 4444ms
Second Account Return Message: Policy check failed
Begin-End: 1592881431584-1592881434060
Time Cost: 2476ms

The 28-th request
First Account Return Message: Access authorized
Begin-End: 1592881554578-1592881559058
Time Cost: 4480ms
Second Account Return Message: Policy check failed
Begin-End: 1592881561578-1592881564059
Time Cost: 2481ms

The 29-th request
First Account Return Message: Access authorized
Begin-End: 1592881684789-1592881689046
Time Cost: 4257ms
Second Account Return Message: Policy check failed
Begin-End: 1592881691752-1592881694083
Time Cost: 2331ms

The 30-th request
First Account Return Message: Access authorized
Begin-End: 1592881814870-1592881819046
Time Cost: 4176ms
Second Account Return Message: Policy check failed
Begin-End: 1592881821751-1592881824041
Time Cost: 2290ms

```

### 2.2 信誉系统加入的影响

隐私交易时使用了 Quorum for Remix 插件，由于该插件在 Remix 中不会返回结果，在非隐私交易时不具备优势，因此访问控制系统时间测量的预准备工作，包括合约部署和交互，使用了 Remix 自己提供的 Deploy and Run 插件，主要利用 Web3 Provider 连接 Quorum 网络。连接端口已默认打开，我们使用的三个节点对应的端口如下

- Node1：22000
- Node2：22001
- Node3：22002

合约部署完成、设备注册完成、属性定义完成及策略定义的完整记录在仓库中，完成后，可以正式开始测试。三种合约部署的 Gas 消耗分别为

- MC：2512367 gas
- RC：1170616 gas
- ACC：4749983 gas

因为我们要进行时间的计算和一些复杂的交互操作，单独安装使用 Web3.js 来进行访问控制

```bash
# 在用户根目录建立web3文件夹
$ mdkir web3
$ cd web3
# 在web3文件夹中本地安装 web3 1.2.8 版本，之前的版本有些部件不再维护，安装会出错，1.2.9某些功能出错
$ npm install web3@1.2.8
```

我们第一次尝试了直接在 JS 脚本中设立循环，使用 setInterval() 函数延时 120s 发起访问请求，但这种方式第一次访问控制的时间和之后的时间存在较大的差距，第一次普遍在 10ms 左右，其后迅速减少，在 1ms 和 2ms 浮动，目前不清楚原因，可能有缓存，因此采用编写 Shell 脚本循环调用 JS 脚本的方式。

注：这里之所以延时 120s，是因为两次访问控制间隔少于 100s 会触发 Too frequent request 错误。这是我们自定义的一种恶意行为，为了获取足够的访问控制时间测量样本，我们设置延时。

完成访问控制的 JS 脚本和循环调用 JS 脚本的 Shell 脚本也放在了仓库中。执行 Shell 脚本之前，首先解锁发起访问的账户，由于我们设置了延时 120s，计划测量 30 个值，因此一次性将账户解锁 1小时以上，这里我们设置 4000s。注意，设备发起访问控制时，应当首先获取目标设备绑定的 ACC 合约地址，该地址可以在 MC 中根据设备账户查询得到。

首先在测试前解锁相应的账户

```js
> personal.unlockAccount(eth.accounts[0],"",3600)
> personal.unlockAccount(eth.accounts[1],"",3600)
```

以第一种方式计，30次合法访问平均访问时间为 10.9 ms，最坏时间为 26ms；30 次非法访问测试30次的平均访问时间 16.7 ms，最坏时间 119ms

而以获取事件为标准，结果如下

```bash
$ ./runscript.sh
Note: Every 2 minutes, send an access request

The 1-th request
First Account Return Message: Access authorized
Begin-End: 1592883410702-1592883413091
Time Cost: 2389ms
Second Account Return Message: Policy check failed
Begin-End: 1592883415679-1592883418060
Time Cost: 2381ms

The 2-th request
First Account Return Message: Access authorized
Begin-End: 1592883538991-1592883543058
Time Cost: 4067ms
Second Account Return Message: Policy check failed
Begin-End: 1592883545743-1592883548053
Time Cost: 2310ms

The 3-th request
First Account Return Message: Access authorized
Begin-End: 1592883668776-1592883673069
Time Cost: 4293ms
Second Account Return Message: Policy check failed
Begin-End: 1592883675659-1592883678070
Time Cost: 2411ms

The 4-th request
First Account Return Message: Access authorized
Begin-End: 1592883798578-1592883803077
Time Cost: 4499ms
Second Account Return Message: Policy check failed
Begin-End: 1592883805549-1592883808081
Time Cost: 2532ms

The 5-th request
First Account Return Message: Access authorized
Begin-End: 1592883928698-1592883933347
Time Cost: 4649ms
Second Account Return Message: Policy check failed
Begin-End: 1592883935926-1592883938111
Time Cost: 2185ms

The 6-th request
First Account Return Message: Access authorized
Begin-End: 1592884058755-1592884063082
Time Cost: 4327ms
Second Account Return Message: Policy check failed
Begin-End: 1592884065655-1592884068105
Time Cost: 2450ms

The 7-th request
First Account Return Message: Access authorized
Begin-End: 1592884188585-1592884193080
Time Cost: 4495ms
Second Account Return Message: Policy check failed
Begin-End: 1592884195653-1592884198091
Time Cost: 2438ms

The 8-th request
First Account Return Message: Access authorized
Begin-End: 1592884318688-1592884323057
Time Cost: 4369ms
Second Account Return Message: Policy check failed
Begin-End: 1592884325596-1592884328072
Time Cost: 2476ms

The 9-th request
First Account Return Message: Access authorized
Begin-End: 1592884448570-1592884453082
Time Cost: 4512ms
Second Account Return Message: Policy check failed
Begin-End: 1592884455601-1592884458065
Time Cost: 2464ms

The 10-th request
First Account Return Message: Access authorized
Begin-End: 1592884578596-1592884583030
Time Cost: 4434ms
Second Account Return Message: Policy check failed
Begin-End: 1592884585559-1592884588043
Time Cost: 2484ms

The 11-th request
First Account Return Message: Access authorized
Begin-End: 1592884708597-1592884713124
Time Cost: 4527ms
Second Account Return Message: Policy check failed
Begin-End: 1592884715840-1592884718041
Time Cost: 2201ms

The 12-th request
First Account Return Message: Access authorized
Begin-End: 1592884838854-1592884843156
Time Cost: 4302ms
Second Account Return Message: Policy check failed
Begin-End: 1592884845940-1592884848098
Time Cost: 2158ms

The 13-th request
First Account Return Message: Access authorized
Begin-End: 1592884968696-1592884973094
Time Cost: 4398ms
Second Account Return Message: Policy check failed
Begin-End: 1592884975668-1592884978101
Time Cost: 2433ms

The 14-th request
First Account Return Message: Access authorized
Begin-End: 1592885098668-1592885103068
Time Cost: 4400ms
Second Account Return Message: Policy check failed
Begin-End: 1592885105627-1592885108061
Time Cost: 2434ms

The 15-th request
First Account Return Message: Access authorized
Begin-End: 1592885228959-1592885233066
Time Cost: 4107ms
Second Account Return Message: Policy check failed
Begin-End: 1592885235785-1592885238073
Time Cost: 2288ms

The 16-th request
First Account Return Message: Access authorized
Begin-End: 1592885358663-1592885363063
Time Cost: 4400ms
Second Account Return Message: Policy check failed
Begin-End: 1592885365668-1592885368093
Time Cost: 2425ms

The 17-th request
First Account Return Message: Access authorized
Begin-End: 1592885488575-1592885493037
Time Cost: 4462ms
Second Account Return Message: Policy check failed
Begin-End: 1592885495515-1592885498053
Time Cost: 2538ms

The 18-th request
First Account Return Message: Access authorized
Begin-End: 1592885618503-1592885623068
Time Cost: 4565ms
Second Account Return Message: Policy check failed
Begin-End: 1592885625534-1592885628090
Time Cost: 2556ms

The 19-th request
First Account Return Message: Access authorized
Begin-End: 1592885748631-1592885753036
Time Cost: 4405ms
Second Account Return Message: Policy check failed
Begin-End: 1592885755591-1592885758081
Time Cost: 2490ms

The 20-th request
First Account Return Message: Access authorized
Begin-End: 1592885878582-1592885883076
Time Cost: 4494ms
Second Account Return Message: Policy check failed
Begin-End: 1592885885587-1592885888045
Time Cost: 2458ms

The 21-th request
First Account Return Message: Access authorized
Begin-End: 1592886008502-1592886013057
Time Cost: 4555ms
Second Account Return Message: Policy check failed
Begin-End: 1592886015619-1592886018081
Time Cost: 2462ms

The 22-th request
First Account Return Message: Access authorized
Begin-End: 1592886138672-1592886148736
Time Cost: 10064ms
Second Account Return Message: Policy check failed
Begin-End: 1592886151352-1592886158603
Time Cost: 7251ms

The 23-th request
First Account Return Message: Access authorized
Begin-End: 1592886282400-1592886285013
Time Cost: 2613ms
Second Account Return Message: Policy check failed
Begin-End: 1592886287515-1592886293054
Time Cost: 5539ms

The 24-th request
First Account Return Message: Access authorized
Begin-End: 1592886413508-1592886418085
Time Cost: 4577ms
Second Account Return Message: Policy check failed
Begin-End: 1592886420543-1592886423137
Time Cost: 2594ms

The 25-th request
First Account Return Message: Access authorized
Begin-End: 1592886543681-1592886548044
Time Cost: 4363ms
Second Account Return Message: Policy check failed
Begin-End: 1592886550578-1592886553043
Time Cost: 2465ms

The 26-th request
First Account Return Message: Access authorized
Begin-End: 1592886673596-1592886678044
Time Cost: 4448ms
Second Account Return Message: Policy check failed
Begin-End: 1592886680590-1592886683046
Time Cost: 2456ms

The 27-th request
First Account Return Message: Access authorized
Begin-End: 1592886803616-1592886808043
Time Cost: 4427ms
Second Account Return Message: Policy check failed
Begin-End: 1592886810652-1592886813046
Time Cost: 2394ms

The 28-th request
First Account Return Message: Access authorized
Begin-End: 1592886933495-1592886938066
Time Cost: 4571ms
Second Account Return Message: Policy check failed
Begin-End: 1592886940528-1592886943155
Time Cost: 2627ms

The 29-th request
First Account Return Message: Access authorized
Begin-End: 1592887063740-1592887068076
Time Cost: 4336ms
Second Account Return Message: Policy check failed
Begin-End: 1592887070528-1592887073060
Time Cost: 2532ms

The 30-th request
First Account Return Message: Access authorized
Begin-End: 1592887193652-1592887198033
Time Cost: 4381ms
Second Account Return Message: Policy check failed
Begin-End: 1592887200536-1592887203073
Time Cost: 2537ms
```

综上，我们得到了下表，表中数字的单位为 ms，可以看到，信誉系统的引入没有什么影响，反而是合法与非法的访问时间相差近一倍，原因我们暂时无法解释。

| 单位/ms | 合法访问时间 | 非法访问时间 | 加入信誉系统合法访问时间 | 加入信誉系统非法访问时间 |
| ------- | ------------ | ------------ | ------------------------ | ------------------------ |
| 平均值  | 4410         | 2434         | 4481                     | 2693                     |
| 最坏    | 4583         | 2537         | 10064                    | 7251                     |
| 最好    | 3376         | 2179         | 2386                     | 2158                     |
| 中位数  | 4479         | 2466         | 4416                     | 2457                     |

### 2.3 传统架构的时间

在下面的论文中，wang 等利用智能合约对传统 ABAC 架构进行了实现，我们认为对比该方案和我们的方案具有较大的意义。

```
P. Wang, Y. Yue, W. Sun, and J. Liu, 
“An Attribute-Based Distributed Access Control for Blockchain-enabled IoT,” 
in 2019 WiMob, Barcelona, Spain, Oct. 2019, pp. 1–6, doi: 10.1109/WiMOB.2019.8923232 .
```

鉴于作者没有提供源码，我们按照论文的描述进行了复现，然后在我们当前的实验平台下测试其访问控制时间，同样的，对合法的访问控制和非法的访问控制，我们各测试30次。

未完待续...