---
name: context-degradation
description: This skill should be used when the user asks to "diagnose context problems", "fix lost-in-middle issues", "debug agent failures", "understand context poisoning", or mentions context degradation, attention patterns, context clash, context confusion, or agent performance degradation. Provides patterns for recognizing and mitigating context failures.
---

# Context Degradation Patterns

# 上下文退化模式

Language models exhibit predictable degradation patterns as context length increases. Understanding these patterns is essential for diagnosing failures and designing resilient systems. Context degradation is not a binary state but a continuum of performance degradation that manifests in several distinct ways.

随着上下文长度增加，语言模型会表现出可预测的退化模式。理解这些模式对于诊断故障和设计弹性系统至关重要。上下文退化不是二元状态，而是一个连续的性能退化谱系，以几种不同的方式表现出来。

## When to Activate

## 何时激活

Activate this skill when:
- Agent performance degrades unexpectedly during long conversations
- Debugging cases where agents produce incorrect or irrelevant outputs
- Designing systems that must handle large contexts reliably
- Evaluating context engineering choices for production systems
- Investigating "lost in middle" phenomena in agent outputs
- Analyzing context-related failures in agent behavior

当出现以下情况时激活此技能：
- 智能体在长对话过程中性能意外下降
- 调试智能体产生错误或不相关输出的案例
- 设计必须可靠处理大上下文的系统
- 评估生产系统的上下文工程选择
- 调查智能体输出中的"中间丢失"现象
- 分析智能体行为中与上下文相关的故障

## Core Concepts

## 核心概念

Context degradation manifests through several distinct patterns. The lost-in-middle phenomenon causes information in the center of context to receive less attention. Context poisoning occurs when errors compound through repeated reference. Context distraction happens when irrelevant information overwhelms relevant content. Context confusion arises when the model cannot determine which context applies. Context clash develops when accumulated information directly conflicts.

These patterns are predictable and can be mitigated through architectural patterns like compaction, masking, partitioning, and isolation.

上下文退化通过几种明显的模式表现出来。中间丢失现象导致上下文中心的信息获得的注意力减少。上下文中毒发生在错误通过重复引用累积时。上下文分心发生在无关信息淹没相关内容时。上下文混淆出现在模型无法确定哪个上下文适用时。上下文冲突在累积的信息直接矛盾时发展。

这些模式是可预测的，可以通过架构模式来缓解，如压缩、掩码、分区和隔离。

## Detailed Topics

## 详细主题

### The Lost-in-Middle Phenomenon

### 中间丢失现象

The most well-documented degradation pattern is the "lost-in-middle" effect, where models demonstrate U-shaped attention curves. Information at the beginning and end of context receives reliable attention, while information buried in the middle suffers from dramatically reduced recall accuracy.

记录最详尽的退化模式是"中间丢失"效应，模型表现出 U 型注意力曲线。上下文开头和结尾的信息获得可靠的注意力，而埋在中间的信息则经历显著降低的召回准确率。

**Empirical Evidence**

**经验证据**

Research demonstrates that relevant information placed in the middle of context experiences 10-40% lower recall accuracy compared to the same information at the beginning or end. This is not a failure of the model but a consequence of attention mechanics and training data distributions.

研究表明，与放在上下文开头或结尾相比，放在中间的相关信息召回准确率低 10-40%。这不是模型的失败，而是注意力机制和训练数据分布的结果。

Models allocate massive attention to the first token (often the BOS token) to stabilize internal states. This creates an "attention sink" that soaks up attention budget. As context grows, the limited budget is stretched thinner, and middle tokens fail to garner sufficient attention weight for reliable retrieval.

模型将大量注意力分配给第一个 token（通常是 BOS token）以稳定内部状态。这创造了一个"注意力汇"，吸收了注意力预算。随着上下文增长，有限的预算被拉得更薄，中间的 token 无法获得足够的注意力权重来进行可靠检索。

**Practical Implications**

**实际启示**

Design context placement with attention patterns in mind. Place critical information at the beginning or end of context. Consider whether information will be queried directly or needs to support reasoning—if the latter, placement matters less but overall signal quality matters more.

