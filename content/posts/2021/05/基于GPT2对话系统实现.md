---
title: "基于GPT2对话系统实现"
date: 2021-05-09T21:43:14+08:00
summary: "对话系统、对话生成，就是给定一句话，系统根据历史上下文对这句话进行回复"
draft: false
tags: [GPT2, Dialogue System, NLP]
---

## 简介

### NLP一般方法

自从预训练模型BERT(Bidirectional Encoder Representation for Transformer)诞生后，当前大多采用大规模通用语料如网页、维基百科，来训练预训练模型(Pretrained Model)，然后再根据下游任务对预训练模型进行精调(Fine Tuning)，这样的方式在许多任务上的表现非常好。

> 精调是个宽泛的说法，通常包括几种方式：不改变模型，只改变数据；根据任务只改变输出层，模型大体不动；对模型做出修改等

这里有几个注意的点：

1. 预训练模型普遍采用编码解码(Encoder-Decoder)模式，这也是一个广泛的说法。广义上指对数据$x$进行处理变成了中间态$z$：如把原始数据做Embedding、对数据处理后得到hidden output，相当于对数据进行了编码；然后对这些编码结果再进行处理，变成了我们想要的结果$y$，这类似与解码的过程。
2. 狭义上指使用了Transformer结构的模型，它包含了具体的Encoder跟Decoder部分，而BERT使用了Encoder部分，GPT使用了Decoder部分，所以上面说BERT含义大概是是双向的Transformer的Encoder表示
{{<figure title="Fig.1 The Transformer" alt="transformer" src="images/基于GPT2对话系统实现/transformer-encoder-decoder.png" width="75%">}}
3. Transformer采用注意力机制(Attention)，所谓注意力机制就是在一个句子中，基本的单位如字、词对另外的字或词是否可见、是否能注意到其它字词，相互是否能得到一个影响，如用概率大小来表示。注意到Encoder跟Decoder的Attention的差别
{{<figure title="Fig.2 Encoder" alt="Encoder" src="images/基于GPT2对话系统实现/transformer-encoder-block-2.png" width="75%">}}
{{<figure title="Fig.3 Decoder" alt="Decoder" src="images/基于GPT2对话系统实现/transformer-decoder-block-2.png" width="75%">}}
4. BERT使用了Self-Attention，而GPT使用了遮罩的注意力机制(Masked Self-Attetion)，只有后面的能注意到前面的
{{<figure title="Fig.4 Self-Attention" alt="Self-Attention" src="images/基于GPT2对话系统实现/self-attention-and-masked-self-attention.png" width="75%">}}

