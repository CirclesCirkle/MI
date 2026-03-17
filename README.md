# 基本介绍

依靠CC/CC:Tweaked Mod在Minecraft中使用Lua语言设计的模仿互联网部分核心特质的模拟系统 
目前名称为：Micro Internetwork (MI) 
目前具有包括路由、服务发现、加密、邮件和分布式计算的简单的模拟实现

需要注意：
MI的消息发送基于Broadcast-Receive而不是Send-Receive
所以说所有通信消息都是在某一网络内公开的，而具体信息都定义在MI的网络协议中

## 关于网络协议

网络协议是MI中传输的信息的通用包装，它的结构和构建很简单

### 它的结构形式如下

- a = target, 目标 ID 或服务名
- b = message, 信息字段
- c = protocol, 主协议名
- d = subprotocol, 子协议名
- info = {}, 其他负载
- 也可以添加些其他的字段

## 详细功能列

### 2. Client 客户端

通用客户端程序，用于访问MI网络中的各种服务。

命令

> client link <CAN> <服务器> <站点> <页面>    访问页面服务    client link CAN:Test server site1 main
> client download <CAN> <服务器> <站点> <文件>    下载文件    client download CAN:Test server site1 file.txt
> client upload <CAN> <服务器> <文件>    上传文件    client upload CAN:Test server myfile.txt

工作流程：

> 自动检测并打开调制解调器
> 解析命令行参数确定目标网络和服务
> 通过lookup服务发现找到目标服务器ID
> 发送请求并等待响应
> 根据响应类型显示内容或保存文件

### 3. CAN 网络划分

CAN (Channel Area Network) 是MI中的逻辑子网概念，类似于现实网络中的VLAN或子网划分。
它的目的：

> 不同CAN中的消息互不可见
> 将不同类型的服务划分到不同网络

### 4. Mail 邮件服务模拟

邮件系统模拟，支持用户注册、加密存储和邮件收发。

组件构成：

> mailserver.lua - 邮件服务器
> mail.lua - 邮件客户端

命令

> mail signup <服务器> <用户名>    注册新邮箱地址
> mail send <服务器> <发件人> <收件人> <邮件文件>    发送加密邮件
> mail list <服务器> <邮箱地址>    列出收件箱
> mail check <服务器> <邮箱地址> <邮件名>    查看并解密邮件

邮件内容使用crackcircle几何加密算法
每个邮件使用随机生成的Key和Salt
服务器存储的是加密后的密文

流程：
发件人 → 加密邮件 → 邮件服务器 → 存储 → 收件人请求 → 解密 → 查看

### 5. Fax 传真服务模拟

模拟现实传真系统的网络服务，支持号码注册和文档传输。

组件构成：

> faxserver.lua - 传真交换中心
> faxprinter.lua - 传真接收终端（需要打印机外设）
> fax.lua - 传真发送客户端

命令

> faxprinter <号码>    注册传真机并开始服务
> fax <目标号码> <文件>    发送传真文档

工作流程：
发件人 → fax.lua → faxserver → 查找号码 → faxprinter → 打印文档

### 6. Netlib 网络库共享

MI的代码分发系统，允许通过网络远程加载Lua库。

组件构成：

> netlibserver.lua - 代码库服务器
> netlib.lua - 客户端加载库

服务器端：注册API密钥（可选）
netlib.registation("libserver", "myapikey123")

客户端：远程加载库
local mylib = netlib.netrequire("libserver", "mylibrary", "myapikey123")

可选的API密钥认证
代码在沙箱环境中执行
支持私有库和公开库

它可以：
共享常用工具函数
分发协议实现库
动态更新客户端代码

### 1. MIF 应用接口

MIF是MI提供给想要使用MI进行创造的人的一个应用程序接口
其提供的函数有：
核心函数详解：
函数   参数说明    功能描述

