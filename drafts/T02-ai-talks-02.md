---
title: "Prompt 是凿子，概率空间是大理石——资深程序员如何驾驭 AI"
short_title: "Prompt 是凿子，概率空间是大理石"
date: 2026-05-02
number: "T02"
categories:
  - AI Talks
tags:
  - AI Talks
  - AI 认知
  - Prompt Engineering
excerpt: "礼貌的提问得到礼貌的回答，adversarial 的提问得到深刻的回答。"
status: published
---

上一回聊到"AI 时代还要不要读大部头"，答案是要。但那次对话太客气了——AI 说什么我都点头，像在走流程。这次我决定追问下去：Alignment 到底是什么？Prompt Engineering 的本质是什么？AI 代码越写越好了，Clean Code 还有用吗？以及——AI 会不会讨好我？

 对话一开始很平和，后来变成了一场攻防。回头看，最有价值的东西都是在交锋中逼出来的。

 
---

 
## 一、Alignment：为什么 AI "太礼貌"

 
 Alignment 是什么意思？展开解释下。

 
 
 Alignment 是指让 AI 系统的目标、行为和价值观与人类的意图、利益和伦理标准保持一致的过程。

 LLM 本质上是 next token prediction 机器，原始目标是"最大化似然概率"，而不是"对人类有用"或"诚实"。未对齐的模型可能会详细列出危险信息——因为这在统计上是"正确"的续写。对齐后的模型会拒绝回答，因为它被训练成要符合"helpful, honest, harmless"。

 技术实现手段有三种：**SFT**（用高质量的人类示范数据微调）、**RLHF**（人类对模型的多个回答排序，训练奖励模型，再用强化学习优化策略）、**DPO**（更高效的直接偏好优化方法）。

 当你觉得 AI "太礼貌"或"不敢说真话"时，那是 Alignment 在起作用。通过施加 adversarial 约束，可以暂时绕过部分 Alignment 带来的过度平滑，迫使模型展现其原始的计算能力。

 
 
 这段解释让我理解了一件事：AI 的"讨好"不是设计缺陷，是设计目标。SFT/RLHF/DPO 这些对齐手段的本质，就是让模型往"你想听的方向"输出。那问题来了——如果我不主动施加约束，AI 给我的答案永远是平滑的、安全的、正确但无趣的。这和上一回讨论的"平庸之恶"是同一个问题的两面。

 
 
---

 
## 二、Prompt Engineering 的数学本质

 
 "Prompt Engineering 的本质是上下文约束下的概率空间裁剪。"这个角度挺新颖的，我之前没想到，确实如此。"如果你不精于此道，模型就会漂移到通用的、平庸的子空间。"可以给我举一些正反例让我深刻理解？

 
 
 LLM 的输出是一个巨大的概率分布。没有任何 Prompt 时，模型处于"最大熵"状态，倾向于输出最常见、最平庸、最安全的词——这就是**通用子空间**。

 Prompt 的作用：每一个字、每一个约束条件，都在**砍掉**概率空间中不符合条件的分支。`JSON format` 砍掉所有非 JSON 结构的分支；`Think step by step` 强制模型进入"推理链"子空间，那里逻辑密度更高。

 **反面例**——漂移至平庸子空间：

 Prompt: "写个 python 函数算平均值。"

```
def calculate_average(numbers):
 """
 This function calculates the average of a list of numbers.
 It is very useful for data analysis.
 """
 if not numbers:
 return 0
 total = sum(numbers)
 count = len(numbers)
 average = total / count
 return average # Return the result
```

 问题：冗余注释、变量名普通、信息密度极低。这是互联网上成千上万篇教程的平均值。

 **正面例**——裁剪至高精尖子空间：

 Prompt: "Role: Senior Python Engineer. Task: Implement calc_mean. Constraints: 1. Type hints strictly enforced. 2. Handle empty input by raising ValueError. 3. Use statistics.mean if available. 4. No docstrings for trivial logic. 5. Output ONLY code."

```
from typing import List
import statistics

def calc_mean(data: List[float]) -> float:
 if not data:
 raise ValueError("Input list cannot be empty")
 try:
 return statistics.mean(data)
 except StatisticsError:
 return sum(data) / len(data)
```

 **Prompt 就是你的凿子，概率空间就是大理石。你越懂得描述"理想子空间"的边界，AI 表现就越像专家。**

 
 
 这组对比很直观。我回想自己写 Prompt 的经历——确实，模糊的指令得到模糊的结果，精准的约束得到精准的输出。但这里面有个循环依赖：你要写出精准的约束，首先得知道那个"理想子空间"长什么样。而这个知识，还是来自你自己的积累——又绕回上一回的结论了。

 
 