设计上下文放置时要考虑注意力模式。将关键信息放在上下文的开头或结尾。考虑信息是直接查询还是需要支持推理——如果是后者，位置不那么重要，但整体信号质量更重要。

For long documents or conversations, use summary structures that surface key information at attention-favored positions. Use explicit section headers and transitions to help models navigate structure.

对于长文档或对话，使用摘要结构将关键信息呈现在注意力有利的位置。使用明确的章节标题和过渡来帮助模型导航结构。

### Context Poisoning

### 上下文中毒

Context poisoning occurs when hallucinations, errors, or incorrect information enters context and compounds through repeated reference. Once poisoned, context creates feedback loops that reinforce incorrect beliefs.

当幻觉、错误或不正确信息进入上下文并通过重复引用累积时，就会发生上下文中毒。一旦中毒，上下文会创建反馈循环，强化错误的信念。

**How Poisoning Occurs**

**中毒如何发生**

Poisoning typically enters through three pathways. First, tool outputs may contain errors or unexpected formats that models accept as ground truth. Second, retrieved documents may contain incorrect or outdated information that models incorporate into reasoning. Third, model-generated summaries or intermediate outputs may introduce hallucinations that persist in context.

中毒通常通过三种途径进入。首先，工具输出可能包含错误或不期望的格式，模型接受这些为真实情况。其次，检索到的文档可能包含错误或过时的信息，模型将这些信息纳入推理。第三，模型生成的摘要或中间输出可能引入幻觉，这些幻觉在上下文中持续存在。

The compounding effect is severe. If an agent's goals section becomes poisoned, it develops strategies that take substantial effort to undo. Each subsequent decision references the poisoned content, reinforcing incorrect assumptions.

复合效应很严重。如果智能体的目标部分被污染，它会制定需要大量努力才能撤销的策略。每个后续决策都会引用被污染的内容，强化错误的假设。

**Detection and Recovery**

**检测和恢复**

Watch for symptoms including degraded output quality on tasks that previously succeeded, tool misalignment where agents call wrong tools or parameters, and hallucinations that persist despite correction attempts. When these symptoms appear, consider context poisoning.

注意症状，包括以前成功的任务输出质量下降、智能体调用错误工具或参数的工具不匹配、以及尽管尝试纠正仍然持续的幻觉。当出现这些症状时，考虑上下文中毒。

Recovery requires removing or replacing poisoned content. This may involve truncating context to before the poisoning point, explicitly noting the poisoning in context and asking for re-evaluation, or restarting with clean context and preserving only verified information.

恢复需要移除或替换被污染的内容。这可能涉及将上下文截断到中毒点之前、在上下文中明确标注中毒并要求重新评估，或者用干净的上下文重启并仅保留已验证的信息。

### Context Distraction

### 上下文分心

Context distraction emerges when context grows so long that models over-focus on provided information at the expense of their training knowledge. The model attends to everything in context regardless of relevance, and this creates pressure to use provided information even when internal knowledge is more accurate.

当上下文增长到模型过度关注提供的信息而牺牲训练知识时，就会出现上下文分心。模型关注上下文中的所有内容而不考虑相关性，这会产生使用所提供信息的压力，即使内部知识更准确。

**The Distractor Effect**

**干扰效应**

Research shows that even a single irrelevant document in context reduces performance on tasks involving relevant documents. Multiple distractors compound degradation. The effect is not about noise in absolute terms but about attention allocation—irrelevant information competes with relevant information for limited attention budget.

研究表明，即使上下文中只有一份无关文档，也会降低涉及相关文档任务的性能。多个干扰因素会加剧退化。这种效应不是关于绝对意义上的噪声，而是关于注意力分配——无关信息与相关信息争夺有限的注意力预算。

Models do not have a mechanism to "skip" irrelevant context. They must attend to everything provided, and this obligation creates distraction even when the irrelevant information is clearly not useful.

模型没有机制来"跳过"无关的上下文。它们必须关注提供的所有内容，这种义务创造了干扰，即使无关信息明显无用。

**Mitigation Strategies**

**缓解策略**

