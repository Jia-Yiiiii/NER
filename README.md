# 基于 BERT 的中文命名实体识别

本项目使用 BERT 模型在 **MSRA** 和 **Weibo** 两个中文数据集上进行命名实体识别（NER）实验，并对比了不同模型和标签对齐策略的效果。

---

## 一、数据分析

### 1.1 数据格式

MSRA 用 `0` 分隔句子，Weibo 用空行。代码中通过判断 `line == '' or line == '0'` 同时兼容两种格式。

两个数据集的标签体系不同：

- **MSRA**：标签为 `B-LOC` 形式，共 7 种。
- **Weibo**：标签为 `B-LOC.NAM` 形式，增加了 `.NAM`（具体名称）和 `.NOM`（指代）的区分，共 17 种。

因此，在代码中为两个数据集**分别建立了独立的 `label2id` 映射**，不共用标签体系。

### 1.2 标签分布

**MSRA 训练集标签分布**

| 标签 | 数量 |
|------|------|
| O | 206412 |
| I-ORG | 9141 |
| I-LOC | 5313 |
| B-LOC | 3952 |
| I-PER | 3612 |
| B-ORG | 2158 |
| B-PER | 1850 |

**Weibo 训练集标签分布**

| 标签 | 数量 |
|------|------|
| O | 68777 |
| I-PER.NOM | 1043 |
| I-PER.NAM | 1041 |
| B-PER.NOM | 766 |
| B-PER.NAM | 574 |
| I-ORG.NAM | 477 |
| I-GPE.NAM | 241 |
| B-GPE.NAM | 205 |
| B-ORG.NAM | 183 |
| I-LOC.NAM | 129 |
| I-LOC.NOM | 66 |
| I-ORG.NOM | 61 |
| B-LOC.NAM | 56 |
| B-LOC.NOM | 51 |
| B-ORG.NOM | 42 |
| B-GPE.NOM | 8 |
| I-GPE.NOM | 8 |

### 1.3 数据处理

<img width="832" height="327" alt="image" src="https://github.com/user-attachments/assets/8facc656-a167-4d23-aa6a-984fbd225345" />
<img width="617" height="330" alt="image" src="https://github.com/user-attachments/assets/c9db921c-50a5-4b22-b3d2-8df0a501a530" />

1. 读取时兼容 MSRA（`0` 分隔）和 Weibo（空行分隔）两种格式。
2. 自动从数据中构建标签映射。MSRA 和 Weibo 标签体系不同，代码分别生成各自的 label2id，不共用。
3. BERT 分词会产生子词，导致标签数量对不上。使用 `word_ids` 对齐，只保留每个词首个子词的原始标签，后续子词标 `O` 忽略。同时也支持标签传播策略（`other`），用于对比实验。

---

## 二、实验结果

### 整体结果对比

| 模型 | 数据集 | 对齐策略 | 最佳验证 F1 | 测试集 F1 |
|------|--------|----------|-------------|-----------|
| bert-base-chinese | MSRA | ignore | 0.9263| 0.9192 |
| chinese-bert-wwm | MSRA | ignore | 0.9403 | 0.9095 |
| bert-base-chinese | Weibo | ignore | 0.7117 | 0.6682 |
| chinese-bert-wwm | Weibo | ignore | 0.7075 | 0.6460 |
| chinese-bert-wwm | Weibo | other | 0.7075 | 0.6460 |

### 2.1 MSRA + bert-base-chinese

测试集详细结果：

| 实体类型 | Precision | Recall | F1-score | Support |
|----------|-----------|--------|----------|---------|
| LOC      | 94.52%    | 93.93% | 94.23%   | 643     |
| ORG      | 80.54%    | 92.26% | 86.00%   | 323     |
| PER      | 96.14%    | 97.39% | 96.76%   | 307     |
| micro avg | 90.98%   | 94.34% | 92.63%   | 1273    |
| macro avg | 90.40%   | 94.53% | 92.33%   | 1273    |
| weighted avg | 91.37% | 94.34% | 92.75% | 1273    |

