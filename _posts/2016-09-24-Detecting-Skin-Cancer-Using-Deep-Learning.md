---
layout:     post
title:      Detecting Skin Cancer Using Deep Learning
date:       2016-07-1  15:31:19
summary:    A description of my summer internship project
categories: Deep Learning
---

<div style="text-align:center" markdown="1">

![desk](http://www.cancerresearch.purdue.edu/files/75/breast%20cancer%20ribbon.png)


</div>
Hullo.

In the summer of 2016 I worked on a project that involved detecting mitosis cells in breast histology images.Mitosis detection in skin cells are an indicator of cancer.
**Define mitosis**.
The project was really interesting because of mainly three reasons.
* Firstly,a successful implementation can reduce doctor hours drastically and automate this procedure by reducing the manual part of the procedure

* Secondly, it was a challenge to do image recognition using deep learning which represents the bleeding edge of tech in this field. Image detection has come a long way and detection of these structures is very feasible.It the perfect application of Convolution Neural Nets
* A chance to dive into one of the excellent Python Deep Learning libraries such as *Theano*,*Keras* etc.



To solve this problem we needed two things, a dataset and a manual of how to go about solving this problem.
We were fortunate enough to get both of these things in the form of the MITOS dataset which consisted of histology images with a magnification of 10x,20x,40x.
As for the manual part, a research paper on the Topic of *Detecting Mitosis in Breast Histology Images using Deep Learning* [*Detecting Mitosis in Breast Histology Images using Deep Learning*](http://people.idsia.ch/~ciresan/data/miccai2013.pdf "Detecting Mitosis in Breast Histology Images using Deep Learning") by DC Ciresan was referred to.

![desk](https://researchweb.iiit.ac.in/~shashank.mujumdar/img/Picture14.jpg)


Now that we had gotten these things, the first objective was to parse the dataset to build our X and y dataset and split it into the test,train and cross validation datasets.

There was an issue encountered,the amount of mitosis cells were greatly outnumbered by non mitosis cells.Roughly to the ratio of 1:90.To solve this, from one image we take 5 random pixels were taken and all the mitosis cells were taken.Patches of the size 101 * 101 were cut centered on the pixels.Mirroring was done when the patch was unable to be cut if the image ran out of space.
To bring the number of mitosis and non mitosis patched to an equal number.Rotation was done to artifically increase the size of the dataset.

After getting these 101*101 patches with their respective category.The dataset was created .THe train,cross-val and test datasets were created in the ratio of 60:20:20 respectively.

Now we move on to the interesting part.Creating and training the model.


![desk](http://deeplearning.net/tutorial/_images/mylenet.png)

A convolutional net was used with a number of alternating filters and max pool layers.The exact details can be looked up in the paper.
After reducing the 101*101 patches to roughly 24*24 patches these were run through a fully connected multi perceptron network which was attached to a softmax layer to output the probabilities of the two classes.

After training for roughly a day,the model was tested and the hyper parameters varied.
We managed to achieve an accuracy of 82% which is roughly equal to what the paper stated.

![desk](http://medicalimaging.spiedigitallibrary.org/data/Journals/JMIOBU/930715/JMI_1_3_034003_f008.png)

After training the model, the final demo had to be created.


The team I was working were great experienced people who really guided me throughout the process and I would like to thank them for the support and advice.It was invaluable.