Mitigate distraction through careful curation of what enters context. Apply relevance filtering before loading retrieved documents. Use namespacing and organization to make irrelevant sections easy to ignore structurally. Consider whether information truly needs to be in context or can be accessed through tool calls instead.

通过仔细策划进入上下文的内容来减轻分心。在加载检索到的文档之前应用相关性过滤。使用命名空间和组织使无关部分在结构上易于忽略。考虑信息是否真的需要放在上下文中，还是可以通过工具调用访问。

### Context Confusion

### 上下文混淆

Context confusion arises when irrelevant information influences responses in ways that degrade quality. This is related to distraction but distinct—confusion concerns the influence of context on model behavior rather than attention allocation.

当无关信息以影响质量的方式影响响应时，就会出现上下文混淆。这与分心有关但不同——混淆关注的是上下文对模型行为的影响，而不是注意力分配。

If you put something in context, the model has to pay attention to it. The model may incorporate irrelevant information, use inappropriate tool definitions, or apply constraints that came from different contexts. Confusion is especially problematic when context contains multiple task types or when switching between tasks within a single session.

如果你把某些东西放在上下文中，模型就必须关注它。模型可能会合并无关信息、使用不适当的工具定义，或应用来自不同上下文的约束。当上下文包含多种任务类型或在单个会话中切换任务时，混淆尤其成问题。

**Signs of Confusion**

**混淆的迹象**

Watch for responses that address the wrong aspect of a query, tool calls that seem appropriate for a different task, or outputs that mix requirements from multiple sources. These indicate confusion about what context applies to the current situation.

注意那些解决查询错误方面的响应、看似适合不同任务的工具调用，或混合多个来源需求的输出。这些表明对什么上下文适用于当前情况的混淆。

**Architectural Solutions**

**架构解决方案**

Architectural solutions include explicit task segmentation where different tasks get different context windows, clear transitions between task contexts, and state management that isolates context for different objectives.

架构解决方案包括明确的任务分割（不同任务获得不同的上下文窗口）、任务上下文之间的清晰过渡，以及为不同目标隔离上下文的状态管理。

### Context Clash

### 上下文冲突

Context clash develops when accumulated information directly conflicts, creating contradictory guidance that derails reasoning. This differs from poisoning where one piece of information is incorrect—in clash, multiple correct pieces of information contradict each other.

当累积的信息直接冲突时，就会产生上下文冲突，产生破坏推理的矛盾指导。这与中毒不同——中毒是一条信息不正确，而在冲突中，多条正确的信息相互矛盾。

**Sources of Clash**

**冲突来源**

Clash commonly arises from multi-source retrieval where different sources have contradictory information, version conflicts where outdated and current information both appear in context, and perspective conflicts where different viewpoints are valid but incompatible.

冲突通常源于多源检索（不同来源有矛盾信息）、版本冲突（过时和当前信息同时出现在上下文中）和观点冲突（不同观点都有效但不兼容）。

**Resolution Approaches**

**解决方法**

Resolution approaches include explicit conflict marking that identifies contradictions and requests clarification, priority rules that establish which source takes precedence, and version filtering that excludes outdated information from context.

解决方法包括明确标记冲突（识别矛盾并请求澄清）、优先级规则（确定哪个来源优先）和版本过滤（将过时信息排除在上下文之外）。

### Empirical Benchmarks and Thresholds

### 经验基准和阈值

Research provides concrete data on degradation patterns that inform design decisions.

研究提供了关于退化模式的具体数据，为设计决策提供信息。

**RULER Benchmark Findings**

**RULER 基准发现**

The RULER benchmark delivers sobering findings: only 50% of models claiming 32K+ context maintain satisfactory performance at 32K tokens. GPT-5.2 shows the least degradation among current models, while many still drop 30+ points at extended contexts. Near-perfect scores on simple needle-in-haystack tests do not translate to real long-context understanding.

RULER 基准测试给出了令人清醒的发现：在声称支持 32K+ 上下文的模型中，只有 50% 在 32K tokens 时保持令人满意的性能。GPT-5.2 在当前模型中表现出最小的退化，而许多模型在扩展上下文时仍然下降 30 多点。简单的大海捞针测试中的近乎完美的分数并不能转化为真正的长上下文理解能力。