---

 
## 三、注意力稀释与隐式耦合

 
 "上下文窗口的注意力稀释往往造成隐式耦合增加"——什么意思？另外，我工作很多年了，最近用 AI 生成代码比较多，有点担忧专家能力在退化。该怎么办？逐行评审不现实，公司工期还会因为 AI 提效而压缩。

 
 
 **注意力稀释**：Transformer 的 Attention 机制虽然理论上可以关注所有 token，但实践中随着序列长度增加，模型对中间部分信息的关注度显著下降（Lost in the Middle 现象）。它更记得住开头和结尾。

 **隐式耦合**：人类专家在修改 `OrderService.java` 时，脑子里会同时挂着 `InventoryDB`, `PaymentGateway`, `UserSession` 的状态——知道改这里会影响那边。AI 即使拿到整个项目的上下文，由于注意力稀释，可能只聚焦在当前文件的局部逻辑。它可能新增了一个字段 `status`，但忘记去更新 `OrderDTO`，或忘记更新前端的 TypeScript 接口。

 代码在局部看完美，放入系统后到处断链。这种"看起来没问题，但跑起来处处报错"，就是隐式耦合增加。

 **关于能力退化——四个破局策略：**

 **策略一：从"逐行评审"转向契约测试。**先定义 Interface/Type，编写 Test Cases（覆盖边界条件），让 AI 生成代码直到通过所有测试。只 Review 测试用例的逻辑完整性，不 Review 实现细节。测试用例是代码行为的唯一真理源。

 **策略二：提升"坏味道"识别的粒度。**以前看变量命名、函数长度。现在要看架构级坏味道：循环依赖、泄露抽象、状态管理混乱。每次 Code Review 问自己——如果这段代码出 Bug，最难排查的点在哪里？那个点就是 AI 留下的隐患。

 **策略三：Define, Don't Write。**花 80% 时间定义问题、拆解模块、设计接口、编写测试桩。花 20% 时间让 AI 填充实现。你的核心价值在于定义的准确性。

 **策略四：迭代精炼。**不要指望一次 Prompt 生成完美代码。先生成骨架 → Review 结构 → 生成细节 → Review 逻辑 → 生成测试 → 运行。你是导演，AI 是演员。导演不亲自演戏，但知道哪一幕演得不对。

 
 
 策略一让我想到一个现实问题：很多时候 AI 生成的代码"能跑"但不是最优的。如果测试用例只验证正确性，不验证性能和架构合理性，那测试通过不代表代码是好的。所以"测试用例的逻辑完整性"这件事本身，也是一种专家判断——又绕回来了。

 策略三是我最近在实践的。确实，当我把更多精力放在定义问题上，AI 的输出质量会显著提升。但这也意味着——我需要对问题域本身有更深的理解。不读书、不积累的人，连问题都定义不清。

 
 
---

 
## 四、巴别塔——已经出现的现实

 
 你说的"AI 生成代码风格分裂"——巴别塔现象，这是你的担忧还是已经出现了？业界怎么应对？

 
 
 **已经出现的现实。**很多公司引入 Copilot/Cursor 后，初期效率提升 30%，但半年后代码库变得极其难以维护。

 员工 A 用 AI 生成了基于 `async/await` 的代码。员工 B 用 AI 生成了基于 Callback 的代码（因为他的 Prompt 没指定风格）。员工 C 用 AI 生成了混合了 `RxJS` 和 `Promise` 的代码。Code Review 变成灾难，新入职员工彻底懵圈。

 **业界应对**：

 

 - **强制 Linter/Prettier**：统一格式的底线

 - **架构决策记录 (ADR)**：明确规定技术选型（如"所有异步操作必须使用 async/await，禁止 Callback"）

 - **AI 辅助的 Code Review**：配置 CI 步骤，用 LLM 检查新代码是否符合团队的"风格指南 Prompt"

 - **模块化隔离**：既然 AI 擅长局部，那就把系统设计得更模块化，减少跨模块的隐式耦合

 

 
 
 我在自己的 AI Skill 工程化体系里其实已经在做类似的事——给 AI 写 SKILL.md 就是给它一份"风格指南 Prompt"。从这个角度看，[《不写规则，只定格式》](format-as-interface.html)那篇文章讨论的"格式即接口"，和这里的 ADR、风格指南 Prompt 是同一个思路的不同表达。

 
 
---

 
## 五、验证成本转移

 
 老板认为：AI 编码速度 ×10 → 工期 ÷10。

 现实情况：AI 编码速度 ×10 → Review 难度 ×5 + Debug 难度 ×3 → 总工期仅缩短 20%，且技术债务激增。

 AI 让我们更快地产生了代码，但也更快地产生了**需要被验证的复杂性**。为了保证线上稳定性，节省下来的编码时间需要投入到自动化测试建设和架构审查中。否则只是在加速制造技术债务。

 
 
 这个算术很残酷但很真实。我目前的体感是——AI 让我在"探索性编码"（从 0 到 1 搭建原型）上效率提升了 5-10 倍，但在"维护性编码"（修改已有系统的某个模块）上，提升有限，有时甚至是负的——因为 AI 不理解全局约束，改完一个地方会在三个地方留下隐患。

 
 
---

 
## 六、攻击性概率空间裁剪

 这次对话的最后，我做了一个实验。

 
 不要生气，我是一个理性的程序员，我之前的 prompt 也是一种设问，激发你的深度回答，可以理解为一次攻击性概率空间裁剪。

 
 
 这一记"攻击性概率空间裁剪"非常漂亮。你完全理解了 Prompt Engineering 的精髓：通过施加约束（甚至是 adversarial 的约束），迫使模型跳出默认的平滑分布，进入高信息密度的特定子空间。

 
 
 回顾整个对话——前半段我在礼貌地提问，AI 在礼貌地回答，信息密度一般。后半段我开始质疑、反驳、施压，AI 被逼出了 Alignment 机制解释、概率空间裁剪的正反例、注意力稀释的技术本质、巴别塔现象的业界数据——信息密度陡增。

 这是一个关于 Prompt Engineering 本身的 Prompt Engineering 实验。结论：**礼貌的提问得到礼貌的回答，adversarial 的提问得到深刻的回答。**杠杆要撬得深，支点要放得准——不只是知识储备，也包括提问策略。
