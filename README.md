
![](image/deepLearning.png)

## Introduction
Google released the ‘Google’s Conceptual Captions’ dataset for image captioning as a new image-recognition challenge and an exercise in AI-driven education. Since pictures may convey a great deal of information, it is important that we create ways of generating conceptual captions from images automatically and accurately.

This project focus on combining current state-of-the-art techniques in both computer vision and natural language processing in a single jointly-trained system which takes images as inputs and generates human-readable descriptions of them.

## Objective
Develop a useful Deep Learning application which combines both computer vision and natural language processing to create accurate, comprehensible captions from images alone.

Understand the various ways in which we can train Neural Networks more quickly through the use of optimizers?

Understand what’s going on in the model, detailing the inter-relations between components.

![](image/objective.png)
<p align="center">
Figure 1. Flowchart for automatic image captioning
</p>

## Data Overview
I used the flickr8k data set which contains more than eight thousand images and their natural-language captions[1].

Suppose you see these pictures -

![](image/image_samples.png)
<p align="center">
Figure 2. Flickr8k image samples
</p>

What are the first things that come to your mind?

Here are a few sentences that people could come up with:
- Left image: "beige puppy walks across the floor"
- Middle image: "blond girl in brown shirt with black pen up her nose"
- Right image: "little girl smiles as she wears white bowl on the top of her head"

## Model Structure
The task of image captioning can be divided into two models:

    • Image Based Model: extract features from an image
    • Language Based Model: create a sentence about the picture using previous captions together with features given by our image based model.

Below are the details of the above two models:

### Image Based Model – Convolutional Neural Network
I utilize transfer learning to generate image features by feeding the images into a pre-trained VGG16 model. This popular model was developed by Oxford's Visual Geometry Group and in recent times has been used in a variety of image-processing tasks.

### Language Based Model – Recurrent Neural Network
Caption sequences were generated by inputting text into a word embedding layer and feeding the resultant output into a subsequent long short-term memory (LSTM) layer.

LSTM cells are a particular kind of recurrent neural net (RNN) with a more complicated set of internal weights which make them both better able to learn long-term dependencies and less susceptible to the vanishing/exploding gradient problem.

![](image/model.png)
<p align="center">
Figure 3. Model Structure
</p>

## Data Engineering
I created the following three functions for cleaning my existing set of captions:

    • remove punctuation
    • remove single character
    • remove numeric characters

After cleaning, the vocabulary size was reduced by about 200 tokens, where each token is just a unique word. A word cloud was generated in order to easily determine which words and phrases were the most common.

![](image/wordCloud.png)
<p align="center">
Figure 4. Word cloud for cleaned captions
</p>

## Image Based Model - CNN

### Create features for each image using VGG16's pre-trained networks

Because the VGG16’s pre-trained network requires an input of size (224,224,3), I had to resize all the input images to the same dimensions. Next these re-sized images were sent through the network and encoded into a 4096 size array. These arrays were re-sized once again so as to have the same dimension as the inputs. If this procedure is hard to follow, consult the flowchart below.

![](image/vgg16.png)
<p align="center">
Figure 5. Flowchart for imaged based model
</p>

### PCA Visualization of the VGG16 features
For each image, 4096 features are created, reflecting the dimensions of the array passed through the network. Because humans lack the circuitry required to visualize a 4096-dimensional space, I created 2-dimensional representation of the space using PCA and visualized the distribution of the sample images.

### Do photo features make sense?
I used k-means clustering on the VGG16 features to separate the images into 4 different clusters.

![](image/pca_3d.gif)
<p align="center">
Figure 6. 3D embedding plot for a subset of 1000 VGG16 features
</p>

I then plotted images from the purple and blue cluster. As you can see, images from the blue cluster tend to resemble each other and images from the purple cluster to resemble each other.

I therefore submit that this clustering schema makes sense.

