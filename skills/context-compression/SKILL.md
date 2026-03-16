---
name: context-compression
description: This skill should be used when the user asks to "compress context", "summarize conversation history", "implement compaction", "reduce token usage", or mentions context compression, structured summarization, tokens-per-task optimization, or long-running agent sessions exceeding context limits.
---

# Context Compression Strategies

# 上下文压缩策略

When agent sessions generate millions of tokens of conversation history, compression becomes mandatory. The naive approach is aggressive compression to minimize tokens per request. The correct optimization target is tokens per task: total tokens consumed to complete a task, including re-fetching costs when compression loses critical information.

当智能体会话产生数百万 token 的对话历史时，压缩变得必不可少。朴素的方法是激进压缩以最小化每次请求的 token 数。正确的优化目标是每次任务的 token 数：完成任务消耗的总 token 数，包括压缩丢失关键信息时的重新获取成本。

## When to Activate

## 何时激活

Activate this skill when:
- Agent sessions exceed context window limits
- Codebases exceed context windows (5M+ token systems)
- Designing conversation summarization strategies
- Debugging cases where agents "forget" what files they modified
- Building evaluation frameworks for compression quality

当出现以下情况时激活此技能：
- 智能体会话超过上下文窗口限制
- 代码库超过上下文窗口（500万+ token 的系统）
- 设计对话摘要策略
- 调试智能体"忘记"修改了哪些文件的案例
- 构建压缩质量评估框架

## Core Concepts

## 核心概念

Context compression trades token savings against information loss. Three production-ready approaches exist:

上下文压缩在节省 token 和信息丢失之间进行权衡。存在三种生产可用的方法：

1. **Anchored Iterative Summarization**: Maintain structured, persistent summaries with explicit sections for session intent, file modifications, decisions, and next steps. When compression triggers, summarize only the newly-truncated span and merge with the existing summary. Structure forces preservation by dedicating sections to specific information types.

1. **锚定迭代摘要**：维护结构化的持久摘要，包含会话意图、文件修改、决策和下一步的明确章节。当压缩触发时，仅摘要新截断的段落并与现有摘要合并。结构通过为特定信息类型分配章节来强制保留信息。

2. **Opaque Compression**: Produce compressed representations optimized for reconstruction fidelity. Achieves highest compression ratios (99%+) but sacrifices interpretability. Cannot verify what was preserved.

2. **不透明压缩**：生成针对重建保真度优化的压缩表示。实现最高压缩率（99%+）但牺牲可解释性。无法验证保留了什么。

3. **Regenerative Full Summary**: Generate detailed structured summaries on each compression. Produces readable output but may lose details across repeated compression cycles due to full regeneration rather than incremental merging.

3. **再生完整摘要**：每次压缩时生成详细的结构化摘要。产生可读的输出，但由于完全再生而非增量合并，可能在重复压缩周期中丢失细节。

The critical insight: structure forces preservation. Dedicated sections act as checklists that the summarizer must populate, preventing silent information drift.

关键洞察：结构强制保留。专用章节充当摘要器必须填充的检查清单，防止静默的信息漂移。

## Detailed Topics

## 详细主题

### Why Tokens-Per-Task Matters

### 为什么每次任务的 Token 数很重要

Traditional compression metrics target tokens-per-request. This is the wrong optimization. When compression loses critical details like file paths or error messages, the agent must re-fetch information, re-explore approaches, and waste tokens recovering context.

传统的压缩指标针对每次请求的 token 数。这是错误的优化目标。当压缩丢失关键细节如文件路径或错误消息时，智能体必须重新获取信息、重新探索方法，并浪费 token 来恢复上下文。

The right metric is tokens-per-task: total tokens consumed from task start to completion. A compression strategy saving 0.5% more tokens but causing 20% more re-fetching costs more overall.

正确的指标是每次任务的 token 数：从任务开始到完成消耗的总 token 数。一个节省 0.5% 更多 token 但导致 20% 更多重新获取的压缩策略，总体成本更高。