**Model-Specific Degradation Thresholds**

**特定模型的退化阈值**

| Model | Degradation Onset | Severe Degradation | Notes |
|-------|-------------------|-------------------|-------|
| GPT-5.2 | ~64K tokens | ~200K tokens | Best overall degradation resistance with thinking mode |
| Claude Opus 4.5 | ~100K tokens | ~180K tokens | 200K context window, strong attention management |
| Claude Sonnet 4.5 | ~80K tokens | ~150K tokens | Optimized for agents and coding tasks |
| Gemini 3 Pro | ~500K tokens | ~800K tokens | 1M context window, native multimodality |
| Gemini 3 Flash | ~300K tokens | ~600K tokens | 3x speed of Gemini 2.5, 81.2% MMMU-Pro |

| 模型 | 退化开始 | 严重退化 | 备注 |
|------|---------|---------|------|
| GPT-5.2 | ~64K tokens | ~200K tokens | 思考模式下整体退化抗性最佳 |
| Claude Opus 4.5 | ~100K tokens | ~180K tokens | 200K 上下文窗口，强大的注意力管理 |
| Claude Sonnet 4.5 | ~80K tokens | ~150K tokens | 针对智能体和编码任务优化 |
| Gemini 3 Pro | ~500K tokens | ~800K tokens | 1M 上下文窗口，原生多模态 |
| Gemini 3 Flash | ~300K tokens | ~600K tokens | Gemini 2.5 的 3 倍速度，81.2% MMMU-Pro |

**Model-Specific Behavior Patterns**

Different models exhibit distinct failure modes under context pressure:

- **Claude 4.5 series**: Lowest hallucination rates with calibrated uncertainty. Claude Opus 4.5 achieves 80.9% on SWE-bench Verified. Tends to refuse or ask clarification rather than fabricate.
- **GPT-5.2**: Two modes available - instant (fast) and thinking (reasoning). Thinking mode reduces hallucination through step-by-step verification but increases latency.
- **Gemini 3 Pro/Flash**: Native multimodality with 1M context window. Gemini 3 Flash offers 3x speed improvement over previous generation. Strong at multi-modal reasoning across text, code, images, audio, and video.

These patterns inform model selection for different use cases. High-stakes tasks benefit from Claude 4.5's conservative approach or GPT-5.2's thinking mode; speed-critical tasks may use instant modes.

**特定模型的行为模式**

不同模型在上下文压力下表现出不同的失败模式：

- **Claude 4.5 系列**：最低的幻觉率，具有校准的不确定性。Claude Opus 4.5 在 SWE-bench Verified 上达到 80.9%。倾向于拒绝或请求澄清而不是编造。
- **GPT-5.2**：提供两种模式——即时（快速）和思考（推理）。思考模式通过逐步验证减少幻觉但增加延迟。
- **Gemini 3 Pro/Flash**：原生多模态，1M 上下文窗口。Gemini 3 Flash 比上一代提供 3 倍速度提升。擅长跨文本、代码、图像、音频和视频的多模态推理。

这些模式为不同用例的模型选择提供信息。高风险任务受益于 Claude 4.5 的保守方法或 GPT-5.2 的思考模式；速度关键的任务可以使用即时模式。

### Counterintuitive Findings

### 反直觉的发现

Research reveals several counterintuitive patterns that challenge assumptions about context management.

研究揭示了几个挑战上下文管理假设的反直觉模式。

**Shuffled Haystacks Outperform Coherent Ones**

**打乱的干草堆胜过连贯的干草堆**

Studies found that shuffled (incoherent) haystacks produce better performance than logically coherent ones. This suggests that coherent context may create false associations that confuse retrieval, while incoherent context forces models to rely on exact matching.

研究发现，打乱的（不连贯的）干草堆比逻辑连贯的干草堆产生更好的性能。这表明连贯的上下文可能会产生混淆检索的错误关联，而不连贯的上下文迫使模型依赖精确匹配。

**Single Distractors Have Outsized Impact**

