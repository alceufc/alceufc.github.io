---
layout: post
title:  "Fractal Dimension from Image"
date:   2013-11-10 16:00:00 -0300
categories: computer vision, image processing, matlab, fractal
excerpt: "How to compute the fractal dimension from an object given its image (Matlab code example).
"
---

Suppose that we want to estimate the fractal dimension from an object
represented by an image:

{% include post_figure.html url="/assets/post_imgs/sierpinski.png"
    caption="Image depicting the Sierpinski  triangle." %}

The image depicts the
[Sierpinski triangle](http://en.wikipedia.org/wiki/Sierpinski_triangle),
a well known fractal object.
Computing the fractal dimension of an object can be useful, for example,
in image [feature extraction](http://www.alceufc.com/2013/09/texture-classification.html).

In our case, the fractal object is represented by a binary image *I*, where
non-zero pixels belong to the object and zero pixels constitute the background.
We can estimate the [Haussdorf fractal dimension](http://en.wikipedia.org/wiki/Hausdorff_dimension)
of the object using the following algorithm:

  1. Pad the image with background pixels so that its dimensions are a power
     of 2;
  2. Set the box size *e* to the size of the image;
  3. Compute *N(e)*, which corresponds to the number of boxes of size *e*
      which contains at least one object pixel;
  4. If *e > 1* then *e = e / 2* and repeat step 3;
  5. Compute the points *log(N(e)) x log(1/e)* and use the least squares
     method to fit a line to the points.
  6. The returned Haussdorf fractal dimension *D* is the slope of the line.

## Matlab code

[This link](http://www.mathworks.com/matlabcentral/fileexchange/30329-hausdorff-box-counting-fractal-dimension)
provides my implementation of the previous algorithm. `The hausDim()` function
can be used to estimate the fractal dimension of the depicted fractal. Note that
first it is necessary to convert the image to a binary matrix using the
[im2bw](http://www.mathworks.com/help/images/ref/im2bw.html) function:

```matlab
% Read the image.
>> I = imread('alceufc.github.io/assets/post_imgs/sierpinksy.png');

% Convert to a binary image and negate pixel values.
>> I = ~im2bw(I);
>> hausDim(I)

ans = 1.5496
```

The estimated value is 1.5496. This is quite close to the
[exact value](http://en.wikipedia.org/wiki/List_of_fractals_by_Hausdorff_dimension)
which is *log(3) / log(2) = 1.58496...*.