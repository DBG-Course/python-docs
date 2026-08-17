# 模型训练和微调

## MiniGPT 文件目录介绍

```bash
MiniGPT
├── README.md 
├── config
│   └── train_config.py # 模型参数文件
├── configurator.py
├── data
│   ├── prepare.py # 准备预训练数据 <TODO>
│   └── prepare_sft.py # 准备微调数据 <TODO>
├── data_utils.py # 构造每一个训练步骤中要输入模型的数据 <TODO>
├── evaluations.py # 实现模型评估相关指标，辅助观察模型效果 <TODO>
├── model.py # 底层模型实现
├── sample.py # 使用训练好的模型进行推理，
├── sample_gradio.py # 使用 gradio 来进行和模型交互的可视化 <TODO> 也可以选用非 gradio 的其他实现
├── train.py # 模型训练实现
└── visualize.py # 模型训练 loss 曲线/评估指标等可视化 <TODO>
```

## 模型结构

本作业**不涉及任何 transformer 模型结构上的实现及修改**。如果你想要了解更多关于transformer或GPT模型结构的知识，可以阅读论文：

* transformers: [https://arxiv.org/pdf/1706.03762.pdf](https://arxiv.org/pdf/1706.03762.pdf)
* GPT2: [https://d4mucfpksywv.cloudfront.net/better-language-models/language-models.pdf](https://d4mucfpksywv.cloudfront.net/better-language-models/language-models.pdf)

* 你也可以采用不同的模型结构，但是需要注意模型训练的时间，资源消耗以及更大的模型可能具有不同的训练性质。

## 模型训练

模型训练包括`预训练`和`微调`两个阶段。
本次作业也**不涉及对于训练流程以及损失函数等部分的实现及修改**。但是需要同学们参考预训练阶段的数据输入，**修改实现微调阶段的数据输入适配**。

### 预训练模型（Pretraing）

一个模型通过**预训练（pretraining）**来从零开始获取海量数据中的知识，自回归（Auto-regressive），即对一个句子从左到右进行预测，是简单但非常有效的训练方式。

MiniGPT采用自回归式的预训练及生成范式。在预训练阶段：

* 给定一段长为$N$的文本token序列。MiniGPT需要预测每一个位置的对应输出。对于位置$k$的token预测基于已给定的前$k-1$个token。
* 对于模型输出的预测使用交叉熵（见知识补充部分）构建损失函数来进行训练。
* 对于每条预训练数据的采集，我们从原始数据中随机采样一个起始位置$i$，然后取$[i, i+N-1]$这一段序列作为本条预训练数据。
  ```python
  ix = torch.randint(len(data) - block_size, (batch_size,))
  x = torch.stack([torch.from_numpy((data[i:i+block_size]).astype(np.int64)) for i in ix])
  y = torch.stack([torch.from_numpy((data[i+1:i+1+block_size]).astype(np.int64)) for i in ix])
  ```


### 有监督微调模型（Supervised Finetuning）

在已经完成预训练的模型的基础上，我们经常会进行**有监督的微调（Supervised Finetuning, SFT）**，使用高质量的人类标注数据继续训练我们的模型，使得模型能够给出更为符合回答要求与更安全的输出（即提高有用性与无害性）。

在本次作业中，我们使用如下格式的SFT数据：
```
问：微观经济学和宏观经济学分别研究哪些内容？ 
答：微观经济学检视一个社会里基本层次的行为，包括个体的行为者（例如个人、公司、买家或卖家）以及与市场的互动；宏观经济学则分析整个经济体和其议题，包括失业、通货膨胀、经济成长、财政和货币政策等。
```

进行SFT训练的流程与预训练阶段几乎相同，但在**数据预处理**与**损失函数计算位置**的设置上略有不同：

- **预处理**：对于单条SFT数据，我们将`问`与`答`两部分合并，截断或填充至固定长度（block size），来输入模型。本作业需要同学们参照预训练阶段，完成这一预处理的实现。
- **损失函数**：对于SFT训练的损失，我们只需要计算`答`部分的序列的预测。这一修改通过调整损失函数的输入参数`loss_mask`来实现。
  `loss_mask`表示需要进行损失计算的位置，是一段长为$L$的0/1序列，其中值为1的位置代表需要计算预测损失的位置。