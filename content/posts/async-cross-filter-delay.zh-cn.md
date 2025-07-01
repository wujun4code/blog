+++
authors = ["Jun Wu"]
title = "真实案例分析：对于“异步、跨系统、多条件判定 + 延迟执行”场景的优雅架构设计"
date = "2025-06-30"
description = ""
tags = [
    "c#",
    "design"
]
categories = [
    "beckend",
    "middleware"
]
series = ["c#"]
+++

# 真实案例分析：对于“异步、跨系统、多条件判定 + 延迟执行”场景的优雅架构设计


---

## 一、背景描述

在开发过程中，我们经常会遇到一类非常复杂的场景：

* 需要根据 **多条异步条件** 判定是否执行一个操作
* 各个条件条件依赖于多个 **外部系统 API**
* 有时需要把 **一个最终操作延迟执行** (如 24 小时后给用户发奖)
* 事后对各步执行进行审计/日志记录

**示例场景**：

> 当用户评价一件商品后，系统需要判断该用户是否符合奖励条件，如果是，则在 24 小时内给他的账户打入 1 美元

---

## 二、异步、多条件判断的主要难点

| 类型       | 描述                       |
| -------- | ------------------------ |
| 多条件上下文不同 | 每条条件可能需要用户ID/评论ID/商品ID之类 |
| 异步分开     | 每条条件需要调用不同服务无法合并         |
| 可扩展/可删除  | 要求后期可以自由地增删规则            |
| 延迟执行要求   | 奖励操作需要在所有判定通过后延迟执行       |

---

## 三、建议架构设计

### 流程图 (Mermaid)

{{<mermaid>}}
flowchart TD
    A[用户提交评论] -->|Push Event| MQ[(Message Queue)]
    MQ --> Review[CommentReviewProcessor]
    Review --> R1[是否购买过该商品]
    Review --> R2[是否是第一次评论]
    Review --> R3[是否 24h 内评论过少于 10 项]
    Review --> R4[是否是 180 天内]
    Review --> R5[是否获得过奖励]
    Review --> R6[评论内容是否合法]
    R1 & R2 & R3 & R4 & R5 & R6 --> AllPass{All Passed?}
    AllPass -->|Yes| Schedule[延迟24h 执行奖励]
{{</mermaid>}}

---

## 四、代码设计：分类规则 + DI 模式

### 规则接口

```csharp
public interface ICommentRewardRule
{
    Task<RuleResult> EvaluateAsync(CommentContext context);
}

public class RuleResult
{
    public bool Passed { get; set; }
    public string? FailReason { get; set; }

    public static RuleResult Pass() => new RuleResult { Passed = true };
    public static RuleResult Fail(string reason) => new RuleResult { Passed = false, FailReason = reason };
}
```

### 上下文 CommentContext

```csharp
public class CommentContext
{
    public string UserId { get; set; }
    public Guid ProductId { get; set; }
    public string Content { get; set; }
    public DateTime CommentTime { get; set; }
    public Guid CommentId { get; set; }
}
```

### 规则实现示例：是否购买过该商品

```csharp
public class PurchasedProductRule : ICommentRewardRule
{
    private readonly IOrderApi _orderApi;

    public PurchasedProductRule(IOrderApi orderApi)
    {
        _orderApi = orderApi;
    }

    public async Task<RuleResult> EvaluateAsync(CommentContext context)
    {
        bool hasPurchased = await _orderApi.HasPurchasedProduct(context.UserId, context.ProductId);
        return hasPurchased ? RuleResult.Pass() : RuleResult.Fail("未购买商品");
    }
}
```

### 规则调用器 CommentRewardEvaluator

```csharp
public class CommentRewardEvaluator
{
    private readonly IEnumerable<ICommentRewardRule> _rules;
    private readonly IRewardService _rewardService;

    public CommentRewardEvaluator(IEnumerable<ICommentRewardRule> rules, IRewardService rewardService)
    {
        _rules = rules;
        _rewardService = rewardService;
    }

    public async Task EvaluateAndRewardAsync(CommentContext context)
    {
        foreach (var rule in _rules)
        {
            var result = await rule.EvaluateAsync(context);
            if (!result.Passed)
            {
                Console.WriteLine($"[Rule Failed] {result.FailReason}");
                return;
            }
        }

        await _rewardService.GrantRewardAsync(context.UserId, 1.00m, context.CommentId);
    }
}
```

---

## 五、时序图解析

{{<mermaid>}}
sequenceDiagram
    participant C as CommentController
    participant MQ as Message Queue
    participant P as CommentProcessor
    participant Rule as Rule Evaluator
    participant Ext as Order/Reward/Content APIs

    C->>MQ: Push CommentCreatedEvent
    MQ->>P: Receive Event
    P->>Rule: Evaluate Rules
    Rule->>Ext: Call Order API
    Rule->>Ext: Call Comment API
    Rule->>Ext: Call Reward API
    Rule->>Ext: Call Moderation API
    Rule-->>P: Result = All Passed
    P->>Ext: Schedule $1 reward via Reward API
{{</mermaid>}}

---

## 六、延迟执行选项

* **Hangfire**:  支持 Delay Job
* **Azure Function Timer / Storage Queue**: 可以设置隐藏时间
* **Kafka 延迟队列**
* **Quartz.NET**: 非常经典的调度器

---

## 七、架构可维护性

| 能力       | 效果                               |
| -------- | -------------------------------- |
| 增加/删除规则  | 规则为 DI + 组件化实现，无需更改主逻辑           |
| 异步/并发执行  | 本设计支持各规则异步调用，实现 Task.WhenAll 很简单 |
| 日志审计     | 每条规则可单独记录日志和失败原因                 |
| 集成策略/缓存性 | 规则可加入内带缓存/定期刷新策略                 |

---

## 八、结言

通过“规则链 + 异步执行 + 分层分类设计”，我们可以有效地应对后期任何规则变化、扩展需求和操作级延迟执行需求。

此架构特别适合如下场景：

* 奖励分发系统
* 评分/缓惊/认证系统
* 分类操作应用(安全检测、网络监控)

---
