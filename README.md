# Name That LLM 本地实验结果

对官方训练集的 2,699 段文本按 `prompt_id` 做固定五折分组。每次用四折训练，在持出的新题目上预测，汇总全部折外预测计算 18 类平均召回率。下表是本地 Balanced Accuracy，Kaggle 公榜和私榜以赛事评分为准。

已记录 **25** 个候选。每个候选目录有五折分数、逐类召回率和混淆矩阵。

| 实验 | Balanced Accuracy | 折 1 | 折 2 | 折 3 | 折 4 | 折 5 |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| [`hybrid-char-lm`](results/hybrid-char-lm) | 0.3860 | 0.3833 | 0.3648 | 0.3558 | 0.4315 | 0.3944 |
| [`hybrid`](results/hybrid) | 0.3838 | 0.3944 | 0.3537 | 0.3614 | 0.4241 | 0.3852 |
| [`external-hybrid`](results/external-hybrid) | 0.3830 | 0.3981 | 0.3556 | 0.3577 | 0.4241 | 0.3796 |
| [`hybrid-family`](results/hybrid-family) | 0.3823 | 0.3981 | 0.3593 | 0.3447 | 0.4259 | 0.3833 |
| [`hybrid-nb-svm`](results/hybrid-nb-svm) | 0.3812 | 0.4019 | 0.3519 | 0.3558 | 0.4241 | 0.3722 |
| [`hybrid-boundaries`](results/hybrid-boundaries) | 0.3808 | 0.4000 | 0.3500 | 0.3521 | 0.4370 | 0.3648 |
| [`hybrid-token-shape`](results/hybrid-token-shape) | 0.3800 | 0.3907 | 0.3556 | 0.3558 | 0.4278 | 0.3704 |
| [`masked-external`](results/masked-external) | 0.3786 | 0.3759 | 0.3556 | 0.3614 | 0.4130 | 0.3870 |
| [`masked`](results/masked) | 0.3782 | 0.3852 | 0.3444 | 0.3522 | 0.4315 | 0.3778 |
| [`svm`](results/svm) | 0.3723 | 0.3759 | 0.3519 | 0.3540 | 0.4111 | 0.3685 |
| [`minilm-hybrid`](results/minilm-hybrid) | 0.3704 | 0.3759 | 0.3370 | 0.3539 | 0.4185 | 0.3667 |
| [`separate`](results/separate) | 0.3645 | 0.3648 | 0.3704 | 0.3596 | 0.3667 | 0.3611 |
| [`tree-hybrid`](results/tree-hybrid) | 0.3619 | 0.3500 | 0.3556 | 0.3521 | 0.3963 | 0.3556 |
| [`wide`](results/wide) | 0.3567 | 0.3611 | 0.3463 | 0.3335 | 0.3833 | 0.3593 |
| [`blend`](results/blend) | 0.3493 | 0.3574 | 0.3500 | 0.3353 | 0.3667 | 0.3370 |
| [`style`](results/style) | 0.3374 | 0.3556 | 0.3315 | 0.3242 | 0.3556 | 0.3204 |
| [`nb-svm`](results/nb-svm) | 0.3275 | 0.3389 | 0.3074 | 0.3225 | 0.3556 | 0.3130 |
| [`word`](results/word) | 0.3252 | 0.3185 | 0.3037 | 0.3282 | 0.3278 | 0.3481 |
| [`char`](results/char) | 0.3212 | 0.3259 | 0.3148 | 0.3226 | 0.3130 | 0.3296 |
| [`family`](results/family) | 0.3134 | 0.3111 | 0.3222 | 0.2853 | 0.3537 | 0.2944 |
| [`token-shape`](results/token-shape) | 0.3037 | 0.3241 | 0.2722 | 0.2872 | 0.3222 | 0.3130 |
| [`char-lm`](results/char-lm) | 0.2826 | 0.3000 | 0.2722 | 0.2557 | 0.2852 | 0.3000 |
| [`boundaries`](results/boundaries) | 0.2801 | 0.2667 | 0.2667 | 0.2946 | 0.2815 | 0.2907 |
| [`style-tree`](results/style-tree) | 0.2226 | 0.2204 | 0.1963 | 0.2409 | 0.2500 | 0.2056 |
| [`minilm`](results/minilm) | 0.1348 | 0.1481 | 0.1259 | 0.1316 | 0.1389 | 0.1296 |

## 版本说明

- `char`、`word`、`style`、`wide` 对应字符 TF-IDF、加入词级 TF-IDF、加入写作风格统计、加入词内字符 n-gram。
- `separate` 分别学习字符、词和风格后融合。 `masked` 再加入保留标点与功能词的内容词遮蔽特征。
- `svm` 使用原文、内容词遮蔽和风格特征训练线性 SVM。 `hybrid` 融合 `svm` 与 `masked`，是只依赖单篇文本的首投基准。
- `char-lm` 比较每类字符语言模型对文本的似然。 `family` 先预测家族再预测具体模型。 `boundaries` 使用段落首尾，`token-shape` 使用词形与标点序列。
- `nb-svm` 用类别条件下的特征出现率给 SVM 特征加权。 `prompt-residual` 根据文本挖掘相似段落并减去共同主题成分，`-thin` 表示用更少的同时预测样本做压力验证。
- `minilm` 使用冻结 MiniLM 表示。带 `external` 的候选使用公开授权补充样本。

## 数据

官方数据页列出训练集规模、字段和评估背景。外部数据对照使用 Zakir Khan 发布的 CC BY 4.0 语料，并按来源说明引用两套公共 prompt。 [官方数据](https://www.kaggle.com/competitions/name-that-llm/data) · [公开补充样本](https://www.kaggle.com/datasets/zakirkhanaleemi/name-that-llm-benchmarks-proxy-generations) · [Seyit Kaan Güneş 的 prompt 来源](https://www.kaggle.com/datasets/seyitkaangunes/ntl-extra-api-passages) · [prvsiyan 的 prompt 来源](https://www.kaggle.com/datasets/prvsiyan/prvsiyans-extended-name-that-llm-dataset)
