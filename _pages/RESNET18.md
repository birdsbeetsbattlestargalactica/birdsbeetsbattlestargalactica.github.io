---
title: Experiments with RESNET18
author: Adina Tung, James Froelich
date: 2023-06-03
category: Jekyll
layout: post
cover: https://birdsbeetsbattlestargalactica.github.io/assets/birds_better.gif
---


We started our initial attempts with RESNET18 mostly copying the example set in the tutorial. We went off the assumption that since the prediction from the tutorial was already doing pretty well to begin with, our only job would be fine-turning the set of parameters to find the sweet spot. With the hyperparameters set the same as the tutorial (momentum = 0.9, decay = 0.0005) and a batch size of 128, we were seeing plateauing at a loss of about 0.5. In an effort to reduce loss and/or possibly prevent overfitting, we added more image augmentation. Our thought process was that if we add another vertical/horizontal flip or adjust the brightness or saturation slightly, we could train the model to detect a bird despite imperfections in the image.  

We tried the following:  
    1. Randomly flipping images vertically (`transforms.RandomVerticalFlip()`), [doc][1]  
    2. Randomly flipping images horizontally (`transforms.RandomHorizontalFlip()`), [doc][2]  
    3. Randomly adjusting the brightness, contrast, saturation and hue  (`transforms.ColorJitter(0.2, 0.2, 0.2, 0.2)`), [doc][3]  
    4. Normalizing the image (`transforms.Normalize((.5, .5, .5), (.5, .5, .5))`), [doc][4]  
    5. Inverting the image (`transforms.RandomInvert(0.125)`), [doc][5]  
    
    
For the following, momentum = 0.9, decay = 0.0005  

<div class="table-wrapper" markdown="block">

|epoch|batch size|schedule|horizontal/vertical flip (p)|random color jitter (p)|normalize|invert (p)|final loss|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:| 
|5|64|{0:.01, 4:0.001}|0.5/0.5|0.2|0.5|-|1.364| <!-- /assets/resnet18_ep5_hv-flip.png  v15 -->   
|5|64|{0:.03, 1:0.01, 4:0.001}|0.5/0.5|0.2|-|-|1.160| <!--/assets/resnet18_ep5_hv-flip_jitter.png v13-->  
|8|64|{0:.01, 4:.001}|0.5/0.5|0.5|-|-|1.516| <!-- /assets/3427.png v'restart, no more invert'-->  
|10|64|{0:.01, 8:.001}|0.5/-|-|-|-|0.528| <!-- /assets/7313.png  v8-->  
|12|64|{0:.01, 6:.001}|0.5/0.5|0.5|-|0.125|1.270|  <!-- /assets/6322.png v'training 8 to 12'-->  

</div>

#### Plots:  

###### epoch = 5, schedule = {0:.01, 4:0.001}, horizontal flip (p) = 0.5, vertical flip (p) = 0.5, color jitter (p) = 0.2 normalize (mean, std) = (0.5, 0.5), invertion (p) = 0, final loss = 1.364  

![Resnet18 with 5 epochs, horizontal and vertical flip](https://birdsbeetsbattlestargalactica.github.io/assets/graphs/resnet18_ep5_hv-flip.png)

###### epoch = 5, schedule = {0:.03, 1:0.01, 4:0.001}, horizontal flip (p) = 0.5, vertical flip (p) = 0.5, color jitter (p) = 0.2 normalize (mean, std) = (0, 0), invertion (p) = 0, final loss = 1.160  

![Resnet18 with 5 epochs, horizontal and vertical flip](https://birdsbeetsbattlestargalactica.github.io/assets/graphs/resnet18_ep5_hv-flip_jitter.png)

###### epoch = 8, schedule = {0:.01, 4:.001}, horizontal flip (p) = 0.5, vertical flip (p) = 0.5, color jitter (p) = 0.5 normalize (mean, std) = (0, 0), invertion (p) = 0, final loss = 1.516  

![Resnet18 with 8 epochs, horizontal and vertical flip](https://birdsbeetsbattlestargalactica.github.io/assets/graphs/3427.png)

###### epoch = 10, schedule = {0:.01, 8:.001}, horizontal flip (p) = 0.5, vertical flip (p) = 0, color jitter (p) = 0 normalize (mean, std) = (0, 0), invertion (p) = 0, final loss = 0.528  

![Resnet18 with 10 epochs, horizontal and vertical flip](https://birdsbeetsbattlestargalactica.github.io/assets/graphs/7313.png)

###### epoch = 12, schedule = {0:.01, 6:.001}, horizontal flip (p) = 0.5, vertical flip (p) = 0.5, color jitter (p) = 0.5 normalize (mean, std) = (0, 0), invertion (p) = 0.125, final loss = 1.270  

![Resnet18 with 12 epochs, horizontal and vertical flip](https://birdsbeetsbattlestargalactica.github.io/assets/graphs/6322.png)

Out of all trials above, the best prediction on the 20% test set never exceeded 79% accuracy. We concluded that since these didn't get stuck on a plateau, the results indicated the dataset was perhaps too complicated for resnet18. We needed additional layers to do feature extraction on a more generalized trainset. This gave us the idea to train with resnet34.


[1]: https://pytorch.org/vision/main/generated/torchvision.transforms.ColorJitter.html
[2]: https://pytorch.org/vision/main/generated/torchvision.transforms.RandomVerticalFlip.html#torchvision.transforms.RandomVerticalFlip
[3]: https://pytorch.org/vision/main/generated/torchvision.transforms.RandomHorizontalFlip.html#torchvision.transforms.RandomHorizontalFlip
[4]: https://pytorch.org/vision/main/generated/torchvision.transforms.Normalize.html
[5]: https://pytorch.org/vision/main/generated/torchvision.transforms.RandomInvert.html#torchvision.transforms.RandomInvert

