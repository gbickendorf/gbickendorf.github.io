---
layout: page
title: Vision Transformers for Particle Physics
description: Using recent vision transformer improvement to see new particles
img: assets/img/avg.jpg
importance: 8
category: science
related_publications: false
---
## Synopsis
Here, we summarize the key points of our <a href="https://arxiv.org/pdf/2406.03096">publication</a>. We discuss the benefits of using machine learning to identify our signal model and explain how images are defined to leverage computer vision techniques. We demonstrate that there is no need to reinvent the wheel in machine learning; instead, significant improvements can be achieved by applying modern models developed by researchers at Google. These improvements are so substantial that they should survive even with a full application.

## Introduction
In this section, we outline the main points of our publication, <a href="https://arxiv.org/pdf/2406.03096">Learning to see R-parity violating scalar top decays</a>. In high-energy particle physics, our goal is to understand the fundamental physics underlying the phenomena observed in experiments. The field is highly advanced, enabling us to accurately predict almost all phenomena observed at the LHC. However, determining whether observed collisions stem from known processes or something new is a complex task that depends on precise statistical modeling. To enhance statistical significance, we must filter out known events while retaining as many potentially interesting events as possible. For some signal models, this is particularly challenging because their signatures in traditional observables are weak. Consequently, more sophisticated methods are required, moving beyond classical, hand-crafted observables.


## Signal model
The signal model of interest belongs to the class of supersymmetric extensions of the Standard Model. The process, illustrated in the accompanying Feynman diagram, can be described as follows: two protons are accelerated to high energies and then collided head-on. The energy from this collision is used to create additional particles, specifically two scalar top quarks (stops). These stops, which have masses between 700 and 1200 GeV, behave similarly to ordinary top quarks but eventually decay into one ordinary top quark and one neutralino. The neutralino, weighing between 100 and 500 GeV, then rapidly decays into three ordinary quarks.
<div class="row justify-content-sm-center">
  <div class="col-sm-5 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/stopfeyn.jpg" title="Stop pair production Feynman diagram" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
Since the signal results in the production of two top quarks, which are relatively easy to isolate, preselections are made to specifically target these top quarks. After preselection, the only significant background process is the pair production of top quarks. The challenge then lies in determining whether a given event is due to stop-pair or top-pair production.

This process is challenging to detect because it is fully hadronic, and the LHC, being a hadron collider, has many known processes that can mimic this signal. Moreover, the signature is quite subtle, typically resulting in two top quarks and two wide jets from the neutralinos. Given the abundance of top quarks at the LHC and the difficulties in reconstructing the wide jets, which are prone to significant errors, a more refined technique is required to distinguish the signal from ordinary physics.

## From Collisions to Images
One approach to addressing this problem is to closely examine what the detector perceives as a jet. Jets are clusters of particles that ideally originate from the same initial particle. When these particles interact with the detector, they deposit energy, which the detector measures. In this sense, a detector functions similarly to a camera. The spatial dimensions of a detector image correspond to angular positions, while the channels of a standard image (typically red, green, and blue) are replaced by subdetectors in this case, such as the hadronic or electromagnetic calorimeter or tracker. The brightness or pixel intensity represents the deposited energy. By analyzing these images, one can attempt to predict the production mechanism, which forms the core of our publication. The average signal and background images are displayed below.
<div class="row justify-content-sm-center">
  <div class="col-sm-5 mt-5 mt-md-0">
    {% include figure.liquid path="assets/img/avg.jpg" title="Average jet image" class="img-fluid rounded z-depth-1" %}
  </div>
</div>

## Machine learning and computer vision
Classifying images is a well-known challenge in machine learning, specifically in computer vision. One of the most successful and widely used methods for this task is the Convolutional Neural Network (CNN). A CNN works by comparing a fixed-size local patch of an image with a learnable filter, performing this comparison across the entire image. The resulting similarity scores create a new image, known as a feature map. For example, when distinguishing between images of cats and dogs, a CNN might initially focus on detecting edges, which highlight the most significant parts of the image. Smooth areas without much variation typically provide less useful information. Consequently, the CNN is likely to first learn a filter that enhances edges in the feature maps. By passing these feature maps through successive convolutional layers, the network can gradually identify more complex features, such as cat-like ears or eyes, which are crucial for recognizing the animal.