**训练曲线**

<img width="467" height="392" alt="image" src="https://github.com/user-attachments/assets/13d9e893-6456-4800-9753-94b4d6a82027" />
<img width="972" height="398" alt="image" src="https://github.com/user-attachments/assets/60285f6a-9f89-4c78-b8f9-18adc9c0685d" />
<img width="653" height="441" alt="image" src="https://github.com/user-attachments/assets/04c449bf-3ba4-4228-b5dd-c077e5712272" />


---

### 2.2 MSRA + chinese-bert-wwm

测试集详细结果：



| 实体类型 | Precision | Recall | F1-score | Support |
|----------|-----------|--------|----------|---------|
| LOC（地点） | 95.72% | 93.93% | 94.82% | 643 |
| ORG（组织） | 87.24% | 91.02% | 89.09% | 323 |
| PER（人名） | 97.41% | 98.05% | 97.73% | 307 |
| **micro avg** | **93.89%** | **94.19%** | **94.04%** | **1273** |
| **macro avg** | **93.46%** | **94.33%** | **93.88%** | **1273** |
| **weighted avg** | **93.98%** | **94.19%** | **94.07%** | **1273** |

**训练曲线**

<img width="468" height="386" alt="image" src="https://github.com/user-attachments/assets/701595da-3848-40a5-9c30-1727fbdf2616" />
<img width="971" height="407" alt="image" src="https://github.com/user-attachments/assets/9b01558d-ff2b-4d9a-a4f8-711f84cacd34" />
<img width="560" height="458" alt="image" src="https://github.com/user-attachments/assets/841072a6-d4a4-4657-88fc-532e31f28cbd" />



---

### 2.3 Weibo + chinese-bert-wwm (align_type='ignore')

测试集详细结果：

| 实体类型 | Precision | Recall | F1-score | Support |
|----------|-----------|--------|----------|---------|
| GPE.NAM  | 68.75%    | 84.62% | 75.86%   | 26      |
| GPE.NOM  | 100.00%   | 100.00%| 100.00%  | 1       |
| LOC.NAM  | 50.00%    | 83.33% | 62.50%   | 6       |
| LOC.NOM  | 40.00%    | 33.33% | 36.36%   | 6       |
| ORG.NAM  | 37.50%    | 44.68% | 40.78%   | 47      |
| ORG.NOM  | 44.44%    | 80.00% | 57.14%   | 5       |
| PER.NAM  | 73.86%    | 73.03% | 73.45%   | 89      |
| PER.NOM  | 77.25%    | 78.37% | 77.80%   | 208     |
| **micro avg** | **68.69%** | **72.94%** | **70.75%** | **388** |
| **macro avg** | **61.48%** | **72.17%** | **65.49%** | **388** |
| **weighted avg** | **69.73%** | **72.94%** | **71.10%** | **388** |

**训练曲线**

<img width="473" height="388" alt="image" src="https://github.com/user-attachments/assets/01121a52-c594-4578-9fff-548ddab8941e" />
<img width="976" height="383" alt="image" src="https://github.com/user-attachments/assets/66f6f57b-f36a-4dd8-ac5f-1b9347bd6854" />
<img width="530" height="412" alt="image" src="https://github.com/user-attachments/assets/bb08ffa9-9cfd-47ca-9d4e-37aa4ce71812" />



---

### 2.4 Weibo + bert-base-chinese (align_type='ignore')

测试集详细结果：