### The Artifact Trail Problem

### 工件追踪问题

Artifact trail integrity is the weakest dimension across all compression methods, scoring 2.2-2.5 out of 5.0 in evaluations. Even structured summarization with explicit file sections struggles to maintain complete file tracking across long sessions.

工件追踪完整性是所有压缩方法中最弱的维度，在评估中得分为 5.0 分制的 2.2-2.5 分。即使有明确文件章节的结构化摘要，也难以在长会话中维护完整的文件追踪。

Coding agents need to know:
- Which files were created
- Which files were modified and what changed
- Which files were read but not changed
- Function names, variable names, error messages

编码智能体需要知道：
- 创建了哪些文件
- 修改了哪些文件以及改了什么
- 读取了哪些文件但没有修改
- 函数名、变量名、错误消息

This problem likely requires specialized handling beyond general summarization: a separate artifact index or explicit file-state tracking in agent scaffolding.

这个问题可能需要超越通用摘要的专门处理：单独的工件索引或智能体脚手架中的显式文件状态追踪。

### Structured Summary Sections

### 结构化摘要章节

Effective structured summaries include explicit sections:

有效的结构化摘要包含明确的章节：

```markdown
## Session Intent
[What the user is trying to accomplish]

## Files Modified
- auth.controller.ts: Fixed JWT token generation
- config/redis.ts: Updated connection pooling
- tests/auth.test.ts: Added mock setup for new config

## Decisions Made
- Using Redis connection pool instead of per-request connections
- Retry logic with exponential backoff for transient failures

## Current State
- 14 tests passing, 2 failing
- Remaining: mock setup for session service tests

## Next Steps
1. Fix remaining test failures
2. Run full test suite
3. Update documentation
```

This structure prevents silent loss of file paths or decisions because each section must be explicitly addressed.

这种结构防止文件路径或决策的静默丢失，因为每个章节必须被明确处理。

### Compression Trigger Strategies

### 压缩触发策略

When to trigger compression matters as much as how to compress:

何时触发压缩与如何压缩同样重要：

| Strategy | Trigger Point | Trade-off |
|----------|---------------|-----------|
| Fixed threshold | 70-80% context utilization | Simple but may compress too early |
| Sliding window | Keep last N turns + summary | Predictable context size |
| Importance-based | Compress low-relevance sections first | Complex but preserves signal |
| Task-boundary | Compress at logical task completions | Clean summaries but unpredictable timing |

| 策略 | 触发点 | 权衡 |
|------|--------|------|
| 固定阈值 | 70-80% 上下文利用率 | 简单但可能压缩过早 |
| 滑动窗口 | 保留最近 N 轮 + 摘要 | 可预测的上下文大小 |
| 基于重要性 | 优先压缩低相关性部分 | 复杂但保留信号 |
| 任务边界 | 在逻辑任务完成时压缩 | 清晰的摘要但时机不可预测 |

The sliding window approach with structured summaries provides the best balance of predictability and quality for most coding agent use cases.

滑动窗口方法配合结构化摘要为大多数编码智能体用例提供了可预测性和质量的最佳平衡。

### Probe-Based Evaluation

### 基于探针的评估

Traditional metrics like ROUGE or embedding similarity fail to capture functional compression quality. A summary may score high on lexical overlap while missing the one file path the agent needs.

传统指标如 ROUGE 或嵌入相似度无法捕获功能性压缩质量。摘要可能在词汇重叠上得分很高，但错过了智能体需要的那个文件路径。

Probe-based evaluation directly measures functional quality by asking questions after compression:

基于探针的评估通过压缩后提问直接测量功能质量：

| Probe Type | What It Tests | Example Question |
|------------|---------------|------------------|
| Recall | Factual retention | "What was the original error message?" |
| Artifact | File tracking | "Which files have we modified?" |
| Continuation | Task planning | "What should we do next?" |
| Decision | Reasoning chain | "What did we decide about the Redis issue?" |

