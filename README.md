# Kaggle比赛总结_LLM - Detect AI Generated Text

# 1. 数据集

## 比赛官方数据集

https://www.kaggle.com/competitions/llm-detect-ai-generated-text/data

| id | prompt_id | text | generated |
| --- | --- | --- | --- |
|  | 0 |  | 0 |
|  | 1 |  | 1 |
- **标签**：student (`0`) or generated by an LLM (`1`)
- **数据量**：1378，整个数据集全为student (`0`)
- **prompt_id：**根据比赛discussion：[https://www.kaggle.com/competitions/llm-detect-ai-generated-text/discussion/453410](https://www.kaggle.com/competitions/llm-detect-ai-generated-text/discussion/453410)，7个prompts如下：

| prompt_id | prompt |
| --- | --- |
| 0 | Car-free cities |
| 1 | Does the electoral college work? |
| 2 | Exploring Venus |
| 3 | The Face on Mars |
| 4 | Facial action coding system |
| 5 | A Cowboy Who Rode the Waves |
| 6 | Driverless cars |

## 补充训练数据

### DAIGT V2

[https://www.kaggle.com/datasets/thedrcat/daigt-v2-train-dataset](https://www.kaggle.com/datasets/thedrcat/daigt-v2-train-dataset)

| text | label | prompt_name | source | RDizzl3_seven |
| --- | --- | --- | --- | --- |
- **数据量：**44868，27371 student (`0`) 【正例少于负例】
- **prompt_name：**包含12% “Distance learning”，12% “Seeking multiple opinions”
- RDizzl3_seven：表示是否属于7个prompts - 20450 True - 6200 (`1`)

### DAIGT V3

[https://www.kaggle.com/datasets/thedrcat/daigt-v3-train-dataset](https://www.kaggle.com/datasets/thedrcat/daigt-v3-train-dataset)

| text | label | prompt_name | source | RDizzl3_seven | model |
| --- | --- | --- | --- | --- | --- |
- **数据量：65267**，27371 student (`0`) 【正例多于负例】
- **prompt_name：**包含12% “Distance learning”，12% “Seeking multiple opinions”，包含18554条7个prompts生成的内容

### DAIGT V4

[https://www.kaggle.com/datasets/thedrcat/daigt-v4-train-dataset](https://www.kaggle.com/datasets/thedrcat/daigt-v4-train-dataset)

- 基本上与**V3**已知 + 8000+ texts with llama-based models finetuned on Persuade corpus

### 辅助数据集：

### mistral

[https://www.kaggle.com/datasets/kkkkkkc/daigt-v2-mistral/data](https://www.kaggle.com/datasets/kkkkkkc/daigt-v2-mistral/data)

### **AI-GA: AI-Generated Abstracts dataset**

https://github.com/panagiotisanagnostou/AI-GA

| title | abstract | label |
| --- | --- | --- |
- 标签：original abstract (labeled as `0`) or AI-generated abstract (labeled as `1`)
- 生成模型：GPT-3

# 2. Baseline选取

## 传统机器学习方法[0.961]：

- [https://www.kaggle.com/code/hubert101/0-960-phrases-are-keys](https://www.kaggle.com/code/hubert101/0-960-phrases-are-keys)
- 数据集：DAIGT V2
- 使用BPE + TF-IDF 做tokenization
- 分类器 - 集成
    - `clf = MultinomialNB(alpha=0.02)`
    - `sgd_model = SGDClassifier(max_iter=8000, tol=1e-4, loss="modified_huber")`
    - `lgb=LGBMClassifier(**p6)`
    - `cat=CatBoostClassifier`

## 预训练大模型Deberta方法：

- [Train]: [https://www.kaggle.com/code/alejopaullier/daigt-deberta-text-classification-train](https://www.kaggle.com/code/alejopaullier/daigt-deberta-text-classification-train)
- [Inference]: [https://www.kaggle.com/code/alejopaullier/daigt-deberta-text-classification-inference](https://www.kaggle.com/code/alejopaullier/daigt-deberta-text-classification-inference)
- 数据集：包含 2421 篇学生生成的论文以及 2421 篇 AI 生成的论文，是原始数据的 4 倍多

# 3. 基于Baseline的方案+Final Submissions

## 数据集

| dataset | Baseline model | partial test score | final submission |
| --- | --- | --- | --- |
| DAIGT V2 | traditional | 0.961 |  |
| DAIGT V2 | deberta | 0.749 |  |
| DAIGT V3 | deberta | 0.737 |  |
| AI-GA | traditional | 0.398 |  |
| DAIGT V2+AI-GA | traditional | 0.937 |  |
| DAIGT V2+mistrial | traditional | 0.961 | ✅ |
| DAIGT V2+gemini+mistrial+len_150+sample_50k | traditional | 0.951 |  |
| DAIGT V2+mistrial all 7 [unclean] | traditional | 0.958 |  |
| DAIGT V2+gemini all data [unclean] | traditional | 0.957 |  |
| DAIGT V3 + persuade | traditional | 0.945 |  |
| DAIGT V3 sample data | traditional | 0.95 |  |
| DAIGT V2 (RDizzl3_seven=True) | traditional | 0.954 |  |
| DAIGT V2 | traditional (MNB+LR+SGD) | 0.942 |  |
| DAIGT V3 | traditional | OOM |  |
| DAIGT V4 | traditional | OOM |  |
| DAIGT V3 (RDizzl3_seven=True) | traditional | 0.947 | ✅ |
| DAIGT V4 (RDizzl3_seven=True) | traditional | 0.92 |  |
| DAIGT V4 sample data | traditional | 0.93 |  |

## 特征工程 [all on DAIGT V2, using the traditional model]

| feature | partial test score | final submission |
| --- | --- | --- |
| counting stop words | 0.895 |  |
| typo count  | 0.927 | ✅ |
| text length | 0.894 |  |
| typo count+text length + stop words | 0.9 |  |
| data cleaning | 0.779 |  |
| data cleaning + typo count + text length + stop words | 0.77 |  |

### Prompt-Text Similarity计算[./prompt_text_sim/]

假设：generated texts与prompt的similarity

数据集：DAIGT V2

方法：使用BERT进行tokenization，然后对比prompt与text的cosine similarity

- 基于当前计算方法，prompt_text_similarity与label相关性不大

![Untitled](Kaggle%E6%AF%94%E8%B5%9B%E6%80%BB%E7%BB%93_LLM%20-%20Detect%20AI%20Generated%20Text%20a007e504bf354d59abeb1bf15c2bdd6d/Untitled.png)

- text和prompt长度差异很大，没有对text进行主题模型的提取