# Rolling Progress Report

*updated Sept. 15, 2017*

## recent experiments and work

### refactor Korean morphological analysis codebase

- switched to `keras-contrib` implementation of CRF
- fixed some errors in `Generator` for data reading
- other minor bugfixes/improvements
- investigated class-based architecture for training, test

### testing CRF 

**English Named Entity Recognition**

- see [this repo (master branch)](https://github.com/SNUDerek/lstm-crf_named_entity)
- bi-LSTM-CRF network architecture
- new `keras-contrib` CRF
- trained on ConLL-2002 NER dataset
- final test accuracy: 97.62%

**English Metaphor Detection**

- see [this repo (metaphor branch)](https://github.com/SNUDerek/lstm-crf_named_entity/tree/metaphor)
- bi-LSTM-CRF network architecture
- new `keras-contrib` CRF
- trained on ??? corpus (Tim's dataset)
- final results:

```
on withheld test sentences:

             precision    recall  f1-score   support

        NON       0.94      0.95      0.95      4968
        MET       0.63      0.58      0.60       662

avg / total       0.91      0.91      0.91      5630

metaphor-positive label info
true positives : 384
false positives: 229
false negatives: 278
rcll (tp/tp+fn): 58.00604229607251
prec (tp/tp+fp): 62.64274061990212
```

according to Tim, this F1 score for metaphor detection (0.60) slightly outperforms previous methods (0.58).

**Wordspacing and EOS labeling in Korean/English**

- (no repo; just testing)
- simulated audio STT outputs of n-character windows
- windows contained only english letters/korean syllables
- windows could start or end mid-word
- task: insert spaces and end-of-sentence markers
- data: crawled data from public websites
- results:

*Korean Data* : decent results, accuracy around 90%

*English Data* : spacing performed well, but end-of-sentence tagging performed poorly, probably due to lack of grammatical information (due to window size and English SVO ordering)

## future plans and projects

### Create refined development dataset for Korean morphological analysis

Goal: create a list of simple sentences (not titles, fragments, etc) of similar length from the Sejong Corpus and/or a totally simulated dataset 

Purpose: to revisit the morphological analysis neural network and analyse its behavior more closely.

### Unsupervised/semisupervised Named Entity *Detection*

From: [Collins and Singer 1999](http://www.aclweb.org/anthology/W99-0613)

Goal: label named entities in *unlabeled* corpus

Concept: Collins and Singer 1999 relied on iterative algorithms (EM, boosting, etc.) to iteratively refine classification based on *hand-engineered* intra-word (spelling) and inter-word (grammar) features. Curious about applying similar methods to distributed features (word and phrase/sentence embeddings).

### Sentence classification, NER, and question-generation

Side project involving a product review dataset. Task is to brainstorm and prototype various 'cool things' to do with the data.

Ideas:

- iterative method for sentence-level sentiment analysis
- per-product review aggregation and mapping pros/cons
- prototyping unsupervised named entity labeling (see above)
- follow-up question generation based on NER/keywords and sentence-level  sentiments (e.g. in a positive review, identify a negative point and ask a follow-up question about the specific problem)

### Reinforcement Learning Basics

Goal: development and refinement of neural networks for reinforcement-learned problems.

Concept: initial project is development of simple tic-tac-toe RL network trained on minimax opponent. Future development will focus on basic chatbot functionality.

Inspirations and Lessons:

[Simple Reinforcement Learning with Tensorflow](https://medium.com/emergent-future/simple-reinforcement-learning-with-tensorflow-part-0-q-learning-with-tables-and-neural-networks-d195264329d0)

[Intel Nervana: Demystifying Deep Reinforcement Learning](https://www.intelnervana.com/demystifying-deep-reinforcement-learning/)

[Reinforcement Learning w/ Keras + OpenAI: DQNs](https://medium.com/towards-data-science/reinforcement-learning-w-keras-openai-dqns-1eed3a5338c)

[Minimax Algorithm in Game Theory](http://www.geeksforgeeks.org/minimax-algorithm-in-game-theory-set-3-tic-tac-toe-ai-finding-optimal-move/)

[OpenAI Gym](https://gym.openai.com)


## notes, papers, links

paper from Amazon Alexa competition from the Montreal Institute for Learning Algorithms:
[Serban et al. 2017: A Deep Reinforcement Learning Chatbot](https://arxiv.org/pdf/1709.02349.pdf)
