# LeanCloud 数据存储服务总览

感谢你选用 LeanCloud 数据存储服务。你可以通过使用我们提供的 SDK，一行后端代码都不用写，而快速完成一个产品（网站或应用）的开发和发布。

## 文档贡献

如果觉得这个文档写的不够好，也可以帮助我们来不断完善。

Github 仓库地址：[https://github.com/leancloud/docs](https://github.com/leancloud/docs)

## 初衷

大部分的产品都是数据驱动的，它们有一个最大的特点，就是对后端的需求在模式上其实是比较统一的：

* 前端负责数据展现和用户交互处理，与后端的 app server 通过网络来交换需要的数据；
* app server 负责业务逻辑处理，生成核心数据存储到 data server，或者聚合 data server 查询到的数据返回给客户端；
* data server 负责核心数据的存储和备份；

这样的模式适合互联网上绝大部分产品，虽然数据结构有差异、业务逻辑不一样，但是前后端交互的主体——「数据」，抽象来看是一致的，后端的架构（譬如 LAMP）也是大同小异的，而且同样的系统在一遍一遍地被重复开发，极大浪费了我们宝贵的技术资源。

## LeanCloud 的解决方案

LeanCloud 对大部分场景下的后端需求进行了抽象和统一，我们通过四大系统来实现一个通用、强大、可定制的 BaaS(Backend as a Service) 服务：

1. **面向对象的海量数据库**
前后端交互的主体，都是「数据」，不管结果多少，属性具体含义如何，它们都可以抽象成统一的「对象」来处理。LeanCloud 支持存储任意类型的对象，支持对象的增、删、改、查等多种操作，并且开发者无需担心数据规模的大小和访问流量的多少，可以简单将 LeanCloud 云端看成是一个面向对象的海量数据库来使用。

2. **大文件存储和分发**
任何一款产品，不管是网站、应用还是游戏，都有一些素材或者文件需要存储和分发。与应用内数据不一样，这些文件因为它的体积较大，为了获得更快捷的用户体验，一般都还需要 [CDN](http://baike.baidu.com/view/21895.htm) 服务。LeanCloud 存储系统完整涵盖了大文件存储和分发的需求。

3. **LeanEngine 完成特定业务逻辑**
LeanCloud 提供的数据操作 API 能覆盖大部分业务的需求，但是凡事总有例外，这些标准 API 有时候并不能完全满足某些特定需求，这时候怎么办？LeanCloud 还提供了「LeanEngine」自定义服务端业务逻辑的功能。LeanEngine 与大家熟知的 [Google App Engine](http://baike.baidu.com/view/1524918.htm) 相似，允许开发者写很少的一部分代码，来完成业务特有逻辑。这些代码会被部署到 LeanCloud 云端，与 LeanCloud 标准服务一起执行，来实现特殊需求。

4. **离线数据分析平台**
对于完全构建在 LeanCloud 上的产品来讲，在运行一段时间之后会积累大量的业务数据，这时候产品和运营层面都会产生一些数据挖掘或商业智能分析的需求，此时如何才能简便地操作云端，看到数据背后隐藏的趋势和价值呢？为此我们推出了分布式的「离线数据分析系统」，支持在应用数据集上进行各种处理和操作。离线分析系统是完全的分布式、实时计算系统，其执行效率和处理的数据规模远在 [Hadoop MapReduce](https://hadoop.apache.org/docs/r1.2.1/mapred_tutorial.html) 之上。

下面我们重点来了解一下存储系统的几个核心概念。

## 对象：AVObject

我们将客户端与服务端交互的纷繁芜杂的数据，统一抽象成「对象」，在我们的数据模型里面，对应的就是「AVObject」，每个 AVObject 都包含与 JSON 相兼容的键值对（key-value）数据，并有一个类别名（Class Name）。LeanCloud 云端对于 AVObject 里面「键」的数量并没有限制，也不要求每一个 AVObject 里面都包含所有的「键」。譬如在一个游戏里面，我们有多种数据需要保存：

* 账户相关的数据，包含用户名、密码、性别等
* 游戏分数相关的数据，包含用户、得分、关联关卡等

我们可以全部存放在一种 AVObject 对象里面（假定类别名：AllInOne）：

```
// 玩家信息，包括姓名、密码、性别等
{"username":"steve", "password":"xxxxx", "gender":"Male"}
// 游戏得分数据，包括分数、玩家、模式、关卡等
{"score":1337, "playerName":"steve", "cheatMode":false, "scene":4}
```

你可以把这两条记录都保存到一种 AVObject 里面，这在 LeanCloud 云端是被允许的。但是我们**强烈建议开发者将不同类型的数据根据用途进行分类，形成不同种类的 「AVObject」**。譬如上面的例子，我们可以抽象出 `User` 这种对象来保存账户信息，抽象出 `GameScore` 这种对象来保存得分信息。

每种「AVObject」对应 LeanCloud 云端的一张「表」（注意：为了便于理解，这里借用了关系型数据库里面表的概念，在 LeanCloud 存储管理控制台，是按 Class 来区分的），同一类对象的所有数据都存放在一起。每一个 AVObject 的实例保存到 LeanCloud 云端之后，LeanCloud 云端会赋予该实例一个全局唯一的 `objectId`，这个 `objectId` 等同于关系型数据库中的主键。同时 LeanCloud 云端还会自动为该实例添加两个属性：`createdAt`和`updatedAt`，分别表示实例创建时间和最后更新时间。

你可以把 LeanCloud 云端看成是一个面向对象的海量数据库，它与传统的关系型数据库的差异有：

* 访问方式不一样，不需要任何 JDBC/ODBC 的驱动，直接通过 HTTP 协议来传输 JSON Object 即可，所以不光服务端可以使用，在客户端也可以直接访问。我们既提供各种平台原生 SDK（iOS, Android, Javascript, Windows Phone, .Net, Unity3D, C++, Python，以及社区贡献的其他语言 SDK）来帮助开发者简单集成数据存储服务，也提供开放的 REST API 供大家直接使用。
* LeanCloud 对于数据的唯一格式要求是满足 JSON Object 的形式，存储新的对象类型时不需要预先在云端定义任何「表结构」，而且同一种数据类型里的键值也是允许随时增加的。这种 schema free 的设计，会给开发者带来最大的便利。
* AVObject 之间没有了主键、外键的概念，也不支持跨表的 join 查询，取而代之的，我们提供另一种数据关联的机制，详见[下文](#数据关联)；
* 既然 AVObject 是面向对象设计的，它的查询就与传统 SQL 不一样，[后文](#数据查询_AVQuery)会有详细说明。不过为了照顾已经习惯了传统关系型数据库查询的开发者，我们也提供了类 SQL 查询的 [Cloud Query Language 查询语法](./cql_guide.html)（简称：CQL）。请注意：LeanCloud 的 CQL 查询语法是 SQL 查询语法的子集和变种，目的是降低大家学习 LeanCloud 查询的 API 的成本，并不是所有 SQL 中可以执行的查询都会在 CQL 中产生相同的结果。

### 有效的数据类型

AVObject 中的`键`，必须是由字母、数字或下划线组成的字符串；开发者自定义的键，不能以 `__`（双下划线）开头。AVObject 中的`值`，可以是字符串、数字、布尔值，或是数组和字典。在平台内部，LeanCloud 将数据存储为 JSON，因此所有能被转换成 JSON 的数据类型都可以保存在 LeanCloud 云端。并且，框架还可以处理日期、Bytes 以及文件类型。总结来说，AVObject 中字段允许的类型包括：

* String 字符串
* Number 数字
* Boolean 布尔类型
* Array 数组
* Object 对象
* Date 日期
* Bytes base64编码的二进制数据
* File  文件
* Null 空值

Object 类型简单地表示每个字段的值都可以由能 JSON 编码的内嵌对象组合而成。对象的 Key（键）包含`$`或者`.`，或者同时有 `__type` 的键，都是系统保留用来做一些额外处理的特殊键，因此请不要在你的对象中使用这样的 Key。另外，`code`、`uuid`、`className`、`keyValues`、`fetchWhenSave`、`running`、`acl`、`ACL`、`isDataReady`、`pendingKeys`、`createdAt`、`updatedAt`、`objectId`、`description` 也都是保留字段，不能作为`键`来使用。

我们的 SDK 会处理原生的 Objective-C 和 Java 类型到 JSON 之间的转换。例如，当你保存一个 NSString 对象的时候，它在我们的系统中会被自动转换成 String 类型。

有两种方式可以存储原生的二进制数据。Bytes 类型允许直接在 AVObject 中关联 NSData 或者 bytes[] 类型的数据。这种方式只推荐用来存储小片的二进制数据。当要保存实际文件（例如图片，视频，文档等），请使用 AVFile 来表示 File 类型，并且 File 类型可以被保存到对象字段中关联起来。

### 数据类型锁定

当一个 Class 初次创建的时候，它不包含任何预先定义并继承的 schema。也就是说对于存储的第一个对象，它的字段可以包含任何有效的类型。

但是，当一个字段被保存至少一次之后，这个字段将被锁定为保存过的数据类型。例如，如果一个 User 对象保存了一个字段 name，类型为 String，那么这个 name 字段将被严格限制为只允许保存 String 类型。（如果你尝试保存其他类型到这个字段，我们的 SDK 会返回一个错误）

一个特例是任何字段都允许被设置为 null，无论它是什么类型。

### 数据管理

[数据管理平台](/data.html?appid={{appid}})是一个允许在任何 app 里更新或者创建对象的 Web UI 管理平台。在这里，你可以看到保存在 Class 里的每个对象的原生 JSON 值。

当使用这个平台的时候，请牢记：

* 输入 "null" 将会设置值为特殊的空值 null，而非字符串 "null"。
* `objectId`, `createdAt` 和 `updatedAt` 不可编辑（它们都是系统自动设置的）。
* 下划线开始的 class 为系统内置 class，不可删除，并且请轻易不要修改它的默认字段，可添加字段。

### 数据导入和导出

我们并不锁定用户和数据，相反，我们提供 API 和工具，来支持用户随时随地导出存放在我们平台上的数据，同样地，数据导入也是可以批量操作的。除了 REST API 之外，我们还提供通过 JSON 文件和 CSV 格式文件的导入数据的功能。具体请参考文档[数据和安全](./data_security.html)。

## 数据关联
上面讲过，AVObject 模型与传统的关系型数据库有一个很大的不同，就是没有了主键、外键的概念，但是对象之间总会存在关联，这时候该如何解决呢？

LeanCloud 中有 4 种方式来构建对象之间的关系：

### Pointer
就是将一个对象 A 存为另一个对象 B 的属性值，这适合用来处理一对一或者一对多的关联关系。譬如

```
AVObject *game= [AVObject objectWithClassName:@"Game"];
[game setObject:[AVUser currentUser] forKey:@"createdBy"];
```

### Array
就是将多个对象 A、B、C 存为另一个对象 D 的属性值，这适合用来处理一对多或者多对多的关联关系。譬如：

```
AVObject *scimitar = ...
AVObject *plasmaRifle = ...
AVObject *grenade = ...
AVObject *bunnyRabbit = ...

NSArray *weapons = @[scimitar, plasmaRifle, grenade, bunnyRabbit];

// 将武器数组存入单个属性之中
[[AVUser currentUser] setObject:weapons forKey:@"weaponsList"];
```

### AVRelation
这是一个专门的关联类，用来建立两种对象之间的关联关系，适合多对多的场景。譬如：

```
// 三个作者
AVObject *authorOne = …
AVObject *authorTwo = …
AVObject *authorThree = …

// 一本书籍
AVObject *book= [AVObject objectWithClassName:@"Book"];

// 建立书籍到作者的关联
AVRelation *relation = [book relationforKey:@"authors"];
[relation addObject:authorOne];
[relation addObject:authorTwo];
[relation addObject:authorThree];

// 保存
[book saveInBackground];
```

### 关联表
使用专门的类，来为两种对象建立关联关系，与 AVRelation 相比它还可以添加更多的附加信息。譬如我们为用户之间关注/被关注的关系建模，就像流行的社交网络那样，一个用户可以关注别的用户。在这里，我们不仅想知道用户 A 是否关注了用户 B，我们还想知道什么时候用户 A 开始关注的用户 B，这时候就适合建立专门的关联表。关联表适合多对多的关联关系。

详细情况请参考我们的技术文章[关系建模指南](./relation_guide.html)。

## 数据查询：AVQuery

AVObject 保存到 LeanCloud 云端之后，如何再次获取到它们呢？这时候需要用到数据查询功能了。前面我们讲过，查询语法因为面向对象的模型不同，与 SQL 语法有较大差异。这时候我们能用的就是 `AVQuery`。AVQuery 的功能非常强大，它可以：

* 检索单一对象，也可以一次检索许多对象；
* 支持各种各样的查询约束；
* 支持分页查询；
* 支持复合查询；
* 支持自动缓存查询结果，保证在网络异常的情况下 UI 也有历史数据可供展现。

除了 AVQuery 之外，我们也提供类 SQL 查询的 [Cloud Query Language 查询语法](./cql_guide.html)（简称：CQL）。请注意：LeanCloud 的 CQL 查询语法是 SQL 查询语法的子集和变种，目的是降低大家学习 LeanCloud 查询的 API 的成本，并不是所有 SQL 中可以执行的查询都会在 CQL 中产生相同的结果。


## 文件存储：AVFile

除了应用内数据存储之外，LeanCloud 云端也支持「文件」类数据的存储。这里的「文件」指的是图片、音乐、视频等常见的文件类型，以及其他任何二进制数据。因为 AVObject 有大小限制，所以超过 **128KB** 的数据不能直接存储到 AVObject 里面；而且，更重要的是，对于图片、音乐、视频类数据，因为他们的体积太大，为了终端用户有快捷的下载体验，都需要额外的 CDN 加速服务，这时候，就需要使用特别的类型「文件」来存储。

LeanCloud 平台用「AVFile」来表示文件。要存储一个文件到 LeanCloud 云端，调用过程也非常简单，譬如：

```
NSData *data = [@"Working with LeanCloud is great!" dataUsingEncoding:NSUTF8StringEncoding];
AVFile *file = [AVFile fileWithName:@"resume.txt" data:data];
[file saveInBackground];
```

## 主要性能指标

LeanCloud 平台保证 99.9% 的高可用性，并且数据访问方面保证了极高的读写性能和极大的数据支持规模：

* 一般数据访问 API 的完成时间都在几十个毫秒内（包括了网络 roundtrip 时间在内）。
* API 访问的并发处理能力无上限。
* 应用内数据支持高达 PB 级别的规模，你完全无需关心传统数据库使用时遇到的「分表」等平行扩展问题；
* 文件存储的网络流量和带宽无限制；

## 原生 SDK

如果你是某个特定平台的开发者，想查看我们原生的 SDK 开发指南，请移步到具体页面：

### iOS 开发指南
详细请参看 [iOS 数据存储开发指南](./leanstorage_guide-ios.html);

### Android 开发指南
详细请参看 [Android 数据存储开发指南](./leanstorage_guide-android.html);

### Javascript 开发指南
详细请参看 [Javascript 数据存储开发指南](./js_guide.html);

### .Net/Unity3D 开发指南
详细请参看 [.Net 数据存储开发指南](./dotnet_guide.html);

### Python 开发指南
详细请参看 [Python 数据存储开发指南](./python_guide.html);

### REST API 说明
详细请参看 [REST API 详细说明](./rest_api.html);


## 常见问题
### 数据安全有保障吗

因为是把自己的核心数据存放到开放的云端，所有用户或多或少都会有这方面的担心：我的数据安全吗？

LeanCloud 其实比你更重视数据安全，我们通过如下技术手段保证你的数据安全可靠：

* 所有数据在云端会存放 3 份拷贝，以保证出现各种硬件、网络故障的时候，数据都是可用的。
* 所有网络请求都是基于 SSL 安全连接（HTTPS）的，保证数据内容不会被轻易截获或者窥探到。
* 应用之间的数据完全隔离，应用 A 数据出现问题不会波及到应用 B，也不会出现应用 A 可以访问到应用 B 的数据的情况（当然明确授权的情况例外）
* 数据访问会进行严格的身份认证。每个应用我们都会生成 appKey 和 masterKey 两套密钥，分别对应开放环境和授信环境的使用场景，所以你可以将某些核心数据的读写只授权给 masterKey。LeanCloud 云端对所有操作都会进行请求发起者身份进行认证，以防御虚假请求的干扰、破坏。
* 在开放环境，譬如 Web 端，我们还支持通过 `Web 安全域名` 来对请求来源做限制，可以简单的防御住 Web 的服务器资源盗取。
* 数据表上提供「行」级别的访问权限控制（ACL），你可以对每一个对象设置不同的读写访问权限（与 Linux 文件系统的 ACL 机制一样，我们也提供 owner、group、public 的读写访问控制）。
* 列级别的访问控制权限。与上面「行」级别的访问权限控制类似，我们还允许你为某张表的部分「列」设定单独的访问控制权限。
* Class 级别的访问控制权限。在一些情况下，设置整个 Class 允许的权限是一种更自然的方式。例如，你可能想设置整个 Class 只读，或者只写。LeanCloud 也允许你设置每个 Class 允许的操作。

具体请参考文档[数据和安全](./data_security.html)。

### 我会被绑定在 LeanCloud 平台吗

除了安全之外，大家担心的第二个问题就是开放。我们把数据存放到这个平台之后，还可以自由导出吗？以后要想搭建自己的后端专属集群了，可以快速地迁出吗？

放心，LeanCloud 秉承「开放、自由」的理念，绝不会故意制造障碍来「绑架」用户。我们提供 API、工具来让用户随时导出或导入自己的数据，我们相信只有可靠的技术和好的服务才能吸引并留住用户。

### 标准 API 满足不了业务需求，怎么办

LeanCloud 标准 API 能满足很多通用的需求，但有时候我有一些特殊的需求，譬如：

* 我要实时查看社交平台内容的热度排名，热度算法是需要融合多种数据来源进行计算的，怎么办？
* 在发生数据变化的时候，要主动触发某一事件。譬如用户 A 评论了用户 B 的照片，这时候我希望能够实时给用户 B 发送一条推送信息，怎么办？

这时候你可以使用我们的 LeanEngine 服务，来扩展标准 API 的功能，具体请参考文档 [云引擎开发指南](leanengine_guide-cloudcode.html)。

### 要对存储的数据进行业务分析，怎么办

我把数据都放到了 LeanCloud 上面，两个月后产品经理想看看某一功能的日均使用情况，或者运营人员要分析用户从注册到消费的时间周期，这时候怎么办？

放心，LeanCloud 提供了 SQL-like 的离线数据分析功能，支持对应用内数据进行任意的处理，来完成数据挖掘和商业智能分析的目的。具体请参考文档 [离线数据分析指南](leaninsight_guide.html)。
