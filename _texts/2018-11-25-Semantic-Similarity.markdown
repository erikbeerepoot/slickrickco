---
layout: post
title:  "Using deep learning to identify semantically similar questions"
date:   2018-11-25 08:24:00 -0700
categories: machine-learning ml lstm
use_math: true
--- 

### Introduction 

A common problem I'm faced with in my day-to-day life is that of knowledge retrieval. A specific question arises, and the answer resides somwhere 
in a knowledge repository. If this repository happens to be indexed by Google, the solution is simple: type a query into the google search bar.
However, there are many repositories that are not indexed by google: My personal email, various knowledge bases I use daily (e.g. Confluence, 
Stack Overflow). Typically, the search functionality provided by these tools leave much to be desired.

Furthermore, websites like Stack Overflow often suffer from a large number of duplicate questions, along with a large of number of duplicate answers -
and often links between them. Assuming innocent motivations, it seems probable that people don't post duplicate questions because they like annoying 
the SO administrators, but simply cannot find the information they are looking for - even though it exists.

One potential solution to this problem is to provided or designate a set of canonical questions and answers and simply map other queries into the 
pre-existing set of questions and answers. Clearly there are many potential user experience issues you might encounter with such an approach. Leaving 
those aside, this seemed like an interesting problem to attempt to solve with Machine Learning.

### Problem analysis