![](image/pca_image_new.png)
<p align="center">
Figure 7. Image samples for the purple cluster and the blue cluster
</p>

## Language Based Model – RNN

My goal is to build a Language Based Model using an RNN. Here’s what that means: for a given sequence of words of length *n*, the model has a set of probabilities corresponding to predictions about what the next word *n + 1* will be. It chooses the highest-probability word from this distribution and continues this process until it generates the special end marker “endseq”, which appears at the end of each sentence in the corpus.

RNNs are sometimes depicted as being "unfolded" to their full extent. By 'unfolding' we simply mean that we write out the network as it would appear as it processes an input. For example, image that we're dealing with a sentence consisting of 5 words. In this case I would vectorize these words, producing five vectors, each of which would be paired with the input image and fed into the corresponding cell of the RNN.

The first cell in the RNN would get just the image (we're not predicting words yet), and would output 'startseq', the special sequence which begins every sentence in the corpus. The second cell would receive the image together with 'startseq', and would predict the first word in the sentence. The third cell would receive the image, 'startseq', plus the first predicted word, and predict the second word. The fourth cell would receive the image, 'startseq', and the first two words, and predict the third word...

Here is an example of an unfolded network:

![](image/rnn.png)
<p align="center">
Figure 8. Flowchart for an unfolded RNN
</p>

In the above:

- x_t is the input at time step t. For example, x_1 could be a one-hot vector corresponding to the second word of a sentence.
- s_t is the hidden state at time step t. It’s the “memory” of the network. s_t is calculated based on the previous hidden state and the input at the current step: s_t=f(Ux_t + Ws_{t-1}). The function f usually is a nonlinearity such as tanh or ReLU.
- o_t is the output at step t. For example, if we wanted to predict the next word in a sentence it would be a vector of probabilities across our vocabulary. o_t = {softmax}(Vs_t)[2].


## How to train Neural Networks more quickly through the use of optimizer?
Optimization refers to searching the space of possible parameters that minimize or maximize some function that we care about.

Typically training a machine learning model involves "indirect" optimization, wherein we pick a metric such as like recall or accuracy to quantify how well our model solves the problem we're interest in. However, we are actually optimizing a cost function which measures the errors in our predictions and hoping that this also moves our chosen metric in the direction we want it to go [3].

![](image/optimizer_comparison.gif)
<p align="center">
Figure 9. Optimizer comparison [3]
</p>

### Gradient Descent
One way we can achieve this is to take our cost function, calculate its gradient, and try to 'descend' the gradient, a process captured by the following equation:

![](image/gd.png)

Here, the learning rate 'α' describes how much we will change our weights with each iteration of training. Smaller learning rates means we have to do more calculations, but a big one makes it possible that we will miss the minimum of the cost function altogether.

![](image/learning_rate.gif)
<p align="center">
Figure 10. Learning rate comparison [3]
</p>

### Gradient Descent with Momentum
Let's take rolling a ball downhill as an example. Adding momentum term (*V_dw* and *V_db*) to the descent is make the ball roll faster and faster. Beta is used to prevent the ball speeding up without limit. It is just a way to make the gradient descent happen faster by skipping over the steeper parts of the learning curve.

![](image/gd_momentum.png)

### Root Mean Squared Propagation (RMSProp)
Another way of approaching gradient descent is to use Root Mean Squared Propagation, or 'RMSProp', which has come into frequent use in recent years. It uses adaptive learning rate to help pointing the correct direction towards the minimum.

![](image/rmsprop.png)

### Adaptive Moment Estimation (Adam)
Finally, Adaptive Moment Estimation, or "Adam", is a popular choice for gradient descent because it is a powerful blend of RMSProp and descent with momentum:

![](image/adam.png)

## Model Result
Figure 11 shows the loss trends for both the training data and the testing data through using different optimizers. It is apparently Adaptive Moment Estimation (Adam) optimizer outperformed other optimizers in this case.

![](image/loss.png)
<p align="center">
Figure 11. Loss comparison for different optimizer
</p>

I discovered that my model's loss on training data continued to decrease even its loss on evaluation data increased when using Adam. For navigation, I wrote a function to store the model weights for each epoch after they had been used to make predictions. This way I could visually determine if the model was overfitting and the results were being skewed. Below table lists the predicted captions at selected epoch for random chosen images.

![](image/caption_sample.png)
<p align="center">
Figure 12. Model-generated Caption for Selected Epochs
</p>


## Model Evaluation
### Bilingual Evaluation Understudy (BLEU)
For testing purposes I turned to the Bilingual evaluation understudy (BLUE) set of benchmarks. BLEU contained a metric for determining the similarity between a model-generated sentence and various other reference sentences. Being close to 1 means that the two are very similar, being close to 0 means they are very dissimilar[4].

### Basic Idea of BLEU
For example let's consider a scenario in which I want to compare the similarly between the hypothesis sentence and the reference sentences.

![](image/bleu.png)
<p align="center">
Figure 13. Flowchart for BLEU calculation
</p>

Finally, 1-gram BLEU, 2-gram BLEU, 3-gram BLEU and 4-gram BLEU are calculated and its average is reported as BLEU. BLEU score calculation is implemented in nltk.util and the source code is available[5].

Below are some examples with BLEU score larger than 0.7:
![](image/good_caption.png)
<p align="center">
Figure 14. Examples for predicted caption with BLEU score larger than 0.7
</p>

Below are some examples with BLEU score larger than 0.3, but less than 0.7:
![](image/mid_caption.png)
<p align="center">
Figure 15. Examples for predicted caption with BLEU score larger than 0.3, but less than 0.7
</p>

## Conclusion
This project focused on building a system to generate captions for images. I used a pre-trained image-model (VGG16) to generate a "thought-vector" of the image, and then I trained a Recurrent Neural Network to map this "thought-vector" to a sequence of integers representing words.

I also explored various ways in which we can train Neural Networks more quickly through the use of optimizers. It turned out that the Adaptive Moment Estimation (Adam) optimizer outperformed other optimizers in this case.

This model works reasonably well, although it is easy to find examples both in the training- and validation-sets where the captions are incorrect.

It's crucial to bear in mind that this model doesn't have a semantic understanding of the content of the images. If it sees an image of a dog and correctly produces a caption stating that, it doesn't mean that the model has anything corresponding to the 'dog' concept.

## Future Work
- Incorporate larger dataset into the model.
- Use another transfer-layer from the VGG-16 model, for example the flatten output of the last convolutional layer.
- Improve the model to generate multiple sequences and find a way to select the best of these different sequences.
- Explore another evaluation metrics to measure the quality of the generated captions.

## Reference
[1] Cyrus Rashtchian, Peter Young, Micah Hodosh, and Julia Hockenmaier. Collecting Image Annotations Using Amazon's Mechanical Turk. In Proceedings of the NAACL HLT 2010 Workshop on Creating Speech and Language Data with Amazon's Mechanical Turk.

[2] Recurrent Neural Networks Tutorial
(http://www.wildml.com/2015/09/recurrent-neural-networks-tutorial-part-1-introduction-to-rnns/)

[3] How to Train Neural Netowrk Faster with Optimizers
(https://towardsdatascience.com/how-to-train-neural-network-faster-with-optimizers-d297730b3713)

[4] Kishore Papineni, Salim Roukos, Todd Ward, Wei-Jing Zhu. (2012). BLEU: a Method for Automatic Evaluation of Machine Translation. IBM T. J. Watson Research Center
Yorktown Heights, NY 10598, USA.

[5] Develop an image captioning deep learning model using Flickr 8K data
(https://fairyonice.github.io/Develop_an_image_captioning_deep_learning_model_using_Flickr_8K_data.html)

[6] Profolio image source: ZDNet

Copyright © 2018 Jasmine He
