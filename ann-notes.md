## Objective

To query for patterns tied to a parcticular word in a code. E.g.: to a variable in C always tied pattern "declaration".

## Algo:
1. take word w
2. remember distance to other words in distance d
3. if the word already happened, check for patterns both in regard to prev. word and the new one.
5. ↑ repeat for the next w.

Preliminary idea: when a word repetition found, check if prev. similar words have same distances to words… which ones?

Shall I may be set in advance possible patterns? Like "any word", "closing word", etc.

## ANNs with regard to source code structure analysis.

### Autoencoder

Learns input specifics with the purpose of restoring the input from broken or noisy samples.

Applications: denoising images, restoring images, coloring black-and-white images *(which is also sort of restoring)*.

Full example: eyes from sequential photos fed to autoencoder layer for denoising, and then to LSTM to detect a motion sign over the series. Been used to control stuff with eyes motion.

How can I use it: to restore code structure for user is typing, so it's in invalid state. But it is better to restore patterns instead of text because it's more semantic and probably cheaper. So autoencoder should get noisy patterns as input, and to output restored ones. Hence patterns in the first place should be learned by other kind of layer.

### Probabilistic

Typically have output nodes representing classes, and, given input, assigns probability of each class to the output.

How can I use it: it's typically supervised learning, and have predefined classes. Unless I gotta find how to use it unsupervisingly, and to make it to create output nodes itself, it's useless for the task.

### A time delay neural network (TDNN)

A usual perceptron working with sequential data. The name have nothing to do with the ANN, it's because the data delayed before feeding to the ANN, so that data from different time is analized together, like if the data happened simulationously.

Useless for me for reasons mentioned in "Probabilistic" ann; useless for anyone because LSTM should do the same without artificial delays.

### Convolutional

Uses math convolution operation. Don't know much details yet, but it don't seem to be used anywhere except of a work with images.

### Regulatory feedback

???

### RBF

Assign points to clusters in n-dimensional data. Requires I think predefined outputs.

Contras mentioned in "Probabilistic" holds.

### Self-organizing map

???

### Learning vector quantization

???

### Echo state

A useless RNN with sparse connections, where only output layer neurons can change.

### Bi-directional

Odd technique of using two RNNs, one processing inputs left-to-right, and the other right-to-left; and then summing the results. Used for labeling and prediction. It's said to be very useful with LSTM.

For my purpose the complain about predefined outputs holds.

### Hierarchical

???


### Stochastic

???

### Neocognitron

??? used for pattern recognition.

### Cascading

Any ANN varying its topology and size.

### Neuro fuzzy

???

### Holographic Associative Memory

Probably akin to LSTM.

# word2vec and GloVe

Both calculate "distance" of a word to every other. Except word2vec is probability-based, and GloVe is count-based. Also that GloVe is more easily paralleled since it takes whole matrix at once, whereas word2vec ANN-based, and uses many passes.

Acc. to internet, final results are more-or-less identical.