> mif.modem()    无    自动检测并打开可用的调制解调器，返回找到的侧边名称    
> mif.idverifyRes()    SENDER: 目标ID, SP: 子协议名, HOST: 服务名    发送ID验证响应，用于服务发现确认     
> mif.lookup()    HOST: 服务名, SP: 子协议, CAN: 目标网络, QUEUE: 是否用队列    在指定网络中查找服务提供者的ID     
> mif.receive()    PS: 协议集, BLACK: 黑名单, WHITE: 白名单, CAN: 网络名    带协议过滤的网络接收函数     
> mif.capturefunc()    SP: 协议集, CAN: 网络名    协议抓包专用接收函数，返回时间戳和包内容     
> mif.post()    ID: 目标, CAN: 网络, PACK: 数据包, ROUTE: 路由方式, SINGLE: 是否等待回包    通用消息发送函数，支持多种路由模式     
> mif.clientLink()    CAN: 网络, INFO: 请求信息, ROUTE: 路由方式    客户端请求页面服务的快捷函数     
> mif.clientDownload()    CAN, INFO, ROUTE    客户端请求下载文件的快捷函数     
> mif.clientUpload()    CAN, INFO, ROUTE    客户端上传文件的快捷函数     
> mif.askSignUp()    CAN: 服务所在网络, SPN: 服务名    向RCS注册自己的服务     
> mif.serverLTP()    SENDER, PACK, ROUTE    服务器处理页面请求的内部函数     
> mif.serverFDP()    SENDER, PACK, ROUTE    服务器处理下载请求的内部函数     
> mif.serverFUP()    PACK, ROUTE    服务器处理上传请求的内部函数     
> mif.createServer()    HOST: 服务名, CAN: 网络, ROUTE: 路由方式    一键创建符合MI规范的服务器     

### 7. Crackcirle 加密

MI自定义的几何加密算法，基于圆上的点映射实现。 
注：你可以找其他算法代替它

算法原理：

> 每个字符被映射到圆上的一个点 (y, b)
> 通过复杂的几何变换（旋转、平移、缩放）进行加密
> 使用Key和Salt作为变换参数
> 只有知道正确参数的接收方才能解密

核心函数：

> crackcircle.encrackcircle(str, key, salt)    加密字符串，返回三维坐标表
> crackcircle.crackcircle(cc, key, salt)    解密坐标表，返回字符坐标
> crackcircle.strd(tab)    将字符坐标表转换为可读字符串

### 8. Cloudfabric 分布式计算

MI上模拟的分布式计算的平台，支持任务分发和分布式执行。

cloudfabric.lua - 分布式计算框架API
ccc.lua - 中央控制器（Computerack）
计算节点 - 运行用户代码的工作节点
数据库节点 - 提供分布式存储

节点类型

> CCC    中央控制器    负责任务调度和节点管理
> Compute    计算节点    执行用户提交的Lua代码
> Database    数据库节点    提供键值对存储服务

提供的沙箱：

> chunkenv - 完整环境，可访问MIF等API
> safeenv - 安全环境，限制系统访问

例子：提交计算任务
local result = cloudfabric.CloudfabricClient({
    code = [[
        local sum = 0
        for i=1,100 do sum = sum + i end
        return sum
    ]],
    func = "compute",
    env = "safe"
}, "Route2", false)

你可以用在：

> 并行计算任务
> 远程代码执行
> 分布式数据处理

### 9. Capturepkt 网络协议捕获

MI的网络分析工具，支持实时抓包、协议分析和加密内容解密。

命令

> capturepkt start <CAN> <协议过滤>    开始抓包（保存原始数据）
> capturepkt crack <CAN> <协议过滤>    抓包并尝试解密加密内容
> capturepkt cloud    监控Cloudfabric节点状态

协议过滤：
通过修改ptable控制要抓取的协议
    local ptable = {
        communicate = true,
        IDverify = true,
        netserver = true,
        crack = true,  -- 开启后会尝试解密
        -- ...
    }

输出格式：
[Clock:1234.56]|addr:123->addr:456
c:mail|msg:signup|e:string|info:take

你可以用它进行网络调试和协议学习。

### 10. Route 2 路由系统

MI的核心路由机制，实现跨CAN通信。

组件
RCS    Route Center Server    本地CAN的路由中心，维护本CAN服务表
RRCS    Root Route Center Server    根路由中心，维护全局RCS表

路由流程：

> 1. 客户端在本地CAN发起跨网请求
> 2. 本地RCS收到请求，查询本CAN服务表（未找到）
> 3. RCS向RRCS询问目标CAN的RCS地址
> 4. RRCS返回目标CAN的RCS ID
> 5. 本地RCS将请求转发给目标RCS
> 6. 目标RCS在目标CAN内广播服务发现
> 7. 目标服务器响应，通信建立
> 8. 返回路径按原路返回

支持跨CAN通信
服务动态注册
路由表自动更新
