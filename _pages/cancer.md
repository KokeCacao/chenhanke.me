---
title: "Histopathological Cancer"
permalink: /pages/cancer/
layout: splash
excerpt: "Histopathological Cancer"
last_modified_at: 2019-04-03T15:15:09-04:00
toc: false
---
# Histopathological Cancer
1. Discover Duplicate: [Discover Duplicate](https://www.kaggle.com/c/histopathologic-cancer-detection/discussion/87367)
2. Discover Leak: [How to get LB 1.000](https://www.kaggle.com/c/histopathologic-cancer-detection/discussion/85424)
3. Invent WSI Normalization: [Better Way to Normalize Slides? WSI Normalization](https://www.kaggle.com/c/histopathologic-cancer-detection/discussion/87069)
4. Full Solution Summary: [How I drop from 0.9805 Private LB to 0.974 (113rd Solution)](https://www.kaggle.com/c/histopathologic-cancer-detection/discussion/87367)
{: .notice--primary}

# My LB 0.980 Solution Method
[Click Here to See Full](https://www.kaggle.com/c/histopathologic-cancer-detection/discussion/87367)

### Single Model
- Private: - 1. 0.9785
- Public: 0.9753
- Network: SEResNeXt50 (modified)
  - Input: 128*128 without maxpooling
  - BN, FC, ELU with 3 layers as a classifier
  - cat maxpooling and avg pooling
- Optimizer: SGD w/ momentum 0.9 (Adam is bad for me)
- LR Scheduler: Triangle lr with Decrease on Plateau
- Fold: 5 fold ensemble
- TTA: 16
- Validation Time Augmentation: 4
- Augmentation: from Kernels + 20% center focus by padding
- Dropout FC: 0.9
- Dropout CNN: 0.2
- Loss: BCE
- Activation: ELU
- Training: Freeze 3 epoch, train to 6th epoch is the best
- Tricks:
  - Snapshot Ensemble (about 2 each fold)
  - Shakeup simulation
  - 8x scheduled Augmentation for faster converge
  - Create a stable CV based on WSI
  - Max and Min lr based on the Paper (1 cycle + gradient warmup)
  - Faster loading with preprocessing images into .npy

### Pseudo labeling (with all probability)
I did not have time to train 5 fold pseudo  
But here are the scores with 4 tta ensemble:  
(They are generally better +0.008 on both public and private)  
Fold, Private, Public  
Fold1: 0.9765, 0.9763  
Fold2: 0.9787, 0.9668  

### Ensemble
Simple weighted average ensemble with 1 fold NasNet, DenseNet...  
Gives Private 0.9805, Public 0.9780  

## Ideas that did not work
Jigsaw puzzle (no unified patterns found)
Average, Geometric, Power ensemble are about the same
Postprocess (setting white 32*32 images in the middle to negative labels)

## Ideas not implemented
- Whole slide image normalization (no enough time + not sure if it works)
- Cheating with 1.000 LB (I don't think I am the first one who discovered this)
- WSI info from test set (I pretend I did not discover this)
- Distill with teacher and student
- Mixup
- Attention layers
- Inceptionv3, v4, X
- attention weighted average pooling
- global max pooling
- global std pooling

## Self Reflection
The only difference between 0.980 and 0.974 is my postprocess: setting white 32*32 images in the middle to negative labels. It gives all my model a stable LB improvement 0.0001. The change is so small that I did not even think whether I should use the postprocess strategy or not.

The mistake comes from not understanding the data: the patches are cropped from WSI. That means even if the image is empty, it may be a huge white cell inside the tumor!  

The other mistake is that I did not evaluate my postprocess through holdout set in CV. I trusted 1/5 LB too much. But looking back, I would still choose to use postprocess because the chance of getting an improvement of 0.0001 in Public LB and getting a huge drop 0.01 in Private LB is pretty low.  

See you in the next cv competition!  



---
