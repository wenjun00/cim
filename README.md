

<div align="center">  

<img src="https://ws4.sinaimg.cn/large/006tNbRwly1fylahz0rrgj30p008ca9x.jpg"  /> 
<br/>

[![Build Status](https://img.shields.io/badge/cim-cross--im-brightgreen.svg)](https://github.com/crossoverJie/cim)
[![QQ群](https://img.shields.io/badge/QQ%E7%BE%A4-787381170-yellowgreen.svg)](https://jq.qq.com/?_wv=1027&k=5HPYvQk)
[![](https://badge.juejin.im/entry/5c2c000e6fb9a049f5713e26/likes.svg?style=flat-square)](https://juejin.im/post/5c2bffdc51882509181395d7)

📘[介绍](#介绍) |📽[视频演示](#视频演示) | 🏖[TODO LIST](#todo-list) | 🌈[系统架构](#系统架构) |💡[流程图](#流程图)|🌁[快速启动](#快速启动)|👨🏻‍✈️[内置命令](#客户端内置命令)|🎤[通信](#群聊私聊)|❓[QA](https://github.com/crossoverJie/cim/blob/master/doc/QA.md)|💌[联系作者](#联系作者)


</div>
<br/>


## 介绍

`CIM(CROSS-IM)` 一款面向开发者的 `IM(即时通讯)`系统；同时提供了一些组件帮助开发者构建一款属于自己可水平扩展的 `IM` 。

借助 `CIM` 你可以实现以下需求：

- `IM` 即时通讯系统。
- 适用于 `APP` 的消息推送中间件。
- `IOT` 海量连接场景中的消息透传中间件。

> 我有在公网部署了一套演示环境，想要体验的可以[联系我](#联系作者)加入内测群获取账号。

## 视频演示

> 点击下方链接可以查看视频版 Demo。

| YouTube | Bilibili|
| :------:| :------: | 
| [群聊](https://youtu.be/_9a4lIkQ5_o) [私聊](https://youtu.be/kfEfQFPLBTQ) | [群聊](https://www.bilibili.com/video/av39405501) [私聊](https://www.bilibili.com/video/av39405821) | 
| <img src="https://ws3.sinaimg.cn/large/006tNbRwly1fys8flaofrj315e0ose81.jpg"  height="295px" />  | <img src="https://ws4.sinaimg.cn/large/006tNbRwly1fys8mpa6wij31240lghdt.jpg" height="295px" />


## TODO LIST

* [x] [群聊](#群聊)。
* [x] [私聊](#私聊)。
* [x] [内置命令](#客户端内置命令)。
* [x] [聊天记录查询](#聊天记录查询)。
* [x] [一键开启价值 2 亿的 `AI` 模式](#ai-模式)。
* [x] 使用 `Google Protocol Buffer` 高效编解码。
* [x] 根据实际情况灵活的水平扩容、缩容。
* [x] 路由(`cim-forward-route`)服务自身是无状态，可用 `Nginx` 代理支持高可用。
* [x] 服务端自动剔除离线客户端。
* [x] 客户端自动重连。
* [ ] 分组群聊。
* [ ] Android SDK。
* [ ] 离线消息。
* [ ] 协议支持消息加密。
* [ ] 更多的客户端路由策略。



## 系统架构

![](https://ws1.sinaimg.cn/large/006tNbRwly1fyldgiizhuj315o0r4n0k.jpg)

- `CIM` 中的各个组件均采用 `SpringBoot` 构建。
-  采用 `Netty` 构建底层通信。
-  `Redis` 存放各个客户端的路由信息、账号信息、在线状态等。
-  `Zookeeper` 用于 `IM-server` 服务的注册与发现。


### cim-server

`IM` 服务端；用于接收 `client` 连接、消息透传、消息推送等功能。

**支持集群部署。**

### cim-forward-route

消息路由服务器；用于处理消息路由、消息转发、用户登录、用户下线以及一些运营工具（获取在线用户数等）。

### cim-client

`IM` 客户端；给用户使用的消息终端，一个命令即可启动并向其他人发起通讯（群聊、私聊）。

## 流程图

![](https://ws1.sinaimg.cn/large/006tNbRwly1fylfxevl2ij30it0etaau.jpg)

- 客户端向 `route` 发起登录。
- 登录成功从 `Zookeeper` 中选择可用 `IM-server` 返回给客户端，并保存登录、路由信息到 `Redis`。
- 客户端向 `IM-server` 发起长连接，成功后保持心跳。
- 客户端下线时通过 `route` 清除状态信息。


## 快速启动

首先需要安装 `Zookeeper、Redis` 并保证网络通畅。

```shell
git clone https://github.com/crossoverJie/cim.git
cd cim
mvn -Dmaven.test.skip=true clean package
```

### 部署 IM-server(cim-server)

```shell
cp /cim/cim-server/target/cim-server-1.0.0-SNAPSHOT.jar /xx/work/server0/
cd /xx/work/server0/
nohup java -jar  /root/work/server0/cim-server-1.0.0-SNAPSHOT.jar --cim.server.port=9000 --app.zk.addr=zk地址  > /root/work/server0/log.file 2>&1 &
```

> cim-server 集群部署同理，只要保证 Zookeeper 地址相同即可。

### 部署路由服务器(cim-forward-route)

```shell
cp /cim/cim-server/cim-forward-route/target/cim-forward-route-1.0.0-SNAPSHOT.jar /xx/work/route0/
cd /xx/work/route0/
nohup java -jar  /root/work/route0/cim-forward-route-1.0.0-SNAPSHOT.jar --app.zk.addr=zk地址 --spring.redis.host=redis地址 --spring.redis.port=6379  > /root/work/route/log.file 2>&1 &
```

> cim-forward-route 本身就是无状态，可以部署多台；使用 Nginx 代理即可。


### 启动客户端

```shell
cp /cim/cim-client/target/cim-client-1.0.0-SNAPSHOT.jar /xx/work/route0/
cd /xx/work/route0/
java -jar cim-client-1.0.0-SNAPSHOT.jar --server.port=8084 --cim.user.id=唯一客户端ID --cim.user.userName=用户名 --cim.group.route.request.url=http://路由服务器:8083/groupRoute --cim.server.route.request.url=http://路由服务器:8083/login
```

![](https://ws2.sinaimg.cn/large/006tNbRwly1fylgxjgshfj31vo04m7p9.jpg)
![](https://ws1.sinaimg.cn/large/006tNbRwly1fylgxu0x4uj31hy04q75z.jpg)

如上图，启动两个客户端可以互相通信即可。

### 本地启动客户端

#### 注册账号
```shell
curl -X POST --header 'Content-Type: application/json' --header 'Accept: application/json' -d '{
  "reqNo": "1234567890",
  "timeStamp": 0,
  "userName": "zhangsan"
}' 'http://路由服务器:8083/registerAccount'
```

从返回结果中获取 `userId`

```json
{
    "code":"9000",
    "message":"成功",
    "reqNo":null,
    "dataBody":{
        "userId":1547028929407,
        "userName":"test"
    }
}
```

#### 启动本地客户端
```shell
# 启动本地客户端
cp /cim/cim-client/target/cim-client-1.0.0-SNAPSHOT.jar /xx/work/route0/
cd /xx/work/route0/
java -jar cim-client-1.0.0-SNAPSHOT.jar --server.port=8084 --cim.user.id=上方返回的userId --cim.user.userName=用户名 --cim.group.route.request.url=http://路由服务器:8083/groupRoute --cim.server.route.request.url=http://路由服务器:8083/login
```

## 客户端内置命令

| 命令 | 描述|
| ------ | ------ | 
| `:q!` | 退出客户端| 
| `:olu` | 获取所有在线用户信息 | 
| `:all` | 获取所有命令 | 
| `:q` | 【:q 关键字】查询聊天记录 | 
| `:ai` | 开启 AI 模式 | 
| `:qai` | 关闭 AI 模式 | 
| `:pu` | 模糊匹配用户 | 
| `:info` | 获取客户端信息 | 
| `:` | 更多命令正在开发中。。 | 

![](https://ws3.sinaimg.cn/large/006tNbRwly1fylh7bdlo6g30go01shdt.gif)

### 聊天记录查询

![](https://ws2.sinaimg.cn/large/006tNc79gy1fz3uwmb5hsj30s8046wm3.jpg)

使用命令 `:q 关键字` 即可查询与个人相关的聊天记录。

> 客户端聊天记录默认存放在 `/opt/logs/cim/`，所以需要这个目录的写入权限。也可在启动命令中加入 `--cim.msg.logger.path = /自定义` 参数自定义目录。



### AI 模式

![](https://ws3.sinaimg.cn/large/006tNc79gy1fz3vf3nsq3j31dc0j01ky.jpg)

使用命令 `:ai` 开启 AI 模式，之后所有的消息都会由 `AI` 响应。

`:qai` 退出 AI 模式。

### 前缀匹配用户名

![](https://ws4.sinaimg.cn/large/006tNc79gy1fz3vo4tgkjj31ni09s41u.jpg)

使用命令 `:qu prefix` 可以按照前缀的方式搜索用户信息。

> 该功能主要用于在移动端中的输入框中搜索用户。 

## 群聊/私聊

### 群聊

![](https://ws1.sinaimg.cn/large/006tNbRwly1fyli54e8e1j31t0056x11.jpg)
![](https://ws3.sinaimg.cn/large/006tNbRwly1fyli5yyspmj31im06atb8.jpg)
![](https://ws3.sinaimg.cn/large/006tNbRwly1fyli6sn3c8j31ss06qmzq.jpg)

群聊只需要在控制台里输入消息回车后即可发送，同时所有在线客户端都可收到消息。

### 私聊

私聊首先需要知道对方的 `userID` 才能进行。

输入命令 `:olu` 可列出所有在线用户。

![](https://ws4.sinaimg.cn/large/006tNbRwly1fyli98mlf3j31ta06mwhv.jpg)

接着使用 `userId;;消息内容` 的格式即可发送私聊消息。

![](https://ws4.sinaimg.cn/large/006tNbRwly1fylib08qlnj31sk082zo6.jpg)
![](https://ws1.sinaimg.cn/large/006tNbRwly1fylibc13etj31wa0564lp.jpg)
![](https://ws3.sinaimg.cn/large/006tNbRwly1fylicmjj6cj31wg07c4qp.jpg)
![](https://ws1.sinaimg.cn/large/006tNbRwly1fylicwhe04j31ua03ejsv.jpg)

同时另一个账号收不到消息。
![](https://ws3.sinaimg.cn/large/006tNbRwly1fylie727jaj31t20dq1ky.jpg)






## 联系作者
- [crossoverJie@gmail.com](mailto:crossoverJie@gmail.com)
- 微信公众号

![](https://ws1.sinaimg.cn/large/006tKfTcly1ftmfdo6mhmj30760760t7.jpg)


### Code Visualization:

Here is a cool visualization of the code evolution

 [![Watch the video](https://img.youtube.com/vi/NhV_brPIG74/0.jpg)](https://www.youtube.com/watch?v=NhV_brPIG74)

 [https://www.youtube.com/watch?v=NhV_brPIG74](https://www.youtube.com/watch?v=NhV_brPIG74)
 
 
 
 前言
记得一年前分享过一篇《一致性 Hash 算法分析》，当时只是分析了这个算法的实现原理、解决了什么问题等。

但没有实际实现一个这样的算法，毕竟要加深印象还得自己撸一遍，于是本次就当前的一个路由需求来着手实现一次。

背景
看过《为自己搭建一个分布式 IM(即时通讯) 系统》的朋友应该对其中的登录逻辑有所印象。

先给新来的朋友简单介绍下 cim 是干啥的：



其中有一个场景是在客户端登录成功后需要从可用的服务端列表中选择一台服务节点返回给客户端使用。

而这个选择的过程就是一个负载策略的过程；第一版本做的比较简单，默认只支持轮询的方式。

虽然够用，但不够优雅😏。

因此我的规划是内置多种路由策略供使用者根据自己的场景选择，同时提供简单的 API 供用户自定义自己的路由策略。

先来看看一致性 Hash 算法的一些特点：

构造一个 0~2^32-1 大小的环。

服务节点经过 hash 之后将自身存放到环中的下标中。

客户端根据自身的某些数据 hash 之后也定位到这个环中。

通过顺时针找到离他最近的一个节点，也就是这次路由的服务节点。

考虑到服务节点的个数以及 hash 算法的问题导致环中的数据分布不均匀时引入了虚拟节点。



自定义有序 Map
根据这些客观条件我们很容易想到通过自定义一个有序数组来模拟这个环。

这样我们的流程如下：

初始化一个长度为 N 的数组。

将服务节点通过 hash 算法得到的正整数，同时将节点自身的数据（hashcode、ip、端口等）存放在这里。

完成节点存放后将整个数组进行排序（排序算法有多种）。

客户端获取路由节点时，将自身进行 hash 也得到一个正整数；

遍历这个数组直到找到一个数据大于等于当前客户端的 hash 值，就将当前节点作为该客户端所路由的节点。

如果没有发现比客户端大的数据就返回第一个节点（满足环的特性）。

先不考虑排序所消耗的时间，单看这个路由的时间复杂度：

最好是第一次就找到，时间复杂度为 O(1)。

最差为遍历完数组后才找到，时间复杂度为 O(N)。

理论讲完了来看看具体实践。

我自定义了一个类： SortArrayMap

他的使用方法及结果如下：





可见最终会按照 key 的大小进行排序，同时传入 hashcode=101 时会按照顺时针找到 hashcode=1000 这个节点进行返回。

下面来看看具体的实现。

成员变量和构造函数如下：



其中最核心的就是一个 Node 数组，用它来存放服务节点的 hashcode 以及 value 值。

其中的内部类 Node 结构如下：



写入数据的方法如下：



相信看过 ArrayList 的源码应该有印象，这里的写入逻辑和它很像。

写入之前判断是否需要扩容，如果需要则复制原来大小的 1.5 倍数组来存放数据。

之后就写入数组，同时数组大小 +1。

但是存放时是按照写入顺序存放的，遍历时自然不会有序；因此提供了一个 Sort 方法，可以把其中的数据按照 key 其实也就是 hashcode 进行排序。



排序也比较简单，使用了 Arrays 这个数组工具进行排序，它其实是使用了一个 TimSort 的排序算法，效率还是比较高的。

最后则需要按照一致性 Hash 的标准顺时针查找对应的节点：



代码还是比较简单清晰的；遍历数组如果找到比当前 key 大的就返回，没有查到就取第一个。

这样就基本实现了一致性 Hash 的要求。

ps:这里并不包含具体的 hash 方法以及虚拟节点等功能（具体实现请看下文），这个可以由使用者来定，SortArrayMap 可作为一个底层的数据结构，提供有序 Map 的能力，使用场景也不局限于一致性 Hash 算法中。

TreeMap 实现
SortArrayMap 虽说是实现了一致性 hash 的功能，但效率还不够高，主要体现在 sort 排序处。

下图是目前主流排序算法的时间复杂度：



最好的也就是 O(N) 了。

这里完全可以换一个思路，不用对数据进行排序；而是在写入的时候就排好顺序，只是这样会降低写入的效率。

比如二叉查找树，这样的数据结构 jdk 里有现成的实现；比如 TreeMap 就是使用红黑树来实现的，默认情况下它会对 key 进行自然排序。

来看看使用 TreeMap 如何来达到同样的效果。运行结果：

127.0
.
0.1000

效果和上文使用 SortArrayMap 是一致的。

只使用了 TreeMap 的一些 API：

写入数据候， TreeMap 可以保证 key 的自然排序。

tailMap 可以获取比当前 key 大的部分数据。

当这个方法有数据返回时取第一个就是顺时针中的第一个节点了。

如果没有返回那就直接取整个 Map 的第一个节点，同样也实现了环形结构。

ps:这里同样也没有 hash 方法以及虚拟节点（具体实现请看下文），因为 TreeMap 和 SortArrayMap 一样都是作为基础数据结构来使用的。

性能对比
为了方便大家选择哪一个数据结构，我用 TreeMap 和 SortArrayMap 分别写入了一百万条数据来对比。

先是 SortArrayMap：



耗时 2237 毫秒。

TreeMap：



耗时 1316毫秒。

结果是快了将近一倍，所以还是推荐使用 TreeMap 来进行实现，毕竟它不需要额外的排序损耗。

cim 中的实际应用
下面来看看在 cim 这个应用中是如何具体使用的，其中也包括上文提到的虚拟节点以及 hash 算法。

模板方法
在应用的时候考虑到就算是一致性 hash 算法都有多种实现，为了方便其使用者扩展自己的一致性 hash 算法因此我定义了一个抽象类；其中定义了一些模板方法，这样大家只需要在子类中进行不同的实现即可完成自己的算法。

AbstractConsistentHash，这个抽象类的主要方法如下：



add 方法自然是写入数据的。

sort 方法用于排序，但子类也不一定需要重写，比如 TreeMap 这样自带排序的容器就不用。

getFirstNodeValue 获取节点。

process 则是面向客户端的，最终只需要调用这个方法即可返回一个节点。

下面我们来看看利用 SortArrayMap 以及 AbstractConsistentHash 是如何实现的。



就是实现了几个抽象方法，逻辑和上文是一样的，只是抽取到了不同的方法中。

只是在 add 方法中新增了几个虚拟节点，相信大家也看得明白。

把虚拟节点的控制放到子类而没有放到抽象类中也是为了灵活性考虑，可能不同的实现对虚拟节点的数量要求也不一样，所以不如自定义的好。

但是 hash 方法确是放到了抽象类中，子类不用重写；因为这是一个基本功能，只需要有一个公共算法可以保证他散列地足够均匀即可。

因此在 AbstractConsistentHash 中定义了 hash 方法。



这里的算法摘抄自 xxl_job，网上也有其他不同的实现，比如 FNV1_32_HASH 等；实现不同但是目的都一样。

这样对于使用者来说就非常简单了：



他只需要构建一个服务列表，然后把当前的客户端信息传入 process 方法中即可获得一个一致性 hash 算法的返回。

同样的对于想通过 TreeMap 来实现也是一样的套路：



他这里不需要重写 sort 方法，因为自身写入时已经排好序了。

而在使用时对于客户端来说只需求修改一个实现类，其他的啥都不用改就可以了。



运行的效果也是一样的。

这样大家想自定义自己的算法时只需要继承 AbstractConsistentHash 重写相关方法即可，客户端代码无须改动。

路由算法扩展性
但其实对于 cim 来说真正的扩展性是对路由算法来说的，比如它需要支持轮询、hash、一致性hash、随机、LRU等。

只是一致性 hash 也有多种实现，他们的关系就如下图：



应用还需要满足对这一类路由策略的灵活支持，比如我也想自定义一个随机的策略。

因此定义了一个接口： RouteHandle

public
 
interface
 
RouteHandle
 
{



    
/**

     * 再一批服务器里进行路由

     * @param values

     * @param key

     * @return

     */

    
String
 routeServer
(
List
<
String
>
 values
,
String
 key
)
 
;

}

其中只有一个方法，也就是路由方法；入参分别是服务列表以及客户端信息即可。

而对于一致性 hash 算法来说也是只需要实现这个接口，同时在这个接口中选择使用 SortArrayMapConsistentHash 还是 TreeMapConsistentHash 即可。



这里还有一个 setHash 的方法，入参是 AbstractConsistentHash；这就是用于客户端指定需要使用具体的那种数据结构。

而对于之前就存在的轮询策略来说也是同样的实现 RouteHandle 接口。



这里我只是把之前的代码搬过来了而已。

接下来看看客户端到底是如何使用以及如何选择使用哪种算法。

为了使客户端代码几乎不动，我将这个选择的过程放入了配置文件。



如果想使用原有的轮询策略，就配置实现了 RouteHandle 接口的轮询策略的全限定名。

如果想使用一致性 hash 的策略，也只需要配置实现了 RouteHandle 接口的一致性 hash 算法的全限定名。

当然目前的一致性 hash 也有多种实现，所以一旦配置为一致性 hash 后就需要再加一个配置用于决定使用 SortArrayMapConsistentHash 还是 TreeMapConsistentHash 或是自定义的其他方案。

同样的也是需要配置继承了 AbstractConsistentHash 的全限定名。

不管这里的策略如何改变，在使用处依然保持不变。

只需要注入 RouteHandle，调用它的 routeServer 方法。

@Autowired

private
 
RouteHandle
 routeHandle 
;

String
 server 
=
 routeHandle
.
routeServer
(
serverCache
.
getAll
(),
String
.
valueOf
(
loginReqVO
.
getUserId
()));

既然使用了注入，那其实这个策略切换的过程就在创建 RouteHandlebean 的时候完成的。



也比较简单，需要读取之前的配置文件来动态生成具体的实现类，主要是利用反射完成的。

这样处理之后就比较灵活了，比如想新建一个随机的路由策略也是同样的套路；到时候只需要修改配置即可。

感兴趣的朋友也可提交 PR 来新增更多的路由策略。

总结
希望看到这里的朋友能对这个算法有所理解，同时对一些设计模式在实际的使用也能有所帮助。

相信在金三银四的面试过程中还是能让面试官眼前一亮的，毕竟根据我这段时间的面试过程来看听过这个名词的都在少数😂（可能也是和候选人都在 1~3 年这个层级有关）。

以上所有源码：

https://github.com/crossoverJie/cim

如果本文对你有所帮助还请不吝转发。

