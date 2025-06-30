+++
authors = ["吴骏"]
title = "关于我"
date = "2024-01-19"
description = "关于我"
+++


# 个人信息

- **姓名：** 吴骏
- **邮箱：** wujun4code@gmail.com  
- **微信：** wing_wj  
- **性别：** 男  
- **出生年份：** 1989  
- **学历：** 辽宁工业大学软件工程本科（2007-2011）  
- **工作经验：** 14年  

- **技术博客：**  
    - [博客](https://junwu.shouyicheng.com/)  
    - [KuberGuide](https://docs.kuberguide.org/)  
- **Github：** [http://github.com/wujun4code](http://github.com/wujun4code)  

## 教育经历

**辽宁科技大学**  
~ 辽宁鞍山  

软件工程 本科  
~ 2007年9月 - 2011年1月  

## 个人简介

我是一名经验丰富的全栈开发工程师，具备全面的 C# 技术背景，涉及多个产品线，包括 Unity、WPF、SystemCenter、WP、ASP.NET 和 .NET Core。在微服务后端开发方面具有丰富经验，熟练使用 Kubernetes 和 Docker。客户端平台经验广泛，涵盖 iOS、Android、Hybrid 和 Web。

我具备快速适应和掌握新框架与新技术的能力，善于工具运用和框架使用，能够准确理解需求并与不同背景的同事进行有效沟通。能够流利使用英语与国际团队沟通协作。具有大型技术文档的维护经验，也擅长技术写作。

## 工作经历

### 软件工程师 @Maersk.com | 2024年5月 - 至今

- 马士基（A.P. 穆勒-马士基 A/S）是一家丹麦航运与物流公司，成立于1904年。

#### 项目 - 角色 - 技术栈

**承运商追踪集成 - 后端开发**

- 在承运商追踪集成项目中，我担任后端开发工程师，项目目标是将马士基与全球不同国家的快递服务集成，实现实时的货运追踪、管理与状态通知。我的职责包括设计和开发共享组件、根据需求编写技术设计文档，并参与开发、运维和交付阶段。
具体的业务内容如下：
1.负责对接全球各个国家的不同的快递服务商的 API，主要的业务内容是生成快递单和追踪快递单的运输状态
2.设计和实现快递集成系统
3.监控，发布，维护生产系统的稳定

工作业绩

1.对接了北美，罗马尼亚，匈牙利等国家的快递服务
2.基于企业业务内容，重构了快递集成系统，使用 GitHub 模板仓库，快速生成快递集成系统的模板代码，将原来的 2个研发/2周交付集成一个快递服务商的开发周期缩短至1个开发/1周
3.重构了整个项目基于 GitHub Actions CI/CD 的 Workflow


- **C#/.NET Core/Kubernetes/CosmosDB/ASP.NET MVC/Kafka/Temporal/Grafana**

### 高级后端工程师 @iHerb.com | 2019年1月 - 2024年1月

- iHerb 是一个销售健康产品的电商平台。期间我担任高级开发工程师，参与多个核心业务项目，涵盖所有与电商相关的功能模块。

#### 项目 - 角色 - 技术栈

**商品目录系统 - 全栈核心开发**

- 担任角色是全栈开发，在不同的业务组件上采用多种语言完成业务功能，使用 TypeScript  基于 [apollographql](https://www.apollographql.com/) 框架的数据查询中间件，为 Web(Desktop+React Mobile Web)/Mobile App /小程序提供统一的 graphql  数据 API ，并且也需要基于 MVC  对 Web  页面进行维护。


- **C#/.NET Core/Kubernetes/SQL Server/React/ASP.NET MVC/Apollo GraphQL/NodeJS/Dgraph/Kafka**

**UGC 用户内容模块 - 后端核心开发**

- 日常任务是维护用户对商品的评论和 Q&A，技术栈结构是 .NET Core 6.0 作为核心的 API  层，底层数据存储同时使用了 ElasticSerach  作为全文检索引擎，Mongodb  作为持久化存储，RabbitMQ  作为消息队列处理用户提交的评论和 Q&A，搭配 Hangfire  进行定期的数据处理和应对新需求的数据改造以及缓存更新。

- **C#/.NET Core/Kubernetes/SQL Server/ASP.NET MVC/Elasticsearch/MongoDB/RabbitMQ/Hangfire**

**网关 API - 后端核心开发**

- 开发服务端 SEO 相关 API，支持多国家/地区/语言的子站跳转，作为整个 UGC 项目的核心 SEO 中间件。独立完成 API 设计、数据源组织、缓存策略、编码、单元测试、发布和部署。
- **C#/.NET Core/Kubernetes/SQL Server/ASP.NET MVC**

**激励评论机制**

- 实现用户提交有效评论后可获得 $1 奖励的功能。设计完整流程，包括评论扫描、评分、有效性判断、历史记录检查等。使用职责链模式提升效率与可读性。
- **C#/.NET Core 6.0/Kubernetes/RabbitMQ/MongoDB**

**BFF（前端后端接口）**

- 作为架构师及后端开发者，为移动 App 和小程序提供数据接口。构建基于 ASP.NET Core 的 Web API 项目，使用 Parse Server 实现数据库微服务，MongoDB 作为存储，Redis 进行前端缓存。
- **C#/.NET Core 2.2/Kubernetes/Jenkins CI/CD/MongoDB**


工作业绩

- Welcome Mat 
  引流新用户注册和给与第一单折扣码的后端实现。
  用户在使用邮箱注册的时候，后端会跑一个完整的校验流程，不同的流程对应的折扣码是不一样的，因此需要设计不同的 workflow  来应对，整个功能从设计，编码和上线发布的后端模块都是独自一个人完成的，并且成功上线。
- Gateway API 
  为服务端的 SEO  做一个 API ，基于不同国家和地区以及语言进行对应子站点的重定向，作为整个 UGC  项目的 SEO  核心服务端中间件，独自完成 API  设计，数据源整理，缓存策略指定，编码，单元测试，发布和上线。


### 开发工程师 @Leancloud.com | 2014年5月 - 2018年12月

- 核心 C# 开发者及官方技术文档维护人员。

#### 项目 - 角色 - 技术栈

**C# SDK - 核心开发者**

- 负责维护 Unity/Windows Phone/Windows Desktop/WPF/Xamarin 的 C# SDK，使用 Visual Studio 开发各种跨平台 SDK。
- **C#/Unity3D**

**实时聊天组件 SDK - 核心开发者**

- 开发基于 WebSocket 的客户端实时聊天组件 C# SDK。服务端使用 Kafka 推送消息，客户端封装服务端的聊天室、群组、私聊等功能，设计接口协议并实现代码。
- **C#/WebSocket/Protobuf**

**LeanCloud 官方文档 - [https://docs.leancloud.cn/] - 核心撰写者**

- 主要负责中文文档的撰写和维护，并编写及审核示例代码。
- **Nunjucks/Markdown/jQuery**

## 中级软件工程师 @微创（北京） | 2013年10月 - 2014年4月

#### 项目 - 角色 - 技术栈

**CloudBox - 后端核心开发**

- 将 System Center 的基础 API 封装为内部 WCF 服务，支持虚拟云桌面的构建与管理。使用 WCF/REST 接口封装 PowerShell API，支持虚拟桌面主机创建、云资源管理、子网监控等功能。项目客户包括银行与大型企业，场景类似 VMware，接触到微软 Azure 公有云与混合云，深入理解 Azure 的 REST API 及异步接口封装机制。
- **C#/.NET/SQL Server/System Center/WCF/Azure REST API**

## 初级开发工程师 @Symbio（北京） | 2011年7月 - 2013年7月

- 主要负责基于 Silverlight 的 Windows Phone 应用开发。

#### 项目 - 角色 - 技术栈

**Nokia App Studio - 应用开发者**

- 将诺基亚收购的部分 iOS 应用移植到 Windows Phone 平台，主要负责其中三个游戏的开发。使用 XNA 引擎和 Silverlight，初步接触微软 XAML 技术栈。当时项目规模大，五六人共同开发20多个游戏，每人负责3~4个。
- **Silverlight for Windows Phone/C#/XNA/XAML/Cocos2d**

## 技能

- **编程语言：** C#（熟练），JavaScript / TypeScript（熟练）  
- **Web 开发：** React（熟练），Angular（基础）  
- **Web 框架：** ASP.NET Core（熟练），Express（基础）  
- **数据库：** MongoDB（熟练），SQL Server（基础），Postgres（基础）  
- **消息队列：** RabbitMQ（熟练），Kafka（基础）  
- **版本控制/部署工具：** Git（熟练），Markdown（熟练）  
- **CI/CD 工具：** Jenkins（熟练），Azure Pipelines&DevOps（熟练），GitHub Actions（熟练），FluxCD（熟练），Harness（熟练）  
- **后端组件：** Kubernetes（熟练），Redis（基础），Elasticsearch（熟练），Docker（熟练）  
- **云平台：** GCP（熟练），Azure（熟练），腾讯云/阿里云/DigitalOcean（基础）  
- **IDE 工具：** Visual Studio（熟练），Visual Studio Code（熟练），IDEA（基础）  
- **操作系统：** Linux（ElementaryOS, Ubuntu）（熟练），macOS（熟练），Windows（熟练）  
- **语言能力：** 中文（母语），英文（流利）