| 探针类型 | 测试内容 | 示例问题 |
|----------|----------|----------|
| 召回 | 事实保留 | "原始错误消息是什么？" |
| 工件 | 文件追踪 | "我们修改了哪些文件？" |
| 延续 | 任务规划 | "我们接下来应该做什么？" |
| 决策 | 推理链 | "关于 Redis 问题我们做了什么决定？" |

If compression preserved the right information, the agent answers correctly. If not, it guesses or hallucinates.

如果压缩保留了正确的信息，智能体会正确回答。否则，它会猜测或产生幻觉。

### Evaluation Dimensions

### 评估维度

Six dimensions capture compression quality for coding agents:

六个维度捕获编码智能体的压缩质量：

1. **Accuracy**: Are technical details correct? File paths, function names, error codes.
2. **Context Awareness**: Does the response reflect current conversation state?
3. **Artifact Trail**: Does the agent know which files were read or modified?
4. **Completeness**: Does the response address all parts of the question?
5. **Continuity**: Can work continue without re-fetching information?
6. **Instruction Following**: Does the response respect stated constraints?

1. **准确性**：技术细节是否正确？文件路径、函数名、错误代码。
2. **上下文感知**：响应是否反映当前对话状态？
3. **工件追踪**：智能体是否知道读取或修改了哪些文件？
4. **完整性**：响应是否解决问题的所有部分？
5. **连续性**：能否在不重新获取信息的情况下继续工作？
6. **指令遵循**：响应是否遵守声明的约束？

Accuracy shows the largest variation between compression methods (0.6 point gap). Artifact trail is universally weak (2.2-2.5 range).

准确性在压缩方法之间显示出最大的差异（0.6 分差距）。工件追踪普遍较弱（2.2-2.5 范围）。

## Practical Guidance

## 实用指南

### Three-Phase Compression Workflow

### 三阶段压缩工作流

For large codebases or agent systems exceeding context windows, apply compression through three phases:

对于超过上下文窗口的大型代码库或智能体系统，通过三个阶段应用压缩：

1. **Research Phase**: Produce a research document from architecture diagrams, documentation, and key interfaces. Compress exploration into a structured analysis of components and dependencies. Output: single research document.

1. **研究阶段**：从架构图、文档和关键接口生成研究文档。将探索压缩为组件和依赖的结构化分析。输出：单一研究文档。

2. **Planning Phase**: Convert research into implementation specification with function signatures, type definitions, and data flow. A 5M token codebase compresses to approximately 2,000 words of specification.

2. **规划阶段**：将研究转化为实现规范，包含函数签名、类型定义和数据流。500万 token 的代码库压缩为约 2000 字的规范。

3. **Implementation Phase**: Execute against the specification. Context remains focused on the spec rather than raw codebase exploration.

3. **实现阶段**：按照规范执行。上下文保持专注于规范而非原始代码库探索。

### Using Example Artifacts as Seeds

### 使用示例工件作为种子

When provided with a manual migration example or reference PR, use it as a template to understand the target pattern. The example reveals constraints that static analysis cannot surface: which invariants must hold, which services break on changes, and what a clean migration looks like.

当提供手动迁移示例或参考 PR 时，将其作为模板来理解目标模式。示例揭示了静态分析无法发现的约束：哪些不变量必须保持，哪些服务在更改时会中断，以及干净的迁移是什么样的。

This is particularly important when the agent cannot distinguish essential complexity (business requirements) from accidental complexity (legacy workarounds). The example artifact encodes that distinction.

当智能体无法区分本质复杂性（业务需求）和偶然复杂性（遗留变通方案）时，这尤其重要。示例工件编码了这种区别。

### Implementing Anchored Iterative Summarization

### 实现锚定迭代摘要

