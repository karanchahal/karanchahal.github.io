---
layout:     post
title:      Detecting Skin Cancer Using Deep Learning
date:       2016-07-24  15:31:19
summary:    My summer fling with deep neural networks
categories: Deep Learning
---

<div style="text-align:center" markdown="1">

![desk](http://www.cancerresearch.purdue.edu/files/75/breast%20cancer%20ribbon.png)


</div>
Hullo.

In the summer of 2016 I worked on a project that involved detecting mitosis cells in breast histology images. Mitosis detection in skin cells are an indicator of cancer. Cancer and mitosis are closely related. Mitosis is the process by which cells reproduce, and without it cancerous cells wouldn't be able to spread through the body and cause tumors.
The project was really interesting because of mainly three reasons.

* Firstly, a successful implementation can reduce doctor hours drastically and automate this procedure by reducing the manual part of the procedure

* Secondly, it was a challenge to do image recognition using deep learning which represents the bleeding edge of tech in this field. Image detection has come a long way and detection of these structures is very feasible. It the perfect application of Convolution Neural Nets

* A chance to dive into one of the excellent Python Deep Learning libraries such as *Theano*,*Keras* etc.



To solve this problem we needed two things, a dataset and a manual of how to go about solving this problem.
We were fortunate enough to get both of these things in the form of the MITOS dataset which consisted of histology images with a magnification of 10x,20x,40x.
As for the manual part, a research paper on the Topic of [*Detecting Mitosis in Breast Histology Images using Deep Learning*](http://people.idsia.ch/~ciresan/data/miccai2013.pdf "Detecting Mitosis in Breast Histology Images using Deep Learning") by DC Ciresan was referred.

To know more about deep learning I would like to refer you to this wonderful resource [here](http://neuralnetworksanddeeplearning.com/) .


<div style="text-align:center" markdown="1">
![desk](https://researchweb.iiit.ac.in/~shashank.mujumdar/img/Picture14.jpg)
</div>

Now that we had gotten these things, the first objective was to parse the dataset to build our X and y dataset. Also split this into the training, testing and cross validation datasets.

A challenge faced was that the amount of mitosis cells greatly outnumbered the non mitosis cells roughly to the ratio of 1:90.To solve this,5 random pixels and all the mitosis cells were taken from the image. Patches of the size 101 * 101 were cut centered on the pixels. Mirroring was done when the patch was unable to be cut off if the image ran out of space. Finally ,to bring the number of mitosis and non mitosis patches to an equal number, rotation was done to artificially increase the size and diversity of the dataset.

After getting these 101*101 patches with their respective category the dataset was divided into the training, cross-validation and testing datasets were created in the ratio of 60:20:20 respectively.

Now we move on to the interesting part. Creating and training the model.


![desk](http://deeplearning.net/tutorial/_images/mylenet.png)
A convolutional neural network is a special type of a neural net which captures spatial information much more clearly while also removing redundant/useless information. This type of network allows for image recognition in a very efficient and effective manner. You can get a very clear explanation if CNN's [here](http://karpathy.github.io/2015/10/25/selfie/) and [here](http://www.wildml.com/2015/11/understanding-convolutional-neural-networks-for-nlp/).

For our project a convolutional net was used with a number of alternating filters and max pool layers. The exact details can be looked up in the paper.
After reducing the 101*101 patches to roughly 24*24 patches which were then run through a fully connected multi perceptron network which was attached to a [softmax](https://en.wikipedia.org/wiki/Softmax_function) layer to output the probabilities of the two classes.

After training for roughly a day, the model was tested and the hyper parameters varied.
We managed to achieve an accuracy of 82% which was roughly equal to what the paper stated.

![desk](http://medicalimaging.spiedigitallibrary.org/data/Journals/JMIOBU/930715/JMI_1_3_034003_f008.png)

After training the model, the final demo had to be created.
The demo took a breast histology image as input and outputted the same image overlaid with indicators at pixel points where a mitosis cell was detected.
To achieve this, after taking the image as input , a subset of points were deemed worthy of further scrutiny by running into a DNN which told with a 60 % probability that the cells were mitosis.
The positive results from this DNN were then fed into the main model that we had trained above, where the final result would be calculated. The probabilities outputted were put in a probability map. This probability map which usually is extremely chaotic was smoothened using a gaussian blur function. Finally the pixel points of the actual mitosis were extracted if they cleared some pre set threshold.



The team I was working were great experienced people who really guided me throughout the process and I would like to thank them for the support and advice. It was invaluable.