[Ref: Illustrated GPT2](http://jalammar.github.io/illustrated-gpt2/)

### 语言建模

对话系统、对话生成，就是要给定一句话，系统根据历史上下文对这句话进行回复。这是一个NLG(Natural Language Generation)任务，或者LM(Language Modeling)语言建模。

所谓语言建模，就是给定一个句子序列$x_1, x_2, \dots, x_{i-1}$，预测$x_i$, 给出预测的概率值。

$$
prob = p(x_{i} | x_{\lt i})
$$

或者，给定$x_1, \dots, x_{k-1}, x_{k+1}, \dots, x_n$，预测$x_k$

$$
prob = p(x_k | x_{i \ne k})
$$

句子生成的过程，其实也是训练模型的过程，所以叫语言建模。

### Why GPT2?

GPT2是一种自回归(Auto Regression, AR)模型，也称CLM(Casual Language Model)模型，两种说法等同，类似于RNN与LSTM。所谓自回归，就是会把输出附加到输入末尾，得到下一次的输入，如此往复直到符合结束条件，如达到句子最大长度、结束标志。这种模型是单向的，从左至右，所以没法知道后面的词对当前词的影响。

相对于自回归模型，另一种是遮罩模型MLM(Masked Language Modeling)，也称AE(Auto Encoding)模型，两种说法等同。这种模型会随机遮盖句子一部分，然后预测被Mask掉的词，如BERT。采用Self Attention机制，每两个字或词之间可以相互联系，所以是双向的，对于理解比较适合，常用来提取特征，然后做句子分类，情感分析之类的任务。

自回归模型比较符合人类语言从左往右的特性，特别适合于生成**流畅的**、**符合人类语言**的句子，但是生成的文本不可控，所以后续需要想办法！

### 术语

- CLM, Casual Language Modeling: 自回归语言建模，把输入附加到输入末尾，得到下一次的输入
- MLM, Masked Language Modeling: 遮罩的语言建模，随机遮罩句子一部分，对其进行预测
- AR, Auto Regression: 自回归，同CLM
- AE, Auto Encoding: 自编码，同MLM。因为采用了Self Attention机制，每两个字或词之间都可以相互联系，所以跟位置没有关系。而句子是有顺序的，要区分位置就引入了位置编码。这种方式类似于编码的过程，所以叫做自编码模型
- Self Attention: 自注意机制，可以对句子进行双向理解
- NLG, Natural Language Generation: 自然语言生成
- Seq2Seq, Sequence to Sequncce: 从一个序列到另一个序列，如机器翻译，文本生成
- token: 字或词，是模型处理的基本单位。要先把句子token化，然后转换成对应的id才能输入进模型，进行处理

[Ref: Terms](https://huggingface.co/transformers/glossary.html)

## 任务理解

我们可以对一个历史对话进行拼接(Concatenate)，形成一个长句子输入进模型，模型每次输出一个字；再添加到长句子末尾，再输入进模型。如此往复直到达到最大长度max_len，或者结束遇到\<eos\>，这个项目中是\<sep\>。如：

```text
今天好点了吗？
一天比一天严重
吃药不管用，去打一针。别拖着

转换为：
<cls>今天好点了吗？<sep>一天比一天严重<sep>吃药不管用，去打一针。别拖着<sep><pad>...
```

- \<cls\>(Classfier)：表示区分，这个是一个整体
- \<sep\>(Seperator)：表示分隔符，区分不同句子
- \<eos\>(End of Sentence)：结束符
- \<pad\>(Padding)：用于填充，因为要批处理，每个输入向量长度要一致

### 训练集

GPT2是使用大量**无标注数据**训练的自回归模型，将自然语言输入进模型就可以进行建模(Language Modeling)了，输出就是下一个字的概率，所以训练集可以这样构造：

```text
# train.txt
# dialog 1
conva
convb
conva
...

# dialog 2
conva
convb
...

...
```

[Ref: OpenAI GPT2](https://huggingface.co/transformers/model_doc/gpt2.html#)

### 项目结构与流程

因为代码里边参数的注释非常详细了，所以先只介绍整个大概的流程，以下是整个项目的结构

```python
project
├── config
│   └── args.py                     # 所有训练参数
├── data
│   ├── test_tokenized.txt          # 已处理的测试集
│   ├── test.txt                    # 原始测试集
│   ├── train_tokenized.txt
│   └── train.txt
├── logs                            # 日志目录
├── model
│   ├── model_config_....json       # 模型的配置信息
│   └── vocab_small.txt             # 字典
├── output
│   ├── dialogue_model
│   │   └── model_epoch1            # 训练好的模型;
│   │       ├── config.json         # 包含模型本身与配置文件
│   │       └── pytorch_model.bin
│   ├── mmi_model                   # 若开起了mmi，则输出mmi模型
│   │   └── model_epoch1
│   │       ├── config.json
│   │       └── pytorch_model.bin
│   ├── sample
│   │   └── output_test.txt         # 测试集输入进模型的输出
│   └── tensorboard_summary         # tensorborad输出
├── src                             # 可以用来可视化训练过程
│   └── chitchat_demo.png
├── utils                           # 工具类
│   ├── dataset.py                  # 包含
│   ├── logger.py
│   └── preprocess.py               # 预处理数据集，具体是将原始文本tokenize，
├── generate_dialogue_subset.py     # 然后转换成id存储起来，以供后续快速加载
├── interact_mmi.py
├── interact.py                     # 交互式，就是跟系统对话的形式，测试模型
├── test.py                         # 测试，输入测试集，包含对话，输出最后的答复
└── train.py                        # 训练
```

**基本流程**

训练：

1. 先将数据集构造成[上述]({{< relref "基于GPT2对话系统实现#训练集" >}})形式，训练集和测试集分别为`data/train.txt`和`data/test.txt`
2. 运行`./train.py`先进行预处理，它会把原始文本token化后转换为id，保存到`data/train_tokenized.txt`；接下来训练的时候会直接读取这个文件，加快了下一次的过程
3. `./train.py`会加载预训练模型`pytorch_model.bin`，可以自动下载或者手动下载
4. 将数据集构造成`dataloader`，其实就是将数据成批地输入模型，这个批处理大小由`batch_size`控制
5. 进行训练的过程，程序会计算每一个batch的`loss`，然后进行反向传播；记录`loss`的变化情况，周期性的选择的模型进行保存，保存在`output/dialogue_model`，这里是每个`epoch`保存一次

测试:

1. 运行`test.py`，同样也会预处理保存在`data/test_tokenized.txt`
2. `test.py`会加载训练好的模型，具体使用哪个模型可以在`config/args.py`里配置
3. 然后根据概率生成对应结果，保存在`output/sample/output_test.txt`中

交互模式：

1. 运行`interact.py`，会打开一个命令行，输入进一句话，系统会进行回复
2. `ctrl+c`退出

## 实验

### 训练

**输入**

数据集预处理过后，生成的token id文件`data/train_tokenized.txt`如下:

```text
# data/train_tokenized.txt
101 872 1962 8024 5496 ... 7270 102
101 5496 2094 5515 3698 ...  511 102
...
```

101代表`[CLS]`，102代表`[SEP]`，这些都在字典里可以找到`vocab_small.txt`。每一行代表一个对话历史，最后这些`input_ids`会被feed进模型，参与计算

输入`input_ids`的维度为`(batch_size, max_len, vocab_size)`，`max_len`应当包括*对话历史本身的长度*加上*生成的句子长度*和填充，即$len(his) + len(gen) \le max\\_len$，所以超过`max_len`需要对其截断

**输出**

模型的输出就是预测出的`prediction_score`，用`logits`表示；n个token生成n个`hidden prediction_score`。用`logits[0, n-2]`去跟`labels[1, n-1]`做计算`CrossEntropy`，**得到`loss`值**。

再比较生成与原始句子之间预测正确的个数，**得到精度**$accuracy = \frac{correct}{num\\_token}$

> 忽略`<pad>`，不对其计算`crossEntropy`与`accuracy`

维度：

- logits: `(batch_size, max_len, vocab_size)`，模型得出的`prediction_score`
- labels: `(batch_size, max_len, vocab_size)`，也就是原句子

比如：

```python
n = 6
labels = "今天天气真好"     # 原始对话
          ↓↗
logits = "天天生真好啊"     # 预测的结果，除了最后一个字

# labels里第一个字"今"预测出下一个字为logits第一个"天"
# 预测出的"天"跟labels里'今'的下一个字比较

shift_logits = logits[0: n-2] # "天天生真"
shift_labels = labels[1: n-1] # "天天气真"

### loss
loss = CrossEntropy(shift_logits, shift_labels)

### accuracy
correct = 3     # 如上述例子
num_token = len(shift_labels)
accuracy = correct/num_token    # 0.75
```

### 测试

也就是预测的过程

1. 针对每一个对话，输出进模型，得到下一个字`prediction_score`。即`logits[ , -1, ]`，长度为`vocab_size`，随后对其进行[采样]({{< relref "基于GPT2对话系统实现#采样方式" >}})。
2. 将采样的结果添加到对话末尾，重复 1.
3. 直到达到预设的最大长度`max_len`，或者采样是`<sep>`，则结束

### 采样方式

将一个长句子输入进模型后，会得到下一个字的概率，有不同的策略对这个字进行采样(Sampling)。若想要生成的句子最符合训练集情况，就选择概率最大，但这样会生成重复的句子，对于聊天机器人可能不太好；想要生成的句子具有多样性，可选取其他采样方式。

- **Greedy Search**，简单地取最大概率
- **Top-K**，选取前k个概率最大的样本，再随机取一个样本
- **Top-P**，选取概率超过一定阈值的样本，如0.9，再从这些样本中随机取一个
- **Beam Seach**，设置一个N值代表N个样本，对当前生成的选择概率最大的N个样本；在下一轮生成时，计算这N个样本与词表所有词组合后的概率值，再选择概率前N个值，如此往复，最后生成概率最大的N个句子
- **Temperture**，$p_j \propto \frac{\exp(p_j / T)}{\sum_i \exp(p_i/T)}$，通过调节Temperture可对全体概率进行平滑或者锐化
- **Penalty**，每次生成一个词后，我们可以添加一个惩罚项，对这个词的概率进行处理，使其减少重复，如 $p_j = \frac{\exp(p_j / T / Penalty)}{\sum_i \exp(p_i/T/Penalty)}$, where $i, j$ has been sampled

> 这些采样方式通常可以组合，比如先选取Top-P取概率超过一定阈值的样本，再使用Top-K取前K个，再随机选取某个样本

## 医疗对话系统改进

上述的对话系统有一个问题，简单地拼接对话历史，模型学习到了语言特征，但这个学到的语言特征包括A和B的，生成的句子也可能是A或B说的话，可以简单地认为A与B的角色是等同的，称为**闲聊系统**。而Task-Oriented聊天机器人，A和B的角色不一样，是**问跟答**的关系。

这个问题可以通过正确地构造训练集来实现。