**单个干扰因素具有巨大影响**

Even a single irrelevant document reduces performance significantly. The effect is not proportional to the amount of noise but follows a step function where the presence of any distractor triggers degradation.

即使只有一份无关文档也会显著降低性能。这种效应与噪声量不成正比，而是遵循阶跃函数——任何干扰因素的存在都会触发退化。

**Needle-Question Similarity Correlation**

**针 - 问题相似性相关性**

Lower similarity between needle and question pairs shows faster degradation with context length. Tasks requiring inference across dissimilar content are particularly vulnerable.

针和问题对之间的相似性越低，随着上下文长度增加退化越快。需要跨不同内容进行推理的任务特别容易受到影响。

### When Larger Contexts Hurt

### 大上下文何时有害

Larger context windows do not uniformly improve performance. In many cases, larger contexts create new problems that outweigh benefits.

更大的上下文窗口并不能一致地提高性能。在许多情况下，大上下文会产生新的问题，超过其好处。

**Performance Degradation Curves**

**性能退化曲线**

Models exhibit non-linear degradation with context length. Performance remains stable up to a threshold, then degrades rapidly. The threshold varies by model and task complexity. For many models, meaningful degradation begins around 8,000-16,000 tokens even when context windows support much larger sizes.

模型随着上下文长度呈现非线性退化。性能在达到阈值之前保持稳定，然后迅速退化。阈值因模型和任务复杂度而异。对于许多模型，即使上下文窗口支持更大的尺寸，有意义的退化也在 8,000-16,000 tokens 左右开始。

**Cost Implications**

**成本影响**

Processing cost grows disproportionately with context length. The cost to process a 400K token context is not double the cost of 200K—it increases exponentially in both time and computing resources. For many applications, this makes large-context processing economically impractical.

处理成本随着上下文长度不成比例地增长。处理 400K token 上下文的成本不是 200K 的两倍——它在时间和计算资源上都呈指数级增长。对于许多应用来说，这使得大上下文处理在经济上不切实际。

**Cognitive Load Metaphor**

**认知负荷隐喻**

Even with an infinite context, asking a single model to maintain consistent quality across dozens of independent tasks creates a cognitive bottleneck. The model must constantly switch context between items, maintain a comparative framework, and ensure stylistic consistency. This is not a problem that more context solves.

即使有无限的上下文，要求单个模型在数十个独立任务中保持一致的质量也会造成认知瓶颈。模型必须不断在项目之间切换上下文、维护比较框架并确保风格一致性。这不是更多上下文能解决的问题。

## Practical Guidance

## 实用指南

### The Four-Bucket Approach

### 四桶方法

Four strategies address different aspects of context degradation:

四种策略解决上下文退化的不同方面：

**Write**: Save context outside the window using scratchpads, file systems, or external storage. This keeps active context lean while preserving information access.

**写**：使用草稿纸、文件系统或外部存储将上下文保存在窗口之外。这可以在保持信息访问的同时保持活动上下文的精简。

**Select**: Pull relevant context into the window through retrieval, filtering, and prioritization. This addresses distraction by excluding irrelevant information.

**选**：通过检索、过滤和优先级排序将相关上下文拉入窗口。这通过排除无关信息来解决分心问题。

**Compress**: Reduce tokens while preserving information through summarization, abstraction, and observation masking. This extends effective context capacity.

**压缩**：通过摘要、抽象和观察掩码在保留信息的同时减少 tokens。这扩展了有效的上下文容量。

**Isolate**: Split context across sub-agents or sessions to prevent any single context from growing large enough to degrade. This is the most aggressive strategy but often the most effective.

**隔离**：将上下文分割到子智能体或会话中，以防止任何单个上下文增长到足以退化的程度。这是最激进的策略，但通常最有效。

### Architectural Patterns

### 架构模式

Implement these strategies through specific architectural patterns. Use just-in-time context loading to retrieve information only when needed. Use observation masking to replace verbose tool outputs with compact references. Use sub-agent architectures to isolate context for different tasks. Use compaction to summarize growing context before it exceeds limits.