1. Define explicit summary sections matching your agent's needs
2. On first compression trigger, summarize truncated history into sections
3. On subsequent compressions, summarize only new truncated content
4. Merge new summary into existing sections rather than regenerating
5. Track which information came from which compression cycle for debugging

1. 定义匹配智能体需求的明确摘要章节
2. 首次压缩触发时，将截断的历史摘要到章节中
3. 后续压缩时，仅摘要新截断的内容
4. 将新摘要合并到现有章节而非重新生成
5. 追踪哪些信息来自哪个压缩周期以便调试

### When to Use Each Approach

### 何时使用每种方法

**Use anchored iterative summarization when:**
- Sessions are long-running (100+ messages)
- File tracking matters (coding, debugging)
- You need to verify what was preserved

**使用锚定迭代摘要的情况：**
- 会话是长时间运行的（100+ 消息）
- 文件追踪很重要（编码、调试）
- 需要验证保留了什么

**Use opaque compression when:**
- Maximum token savings required
- Sessions are relatively short
- Re-fetching costs are low

**使用不透明压缩的情况：**
- 需要最大 token 节省
- 会话相对较短
- 重新获取成本低

**Use regenerative summaries when:**
- Summary interpretability is critical
- Sessions have clear phase boundaries
- Full context review is acceptable on each compression

**使用再生摘要的情况：**
- 摘要可解释性至关重要
- 会话有清晰的阶段边界
- 每次压缩时完整上下文审查是可接受的

### Compression Ratio Considerations

### 压缩率考虑

| Method | Compression Ratio | Quality Score | Trade-off |
|--------|-------------------|---------------|-----------|
| Anchored Iterative | 98.6% | 3.70 | Best quality, slightly less compression |
| Regenerative | 98.7% | 3.44 | Good quality, moderate compression |
| Opaque | 99.3% | 3.35 | Best compression, quality loss |

| 方法 | 压缩率 | 质量分数 | 权衡 |
|------|--------|----------|------|
| 锚定迭代 | 98.6% | 3.70 | 最佳质量，稍低压缩率 |
| 再生 | 98.7% | 3.44 | 良好质量，中等压缩 |
| 不透明 | 99.3% | 3.35 | 最佳压缩，质量损失 |

The 0.7% additional tokens retained by structured summarization buys 0.35 quality points. For any task where re-fetching costs matter, this trade-off favors structured approaches.

结构化摘要额外保留的 0.7% token 换取了 0.35 个质量分数点。对于任何重新获取成本重要的任务，这种权衡有利于结构化方法。

## Examples

## 示例

**Example 1: Debugging Session Compression**

**示例 1：调试会话压缩**

Original context (89,000 tokens, 178 messages):
- 401 error on /api/auth/login endpoint
- Traced through auth controller, middleware, session store
- Found stale Redis connection
- Fixed connection pooling, added retry logic
- 14 tests passing, 2 failing

原始上下文（89,000 token，178 条消息）：
- /api/auth/login 端点上的 401 错误
- 追踪通过认证控制器、中间件、会话存储
- 发现陈旧的 Redis 连接
- 修复连接池，添加重试逻辑
- 14 个测试通过，2 个失败

Structured summary after compression:

压缩后的结构化摘要：

```markdown
## Session Intent
Debug 401 Unauthorized error on /api/auth/login despite valid credentials.

## Root Cause
Stale Redis connection in session store. JWT generated correctly but session could not be persisted.

## Files Modified
- auth.controller.ts: No changes (read only)
- middleware/cors.ts: No changes (examined)
- config/redis.ts: Fixed connection pooling configuration
- services/session.service.ts: Added retry logic for transient failures
- tests/auth.test.ts: Updated mock setup

## Test Status
14 passing, 2 failing (mock setup issues)

## Next Steps
1. Fix remaining test failures (mock session service)
2. Run full test suite
3. Deploy to staging
```

**Example 2: Probe Response Quality**

**示例 2：探针响应质量**

