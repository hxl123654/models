## 简介

对话情绪识别（Emotion Detection，简称EmoTect），专注于识别智能对话场景中用户的情绪，针对智能对话场景中的用户文本，自动判断该文本的情绪类别并给出相应的置信度，情绪类型分为积极、消极、中性。

对话情绪识别适用于聊天、客服等多个场景，能够帮助企业更好地把握对话质量、改善产品的用户交互体验，也能分析客服服务质量、降低人工质检成本。可通过[AI开放平台-对话情绪识别](http://ai.baidu.com/tech/nlp_apply/emotion_detection) 线上体验。

效果上，我们基于百度自建测试集（包含闲聊、客服）和nlpcc2014微博情绪数据集，进行评测，效果如下表所示：
 
| 模型 | 闲聊 | 客服 | 微博 |
| :------| :------ | :------ | :------ |
| BOW | 90.2% | 87.6% | 74.2% |
| LSTM | 91.4% | 90.1% | 73.8% |
| Bi-LSTM | 91.2%  | 89.9%  | 73.6% |
| CNN | 90.8% |  90.7% | 76.3%  |
| TextCNN |  91.1% | 91.0% | 76.8% |
| BERT | 93.6% | 92.3%  | 78.6%  |
| ERNIE | 94.4% | 94.0% | 80.6% |


## 快速开始

本项目依赖于 Paddlepaddle Fluid 1.3.2，请参考 [安装指南](http://www.paddlepaddle.org/#quick-start) 进行安装

#### 数据准备

下载经过预处理的数据，运行该脚本之后，会生成data目录，data目录下有训练集数据（train.tsv）、开发集数据（dev.tsv）、测试集数据（test.tsv）、 待预测数据（infer.tsv）以及对应词典（vocab.txt）
```shell
sh download_data.sh
```

#### 模型下载

我们开源了基于海量数据训练好的对话情绪识别模型（基于TextCNN模型训练），可供用户直接使用，运行脚本后，会生成models目录，models目录下会有预训练的模型文件
```shell
sh download_model.sh
```
#### 模型评估

基于已有的预训练模型和数据，可以运行下面的命令进行测试，查看预训练的模型在测试集（test.tsv）上的评测结果
```shell
sh run.sh eval
```

#### 模型训练

基于示例的数据集，可以运行下面的命令，在训练集（train.tsv）上进行模型训练，并在开发集（dev.tsv）验证
```shell
sh run.sh train
```
训练完成后，可修改```run.sh```中init_checkpoint参数，进行模型评估和预测

#### 模型预测

基于预训练模型，可在新的数据集（infer.tsv）上进行预测，得到模型预测结果及概率
```shell
sh run.sh infer
```

## 进阶使用

#### 任务定义

对话情绪识别任务输入是一段用户文本，输出是检测到的情绪类别，包括消极、积极、中性，这是一个经典的短文本三分类任务。

#### 模型原理介绍

本项目针对对话情绪识别问题，开源了一系列分类模型，供用户可配置地使用：

+ BOW：Bag Of Words，是一个非序列模型，使用基本的全连接结构；
+ CNN：浅层CNN模型，能够处理变长的序列输入，提取一个局部区域之内的特征；；
+ TextCNN：多卷积核CNN模型，能够更好地捕捉句子局部相关性；
+ LSTM：单层LSTM模型，能够较好地解决序列文本中长距离依赖的问题；
+ BI-LSTM：双向单层LSTM模型，采用双向LSTM结构，更好地捕获句子中的语义特征；
+ ERNIE：百度自研基于海量数据和先验知识训练的通用文本语义表示模型，并基于此在对话情绪分类数据集上进行fine-tune获得。

#### 数据格式说明

训练、预测、评估使用的数据示例如下，数据由两列组成，以制表符（'\t'）分隔，第一列是情绪分类的类别（0表示消极；1表示中性；2表示积极），第二列是以空格分词的中文文本，文件为utf8编码。

```text
0   谁 骂人 了 ？ 我 从来 不 骂人 ， 我 骂 的 都 不是 人 ， 你 是 人 吗 ？
1   我 有事 等会儿 就 回来 和 你 聊
2   我 见到 你 很高兴 谢谢 你 帮 我
```
注：本项目额外提供了分词预处理脚本（在preprocess目录下），可供用户使用，具体使用方法如下：
```shell
python tokenizer.py --test_data_dir ./test.txt.utf8 --batch_size 1 > test.txt.utf8.seg
```

#### 代码结构说明

```text
.
├── config.json             # 模型配置文件
├── config.py               # 定义了该项目模型的相关配置，包括具体模型类别、以及模型的超参数
├── reader.py               # 定义了读入数据，加载词典的功能
├── run_classifier.py       # 该项目的主函数，封装包括训练、预测、评估的部分
├── run_ernie_classifier.py # 基于ERNIE表示的项目的主函数
├── run_ernie.sh            # 基于ERNIE的训练、预测、评估运行脚本
├── run.sh                  # 训练、预测、评估运行脚本
├── utils.py                # 定义了其他常用的功能函数
```

#### 如何组建自己的模型

可以根据自己的需求，组建自定义的模型，具体方法如下所示：

1. 定义自己的网络结构 
用户可以在 ```models/classify/nets.py``` 中，定义自己的模型，只需要增加新的函数即可。假设用户自定义的函数名为```user_net```
2. 更改模型配置
在 ```config.json``` 中需要将 ```model_type``` 改为用户自定义的 ```user_net```
3. 模型训练，运行训练、评估、预测脚本即可（具体方法同上）

#### 使用ERNIE进行finetune

1. 下载 ERNIE 预训练模型
```
mkdir -p models/ernie
cd models/ernie
wget --no-check-certificate https://baidu-nlp.bj.bcebos.com/ERNIE_stable-1.0.1.tar.gz
tar xvf ERNIE_stable-1.0.1.tar.gz
rm ERNIE_stable-1.0.1.tar.gz
```
2. 配置 ERNIE 模型及数据
通过 ```run_ernie.sh``` 配置ERNIE模型路径及数据路径，例如
```
MODEL_PATH=./models/ernie
TASK_DATA_PATH=./data
```
3. 模型训练
```
sh run_ernie.sh train
```
训练、评估、预测详细配置，请查看 ```run_ernie.sh```

## 如何贡献代码

如果你可以修复某个issue或者增加一个新功能，欢迎给我们提交PR。如果对应的PR被接受了，我们将根据贡献的质量和难度进行打分（0-5分，越高越好）。如果你累计获得了10分，可以联系我们获得面试机会或者为你写推荐信。