There are a few reasons why this is a nice problem to solve with Machine Learning:
- It's a well researched problem, and has both classical ML solutions and Neural solutions.
- State of the art approaches yield [good results](https://nlpprogress.com/english/semantic_textual_similarity.html).
- A variety of datasets are available, ranging from [small](http://clic.cimec.unitn.it/composes/sick.html) to [large](https://www.kaggle.com/c/quora-question-pairs/data).
- Simple approaches like the [Manhattan LSTM](https://www.aaai.org/ocs/index.php/AAAI/AAAI16/paper/download/12195/12023) are quite accessible even for an ML beginner.
- A ML based approach can augment a search engine like ElasticSearch for even better results.

### Model & Methodology

As I pointed out above, the Manhattan LSTM model is a simple way to get started. The [paper](https://www.aaai.org/ocs/index.php/AAAI/AAAI16/paper/download/12195/12023) explains 
the relatively simple approach:
- The model is a "Siamese LSMT." There are two LSTMs, termed $LSTM_a$ and $LSTM_b$, with the weights being the same between them.
- The cost function is the negative exponent of the $l_1$ norm of the LSTM outputs:

 $$g(h^{(a)}_{T_a},h^{(b)}_{T_b}) = e^{(-\lVert{h^{(a)}_{T_a} - h^{(b)}_{T_b}}\rVert_{1})}$$)

 - The authors use pre-trained word embeddings (word2vec) as input.
 - The model weights are initialized to random gaussian values.
 - Finally, the model is pre-trained on the SICK dataset and trained on the Quora question answering dataset.
 - The authors use the AdaDelta optimizer to train the model & employ early stopping.


For our approach, we start with a more basic and slightly different approach. First, we use Keras and [GloVe word embeddings](https://nlp.stanford.edu/projects/glove/). GloVe is similar
to word2vec, but with better performance. Keras is a high-level API built on top of TensorFlow to allow rapid development. As shown below, creating an LSTM model that runs on the GPU is
straightfoward:

Setting up the model: 
 {% highlight python %}

from keras.layers import Input, Dense, Embedding, CuDNNLSTM, Lambda
from keras.models import Model
import keras.backend as K

def exponent_neg_manhattan_distance(left, right):
    return K.exp(-K.sum(K.abs(left-right), axis=1, keepdims=True))

n_hidden_units = 32;

left_input = Input(shape=(max_question_length,), dtype='int32')
right_input = Input(shape=(max_question_length,), dtype='int32')

embedding_layer = Embedding(
    len(embeddings), 
    embedding_dim, 
    input_length=max_question_length,
    weights = [embeddings],
    trainable = False,
)
embedded_left = embedding_layer(left_input)
embedded_right = embedding_layer(right_input)

lstm = CuDNNLSTM(n_hidden_units)

left_out = lstm(embedded_left)
right_out = lstm(embedded_right)

malstm_distance = Lambda(function=lambda x: exponent_neg_manhattan_distance(x[0], x[1]),output_shape=lambda x: (x[0][0], 1))([left_out, right_out])

model = Model(inputs=[left_input, right_input], outputs=[malstm_distance])

{% endhighlight %}

One thing to note is that the Keras optimizers are not capable of sparse gradient upddates, so we fall back to a TensorFlow optimizer. This is important because
backprop on dense matrices incurs a large performance penality. We also choose to use Stochastic Gradient Descent (for now):

{% highlight python %}

from keras.optimizers import Adadelta
from time import time
import datetime 
import tensorflow as tf

batch_size = 64
num_epoch = 10

model.compile(loss='mean_squared_error', optimizer=tf.train.GradientDescentOptimizer(0.5), metrics=['accuracy'])

start_time = time()
trained = model.fit(
    [X_train['left'], X_train['right']], 
    Y_train, 
    batch_size=batch_size, 
    epochs=num_epoch,
    validation_data=([X_validation['left'], X_validation['right']], Y_validation)
)
print("Training time finished.\n{} epochs in {}".format(num_epoch, datetime.timedelta(seconds=time()-start_time)))

{% endhighlight %}

### Training

The full notebook can be found on [github](https://github.com/erikbeerepoot/machine-learning/blob/master/notebooks/semantic-similarity/Manhattan-LSTM.ipynb). The notebook along
with the pre-trained word vectors was copied to an AWS EC2 p2.large instance, and run in a `tensorflow_p36` virtuelenv. Training progress:


```
Train on 364290 samples, validate on 40000 samples
Epoch 1/10
364290/364290 [==============================] - 773s 2ms/step - loss: 0.0902 - acc: 0.8906 - val_loss: 0.1231 - val_acc: 0.8329
Epoch 2/10
364290/364290 [==============================] - 775s 2ms/step - loss: 0.0878 - acc: 0.8950 - val_loss: 0.1262 - val_acc: 0.8283
Epoch 3/10
364290/364290 [==============================] - 775s 2ms/step - loss: 0.0860 - acc: 0.8981 - val_loss: 0.1228 - val_acc: 0.8339
Epoch 4/10
364290/364290 [==============================] - 774s 2ms/step - loss: 0.0845 - acc: 0.9012 - val_loss: 0.1221 - val_acc: 0.8348
Epoch 5/10
364290/364290 [==============================] - 773s 2ms/step - loss: 0.0832 - acc: 0.9036 - val_loss: 0.1226 - val_acc: 0.8341
Epoch 6/10
364290/364290 [==============================] - 773s 2ms/step - loss: 0.0827 - acc: 0.9045 - val_loss: 0.1248 - val_acc: 0.8306
Epoch 7/10
364290/364290 [==============================] - 774s 2ms/step - loss: 0.0889 - acc: 0.8959 - val_loss: 0.1282 - val_acc: 0.8244
Epoch 8/10
364290/364290 [==============================] - 773s 2ms/step - loss: 0.0883 - acc: 0.8968 - val_loss: 0.1246 - val_acc: 0.8316
Epoch 9/10
364290/364290 [==============================] - 777s 2ms/step - loss: 0.0857 - acc: 0.9012 - val_loss: 0.1258 - val_acc: 0.8287
Epoch 10/10
364290/364290 [==============================] - 776s 2ms/step - loss: 0.0845 - acc: 0.9028 - val_loss: 0.1255 - val_acc: 0.8296
Training finished.
10 epochs in 2:09:03.967721
```

The main thing to note about the above results is the gap in accuracy between the training set and the validation set. This is typically 
and indication of overfitting (high variance). Hence, we should expect that questions from (or similar to) the questions in the dataset
will yield good predictions, but the model won't generalize well. We ran a simple qualitative experiment:

{% highlight python %}

import numpy as np

# q1 -> q3 are from the WikiAnswers corpus
q1 = "Are fats and oils constructed from glycerol and fatty acids?"
q2 = "What are the countries in which the islam worshippers live today?"
q3 = "What produces glycerol for fatty acid synthesis?"
# q4 -> q6 are from the Quora corpus
q4 = "What are some of the best romantic movies in English"
q5 = "What is the best way to learn c programming?"
q6 = "What is the best romantic movie you have ever seen"

q1_sequence = process_sentence(q1)
q2_sequence = process_sentence(q2)
q3_sequence = process_sentence(q3)
q4_sequence = process_sentence(q4)
q5_sequence = process_sentence(q5)
q6_sequence = process_sentence(q6)

[p1, p2, p3, p4, p5, p6] = pad_sequences(
    [q1_sequence, q2_sequence, q3_sequence, q4_sequence, q5_sequence, q6_sequence], len(X_train['left'][0])
)
sequence_1 = np.array([p1, p5, p3, p4, p5, p6])
sequence_2 = np.array([p3, p1, p2, p6, p5, p2])

model.predict( [sequence_1, sequence_2], verbose = 1)

{% endhighlight %}

with the output:

```
array([[0.14649092],
       [0.01732459],
       [0.07154535],
       [0.7385108 ],
       [1.        ],
       [0.22897404]], dtype=float32)
```

This simple experiment confirms our suspicions: the model does not generalize well. In the next post, we will look at how we can improve these results.
