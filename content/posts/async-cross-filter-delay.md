+++
authors = ["Jun Wu"]
title = "Developer’s Guide: Elegant Architecture for Asynchronous, Multi-Condition Judgments with External Dependencies"
date = "2024-02-23"
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

# Developer’s Guide: Elegant Architecture for Asynchronous, Multi-Condition Judgments with External Dependencies

---

## 1. Background

In backend development, we often encounter complex scenarios that involve:

- **Multiple asynchronous conditions** to determine whether to execute a business logic
- Each condition may rely on a different **external system/API** (e.g., order, comment, reward)
- **Delayed execution** after a condition check passes (e.g., reward sent 24 hours after comment)
- **Auditability and traceability** of each rule/check

### Example Use Case

> After a user submits a product comment, the system must check if the user is eligible for a $1 reward. If eligible, the system will issue the reward within 24 hours.

---

## 2. Challenges in Asynchronous Multi-Condition Judgments

| Challenge | Description |
|----------|-------------|
| Context-Specific Conditions | Each rule may require different inputs like userId, productId, commentId |
| Async Split Execution | Different services need to be called independently (e.g., order, reward, moderation) |
| Maintainability and Scalability | Rules should be easy to add/remove independently |
| Delayed Execution | Reward dispatch needs to occur 24 hours later if all rules pass |

---

## 3. Recommended Architecture Design

### Architecture Flow (Mermaid Diagram)

{{<mermaid>}}
flowchart TD
    A[User Submits Comment] -->|Push Event| MQ[(Message Queue)]
    MQ --> Review[CommentReviewProcessor]
    Review --> R1[Purchased Product]
    Review --> R2[First-Time Comment]
    Review --> R3[Less than 10 comments in last 24h]
    Review --> R4[Within 180 days of purchase]
    Review --> R5[No previous reward for this product]
    Review --> R6[Content passes moderation]
    R1 & R2 & R3 & R4 & R5 & R6 --> AllPass{All Passed?}
    AllPass -->|Yes| Schedule[Schedule $1 reward in 24h]
{{</mermaid>}}

---

## 4. Code Structure: Rule Interface + DI Composition

### Rule Interface

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

### Context Model

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

### Example Rule: Check if User Purchased Product

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
        return hasPurchased ? RuleResult.Pass() : RuleResult.Fail("Product not purchased");
    }
}
```

### Rule Executor: CommentRewardEvaluator

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

## 5. Sequence Diagram

{{<mermaid>}}
sequenceDiagram
    participant C as CommentController
    participant MQ as Message Queue
    participant P as CommentProcessor
    participant Rule as Rule Evaluator
    participant Ext as External APIs

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

## 6. Delayed Reward Execution (Options)

- **Hangfire**: schedule delayed jobs
- **Azure Functions with Timer/Storage Queue**
- **Kafka/Redis Delay Queues**
- **Quartz.NET** for classic .NET scheduling

---

## 7. Maintainability Checklist

| Feature | Benefit |
|--------|---------|
| Rule add/remove | DI + modular rule classes allow easy updates |
| Async/Parallel Execution | Can use `Task.WhenAll()` for rules that don't depend on each other |
| Auditing | Each rule returns reason on failure for logging/tracing |
| Extensible for caching/policy | Rules can implement internal caching or throttling strategies |

---

## 8. Summary

By combining **rule chaining**, **asynchronous execution**, and **layered design**, this architecture effectively supports:

- Reward systems
- Behavior scoring, fraud checks, moderation
- Classification decisions in security, monitoring, or analytics