| 实体类型 | Precision | Recall | F1-score | Support |
|----------|-----------|--------|----------|---------|
| GPE.NAM  | 67.65%    | 88.46% | 76.67%   | 26      |
| GPE.NOM  | 100.00%   | 100.00%| 100.00%  | 1       |
| LOC.NAM  | 55.56%    | 83.33% | 66.67%   | 6       |
| LOC.NOM  | 35.71%    | 83.33% | 50.00%   | 6       |
| ORG.NAM  | 43.14%    | 46.81% | 44.90%   | 47      |
| ORG.NOM  | 44.44%    | 80.00% | 57.14%   | 5       |
| PER.NAM  | 71.88%    | 77.53% | 74.59%   | 89      |
| PER.NOM  | 73.13%    | 79.81% | 76.32%   | 208     |
| **micro avg** | **66.89%** | **76.03%** | **71.17%** | **388** |
| **macro avg** | **61.44%** | **79.91%** | **68.29%** | **388** |
| **weighted avg** | **67.69%** | **76.03%** | **71.40%** | **388** |

**训练曲线**
<img width="480" height="381" alt="image" src="https://github.com/user-attachments/assets/420b2081-73d8-401c-8270-11a60ebbe768" />
<img width="978" height="402" alt="image" src="https://github.com/user-attachments/assets/51c0c03a-4f05-472d-ac79-7f8773b03fd7" />
<img width="540" height="402" alt="image" src="https://github.com/user-attachments/assets/71e756cb-7c67-4206-8209-9d2d1a0902f3" />

### 2.5 Weibo + chinese-bert-wwm (align_type='other')

测试集详细结果：
| 实体类型 | Precision | Recall | F1-score | Support |
|----------|-----------|--------|----------|---------|
| GPE.NAM  | 68.75%    | 84.62% | 75.86%   | 26      |
| GPE.NOM  | 100.00%   | 100.00%| 100.00%  | 1       |
| LOC.NAM  | 50.00%    | 83.33% | 62.50%   | 6       |
| LOC.NOM  | 40.00%    | 33.33% | 36.36%   | 6       |
| ORG.NAM  | 37.50%    | 44.68% | 40.78%   | 47      |
| ORG.NOM  | 44.44%    | 80.00% | 57.14%   | 5       |
| PER.NAM  | 73.86%    | 73.03% | 73.45%   | 89      |
| PER.NOM  | 77.25%    | 78.37% | 77.80%   | 208     |
| micro avg | 68.69%   | 72.94% | 70.75%   | 388     |
| macro avg | 61.48%   | 72.17% | 65.49%   | 388     |
| weighted avg | 69.73% | 72.94% | 71.10% | 388     |

**训练曲线**
<img width="562" height="402" alt="image" src="https://github.com/user-attachments/assets/3c4d86af-5c8f-46b8-8808-957abcf576f2" />
<img width="1110" height="402" alt="image" src="https://github.com/user-attachments/assets/93fa3c2e-d77d-4e2d-86da-f6fb915f5892" />
<img width="555" height="385" alt="image" src="https://github.com/user-attachments/assets/584ff5cd-8a7c-43d2-aaea-5a5ac10713be" />



---

## 三、总结

从结果看，在规范的MSRA数据集上，bert-base-chinese和chinese-bert-wwm的表现都很好，测试F1都在0.91以上。但在Weibo数据上，性能下降很明显，F1值普遍在0.66-0.69之间，说明社交媒体文本的NER任务难度大很多。

对比模型，chinese-bert-wwm在Weibo上的表现比bert-base-chinese好一些，说明全词掩码在非规范文本上可能有点帮助。ignore和other两个方式结果完全一样，可能在这个场景下影响不大。

## 项目结构

```text
BERT-NER-DEMO2/
├── DATA/
│ ├── MSRA/
│ │ ├── train.txt
│ │ ├── dev.txt
│ │ └── test.txt
│ └── weibo/
│ ├── train.txt
│ ├── dev.txt
│ └── test.txt
├── configs/
│ └── Bert_Config_exp1.json
├── data.py
├── model.py
├── trainer.py
├── utils.py
├── requirements.txt
|——label2id.json
└── README.md
