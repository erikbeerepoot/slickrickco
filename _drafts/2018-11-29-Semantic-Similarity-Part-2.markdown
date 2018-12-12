---
layout: post
title:  "Using deep learning to identify semantically similar questions (Part 2)"
date:   2018-11-25 08:24:00 -0700
categories: machine-learning ml lstm
use_math: true
--- 

### Introduction 

In the [last](http://slickrick.co/texts/2018-11-25-Semantic-Similarity/) post, we looked at a very basic Manhattan LSTM to rank the similarity of two sentences. 
We saw that not only the accuracy left something to be desired, the model did not generalize well beyond the dataset it was trained on. In this post, we're going to:
- Speed up the training by reducing the size of the vocabulary.
- Explore some improvements to the model, such as pre-training.
- Explore alternative optimization algorithms.
- Tune the model's hyperparameters to see if we can get better results.


#### Speeding up training

As you may have noticed in the last post, we used the entire set of GloVe word embeddings as the embedding matrix. This is not particularly efficient in terms of the 
memory use and emperically seems to lead to significantly reduced training performance. Instead, we can build a reduced embedding matrix:

{% highlight python linenos %}
import numpy as np

def build_embedding_matrix(vocab, embedding_dim):    
    embedding_matrix = np.zeros((len(vocab) , embedding_dim))
    for index in range(1, len(vocab)):
        try:
            word = vocab[index]
            vector = embeddings.loc[word]
            embedding_matrix[index] = vector
        except:
            continue
    return embedding_matrix

embedding_matrix = build_embedding_matrix(vocab, embedding_dim)
{% endhighlight %}

As you can see, we pre-allocate a numpy array of the appropriate size and we iterate over the words in the vocab to fill it with GloVe embeddings for each word we find.
If the current word is not in the GloVe embeddings, we skip it. This means our embedding matrix will have some 0's for the unknown words we will undoubtedly have in our 
dataset.

 