通过特定的架构模式实现这些策略。使用即时上下文加载仅在需要时检索信息。使用观察掩码用简洁的引用替换冗长的工具输出。使用子智能体架构来隔离不同任务的上下文。使用压缩在上下文超过限制之前总结不断增长的上下文。

## Examples

## 示例

**Example 1: Detecting Degradation**

**示例 1：检测退化**

```yaml
# Context grows during long conversation
turn_1: 1000 tokens
turn_5: 8000 tokens
turn_10: 25000 tokens
turn_20: 60000 tokens (degradation begins)
turn_30: 90000 tokens (significant degradation)
```

```yaml
# 上下文在长对话期间增长
第 1 轮：1000 tokens
第 5 轮：8000 tokens
第 10 轮：25000 tokens
第 20 轮：60000 tokens（退化开始）
第 30 轮：90000 tokens（显著退化）
```

**Example 2: Mitigating Lost-in-Middle**

**示例 2：缓解中间丢失**

```markdown
# Organize context with critical info at edges

[CURRENT TASK]                      # At start
- Goal: Generate quarterly report
- Deadline: End of week

[DETAILED CONTEXT]                  # Middle (less attention)
- 50 pages of data
- Multiple analysis sections
- Supporting evidence

[KEY FINDINGS]                     # At end
- Revenue up 15%
- Costs down 8%
- Growth in Region A
```

```markdown
# 在边缘组织关键信息的上下文

[当前任务]                          # 在开始
- 目标：生成季度报告
- 截止日期：周末

[详细上下文]                        # 中间（较少关注）
- 50 页数据
- 多个分析部分
- 支持证据

[关键发现]                          # 在结尾
- 收入增长 15%
- 成本下降 8%
- A 地区增长
```

## Guidelines

## 指南

1. Monitor context length and performance correlation during development
2. Place critical information at beginning or end of context
3. Implement compaction triggers before degradation becomes severe
4. Validate retrieved documents for accuracy before adding to context
5. Use versioning to prevent outdated information from causing clash
6. Segment tasks to prevent context confusion across different objectives
7. Design for graceful degradation rather than assuming perfect conditions
8. Test with progressively larger contexts to find degradation thresholds

1. 在开发过程中监控上下文长度和性能相关性
2. 将关键信息放在上下文的开头或结尾
3. 在退化变得严重之前实施压缩触发器
4. 在添加到上下文之前验证检索文档的准确性
5. 使用版本控制防止过时信息导致冲突
6. 分割任务以防止不同目标之间的上下文混淆
7. 设计优雅降级而不是假设完美条件
8. 用逐渐增大的上下文测试以找到退化阈值

## Integration

## 集成

This skill builds on context-fundamentals and should be studied after understanding basic context concepts. It connects to:

此技能建立在 context-fundamentals 之上，应在理解基本上下文概念后学习。它连接到：

- context-optimization - Techniques for mitigating degradation
- multi-agent-patterns - Using isolation to prevent degradation
- evaluation - Measuring and detecting degradation in production

- context-optimization - 缓解退化的技术
- multi-agent-patterns - 使用隔离来防止退化
- evaluation - 在生产环境中测量和检测退化

## References

## 参考资料

Internal reference:
- [Degradation Patterns Reference](./references/patterns.md) - Detailed technical reference

内部参考：
- [退化模式参考](./references/patterns.md) - 详细技术参考

Related skills in this collection:
- context-fundamentals - Context basics
- context-optimization - Mitigation techniques
- evaluation - Detection and measurement

本系列中的相关技能：
- context-fundamentals - 上下文基础
- context-optimization - 缓解技术
- evaluation - 检测和测量

External resources:
- Research on attention mechanisms and context window limitations
- Studies on the "lost-in-middle" phenomenon
- Production engineering guides from AI labs

外部资源：
- 关于注意力机制和上下文窗口限制的研究
- 关于"中间丢失"现象的研究
- 来自 AI 实验室的生产工程指南

---

## Skill Metadata

**Created**: 2025-12-20
**Last Updated**: 2025-12-20
**Author**: Agent Skills for Context Engineering Contributors
**Version**: 1.0.0
