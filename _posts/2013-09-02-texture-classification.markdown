---
layout: post
title:  "Texture Classification"
date:   2013-09-02 16:00:00 -0300
categories: classification, computer vision, descriptor, feature extraction, image processing, matlab, texture descriptor
---

In many applications, texture has a close relationship with the semantics.
For example, Fig. 1(a) shows a CT scan of a lung. In this case, there are
differences between the textures of healthy and unhealthy tissues.
In satellite images - Fig. 1(b) - texture can be employed to differentiate
terrains such as forests, buildings and sea.

<table align="center" cellpadding="0" cellspacing="0" class="tr-caption-container" style="margin-left: auto; margin-right: auto; text-align: center;">
    <tbody>
        <tr><td style="text-align: center;">
            <a href="http://2.bp.blogspot.com/-E6DhyCwMd1s/UiN29YaR5PI/AAAAAAAAGl4/7_Zw-RZ9VYw/s1600/aplicacoes_textura.png" imageanchor="1" style="margin-left: auto; margin-right: auto;"><img border="0" src="http://2.bp.blogspot.com/-E6DhyCwMd1s/UiN29YaR5PI/AAAAAAAAGl4/7_Zw-RZ9VYw/s1600/aplicacoes_textura.png" /></a>
        </td></tr>
        <tr>
            <td class="tr-caption">Figure 1 - Texture information can be used to (a) detect unhealthy lung tissue or (b) different types of terrains.</td>
        </tr>
    </tbody>
</table>

<p>The goal of this post is to provide a brief overview of how texture information can be used to classify images using Matlab. Basically, what we want to do is to learn a texture classifier from a set of labeled images depicting textures. Then, we will use the learned classifier to provide a class label for an unlabeled image.</p>

<p>Texture classification can be divided into 3 phases which are discussed in the following sections:</p>

<ol>
    <li>Extracting texture features;</li>
    <li>Training a classifier;</li>
    <li>Classification of an unlabeled texture image;</li>
</ol>

<h2>Downloading the image set</h2>

<p>In order to classify texture images, we will use as example the <a href="http://www-cvr.ai.uiuc.edu/ponce_grp/data/" target="_blank">Textured Surfaces Dataset</a> from Prof. Jean Ponce's research group. Just click on the link and search for "Texture Database". Then download the 5 provided zip files (~40MB each).</p>

<p>Unzip all the images into a single <code>imgs</code> directory. We we will also use a text file (<a href="https://docs.google.com/file/d/0ByOACbBQiCq4YzRwQjN1YUNHdlU/edit?usp=sharing" target="_blank">image_class.txt</a>) where each line contains the file name of each image and the corresponding class. Organize the images and image_class.txt under the following directory structure:</p>

<blockquote class="tr_bq">
<span lang="EN-US">.<br />|<br />|--imgs<br />| &nbsp; &nbsp;|--img1.jpg<br />| &nbsp; &nbsp;|--img2.jpg<br />| &nbsp; &nbsp;...<br />| &nbsp; &nbsp;|--img1000.jpg<br />|<br />|--image_class.txt&nbsp;</span>
</blockquote>


<h2>Texture feature extraction</h2>

<p>There are many feature extraction algorithms available for texture analysis. In this post I will use the SFTA Texture Extractor, (<a href="http://www.mathworks.com/matlabcentral/fileexchange/37933-sfta-texture-extractor" target="_blank">Matlab code available here</a>) that I have developed as <a href="https://drive.google.com/file/d/0ByOACbBQiCq4Ymh6WnJsMWhaUWs/edit?usp=sharing" target="_blank">part of my PhD research</a>. Download the SFTA code and unzip into the working directory:</p>

<blockquote class="tr_bq">
.<br />
|<br />
|--imgs<br />
| &nbsp; &nbsp;|--img1.jpg<br />
| &nbsp; &nbsp;|--img2.jpg<br />
| &nbsp; &nbsp;...<br />
| &nbsp; &nbsp;|--img1000.jpg<br />
|<br />
|--image_class.txt<br />
|--findBorders.m<br />
|--hausDim.m<br />
|--otsurec.m<br />
|--sfta.m
</blockquote>

<p>To extract features we will use the <code>sfta(I, nt)</code> function, where <code>I</code> corresponds to the input image depicting a texture and <code>nt</code> is a parameter that defines the size of the feature vector.</p>

<h2>Training the classifier</h2>

<p>There are many good libraries and software that provide implementation of classification algorithms such as <a href="http://www.cs.waikato.ac.nz/ml/weka/" target="_blank">Weka</a> for Java or <a href="http://www.csie.ntu.edu.tw/~cjlin/libsvm/" target="_blank">LIBSVM</a>. In this example, however, I will use the classifiers provided by Matlab. More specifically, I will use the <a href="http://www.mathworks.com/help/stats/naivebayes.fit.html" target="_blank">Naive Bayes classifier</a>.</p>

<p>In order to train the Naive Bayes we need to organize the images' feature vectors into a single NxM matrix, where N is the number of images and M is the size of the feature vector. Also, we will need a Nx1 cell array containing the images' labels. The following code reads the <code>image_class.txt</code> file and generates the <code>F</code> matrix with the feature vectors and the <code>L</code> cell array with the images' labels:</p>


<blockquote class="tr_bq">
imgListFile = fopen('image_class.txt', 'r');<br />F = zeros(1000, 24); L = cell(1000, 1);<br />tline = fgetl(imgListFile); currentLine = 1;<br />while ischar(tline)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <br />&nbsp;&nbsp;&nbsp; disp(tline)<br />&nbsp;&nbsp;&nbsp; splittedLine = regexp(tline, ',[ ]*', 'split');<br />&nbsp;&nbsp;&nbsp; <br />&nbsp;&nbsp;&nbsp; % Extract the SFTA feature vector from the image.<br />&nbsp;&nbsp;&nbsp; imagePath = fullfile('imgs', splittedLine{1});<br />&nbsp;&nbsp;&nbsp; I = imread(imagePath);<br />&nbsp;&nbsp;&nbsp; f = sfta(I, 4);<br />&nbsp;&nbsp;&nbsp; F(currentLine, :) = f;<br />&nbsp;&nbsp;&nbsp; <br />&nbsp;&nbsp;&nbsp; % Store the image label.<br />&nbsp;&nbsp;&nbsp; L{currentLine} = splittedLine{2};<br />&nbsp;&nbsp;&nbsp; <br />&nbsp;&nbsp;&nbsp; tline = fgetl(imgListFile);<br />&nbsp;&nbsp;&nbsp; currentLine = currentLine + 1;<br />end;&nbsp;&nbsp; <br />fclose(imgListFile);
</blockquote>

<p>Now that we have the <code>F</code> matrix and <code>L</code> cell array, we can train a Naive Bayes classifier <code>nb</code> using the following command:</p>

<blockquote class="tr_bq">
&gt;&gt; nb = NaiveBayes.fit(F, L);
</blockquote>

<h2>Classification</h2>

<p>The <code>nb</code> classifier can be used to classify an image <code>Itest</code> using the following commands:</p>

<blockquote class="tr_bq">
&gt;&gt; Itest = imread(testImagePath)<br />
&gt;&gt; features = sfta(Itest, 4)<br />
&gt;&gt; predict(nb, features)<br />
ans = "knit"
</blockquote>
