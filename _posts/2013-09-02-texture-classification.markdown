---
layout: post
title:  "Texture Classification"
date:   2013-09-02 16:00:00 -0300
categories: classification, computer vision, descriptor, feature extraction, image processing, matlab, texture descriptor
excerpt: "How to use Matlab to extract texture features and classify images."
---

In many applications, texture has a close relationship with the semantics.
For example, Fig. 1(a) shows a CT scan of a lung. In this case, there are
differences between the textures of healthy and unhealthy tissues.
In satellite images - Fig. 1(b) - texture can be employed to differentiate
terrains such as forests, buildings and sea.

{% include post_figure.html url="/assets/post_imgs/aplicacoes_textura.png"
    caption="Figure 1 - Texture information can be used to (a) detect unhealthy lung tissue or (b) different types of terrains." %}

The goal of this post is to provide a brief overview of how texture information
can be used to classify images using Matlab. Basically, what we want to do is
to learn a texture classifier from a set of labeled images depicting textures.
Then, we will use the learned classifier to provide a class label for an
unlabeled image.

Texture classification can be divided into 3 phases which are discussed in the
following sections:

 1. Extracting texture features;
 2. Training a classifier;
 3. Classification of an unlabeled texture image;

## Downloading the image set

In order to classify texture images, we will use as example the
[Textured Surfaces Dataset](http://www-cvr.ai.uiuc.edu/ponce_grp/data/)
from Prof. Jean Ponce's research group. Just click on the link and search for
"Texture Database". Then download the 5 provided zip files (~40MB each).

Unzip all the images into a single `imgs` directory. We we will also use a text
file
([image_class.txt](https://docs.google.com/file/d/0ByOACbBQiCq4YzRwQjN1YUNHdlU/edit?usp=sharing))
where each line contains the file name of each image and the corresponding
class. Organize the images and `image_class.txt` under the following directory
structure:

```
.
|
|--imgs
|    |--img1.jpg
|    |--img2.jpg
|    ...
|    |--img1000.jpg
|
|--image_class.txt
```

## Texture feature extraction

There are many feature extraction algorithms available for texture analysis.
In this post I will use the SFTA Texture Extractor
([Matlab code available here](http://www.mathworks.com/matlabcentral/fileexchange/37933-sfta-texture-extractor)
that I have developed as [part of my MSc. research](/assets/papers/Costa_SIBGRAPI_2012.pdf).
Download the SFTA code and unzip into the working directory:

```
.
|
|--imgs
|    |--img1.jpg
|    |--img2.jpg
|    ...
|    |--img1000.jpg
|
|--image_class.txt
|--findBorders.m
|--hausDim.m
|--otsurec.m
|--sfta.m
```

To extract features we will use the `sfta(I, nt)` function, where `I`
corresponds to the input image depicting a texture and `nt` is a parameter that
defines the size of the feature vector.

## Training the classifier

There are many good libraries and software that provide implementation of
classification algorithms such as [Weka](http://www.cs.waikato.ac.nz/ml/weka/)
for Java or [LIBSVM](http://www.csie.ntu.edu.tw/~cjlin/libsvm/).
In this example, however, I will use the classifiers provided by Matlab. More
specifically, I will use the
[Naive Bayes classifier](http://www.mathworks.com/help/stats/naivebayes.fit.html).

In order to train the Naive Bayes we need to organize the images' feature
vectors into a single *NxM* matrix, where *N* is the number of images and *M*
is the size of the feature vector. Also, we will need a Nx1 cell array
containing the images' labels. The following code reads the `image_class.txt`
file and generates the `F` matrix with the feature vectors and the `L` cell
array with the images' labels:

```matlab
imgListFile = fopen('image_class.txt', 'r');
F = zeros(1000, 24); L = cell(1000, 1);
tline = fgetl(imgListFile); currentLine = 1;
while ischar(tline)
    disp(tline)
    splittedLine = regexp(tline, ',[ ]*', 'split');

    % Extract the SFTA feature vector from the image.
    imagePath = fullfile('imgs', splittedLine{1});
    I = imread(imagePath);
    f = sfta(I, 4);
    F(currentLine, :) = f;

    % Store the image label.
    L{currentLine} = splittedLine{2};

    tline = fgetl(imgListFile);
    currentLine = currentLine + 1;
end;
fclose(imgListFile);
```

Now that we have the `F` matrix and the `L` cell array, we can train a Naive
Bayes classifier `nb` using the following command:

```matlab
>> nb = NaiveBayes.fit(F, L);
```

## Classification

The `nb` classifier can be used to classify an image `Itest` using the
following commands:

```matlab
>> Itest = imread(testImagePath)
>> features = sfta(Itest, 4)
>> predict(nb, features)
ans = "knit"
```
