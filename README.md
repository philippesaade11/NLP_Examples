# NLP_Examples
Contains functions and models I use for Natural Language Processing.

# Cleaning Sentences
clean_sentence.ipynb file contains functions to clean the sentences, removing special characters, expand contraction, lemmation, stemming, tokenizing and sentence splitting.

the split_sentences function simply splits text into multiple sentences, it contains multiple conditions and designed for specific text format.

the split_sentences_2 takes into consideration a list of topics to split the text. Depending on whether the sentence or chunk contains topics, the function decides on the split.
the function constructs a tree where each node is a cut point and each leaf is a sentence or chunk. it then calculates a score for every node, using entropy on the number of topics inside the chunk and the type of split (split on a period get a greater score than a split on a conjunction).
Finally using a specific threshold, the function outputs the splits with scores larger than the threshold.

(both split_sentences and split_sentences_2 split on conjunctions and extreme points to ensure that each chunk contains 1 sentiment to train the model inside train_CNN_w_topic.ipynb script)

# Train a Word2Vec model
train_word2vec.ipynb simply cleans sentences and trains a word2vec model using gensim, this is used to create embeddings for the following CNN models.

# Train a CNN sentiment model
after training the word2vec model and cleaning the sentences, a Convolutional Neural Network is trained to classify the sentiments of each sentence. we ue Keras and Tensorflow to create the CNN.

the problem with this model is that some sentences contain multiple sentiments and the ouput is the general sentiment without considering the topic.

for example, in the sentence "I like pizza but I hate fish" the model might output positive or negative.

The next model however, will take as input the sentence along with the index of the word to focus on and output a sentiment.
In our example if the input was the sentence and a mask like so: [0, 0, 1, 0, 0, 0, 0] the output would be positive but with the same sentence if the mask was [0, 0, 0, 0, 0, 0, 1] the output would be negative.

# Train a CNN sentiment model with topics
We first need to create data for this model, here are a few steps to generate data using the texts and sentiments we have:
- we first need to create a list of words to identify our topics (In clean_sentence.ipynb there's an example for topics when sentiments are about cars).
- we then use the sentence splitters in clean_sentence.ipynb along with the topics to split the text into multiple chunks each with 1 or fewer topics.
- the data we had (that we used to train the previous CNN) consisted of a sentiment for all the text and not the current chunks we have. Some chunks might have wrong labels now, that is why we use the first CNN model we trained that outputs the general sentiment to label the new data.
(In my case, I only took the chunks where the CNN output agreed with the label of the full text)

Now we somewhat ensured that we have smaller sentences containing less topics and 1 sentiment.

The above steps are not included in the scripts. However all the functions needed are (the sentence splitter and CNN model)
While training, a generator is used to stick sentences back together with conjunctions or commas and the model is trained multiple times with the newly constructed sentences, each time with a new topic to focus on and a new sentiment for the chosen topic.
