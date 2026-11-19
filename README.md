# Name That LLM 本地实验结果

对官方训练集的 2,699 段文本按 `prompt_id` 做固定五折分组。每次用四折训练，在持出的新题目上预测，汇总全部折外预测计算 18 类平均召回率。下表是本地 Balanced Accuracy，Kaggle 公榜和私榜以赛事评分为准。

## Kaggle 提交成绩

| 版本 | 本地五折 Balanced Accuracy | Kaggle 公榜 |
| --- | ---: | ---: |
| v1 · 初次提交 | 0.4101 | 0.384 |
| v2 · Paired Style | 0.4286 | 0.400 |
| v3 · 冻结 ModernBERT 特征融合 | 0.4382 | 0.419 |
| v4 · 加入题目相关的写作风格 | 0.4445 | 0.414 |
| v5 · v4 与额外训练的 ModernBERT 融合 | 0.4486 | 尚无公榜分 |
| v6 · v3 与文本近邻校准 | 0.4612 | 尚无公榜分 |
| 双 Qwen · 两阶段 LoRA 与文本近邻校准 | 无独立同折分数 | **0.593** |

11 月 19 日北京时间 21:55:19，双 Qwen 版本成为本队第五次正式提交，距上次提交 3 小时 55 分 03 秒。Kaggle 显示该次评测成功，公榜为 **0.593**，比此前本队最佳分数提高 0.174，比首次提交提高 0.209。仓库收录 **136** 个本地实验。

## 目标 0.700 的实验