The effectiveness of CNNs largely stems from their focus on local information, allowing them to deliver impressive results with relatively low computational costs. However, recent advancements in computational power at consumer-level prices have enabled the development of more sophisticated techniques. One such innovation is the attention mechanism, which, although computationally demanding, allows for the use of global information—where every input is connected to all other inputs without any inherent ordering. This mechanism is central to the functionality of transformers, a technology that has gained significant attention, especially with the rise of models like ChatGPT (where the 'T' stands for transformer). Transformers have revolutionized natural language processing, and their success has sparked interest in applying them to image processing as well.


We employ two models, <a href="https://arxiv.org/abs/2106.04803">CoAtNet</a> and <a href="https://arxiv.org/abs/2204.01697">MaxViT</a>, both of which incorporate transformers. Studies introducing these models have demonstrated significant performance improvements over CNNs, particularly with large datasets. Given that physicists excel at working with large datasets, we train both models on jet images and compare their performance against CNNs.

## Training
Our Monte Carlo simulations, which model proton collisions, are highly accurate, allowing us to train on simulated images effectively. To ensure that the model distinguishes between signal and background jets without being influenced by unrelated correlations, we ensure that the signal jets originate from a direction close to the truth-level neutralino. We normalize the pixel intensity by dividing by the maximum value, thereby removing scale-related information, which we reintroduce by feeding the jet mass into the classification network. This is achieved by appending the mass value to the output of the vision component of the model. In total, we train on nearly 6 million images.

## Results
In a practical application, these models would typically serve as inputs to a more sophisticated statistical analysis. However, our proof of concept is focused on demonstrating the models' effectiveness, so we quantify performance using the metric $$\epsilon_S/ \sqrt{\epsilon_B}$$, where $$\epsilon_S$$ and $$\epsilon_B$$ represent the well-known true-positive and false-positive rates, respectively, after applying a threshold to the output neuron.


The performance results for the test set are illustrated below. The labels AK08, AK10, and AK14 correspond to the allowed radii of the respective jets, which essentially represent progressively larger active areas of the detector image. 
<div class="row justify-content-sm-center">
  <div class="col-sm-6 mt-5 mt-md-0">
    {% include figure.liquid path="assets/img/perf1.jpg" title="SIC of all models" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
It is evident that the largest jet images yield the most powerful classifications, with both transformer-based models significantly outperforming the traditional CNN. The improvements observed in the original publications translate well into the context of particle physics, which is a key result of our study.

To assess the raw performance of these models, we conducted a simple mock analysis. In this analysis, we combined additional classical features with the predictions from the largest AK14 jets and fed them into a gradient boosted decision tree. Both background and signal events were normalized to the expected event counts used in current analyses by the CMS and ATLAS collaborations. We then compared the 95% confidence level (C.L.) excluded masses for each of the three computer vision models against those obtained from a decision tree that only used classical features.
<div class="row justify-content-sm-center">
  <div class="col-sm-6 mt-5 mt-md-0">
    {% include figure.liquid path="assets/img/perf2.jpg" title="Entension of the excluded stop mass range" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
The CNN demonstrates relatively weak performance, contributing only marginally to expanding the excluded parameter space. In contrast, both transformer-based models show a significant advantage, excluding stop masses that are 60 GeV to even 100 GeV higher. This substantial improvement highlights the potential of simply switching to a modern transformer-based architecture to achieve far superior results.

## Conclusion
In conclusion, our study demonstrates that modern transformer-based models significantly outperform traditional CNNs in classifying jet images, leading to a substantial increase in the excluded stop masses. This improvement underscores the value of adopting cutting-edge machine learning techniques in high-energy physics. For a more detailed exploration of our methods and findings, we encourage you to read the full publication.