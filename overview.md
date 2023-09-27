# 数伽

### 以entity为中心，多级扩展搜索方式的思路：

* query-->entities-->subgraph+prompt-->LLM-finetuned-->query+cot-->answer

**参考内容：**

1. [wey-gu/NebulaGraph-GPT: Help Compose your NebulaGraph Queries (github.com)](https://github.com/wey-gu/NebulaGraph-GPT)
2. [NebulaGraphQAChain | 🦜️🔗 Langchain](https://python.langchain.com/docs/use_cases/more/graph/graph_nebula_qa)
3. [Graph RAG: 知识图谱结合 LLM 的检索增强 - siwei.io](https://siwei.io/graph-rag/)
4. [Text2Cypher：大语言模型驱动的图谱查询生成 - siwei.io](https://siwei.io/llm-text-to-nebulagraph-query/)
5. [FudanDISC/DISC-MedLLM: Repository of DISC-MedLLM, it is a comprehensive solution that leverages Large Language Models (LLMs) to provide accurate and truthful medical response in end-to-end conversational healthcare services.](https://github.com/FudanDISC/DISC-MedLLM)

### 关键点：

1. query-->entities：
   1. 使用什么抽取方法兼顾效率和精度
   2. 意图识别加入？
2. finetune LLM：
   1. 建议若用中文模型，用BaiChuan-13B（v2），英文用LLaMA2-13B，70b级别用OpenBuddy-LLaMA2-70B
   2. 模型通用快速训练库：[https://github.com/hiyouga/LLaMA-Efficient-Tuning](https://github.com/hiyouga/LLaMA-Efficient-Tuning)
   3. lora模型权重合并（发代码）
   4. qlora（发代码和开源项目）
3. LLM（finetuned）：
   1. 根据三元组构造数据集：
      1. [https://huggingface.co/datasets/Open-Orca/OpenOrca](https://huggingface.co/datasets/Open-Orca/OpenOrca) 这个数据集是业内数据构造的标杆，保证了数据的丰富性和有效性。可以按照这个数据集的整理规则来。paper链接：[https://arxiv.org/abs/2306.02707](https://arxiv.org/abs/2306.02707)
      2. 构造数据生成的prompt（发代码）
      3. 多线程并发调用GPT（发代码）
      4. 数据清洗
   2. 评估标准
      1. 通用评估标准：
         1. [大语言模型评估全解：评估流程、评估方法及常见问题 - 知乎](https://zhuanlan.zhihu.com/p/644030637)
         2. [https://arxiv.org/pdf/2307.03109v6.pdf](https://arxiv.org/pdf/2307.03109v6.pdf)
         3. [可复现、自动化、低成本、高评估水平，首个自动化评估大模型的大模型 PandaLM 来了 | 机器之心](https://www.jiqizhixin.com/articles/2023-05-08-3)
      2. 人工评估：
         1. 构建数据集、明确评估方法，视业务情况而定
4. cot运用：
   1. 原理：[https://cloud.tencent.com/developer/article/2298660](https://cloud.tencent.com/developer/article/2298660)
   2. 运用：[解密 Prompt 系列 9. 模型复杂推理 - 思维链基础和进阶玩法 - 腾讯云开发者社区 - 腾讯云](https://cloud.tencent.com/developer/article/2296079?from=20421&from_column=20421)
   3. [https://github.com/PhoebusSi/Alpaca-CoT](https://github.com/PhoebusSi/Alpaca-CoT) ，这个库是个综合库，内部包含很多综合内容，例如数据集，prompt，方法等。

## 模型量化、推理加速

1. exllama： exllama刚开源了第2代，[turboderp/exllamav2: A fast inference library for running LLMs locally on modern consumer-class GPUs](https://github.com/turboderp/exllamav2) 其原理是采用混合量化模式加校准的方法，可显著加速LLaMA系模型的推理速度，目前测试2张3090可以加载openbuddy-70B级别的模型（基于LLaMA2-70b），推理速度为：11~12tokens/s，按照llama的词表约3个汉字每秒，显存占用40G。如果换用baichuan-13B模型，单卡甚至可以加载多个量化后的模型，速度也能得到保证，不失为一种部署方案。
2. gptq： 一种量化技术，现在已经被huggingface库收入，可结合上面的exllama使用，加速推理，目前在基本没损失的情况下最大量化为8bit,建议模型若不大于13b，量化级别不要低于4bit，再缩小会导致较大的损失。
3. vllm：[vllm-project/vllm: A high-throughput and memory-efficient inference and serving engine for LLMs](https://github.com/vllm-project/vllm#h5o-25) 基于Pagedattention的模型加速架构，目前适配大多都没问题。
4. ggml/fastlm： 简单的暴力量化，基本就是参数都不管，然后通通量化，量化后无校准过程，直接将量化后的结果作为模型结果，适用性强，但有点得不偿失，在测试超大模型但上述其他没有适配的时候可以尝试。

## 其他

1. 模型和图谱结合综述：[RManLuo/Awesome-LLM-KG: Awesome papers about unifying LLMs and KGs](https://github.com/RManLuo/Awesome-LLM-KG#kg-enhanced-llms)
2. 其他思路：[xttxECNU/Text2DT_Baseline](https://github.com/xttxECNU/Text2DT_Baseline)[2307.07306.pdf (arxiv.org)](https://arxiv.org/pdf/2307.07306.pdf)
3. 多轮对话： 就是传统对话系统里面，如果要判断是否跟上下文关联的话，通常需要训练一个分类模型，输入上文和本轮query，输出是否关联，如果关联就把上文内容拼接，如果不关联，就不管上文了，一般我们会设定关联历史轮次的最大值（比如3轮），不然没法做了。
4. 模型认知： 目前的模型大多都属于有各自的缺陷的，比如baichuan-13b，虽然中文能力足够，但是由于模型大小限制其智商有限，有机推理能力也不足，但是例如基于LLaMA2-70b做中文语料增量训练的一些模型，虽然逻辑能力足够，但是由于中文预料训练数据不足，所以针对中文通识知识存在很多认知误区。*所以如何在自己的专业领域内运用好模型，如何用领域数据合理的训练合适的基座模型，是在业务场景上至关重要的。