After compression, asking "What was the original error?":

压缩后，询问"原始错误是什么？"：

Good response (structured summarization):
> "The original error was a 401 Unauthorized response from the /api/auth/login endpoint. Users received this error with valid credentials. Root cause was stale Redis connection in session store."

良好的响应（结构化摘要）：
> "原始错误是来自 /api/auth/login 端点的 401 未授权响应。用户在提供有效凭据的情况下收到此错误。根本原因是会话存储中陈旧的 Redis 连接。"

Poor response (aggressive compression):
> "We were debugging an authentication issue. The login was failing. We fixed some configuration problems."

糟糕的响应（激进压缩）：
> "我们在调试一个认证问题。登录失败了。我们修复了一些配置问题。"

The structured response preserves endpoint, error code, and root cause. The aggressive response loses all technical detail.

结构化响应保留了端点、错误代码和根本原因。激进响应丢失了所有技术细节。

## Guidelines

## 指南

1. Optimize for tokens-per-task, not tokens-per-request
2. Use structured summaries with explicit sections for file tracking
3. Trigger compression at 70-80% context utilization
4. Implement incremental merging rather than full regeneration
5. Test compression quality with probe-based evaluation
6. Track artifact trail separately if file tracking is critical
7. Accept slightly lower compression ratios for better quality retention
8. Monitor re-fetching frequency as a compression quality signal

1. 优化每次任务的 token 数，而非每次请求的 token 数
2. 使用带有明确文件追踪章节的结构化摘要
3. 在 70-80% 上下文利用率时触发压缩
4. 实现增量合并而非完全再生
5. 使用基于探针的评估测试压缩质量
6. 如果文件追踪至关重要，单独追踪工件轨迹
7. 接受稍低的压缩率以获得更好的质量保留
8. 将重新获取频率作为压缩质量信号进行监控

## Integration

## 集成

This skill connects to several others in the collection:

此技能与集合中的其他几个技能相关联：

- context-degradation - Compression is a mitigation strategy for degradation
- context-optimization - Compression is one optimization technique among many
- evaluation - Probe-based evaluation applies to compression testing
- memory-systems - Compression relates to scratchpad and summary memory patterns

- context-degradation - 压缩是退化的缓解策略
- context-optimization - 压缩是众多优化技术之一
- evaluation - 基于探针的评估适用于压缩测试
- memory-systems - 压缩与便笺和摘要记忆模式相关

## References

## 参考资料

Internal reference:
- [Evaluation Framework Reference](./references/evaluation-framework.md) - Detailed probe types and scoring rubrics

内部参考：
- [评估框架参考](./references/evaluation-framework.md) - 详细的探针类型和评分标准

Related skills in this collection:
- context-degradation - Understanding what compression prevents
- context-optimization - Broader optimization strategies
- evaluation - Building evaluation frameworks

此集合中的相关技能：
- context-degradation - 理解压缩防止什么
- context-optimization - 更广泛的优化策略
- evaluation - 构建评估框架

External resources:
- Factory Research: Evaluating Context Compression for AI Agents (December 2025)
- Research on LLM-as-judge evaluation methodology (Zheng et al., 2023)
- Netflix Engineering: "The Infinite Software Crisis" - Three-phase workflow and context compression at scale (AI Summit 2025)

外部资源：
- Factory Research: 评估 AI 智能体的上下文压缩（2025年12月）
- LLM-as-judge 评估方法研究（Zheng et al., 2023）
- Netflix Engineering: "无限软件危机" - 大规模三阶段工作流和上下文压缩（AI Summit 2025）

---

## Skill Metadata

## 技能元数据

**Created**: 2025-12-22
**Last Updated**: 2025-12-26
**Author**: Agent Skills for Context Engineering Contributors
**Version**: 1.1.0

**创建日期**：2025-12-22
**最后更新**：2025-12-26
**作者**：Agent Skills for Context Engineering Contributors
**版本**：1.1.0

