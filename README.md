# Passage Retrieval

## Introduction
Passage Retrieval is a crucial part of modern open-domain question-answering systems that rely on precise and efficient retrieval components to find passages containing correct answers.

Traditionally, lexical methods like TF-IDF or BM25 were commonly used to power the retrieval systems. They are fast, interpretable, and don’t require any training (and therefore a training set). However, they can only return a document if it contains a keyword present in a query. Moreover, their text understanding is limited because they ignore the word order.

Recently, neural retrieval systems (e.g. Dense Passage Retrieval) surpassed those traditional methods by fine-tuning the pre-trained language models on a large number of (query, document) pairs. They solve the aforementioned problems of lexical methods, but at the cost of the necessity to label training sets and poor generalizability to other domains. As a result, in a zero-shot setup (i.e. no training set) the lexical methods are still competitive or even better than neural models.


## Task Definition
The goal of this task is to develop a system for cross-domain question-answering retrieval.

The participant will be given:

1. Training set consisting of (question, passage) pairs from trivia domain, i.e. general-knowledge questions typical for popular TV quiz shows, such as [Fifteen to One](https://en.wikipedia.org/wiki/Fifteen_to_One) (PL: [Jeden z dziesięciu](https://pl.wikipedia.org/wiki/Jeden_z_dziesi%C4%99ciu)).
2. Three separate test sets with unpaired questions and passages from different domains: trivia, legal, and customer support. Note that for legal and customer support we won’t provide training sets.

For each test question, the system is supposed to retrieve an ordered list of the ten most relevant passages (i.e. containing the answer) from the provided corpus. The system will be scored based on its performance on all three test sets.

The participants are free to use any publicly available datasets to develop their systems with the exception of the test set B from [the PolEval 2021 QA competition](http://2021.poleval.pl/tasks/task4). It is also forbidden to manually label the test examples.


## Dataset

### Training set
The training set consists of 5000 trivia questions, i.e. general-knowledge questions typical for popular TV quiz shows, such as [Fifteen to One](https://en.wikipedia.org/wiki/Fifteen_to_One) (PL: [Jeden z dziesięciu](https://pl.wikipedia.org/wiki/Jeden_z_dziesi%C4%99ciu)). For each question, we manually found up to five passages from Polish Wikipedia that contain the answer to the question. Each passage is accompanied by the title of the article in which it was found. Overall, the training set consists of 16389 question-passage pairs.

Additionally, we release a Wikipedia corpus of 7097322 passages. The raw Wikipedia dump was parsed using [WikiExtractor](https://github.com/attardi/wikiextractor) and split into passages at the ends of the paragraphs or if the passage was longer than 500 characters.


### Test sets
There are three test sets with questions from different domains. The first dataset consists of 1291 trivia questions that are similar to those from the training set.

The second dataset consists of 900 questions and 921 passages regarding the large e-commerce platform – Allegro.pl. The dataset was created based on help articles and lists of frequently asked questions available on the Allegro website. Each question-passage pair was manually verified and edited if necessary. In contrast to other test sets, each provided passage is relevant to some question, i.e. there are no passages that don’t match any test question.

The third dataset contains over 700 questions from the legal domain. The dataset is a bit artificial, since part of the questions were created by randomly selecting the provisions and asking questions based on the content of these provisions. The dataset is thus similar to SQuAD, yet the task – like in the other datasets – only requires to identify the relevant passages, rather than answering the question. The questions will be supplemented by approx. 26 thousand provisions extracted from more than one thousand laws published between 1993 and 2004. An example question from the legal domain is: “Czy Najwyższa Izba Kontroli przeprowadza kontrolę pod względem dochodowości?”


### Dataset format
The datasets are stored in four directories, one for each dataset (training and three testing sets). Each dataset has the same format.


#### Questions
The questions are stored in “questions.jl” which is a JSON lines file with two fields: “id” containing the question identifier and “text” containing the question itself.

Example:

```
{"id": "123", "text": "W którym państwie leży miasto Bangkok?"}
```



#### Passages
The corpus of all available passages is stored in “passages.jl”, which is also a JSON lines file with the same two fields: “id” and “text”. For trivia questions and the legal questions, there is also an additional field “title”. In the first case, it contains the title of the Wikipedia article in which the passage was found and in the second case it contains the title of the law, the provisions come from. The optional “meta” field contains additional information about the passage.

Example:

```
{"id": "2-0", "title": "Miss Grand International", "text": "Główna uroczystość odbywała się w Tajlandii do roku 2015, kiedy to odbyła się w mieście Bangkok.", "meta": {"article_id": 2, "passage_id": 0}}
```

#### Pairs
Additionally, the training set contains the pairing between questions and passages, stored in “pairs.tsv”. It is a tab-separated file with three columns:

1. “question-id” which matches the question identifier,
2. “passage-id” which matches the passage identifier,
3. “score” which indicates how relevant is the passage to the question. It is always equal to 1 as we only include the positive pairs in the training set.

Example:

```
question-id    passage-id    score
123            2-0           1
```

The “pairs.tsv” file contains only positive question-passage pairs, i.e. the passages containing the answer to the question. It doesn’t contain negative pairs, and it is not guaranteed that it contains all possible positive pairs.


### Downloading datasets
All datasets can be found [here](https://huggingface.co/datasets/piotr-rybak/poleval-passage-retrieval/tree/main).


## Evaluation
The submitted system will be evaluated using Normalized Discounted Cumulative Gain for the top 10 most relevant passages (NDCG@10).

### Submission format
The goal of the task is to find an ordered list of the ten most relevant passages for each question. The submission should consist of a single tab-separated file. Each of the ten columns should contain one `passage-id` from `passages.jl` file with the left-most column being the most relevant passage. Each line should contain passages relevant to the matching question from the [`in.tsv`](https://github.com/poleval/2022-passage-retrieval/blob/main/test-A/in.tsv) file.

Example:

```
235415-0  17326-2	2490065-0	69774-0	17332-1 37931-0	1152562-0	3861237-0	2294984-0	407279-0
```

## Baseline
We provide a simple baseline system based on the BM25 algorithm [here](https://github.com/360er0/poleval-passage-retrieval).

## References
1. Nandan Thakur, Nils Reimers, Andreas Rücklé, Abhishek Srivastava, Iryna Gurevych. 2021. [BEIR: A Heterogeneous Benchmark for Zero-shot Evaluation of Information Retrieval Models](https://openreview.net/forum?id=wCu6T5xFjeJ)
2. Vladimir Karpukhin, Barlas Oguz, Sewon Min, Patrick Lewis, Ledell Wu, Sergey Edunov, Danqi Chen, and Wen-tau Yih. 2020. [Dense Passage Retrieval for Open-Domain Question Answering](https://aclanthology.org/2020.emnlp-main.550/)
3. Lee, Kenton, Ming-Wei Chang, and Kristina Toutanova. "Latent retrieval for weakly supervised open domain question answering." arXiv:1906.00300 (2019).
4. Kelvin Guu, Kenton Lee, Zora Tung, Panupong Pasupat, and Ming-Wei Chang. 2020. REALM: Retrieval-augmented language model pre-training. ArXiv, abs/2002.08909.

## Challenge metadata

Tags: poleval-2022, information-retrieval