Kaggle 在 Alex Chilton 的固定 V30 上显示作者公榜 0.702。本队派生版在公开样例上也运行模型，并保留完整评分规模上的模型、权重和后处理。45 项依赖从 Kaggle 原资源挂载。本队私有版本 371475859 已在两张 T4 上实际运行 1,868.6 秒，下载的预测文件含三个唯一 ID 和有效的模型名称。小样本未触发 GLM 全量似然阶段。[作者 V30](https://www.kaggle.com/code/alexchilton/name-that-llm-0-702-all-public-what-failed?scriptVersionId=370591902) · [固定输入及代码哈希](reports/alex-v30-inputs.json) · [本队云端预览](reports/v30-cloud-previews.json)

本队逐词元版本 371492434 也已在双 T4 上运行完成，耗时 2,033.8 秒。小批次下两个版本下载的三行文件逐字节一致，均通过官方样例 ID 与类别检查。逐词元模型在最终单元启用了修正后的分类顺序，实际 GLM 似然计算仍需更大批次触发。[本队两个云端运行记录](reports/v30-cloud-previews.json)

在自有 v6 模型上，加入 V30 的三条排版规则后，按题目分组的五折平衡准确率从 0.46489 升至 0.46674。在四套已有预测上，一次性应用所有规则和按训练折学习惩罚力度均未超过作者原有顺序，暂时保留原顺序。该实验只评估末端规则。[折内训练和配对比较](reports/typography-rule-overlap.json)

混合语料 ModernBERT 训练完成了五折。以 18% 权重加入 v5 后，平衡准确率由 0.44859 降至 0.44081，五折中只有一折上升，本轮未晋级。[逐折指标](reports/mixed40-v5-comparison.json)

CHEN Han 公开了 Qwen3-Embedding-8B 的官方训练集折外预测。2,699 行的顺序、18 个类别和五个训练折均已核对，每个验证题目都与对应训练折分离。只用单篇文本的分类头平衡准确率为 **0.65164**。加入 5% 的写作标记后为 0.65237。只在同系列两个候选接近时重排为 0.65238，纠正 28 篇、改错 26 篇，按题目重采样的差值区间跨过零。[原作者固定版本](https://www.kaggle.com/code/concyclics/0-679?scriptVersionId=365653660) · [固定权重与同系列重排实验](reports/native-qwen-sibling-margin.json)

在同一组折外预测上，我们用文本相似度寻找互为最近邻的文章，轻微降低两篇同时来自同一型号的概率。每题保留一篇时预测不变；每题三篇时从 0.65164 到 0.65238；完整题组上达到 0.65423。隐藏集的题组大小未知。[逐折分数、匹配数量与误判变化](reports/native-qwen-mutual-pairs.json)

另外尝试了根据输出字数推断模型及按缩写、标点使用机会建模。两种修正一起只用官方语料拟合，自有基线五折从 0.46118 到 0.46415。在 216 篇公开外部文章上，使用轻量标记分类器的对照从 0.20370 降到 0.19444，暂不进入提交。[语料来源、各类召回与外部确认](reports/source-response-experiments.json)

### 把 GLM 的文本似然传到最终分类

原方案只保留部分 GLM 分类结果。第二个候选让末端分类器读取已经算好的两种 GLM 的文本似然，改动一处参数，推理次数相同。我们用 José Padilla 公布的逐词元似然及 CHEN Han 的折外预测测试这一变化，按原始五个训练折重新拟合 GLM 分类头和风格分类器。[GLM 似然来源](https://www.kaggle.com/code/estopadilla/ntl-glm-likelihood-train) · [候选文件哈希及小批量验证](reports/glm-corrected-candidate.json)

| 同一批 2,699 篇文章的验证条件 | 原末端处理 | 传入 GLM 似然 | 再加同系列重排 |
| --- | ---: | ---: | ---: |
| 单篇 Qwen 分数 | 0.67570 | 0.68054 | 0.68054 |
| 同一批内每题最多两篇 | 0.68237 | 0.68571 | 0.68793 |
| 同一题的回答集中在同一批 | 0.70793 | 0.71017 | 0.71239 |

单篇条件下，传入似然纠正 17 篇、改错 4 篇，五折都上升。进步主要来自区分 Claude 与 GLM，两种 GLM 的平均召回从 0.5833 降到 0.5700。三种条件共用相同文章和 Qwen 基础模型，隐藏集的题目密度未知。表中分数仅用于候选排序。[完整对照与按题目重采样区间](reports/glm-sibling-factorial.json) · [各类别召回](reports/glm-cross-fit-plain.json)

### 只在低置信文本上运行 ATLAS 4B

采用 Harshita Kumari 的 Qwen3-4B ATLAS，给主模型最难区分的四分之一文本补算来源似然。固定 10% 融合权重，每篇分别计算全部 18 个来源或主模型排名前四的来源。四来源版保留候选集合原有的总概率。每折选样和排序只用基础模型的预测。[作者推理版本](https://www.kaggle.com/code/harshitakumari256/ntl-atlas-qwen3-4b-peft-inference?scriptVersionId=361750379) · [作者权重及折外似然](https://www.kaggle.com/datasets/harshitakumari256/ntl-atlas-bundle/versions/1)

| 同一批 2,699 篇文章 | 逐词元 GLM 基线 | 低置信文本算 18 个来源 | 低置信文本算前四个来源 |
| --- | ---: | ---: | ---: |
| 单篇 Qwen 分数 | 0.68425 | 0.69091 | 0.69239 |
| 每题最多两篇 | 0.68980 | 0.69349 | 0.69423 |
| 同题回答集中 | 0.71500 | 0.72018 | 0.71721 |

三个场景各有 674 篇进入 ATLAS。四来源版把需计算的文本与来源组合从 12,132 个降到 2,696 个。稀疏组相对基线改对 62 篇、改错 50 篇，增益的题目重采样区间跨过零。作者给出的五组验证 ID 与按题目分组的固定划分一致，现有材料无法独立核验作者每次训练的日志。[逐折分数、纠错数与配对区间](reports/atlas-four-prefixes.json)

Kaggle 私有版本 371505276 在双 T4 上运行了 189.5 秒。两张卡各预热 8 篇，再用同一批共 64 篇比较两种方法，并交换先后顺序。18 来源耗时 59.74 秒，四来源耗时 13.74 秒，单步实测加速 **4.35 倍**。两种方法对应的 320 个来源分数中，最大绝对误差 0.00312，全部在预设 FP16 容差内。这次计时包含原作者 18 来源版本额外计算的 clipped S 分数。[双 T4 吞吐、显存和数值误差](reports/atlas-prefix-t4-timing.json)

把四来源步骤放在 GLM 完成后，每题最多两篇时到达 **0.69534**，单篇为 0.69164，密集组为 0.71647。稀疏组相对逐词元基线改对 62 篇、改错 47 篇，按题目重采样的 95% 增益区间为 −0.00185 至 +0.01292。这个顺序允许先写好完整的 GLM 预测，再利用剩余时间加算 ATLAS。交付笔记本已用五折真实似然重放，得到同一个 0.69534。模拟 GPU 失败、输出缺失和时间不足时，原预测文件的哈希均保持一致。[两种执行顺序](reports/atlas-after-glm.json) · [交付笔记本重放与超时验证](reports/atlas-after-glm-deployment.json)

在单篇 Qwen 验证上，分别让两种 GLM 变少、变多，或随机改变 18 类的比例，按新批次重新选择低置信文章及 GLM 计算对象。共 18 次抽样，追加四来源步骤相对逐词元基线的增益为 +0.00443 至 +0.00851。这些抽样复用了原来的文章和模型。[类别比例压力测试](reports/atlas-after-glm-prevalence.json)

### 比较两个 GLM 对同一个词元的判断

我们核对了 GLM-5.3 与 Flash 的 395,382 个词元 ID、段落边界和前缀长度。原有分类头先分别汇总两个模型的似然；新分类头增加 22 个统计量，直接比较两个模型在同一位置赋给实际词元的概率，包括概率差的分位数、高难度词元的差值，以及两者首选词元不一致的比例。新增特征复用已有的两次前向计算。每折的分类头只用另外四折训练。[逐词元源数据](https://www.kaggle.com/code/estopadilla/ntl-glm-likelihood-train/versions/3)

额外按训练折词频分为四组，分别计算两个 GLM 的相对表现。稀疏组从 0.68980 降到 0.68794，单篇与密集组在五位小数上持平。暂保留 76 维逐词元版本。[词频实验与逐折结果](reports/glm-frequency-bands.json)

### Gemini 3.7 与 3.8 的同题对照

稀疏题组中，原始 Qwen 把 48 篇 Gemini 3.7 回答判作 3.8，又把 46 篇 3.8 回答判作 3.7。我们用训练折中同一题的两份回答学习文字差异，模型第一、第二名恰为这两个型号时才调节二者的相对概率。单篇五折从 0.68425 到 0.68351，稀疏组持平于 0.68980，密集组从 0.71500 到 0.71463。这个分类器未加入提交版本。[五折与外部盲评](reports/gemini-paired-specialist.json)

另封存了 1,698 对同题 Vertex API 回答。它们的服务型号和生成时间均有逐条记录，均晚于两个早期 Qwen 组件发布；两个文本分类器在 3,396 篇两类盲评中均判对 1,800 篇（53.00%）。这些训练外记录适用于已核查的早期组件。完整 V30 的 SVC4 已见全部 3,396 个段落 ID。[数据来源与组件重合审计](reports/atlas4b-overlap-audit.json)

| 验证条件 | 已传入 GLM 似然 | 增加逐词元比较 | 纠正 / 改错 |
| --- | ---: | ---: | ---: |
| 单篇 | 0.68054 | 0.68425 | 21 / 11 |
| 每题最多两篇 | 0.68571 | 0.68980 | 22 / 11 |
| 同题集中 | 0.71017 | 0.71500 | 19 / 6 |

我们还改变了验证批次中各类文章的比例。分别减少 GLM、提高 GLM 的相对占比，以及随机改变 18 类比例，每种情形抽取六次，并对每个样本批次重新选取最需要计算 GLM 似然的文章。逐词元候选在 18 次配对比较中均上升，差值范围为 0.00186 至 0.00631。这些实验复用了官方训练文章，公榜仍需本队直接提交验证。[五折数据](reports/glm-token-contrast.json) · [类别占比测试](reports/glm-prevalence-stress.json)

逐词元候选已经过 480 篇真实词元分数的推理重放。72 篇被选中，调换输入文件行序和底模类别顺序后，最终 Parquet 仍与独立计算一致。另做了固定计算量的选文实验，将不确定性纳入排序，单篇条件只多判对 3 篇，密集题组净少判对 2 篇，交付版保留按 GLM 概率排序。[部署核验和文件哈希](reports/glm-token-deployment.json) · [选文策略对照](reports/glm-selection-policy.json)

同样的估算计算预算下，将每篇上限由 160 词元降到 128 或 96，分别扩大到最多 18% 或 21% 的文章，并按新的词元长度重训分类头。128 词元只改善了单篇条件，96 词元三组均下降，保留 160。逐折实测词元位置另列，GPU 耗时仍需平台记录。[截断长度、覆盖率和实际位置数](reports/glm-coverage.json)

本队公榜 0.593 双 Qwen 的第二次私有外部诊断返回了 270 篇文章的完整概率。单独测试同系列重排，小样本 54 篇从 34 篇判对变为 35 篇，但 216 篇确认样本从 93 篇判对降到 92 篇。排版规则在两组上均未提高平衡准确率。外部语料的生成来源与官方测试可能不同。[双 Qwen 的独立外部对照](reports/paired-qwen-external-followup.json)

审计 V30 的训练来源后，270 篇中有 **233 篇**与其实际启用的 SVC4 监督训练文本匹配，**270 个题目指纹均在该训练池中**。另外，Nishant handwriting-blend 的训练池有 205 篇同文，与前述 233 篇存在重叠。我们复现了原预拟合 TF-IDF 和 Distort 的抽样，24,443 个训练 ID 的顺序哈希与作者记录相符，其中有 33 篇与诊断集同文。原始双 Qwen 使用另一份训练池，两个 GLM 分类头只用官方训练文章。由此将这组外部结果限定为相应组件的诊断；用于整个 V30 的可核验独立样本数为零。[逐组件来源、抽样验证和重叠统计](reports/v30-external-overlap.json)

## 实验说明

两套 Qwen 适配器均已用官方 2,699 行全量微调。表中保留本队实测成绩；作者报告其原方案公榜为 0.592。我们继承两套 Qwen 的输出、词法分类器和融合，新增 15% 权重的文本近邻校准。本队预览在两张 T4 上实际运行两套模型并生成三行输出。[作者 Notebook](https://www.kaggle.com/code/nishantdahal/two-stage-lora-on-the-public-pool) · [公开权重与数据来源](https://www.kaggle.com/datasets/nishantdahal/ntl-two-stage-lora/versions/1)

我们也在自有 v3 的折外预测上比较了两档类别分布校准强度。把强度减半时，完整批次的平衡准确率从 0.44377 降至 0.44266，每题约三段时从 0.42673 升至 0.43043。该对照保留两种设备无关的轻量后处理用于独立语料验证。[逐折与类别比例变化测试](results/v3-marginal-half-vs-full/comparison.json)

从 v1 出发，加入写作标记和外部语料词形分类器后，本地五折得分达到 0.4208。v2 根据同批预测的类别分布做轻量校准。v3 再加入冻结 ModernBERT 各层的词元均值和标准差特征。v4 通过相似训练题目估计一段文字通常具备的写作风格，把实际风格与该预期的差值用于分类。新分支以 8% 的权重与 v3 融合。

将 v3 的各验证折拆成六个小批次后，得分为 0.4378。相对于 v1 的本地预测，v3 改对 230 条，另有 154 条由正确变错，净增 76 条。按题目成组重采样 3,000 次，分差的 95% 百分位区间为 +0.0148 至 +0.0411。该区间描述固定预测在这批题目上的波动。[压力测试](results/postlb-paired-cal/stress.json)

本轮还对照了冻结表示的判别分类器和 Will Rice 发布的原始 API 语料。后者经路由、文本长度和重复检查后，抽取 3,620 条加入轻量分类器，按 10% 融入 v3 后得分为 0.43824。同样权重的纯官方训练数据对照为 0.43489。[Will Rice 原始语料](https://www.kaggle.com/datasets/willrice/name-that-llm-openrouter-harvest)

v5 的新编码器用每类 40 条外部样本补充训练，固定融合权重为 18%。分别与 v1、v3、v4 融合，五折分数为 0.41786、0.44304 和 0.44859。v5 对应 v4 融合方案，相对 v4 提高 0.00408，五折中三折提高、一折持平、一折下降，达到预设的 0.003 总分增幅及三折改善门槛。[v1 与 v3 的对照](results/hourly-encoder/s42-extra40-ep3/complete.json) · [v4 与 v5 的逐折对照](results/extra40-v4-blend-18/comparison.json)

### 文本近邻校准

取同一批次中词语最相近的段落，若许多相似段落都被高置信度判为同一个模型，则轻微下调该模型的概率。每折仅用训练文本构建词表。每题约三段时，从原始折外预测重算各小批次的两级校准，v3 得到 0.00630 的净增益，v4 加 Extra40 的分数轻微下降。每题仅一段时，三种配置的平衡准确率均与各自基线相同。

| 基线 | 完整批次基线 | 完整批次校准 | 稀疏批次基线 | 稀疏批次校准 | 完整批次提升折数 |
| --- | ---: | ---: | ---: | ---: | ---: |
| [v3（本轮 v6）](results/v3-text-neighbor-calibration/comparison.json) | 0.43821 | 0.46118 | 0.43303 | 0.43933 | 5/5 |
| [v3 + 18% Extra40](results/v5-v3-text-neighbor-calibration/comparison.json) | 0.44304 | 0.46378 | 0.43860 | 0.44304 | 5/5 |
| [v4 + 18% Extra40](results/v5-text-neighbor-calibration/comparison.json) | 0.44859 | 0.46156 | 0.44600 | 0.44526 | 4/5 |

另两项对照同以 v4 + Extra40 的 0.44859 为基准。对易混淆型号做成对重排得到 0.44600，加入段落内部重复模式得到 0.44229。成对重排的二级模型复用了折外预测，报告保留了该验证设计的限制。[成对重排](results/v5-confusion-pairs/comparison.json) · [段落重复模式](results/passage-recurrence-v5-08/comparison.json)

### 冻结表示与外部语料对照

用同批相近段落估计共同的主题表示，再对冻结 ModernBERT 向量做中心化。完整批次中，v3 从 0.43821 提升至 0.44636，五折均有改善；v6 从 0.46118 提升至 0.46489，三折提高、一折持平、一折下降。每题约三段时，v3 从 0.43303 提升至 0.43488，v6 维持 0.43933。各小批次都重新计算了适用的上游校准。[逐折对照](results/frozen-twins-v6-full/comparison.json) · [方法来源](https://huggingface.co/datasets/yauwes/name-that-llm-sota-artifacts)

将文本近邻的校准改成三轮成对消息传递后，完整批次从 v6 的 0.46118 降至 0.45304，稀疏批次从 0.43933 降至 0.43674。[消息传递对照](results/v7-soft-topic-messages/comparison.json)

在 v5 上以 10% 权重融合相同架构的线性分类器，纯官方数据得到 0.44230，补入 Will Rice 数据后为 0.44491，v5 基线为 0.44859。抽取的 3,620 段外部文本中，3,481 段能映射到题目。与既有补充数据的可映射题目去重后，仅余三个样本、两个不同题目。Maniceet 数据卡记录 7,753 段、十个型号，原始文件下载返回 HTTP 403，本轮训练使用量为零。[来源审计和逐折结果](reports/external-source-controls-v5.json) · [Will Rice 数据](https://www.kaggle.com/datasets/willrice/name-that-llm-openrouter-harvest) · [Maniceet 数据卡](https://www.kaggle.com/datasets/maniceet/name-that-llm-openrouter-openai-generations)

## 双 Qwen 的独立来源验证

同一套双 Qwen 模型在 Kaggle 上实际运行两组外部文本。第一组为 54 段，每类 3 段。第二组用不同题目和文本做确认，每类 12 段，共 216 段。所有文本均与官方训练集和作者的 34,940 行训练池做了去重。两组各自计算主题聚类和类别概率校准。第二组的 216 段中，同一发布者提供了 166 段，另有 12 段 Gemini 3.6 只有发布者记录的直接 API 型号。

| 方法 | 54 段 | 216 段 | 逐行确认服务端型号的 204 段 |
| --- | ---: | ---: | ---: |
| 双 Qwen，本队提交配方 | 0.62963 | 0.43056 | 0.43137 |
| 双 Qwen，类别分布校准强度减半 | 0.59259 | 0.44907 | 0.45098 |
| 自有写作风格标记对照 | 0.25926 | 0.20370 | 0.21078 |

类别分布校准减半，在 216 段上多判对 4 段，在 54 段上少判对 2 段。在两组里，将文本近邻权重设为 0 或 15% 得到相同的平衡准确率。逐类统计显示，提交配方在 216 段上正确识别了 12 段 GLM-5.3 中的 1 段，以及 12 段 Gemini-3.8-Flash 中的 10 段。[真实云端逐类结果与原始数据来源](reports/two-qwen-external-audit.json) · [写作标记对照](reports/writing-markers-external.json)

在官方训练集的固定折外预测上，按题目有放回抽样 10,000 次，每次重算 18 类平均召回率。v6 相对 v3 的增幅为 0.02298，95% 百分位区间为 0.01481 至 0.03150。中心化表示相对 v6 增加 0.00371，对应区间为 −0.00221 至 0.01000。该估计只反映这批固定折外预测对题目组成的敏感程度。[成对重采样报告](reports/grouped-comparisons-v6.json)

## 全部本地五折结果

| 实验 | Balanced Accuracy | 折 1 | 折 2 | 折 3 | 折 4 | 折 5 |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| [`frozen-twins-v6-full`](results/frozen-twins-v6-full) | 0.4649 | 0.4519 | 0.4574 | 0.4522 | 0.4889 | 0.4741 |
| [`v5-v3-text-neighbor-calibration`](results/v5-v3-text-neighbor-calibration) | 0.4638 | 0.4519 | 0.4426 | 0.4485 | 0.4926 | 0.4833 |
| [`v5-text-neighbor-calibration`](results/v5-text-neighbor-calibration) | 0.4616 | 0.4463 | 0.4537 | 0.4522 | 0.4796 | 0.4759 |
| [`v3-text-neighbor-calibration`](results/v3-text-neighbor-calibration) | 0.4612 | 0.4481 | 0.4444 | 0.4541 | 0.4852 | 0.4741 |
| [`v7-soft-topic-messages`](results/v7-soft-topic-messages) | 0.4530 | 0.4500 | 0.4370 | 0.4356 | 0.4796 | 0.4630 |
| [`extra40-v4-blend-18`](results/extra40-v4-blend-18) | 0.4486 | 0.4500 | 0.4389 | 0.4281 | 0.4704 | 0.4556 |
| [`frozen-twins-v3-full`](results/frozen-twins-v3-full) | 0.4464 | 0.4333 | 0.4333 | 0.4337 | 0.4685 | 0.4630 |
| [`v5-confusion-pairs`](results/v5-confusion-pairs) | 0.4460 | 0.4389 | 0.4389 | 0.4318 | 0.4741 | 0.4463 |
| [`source-control-v5-willrice`](results/source-control-v5-willrice) | 0.4449 | 0.4519 | 0.4370 | 0.4245 | 0.4648 | 0.4463 |
| [`topic-style-length-gate`](results/topic-style-length-gate) | 0.4445 | 0.4407 | 0.4426 | 0.4244 | 0.4704 | 0.4444 |
| [`topic-style-v4-blend-8`](results/topic-style-v4-blend-8) | 0.4445 | 0.4407 | 0.4426 | 0.4244 | 0.4704 | 0.4444 |
| [`extra40 + postlb-paired-frozen-cal (v3)`](results/hourly-encoder/s42-extra40-ep3/on-postlb-paired-frozen-cal) | 0.4430 | 0.4463 | 0.4333 | 0.4170 | 0.4667 | 0.4519 |
| [`v3-marginal-half-vs-full`](results/v3-marginal-half-vs-full) | 0.4427 | 0.4370 | 0.4333 | 0.4317 | 0.4611 | 0.4500 |
| [`source-control-v5-official`](results/source-control-v5-official) | 0.4423 | 0.4426 | 0.4278 | 0.4282 | 0.4556 | 0.4574 |
| [`passage-recurrence-v5-08`](results/passage-recurrence-v5-08) | 0.4423 | 0.4370 | 0.4352 | 0.4225 | 0.4685 | 0.4481 |
| [`topic-style-v4-blend-16`](results/topic-style-v4-blend-16) | 0.4404 | 0.4352 | 0.4426 | 0.4151 | 0.4667 | 0.4426 |
| [`frozen-twins-v6-thin`](results/frozen-twins-v6-thin) | 0.4393 | 0.4352 | 0.4278 | 0.4151 | 0.4741 | 0.4444 |
| [`postlb-frozen-disc-length-6`](results/postlb-frozen-disc-length-6) | 0.4389 | 0.4278 | 0.4352 | 0.4151 | 0.4611 | 0.4556 |
| [`harvest-v3-blend-10`](results/harvest-v3-blend-10) | 0.4382 | 0.4259 | 0.4315 | 0.4245 | 0.4648 | 0.4444 |
| [`postlb-paired-frozen-cal`](results/postlb-paired-frozen-cal) | 0.4382 | 0.4296 | 0.4278 | 0.4188 | 0.4630 | 0.4519 |
| [`harvest-v3-control-blend-10`](results/harvest-v3-control-blend-10) | 0.4349 | 0.4296 | 0.4259 | 0.4208 | 0.4537 | 0.4444 |
| [`frozen-twins-v3-thin`](results/frozen-twins-v3-thin) | 0.4349 | 0.4278 | 0.4241 | 0.4132 | 0.4667 | 0.4426 |
| [`harvest-v3-control-blend-20`](results/harvest-v3-control-blend-20) | 0.4301 | 0.4463 | 0.4204 | 0.4190 | 0.4426 | 0.4222 |
| [`harvest-v3-blend-20`](results/harvest-v3-blend-20) | 0.4294 | 0.4259 | 0.4148 | 0.4192 | 0.4519 | 0.4352 |
| [`postlb-paired-cal`](results/postlb-paired-cal) | 0.4286 | 0.4259 | 0.4130 | 0.4153 | 0.4556 | 0.4333 |
| [`markers-frozen-ext-30`](results/markers-frozen-ext-30) | 0.4282 | 0.4148 | 0.4056 | 0.4281 | 0.4611 | 0.4315 |
| [`markers-frozen-ext-15`](results/markers-frozen-ext-15) | 0.4245 | 0.4130 | 0.4074 | 0.4059 | 0.4648 | 0.4315 |
| [`markers-adapter-20`](results/markers-adapter-20) | 0.4215 | 0.4111 | 0.4093 | 0.4042 | 0.4648 | 0.4185 |
| [`markers-v2-12-cal-25`](results/markers-v2-12-cal-25) | 0.4212 | 0.4148 | 0.4167 | 0.3987 | 0.4685 | 0.4074 |
| [`seq-v2-corpus-marker-10`](results/seq-v2-corpus-marker-10) | 0.4208 | 0.4204 | 0.4037 | 0.4078 | 0.4556 | 0.4167 |
| [`postlb-paired`](results/postlb-paired) | 0.4208 | 0.4204 | 0.3981 | 0.4059 | 0.4537 | 0.4259 |
| [`markers-adapter-5`](results/markers-adapter-5) | 0.4182 | 0.4019 | 0.3981 | 0.4061 | 0.4722 | 0.4130 |
| [`markers-v2-12-cal-50`](results/markers-v2-12-cal-50) | 0.4179 | 0.4111 | 0.4056 | 0.4025 | 0.4630 | 0.4074 |
| [`extra40 + hybrid-modernbert (v1)`](results/hourly-encoder/s42-extra40-ep3/on-hybrid-modernbert) | 0.4179 | 0.4222 | 0.4019 | 0.3929 | 0.4574 | 0.4148 |
| [`markers-v2-12`](results/markers-v2-12) | 0.4156 | 0.4093 | 0.4056 | 0.3968 | 0.4630 | 0.4037 |
| [`markers-v2-syntax-marker-10`](results/markers-v2-syntax-marker-10) | 0.4152 | 0.4111 | 0.3963 | 0.3930 | 0.4537 | 0.4222 |
| [`seq-v2-corpus-fusion-15`](results/seq-v2-corpus-fusion-15) | 0.4145 | 0.4241 | 0.3981 | 0.4003 | 0.4352 | 0.4148 |
| [`seq-v2-two-corpora-fusion-15`](results/seq-v2-two-corpora-fusion-15) | 0.4134 | 0.4204 | 0.3981 | 0.3966 | 0.4370 | 0.4148 |
| [`markers-v2-ext-12`](results/markers-v2-ext-12) | 0.4123 | 0.4037 | 0.4074 | 0.3874 | 0.4500 | 0.4130 |
| [`hybrid-modernbert-cal-50`](results/hybrid-modernbert-cal-50) | 0.4116 | 0.4167 | 0.3944 | 0.3855 | 0.4537 | 0.4074 |
| [`markers-v2-12-cal-100`](results/markers-v2-12-cal-100) | 0.4112 | 0.4019 | 0.3926 | 0.3988 | 0.4463 | 0.4167 |
| [`hybrid-modernbert-cal-25`](results/hybrid-modernbert-cal-25) | 0.4112 | 0.4185 | 0.3944 | 0.3856 | 0.4463 | 0.4111 |
| [`hybrid-moments-v2-30`](results/hybrid-moments-v2-30) | 0.4108 | 0.4056 | 0.3907 | 0.3854 | 0.4352 | 0.4370 |
| [`hybrid-modernbert`](results/hybrid-modernbert) | 0.4101 | 0.4185 | 0.3963 | 0.3838 | 0.4426 | 0.4093 |
| [`hybrid-moments-v2-15`](results/hybrid-moments-v2-15) | 0.4101 | 0.4167 | 0.3944 | 0.3854 | 0.4389 | 0.4148 |
| [`seq-v2-length-short65`](results/seq-v2-length-short65) | 0.4097 | 0.4185 | 0.4019 | 0.3745 | 0.4444 | 0.4093 |
| [`markers-adapter-1`](results/markers-adapter-1) | 0.4090 | 0.3963 | 0.3833 | 0.3856 | 0.4667 | 0.4130 |
| [`markers-v2-22`](results/markers-v2-22) | 0.4090 | 0.3981 | 0.3944 | 0.3950 | 0.4519 | 0.4056 |
| [`seq-v2-length-fusion-7`](results/seq-v2-length-fusion-7) | 0.4086 | 0.4204 | 0.3963 | 0.3800 | 0.4500 | 0.3963 |
| [`markers-v2-syntax-base-10`](results/markers-v2-syntax-base-10) | 0.4082 | 0.4093 | 0.4000 | 0.3762 | 0.4426 | 0.4130 |
| [`markers-v2-ext-22`](results/markers-v2-ext-22) | 0.4064 | 0.3889 | 0.3981 | 0.3838 | 0.4556 | 0.4056 |
| [`markers-v2-syntax-marker-20`](results/markers-v2-syntax-marker-20) | 0.4052 | 0.3852 | 0.3870 | 0.3985 | 0.4296 | 0.4259 |
| [`contrast-v2-phrase-15`](results/contrast-v2-phrase-15) | 0.4049 | 0.4056 | 0.3981 | 0.3837 | 0.4370 | 0.4000 |
| [`contrast-v2-bpe-15`](results/contrast-v2-bpe-15) | 0.4041 | 0.4056 | 0.3926 | 0.3762 | 0.4407 | 0.4056 |
| [`markers-v2-syntax-base-20`](results/markers-v2-syntax-base-20) | 0.4037 | 0.3944 | 0.3815 | 0.3909 | 0.4296 | 0.4222 |
| [`contrast-v2-15`](results/contrast-v2-15) | 0.4034 | 0.4093 | 0.3907 | 0.3762 | 0.4481 | 0.3926 |
| [`hybrid-modernbert-cal-100`](results/hybrid-modernbert-cal-100) | 0.4027 | 0.4019 | 0.3889 | 0.3781 | 0.4481 | 0.3963 |
| [`contrast-v2-tokens-15`](results/contrast-v2-tokens-15) | 0.4026 | 0.4148 | 0.3907 | 0.3706 | 0.4352 | 0.4019 |
| [`seq-v2-fusion-12`](results/seq-v2-fusion-12) | 0.4015 | 0.3963 | 0.3889 | 0.3874 | 0.4333 | 0.4019 |
| [`hybrid-glm-refine-25`](results/hybrid-glm-refine-25) | 0.3986 | 0.4056 | 0.3833 | 0.3781 | 0.4444 | 0.3815 |
| [`hybrid-two-corpora`](results/hybrid-two-corpora) | 0.3986 | 0.4093 | 0.3852 | 0.3744 | 0.4407 | 0.3833 |
| [`contrast-v2-tokens-30`](results/contrast-v2-tokens-30) | 0.3982 | 0.4074 | 0.3963 | 0.3724 | 0.4204 | 0.3944 |
| [`hybrid-glm-refine-50`](results/hybrid-glm-refine-50) | 0.3978 | 0.4056 | 0.3852 | 0.3743 | 0.4426 | 0.3815 |
| [`hybrid-claude-augment-50`](results/hybrid-claude-augment-50) | 0.3967 | 0.4056 | 0.3796 | 0.3782 | 0.4352 | 0.3852 |
| [`hybrid-claude-only-refine`](results/hybrid-claude-only-refine) | 0.3967 | 0.4074 | 0.3741 | 0.3762 | 0.4407 | 0.3852 |
| [`hybrid-claude-augment-25`](results/hybrid-claude-augment-25) | 0.3956 | 0.4056 | 0.3778 | 0.3726 | 0.4389 | 0.3833 |
| [`hybrid-extended-augment-50`](results/hybrid-extended-augment-50) | 0.3945 | 0.4000 | 0.3889 | 0.3687 | 0.4370 | 0.3778 |
| [`hybrid-glm-claude-official`](results/hybrid-glm-claude-official) | 0.3938 | 0.4000 | 0.3796 | 0.3726 | 0.4352 | 0.3815 |
| [`contrast-v2-phrase-30`](results/contrast-v2-phrase-30) | 0.3930 | 0.3944 | 0.3926 | 0.3761 | 0.4093 | 0.3926 |
| [`hybrid-extended-augment-25`](results/hybrid-extended-augment-25) | 0.3930 | 0.4019 | 0.3778 | 0.3669 | 0.4370 | 0.3815 |
| [`hybrid-style-mix-thin`](results/hybrid-style-mix-thin) | 0.3927 | 0.3963 | 0.3796 | 0.3615 | 0.4426 | 0.3833 |
| [`hybrid-style-mix`](results/hybrid-style-mix) | 0.3923 | 0.3981 | 0.3796 | 0.3596 | 0.4407 | 0.3833 |
| [`contrast-v2-30`](results/contrast-v2-30) | 0.3923 | 0.3870 | 0.3852 | 0.3780 | 0.4259 | 0.3852 |
| [`hybrid-glm-augment-50`](results/hybrid-glm-augment-50) | 0.3919 | 0.4037 | 0.3778 | 0.3688 | 0.4278 | 0.3815 |
| [`hybrid-glm-augment-25`](results/hybrid-glm-augment-25) | 0.3919 | 0.4056 | 0.3759 | 0.3669 | 0.4296 | 0.3815 |
| [`hybrid-char-typography`](results/hybrid-char-typography) | 0.3912 | 0.4000 | 0.3704 | 0.3651 | 0.4352 | 0.3852 |
| [`hybrid-glm-claude-augmented`](results/hybrid-glm-claude-augmented) | 0.3893 | 0.3907 | 0.3778 | 0.3595 | 0.4370 | 0.3815 |
| [`hybrid-prompt-residual`](results/hybrid-prompt-residual) | 0.3867 | 0.3889 | 0.3685 | 0.3522 | 0.4389 | 0.3852 |
| [`hybrid-typography`](results/hybrid-typography) | 0.3860 | 0.3981 | 0.3796 | 0.3651 | 0.4259 | 0.3611 |
| [`hybrid-char-lm`](results/hybrid-char-lm) | 0.3860 | 0.3833 | 0.3648 | 0.3558 | 0.4315 | 0.3944 |
| [`contrast-v2-bpe-30`](results/contrast-v2-bpe-30) | 0.3838 | 0.3722 | 0.3907 | 0.3614 | 0.4037 | 0.3907 |
| [`hybrid-prompt-residual-thin`](results/hybrid-prompt-residual-thin) | 0.3838 | 0.3833 | 0.3741 | 0.3541 | 0.4278 | 0.3796 |
| [`hybrid`](results/hybrid) | 0.3838 | 0.3944 | 0.3537 | 0.3614 | 0.4241 | 0.3852 |
| [`external-hybrid`](results/external-hybrid) | 0.3830 | 0.3981 | 0.3556 | 0.3577 | 0.4241 | 0.3796 |
| [`claude-augment-25`](results/claude-augment-25) | 0.3827 | 0.3852 | 0.3685 | 0.3597 | 0.4259 | 0.3741 |
| [`prompt-residual`](results/prompt-residual) | 0.3823 | 0.3759 | 0.3685 | 0.3727 | 0.4185 | 0.3759 |
| [`hybrid-family`](results/hybrid-family) | 0.3823 | 0.3981 | 0.3593 | 0.3447 | 0.4259 | 0.3833 |
| [`extended-augment-25`](results/extended-augment-25) | 0.3819 | 0.3889 | 0.3704 | 0.3576 | 0.4185 | 0.3741 |
| [`hybrid-nb-svm`](results/hybrid-nb-svm) | 0.3812 | 0.4019 | 0.3519 | 0.3558 | 0.4241 | 0.3722 |
| [`hybrid-boundaries`](results/hybrid-boundaries) | 0.3808 | 0.4000 | 0.3500 | 0.3521 | 0.4370 | 0.3648 |
| [`claude-augment-50`](results/claude-augment-50) | 0.3804 | 0.3815 | 0.3630 | 0.3709 | 0.4130 | 0.3741 |
| [`hybrid-token-shape`](results/hybrid-token-shape) | 0.3800 | 0.3907 | 0.3556 | 0.3558 | 0.4278 | 0.3704 |
| [`extended-augment-50`](results/extended-augment-50) | 0.3797 | 0.3759 | 0.3630 | 0.3613 | 0.4222 | 0.3759 |
| [`hybrid-family-refine`](results/hybrid-family-refine) | 0.3797 | 0.3833 | 0.3500 | 0.3522 | 0.4315 | 0.3815 |
| [`masked-external`](results/masked-external) | 0.3786 | 0.3759 | 0.3556 | 0.3614 | 0.4130 | 0.3870 |
| [`masked`](results/masked) | 0.3782 | 0.3852 | 0.3444 | 0.3522 | 0.4315 | 0.3778 |
| [`prompt-residual-thin`](results/prompt-residual-thin) | 0.3756 | 0.3667 | 0.3667 | 0.3709 | 0.4204 | 0.3537 |
| [`seq-v2-two-corpora`](results/seq-v2-two-corpora) | 0.3738 | 0.3759 | 0.3944 | 0.3635 | 0.3759 | 0.3593 |
| [`svm`](results/svm) | 0.3723 | 0.3759 | 0.3519 | 0.3540 | 0.4111 | 0.3685 |
| [`minilm-hybrid`](results/minilm-hybrid) | 0.3704 | 0.3759 | 0.3370 | 0.3539 | 0.4185 | 0.3667 |
| [`seq-v2-corpus`](results/seq-v2-corpus) | 0.3689 | 0.3648 | 0.3926 | 0.3596 | 0.3667 | 0.3611 |
| [`separate`](results/separate) | 0.3645 | 0.3648 | 0.3704 | 0.3596 | 0.3667 | 0.3611 |
| [`tree-hybrid`](results/tree-hybrid) | 0.3619 | 0.3500 | 0.3556 | 0.3521 | 0.3963 | 0.3556 |
| [`frozen-disc-pooled`](results/frozen-disc-pooled) | 0.3608 | 0.3593 | 0.3333 | 0.3562 | 0.3833 | 0.3722 |
| [`wide`](results/wide) | 0.3567 | 0.3611 | 0.3463 | 0.3335 | 0.3833 | 0.3593 |
| [`glm-augment-25`](results/glm-augment-25) | 0.3548 | 0.3630 | 0.3444 | 0.3315 | 0.3870 | 0.3481 |
| [`glm-augment-50`](results/glm-augment-50) | 0.3522 | 0.3574 | 0.3389 | 0.3315 | 0.3833 | 0.3500 |
| [`blend`](results/blend) | 0.3493 | 0.3574 | 0.3500 | 0.3353 | 0.3667 | 0.3370 |
| [`frozen-external-v2`](results/frozen-external-v2) | 0.3471 | 0.3296 | 0.3296 | 0.3354 | 0.3630 | 0.3778 |
| [`frozen-disc-length`](results/frozen-disc-length) | 0.3445 | 0.3222 | 0.3185 | 0.3580 | 0.3574 | 0.3667 |
| [`topic-style-v4`](results/topic-style-v4) | 0.3378 | 0.3426 | 0.3204 | 0.3280 | 0.3519 | 0.3463 |
| [`style`](results/style) | 0.3374 | 0.3556 | 0.3315 | 0.3242 | 0.3556 | 0.3204 |
| [`frozen-moments-v2`](results/frozen-moments-v2) | 0.3337 | 0.3296 | 0.3111 | 0.3169 | 0.3574 | 0.3537 |
| [`nb-svm`](results/nb-svm) | 0.3275 | 0.3389 | 0.3074 | 0.3225 | 0.3556 | 0.3130 |
| [`contrast-v2-solo`](results/contrast-v2-solo) | 0.3260 | 0.3296 | 0.3130 | 0.3466 | 0.3278 | 0.3130 |
| [`word`](results/word) | 0.3252 | 0.3185 | 0.3037 | 0.3282 | 0.3278 | 0.3481 |
| [`char`](results/char) | 0.3212 | 0.3259 | 0.3148 | 0.3226 | 0.3130 | 0.3296 |
| [`contrast-v2-tokens-solo`](results/contrast-v2-tokens-solo) | 0.3208 | 0.3278 | 0.3000 | 0.3056 | 0.3278 | 0.3426 |
| [`seq-v2-kernel`](results/seq-v2-kernel) | 0.3148 | 0.3278 | 0.3259 | 0.3207 | 0.3278 | 0.2722 |
| [`family`](results/family) | 0.3134 | 0.3111 | 0.3222 | 0.2853 | 0.3537 | 0.2944 |
| [`token-shape`](results/token-shape) | 0.3037 | 0.3241 | 0.2722 | 0.2872 | 0.3222 | 0.3130 |
| [`markers-v2-ext-standalone`](results/markers-v2-ext-standalone) | 0.2971 | 0.2630 | 0.2981 | 0.3041 | 0.3278 | 0.2926 |
| [`modernbert-standalone`](results/modernbert-standalone) | 0.2934 | 0.2981 | 0.2759 | 0.3057 | 0.2889 | 0.2981 |
| [`markers-v2-standalone`](results/markers-v2-standalone) | 0.2897 | 0.2685 | 0.2889 | 0.2818 | 0.3074 | 0.3019 |
| [`char-lm`](results/char-lm) | 0.2826 | 0.3000 | 0.2722 | 0.2557 | 0.2852 | 0.3000 |
| [`boundaries`](results/boundaries) | 0.2801 | 0.2667 | 0.2667 | 0.2946 | 0.2815 | 0.2907 |
| [`contrast-v2-phrase-solo`](results/contrast-v2-phrase-solo) | 0.2711 | 0.2630 | 0.2648 | 0.2593 | 0.2833 | 0.2852 |
| [`typography`](results/typography) | 0.2552 | 0.2296 | 0.2556 | 0.2817 | 0.2741 | 0.2352 |
| [`harvest-v3-control-raw`](results/harvest-v3-control-raw) | 0.2463 | 0.2352 | 0.2648 | 0.2580 | 0.2185 | 0.2556 |
| [`seq-v2-length`](results/seq-v2-length) | 0.2456 | 0.2593 | 0.2611 | 0.2114 | 0.2574 | 0.2389 |
| [`harvest-v3-raw`](results/harvest-v3-raw) | 0.2405 | 0.2463 | 0.2500 | 0.2363 | 0.2389 | 0.2315 |
| [`passage-recurrence-solo`](results/passage-recurrence-solo) | 0.2315 | 0.2056 | 0.2296 | 0.2446 | 0.2407 | 0.2370 |
| [`markers-v2-syntax-standalone`](results/markers-v2-syntax-standalone) | 0.2293 | 0.2000 | 0.2370 | 0.2278 | 0.2241 | 0.2574 |
| [`style-tree`](results/style-tree) | 0.2226 | 0.2204 | 0.1963 | 0.2409 | 0.2500 | 0.2056 |
| [`contrast-v2-bpe-solo`](results/contrast-v2-bpe-solo) | 0.2037 | 0.2000 | 0.2056 | 0.1928 | 0.2056 | 0.2148 |
| [`minilm`](results/minilm) | 0.1348 | 0.1481 | 0.1259 | 0.1316 | 0.1389 | 0.1296 |

## 版本说明

- `char`、`word`、`style`、`wide` 对应字符 TF-IDF、加入词级 TF-IDF、加入写作风格统计、加入词内字符 n-gram。
- `separate` 分别学习字符、词和风格后融合。 `masked` 再加入保留标点与功能词的内容词遮蔽特征。
- `svm` 使用原文、内容词遮蔽和风格特征训练线性 SVM。 `hybrid` 融合 `svm` 与 `masked`，是只依赖单篇文本的首投基准。
- `char-lm` 比较每类字符语言模型对文本的似然。 `family` 先预测家族再预测具体模型。 `boundaries` 使用段落首尾，`token-shape` 使用词形与标点序列。
- `nb-svm` 用类别条件下的特征出现率给 SVM 特征加权。 `prompt-residual` 根据文本挖掘相似段落并减去共同主题成分，`-thin` 表示用更少的同时预测样本做压力验证。
- `minilm` 使用冻结 MiniLM 表示。带 `external` 的候选使用公开授权补充样本。
- `typography` 用排版、标点和写作统计估计每类的特征概率。 `hybrid-char-typography` 只用单篇文本，`hybrid-style-mix` 同时用同批次的相似文本做主题残差。
- `hybrid-family-refine` 仅重排同系列型号，保留每个家族原有的概率总量。

公开方法参考包括字符语言模型、遮蔽、主题居中和排版指纹。 [字符作者语言模型](https://aclanthology.org/E03-1053/) · [内容词遮蔽](https://aclanthology.org/E17-1107/) · [主题居中](https://github.com/jerrythomas/name-that-llm) · [排版与家族重排](https://huggingface.co/datasets/yauwes/name-that-llm-sota-artifacts)
- `claude-augment` 和 `extended-augment` 是加权 SVM。25 和 50 是外部样本占对应类别训练权重的百分比；各类别的总权重相同。
- `hybrid-two-corpora` 融合单篇版与两套加权 SVM。新增样本只加入训练折，筛除由重建赛题 prompt 或 Kaggle Proxy 路由产生的记录。 [Claude API 语料与加权实验](https://www.kaggle.com/datasets/seyitkaangunes/ntl-extra-api-passages) · [18 类扩展语料](https://www.kaggle.com/datasets/prvsiyan/prvsiyans-extended-name-that-llm-dataset)
- `glm-augment` 单独对两个 GLM 型号补样；`glm-refine` 仅重排两个 GLM 型号；`glm-claude` 对七个 GLM 与 Claude 型号做专项对照。
- [ModernBERT 两折筛选](reports/modernbert-two-fold.json)记录每轮分数和 15% 与 25% 融合结果。完整五折实验列入上表。
- [ModernBERT 五折复核](reports/modernbert-five-fold.json)保留各轮学习曲线和同折对照。`hybrid-modernbert` 采用前两折选定的第 3 轮与 25% 融合权重，后三折沿用该配方。

## 数据

官方数据页列出训练集规模、字段和评估背景。外部数据对照使用 Zakir Khan 发布的 CC BY 4.0 语料，并按来源说明引用两套公共 prompt。 [官方数据](https://www.kaggle.com/competitions/name-that-llm/data) · [公开补充样本](https://www.kaggle.com/datasets/zakirkhanaleemi/name-that-llm-benchmarks-proxy-generations) · [Seyit Kaan Güneş 的 prompt 来源](https://www.kaggle.com/datasets/seyitkaangunes/ntl-extra-api-passages) · [prvsiyan 的 prompt 来源](https://www.kaggle.com/datasets/prvsiyan/prvsiyans-extended-name-that-llm-dataset)
