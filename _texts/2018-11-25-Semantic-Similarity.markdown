---
layout: post
title:  "Using deep learning to identify semantically similar questions"
date:   2018-11-25 08:24:00 -0700
categories: machine-learning ml lstm
use_math: true
--- 

#### Introduction 

A common problem I'm faced with in my day-to-day life is that of knowledge retrieval. A specific question arises, and the answer resides somwhere 
in a knowledge repository. If this repository happens to be indexed by Google, the solution is simple: type a query into the google search bar.
However, there are many repositories that are not indexed by google: My personal email, various knowledge bases I use daily (e.g. Confluence), 
Stack Overflow, etc. Typically, the search functionality provided by these tools leave much to be desired.

Furthermore, websites like Stack Overflow often suffer from a large number of duplicate questions, along with a large of number of duplicate answers -
and often links between them. Assuming innocent motivations, it seems probable that people don't post duplicate questions because they like annoying 
the SO administrators, but simply cannot find the information they are looking for - even though it exists.

One potential solution to this problem is to provided or designate a set of canonical questions and answers and simply map other queries into the 
pre-existing set of questions and answers. Clearly there are many potential user experience issues you might encounter with such an approach. Leaving 
those aside, this seemed like an interesting problem to attempt to solve with Machine Learning.

#### Benefits of this problem

There are a few reasons why this is a nice problem to solve with Machine Learning:
- It's a well researched problem, and has both classical ML solutions and Neural solutions.
- State of the art approaches yield [good results](https://nlpprogress.com/english/semantic_textual_similarity.html).
- A variety of datasets are available, ranging from [small](http://clic.cimec.unitn.it/composes/sick.html) to [large](https://www.kaggle.com/c/quora-question-pairs/data).
- Simple approaches like the [Manhattan LSTM](https://www.aaai.org/ocs/index.php/AAAI/AAAI16/paper/download/12195/12023) are quite accessible even for an ML beginner.
- A ML based approach can augment a search engine like ElasticSearch for even better results.

#### A first attempt

As I pointed out above, the Manhattan LSTM model is a simple way to get started. The [paper](https://www.aaai.org/ocs/index.php/AAAI/AAAI16/paper/download/12195/12023) explains 
the relatively simple approach:
- The model is a "Siamese LSMT." There are two LSTMs, termed LSTMa and LSTMb, with the weights being the same between them.
- The cost function is the negative exponent of the l1 norm of the LSTM outputs.

Let's test some inline math $x$, $y$, $x_1$, $y_1$.