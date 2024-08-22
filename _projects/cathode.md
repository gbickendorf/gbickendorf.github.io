---
layout: page
title: Learning the Density of Physics
description: An easy introduction to resonant anomaly detection
img: assets/img/Designer.jpeg
importance: 4
category: science
---
## Synopsis
Here I will give an overview of the main points of resonant anomaly detection. I tried to keep it as simple as possible so that computer scientists get the gist of the physics content and physicists can follow the machine learning part.

## Introduction
In this writeup we will cover the inner workings of my <a href="https://journals.aps.org/prd/abstract/10.1103/PhysRevD.109.096031">publication on resonant anomaly detection</a>. I will start with the very basics, so even non-physicists will hopefully be able to follow along and understand the main points.

## Particle Physics
Let us start by setting the stage for modern particle physics. This chapter is aimed at non-physicists and can be skipped otherwise. We will give the proper physics terms for analogies in parentheses if necessary.

Imagine that you are given a bag of marbles of different colors and materials and are tasked with understanding the composition and grouping the marbles accordingly.
The physicist's way of tackling this is to smash the marbles (particles) at near-light speed to get an insight into the composition. When the marbles collide, they will break up and scatter the innards everywhere. These scattering products will be recorded by different cameras (detectors), such as a color camera or a radar gun that measures speed (calorimetric detector, tracker, etc.). From the angular and weight distributions (eta, phi, particle mass etc.) one might then infer the properties of the original marble. Many fine red shards might hint at a red glass marble, while a few black gooey specks might indicate that the original marble was a black rubber ball. The main point from here on is that the inference from the broken marble particles to the full marble is imperfect and has to rely on statistical methods.

To get closer to the original story, consider the slightly different classification task: Now we are tasked with deciding if there are *special* marbles (supersymmetric particles) among the ordinary marbles (known standard model particles). Ordinary marbles are just solid red, green and orange glass marbles and are really common. To make the statistical inference less complex, one usually groups the low-level detector observables into a few high-level features. This might be the speed of a group of co-moving shards (jet $$p_T$$ ), the sum of all shards ($$H_T$$) or the number of shards ($$N_J$$).

The aim is to build a statistical model out of these features that predicts whether the bag contains *special* marbles or not. The subsection, **The Dataset** boils down to why we chose which features to achieve the goal as efficiently as possible. The most important point is that the background is smooth in one feature, while the signal process produces a localized excess.

### The Dataset
In the publication, we demonstrate resonant anomaly detection (see below) for the first time for signal models, that are found on the tails of some distributions. A model that is commonly found at the tails of the $$p_T^{\rm miss}$$ distribution are supersymmetric models. Specifically, we will cover the pair production of heavy gluinos that each undergo a decay chain into a Z boson, a light neutralino and some soft jets. The neutralino escapes the detector and only leaves $$p_T^{\rm miss}$$ while the Z produces fat jets, whenever it decays hadronically.

For the non-specialists: our signal model produces mainly a lot of missing momentum ($$p_T^{\rm miss}$$) that forms the tail of the distribution and two localized bumps in two jet mass distributions.

A baseline selection is therefore (among others) to require large $$p_T^{\rm miss}$$, consequently large $$H_T$$ and two large jets with masses in the Z-mass region. To reach the large $$H_T$$ each collision has to contain many hard jets. To produce the required large amount of missing momentum, we need neutrinos, which escape the detector. Therefore, the main backgrounds will be Z, W and top plus jets. The Z and W bosons will almost never decay hadronically since the neutrino is required to satisfy the $$p_T^{\rm miss}$$ requirement. Consequently, the background is smooth in the jet mass since there is no resonance that decays hadronically. The signal, on the other hand, gets its $$p_T^{\rm miss}$$ by the neutralino, therefore many times the Z decays  visibly and leads to the bump.

The signal and background events are distributed as follows: For the moment, ignore the yellow lines. This will be explained below. 
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Mj1_woSR.jpg" title="MJ1" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/MJ2" title="MJ2" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
The left figure shows the resonant feature, that will take the place of $$m$$. We can define a signal region and a sideband that is signal enriched and depleted respectively. The right plot shows the other jet mass. This is the first of the $$x$$ features and will be useful for classification because the signal is again concentrated at the Z masss
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/HT.jpg" title="HT" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/PTMiss" title="PTMiss" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/TauJ1" title="TauJ1" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/TauJ2" title="TauJ2" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
These are the remaining $$x$$ features. Notably, the two leftmost plots show the signal being at the tail of the respective distribution. This is the property that lies at the core of this publication.

## Anomaly Detection
The knowledge (or assumption) of a localized bump with signal events while, for backgrounds, the distribution is smooth in the jet mass, gives us a huge advantage. Let us split the feature set into two subsets: let $$m$$ be the feature where the bump is, while $$x$$ contains all other features. 
The strategy for resonant anomaly detection is simple. When $$m$$ is far away from the Z-mass (namely in the sideband or SB), there will be almost exclusively background events. Therefore, here, the distribution of $$x$$ is also mostly the distribution of the background events. One might learn the distribution of $$x$$ here, therefore getting a template of what a background event should look like. This template can then be interpolated into the region near the Z-mass (namely the signal region, or SR) and compared to the events that are found there. If events do not look like the template at all, they are anomalous and might be interesting.
The specific technique is called <a href="https://journals.aps.org/prd/abstract/10.1103/PhysRevD.106.055006">CATHODE</a>. 

#### Step 1: Density Estimation
The first step sounds trivial. One learns the density of $$x$$ in the sideband $$p(x\vert m\in \rm SB)$$, i.e., where there are expected to be no signal events present. The remainder of this subchapter concerns how this is done and can, in principle, be skipped.

Let us take the physicists approach to machine learning. We want to find a function that solves a given problem. In this case, we want to find the function $$p(x\vert m\in \rm SB)$$ that depends on $$m$$ and $$x$$. We might simply construct a function that depends on possibly millions of parameters. The function is so complex that for some values of the parameters, the problem is solved to a sufficient degree. To quantify how good the solution is, one constructs a cost function. A low value signifies that the function behaves sufficiently closely to the desired solution.

General functions are not known to behave randomly. The trick we will use is known as a **normalizing flow**. If one wanted to model an N-dimensional random variable $$z$$, one could start at a N dimensional normal distribution and use its samples $$y$$ which are transformed by some function. The result follows no longer the normal distribution. One needs to find the transformation function $$T$$ such that $$T(y)$$ is distributed like $$z$$. Since it is hard to find a loss function for this direction, one inverts the expression to $$y\sim T^{-1}(z)$$. In other words: we want to find the inverse transformation, that changes the distribution of $$z$$ to be as closely as possible to that of $$y$$, i.e., a normal distribution. This can easily be quantified by the likelihood of each sample under the hypothesis of a normal distribution. This likelihood is numerically maximized, which is equivalent to learning the density in the reverse order. The neat part is that we can readily sample from the Gaussian, pass the samples through $$T$$ and get artificial samples of the target distribution.

#### Step 2: Interpolation
This step is trivial. Nowhere did we explicitly restrict the estimated density to the region where we learned it, i.e., the SB. Therefore, we are allowed to use the estimate even in the signal region. To get the distribution of $$m$$ right, one can use simple techniques such as kernel density estimation (KDE).

#### Step 3: Sampling
One next samples a desired number of values of $$m$$ using the KDE. These are then used to sample a large number of artificial events from the learned distribution of $$x$$ inside the signal region. If all goes well, these should follow the distribution of true background samples in this region without touching the real $$x$$ samples in the signal region at all.

#### Step 4: Classification
The last step is where the magic happens. Here, one trains a neural network to classify artificial and real events inside the signal region. If a real event can not be reliably classified as real, it looks just like an artificial event. The artificial event, in turn, is built to look like background events. Therefore, the real event is indistinguishable from a background event and is not anomalous. On the other hand, if the classifier is very certain that a given $$x$$ is a real event, it looks nothing like an artificial background event and is highly anomalous. One can therefore create a subset of anomalous events simply by observing how certain the classifier is that it is a real event. The anomaly score $$R$$ is simply the degree of certainty that an event is real, i.e. anomalous. This can be used for the selection.

The main takeaways are:
- The technique is completely data-driven. No simulation is needed.
- The only assumption about the signal is that it is localized in $$m$$.
- We did not specify in which subset of $$x$$ the signal is found. We could provide many features, to cover many signal models. The classifier will pick what is useful on its own.
- The opportunity cost is quite low. Picking $$m$$ and $$x$$ is easy. Handcrafting an analysis takes time.

## What is novel about our publication?
Up until our publication, the only models for which the technique has been shown to work contained fully localized features $$x$$. For our model, the density estimation has to be accurate mostly at large $$p_T^{\rm miss}$$, where there are few background events available. If it fails to model the behavior there, it might hide the signal events. Additionally, the classifier has to cope with these extreme values. Only if both steps work reasonably well, we get a useful performance.

## Results
In a realistic application, one would construct a statistical test, to judge whether there is an excess of anomalous events or not. Since our work is a proof-of-concept, we opt for a simpler figure, namely 
$$ Z = \frac{S}{\sqrt{B}}$$ with the number of signal events $$S$$ and background events $$B$$ that are classified as anomalous. This essentially describes how obvious a signal presence is among the background events. A value much larger than one would certainly be seen. Framed in this context, the goal is to raise the value of $$Z$$ from undetectable values before the application of CATHODE to larger, more obvious values.

Let us see how the technique works. To compare the results, we show numerous extrapolations of full proper analyses. Keep in mind that each full analysis most likely took a PhD student multiple years of work. This emphasizes how much less time we need to apply CATHODE.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/ZZ-1.jpg" title="Performance ZZ model" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/HH-1.jpg" title="Performance HH model" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
The x-axis is the gluino mass, while the y-axis is $$Z$$. There are fewer signal events at higher gluino masses, simply because the production process is rarer.
The orange line denotes in both graphs the value of $$Z$$ before any use of the anomaly score. The blue lines are what can be expected by an application of CATHODE on real data that uses the anomaly assignments. CATHODE indeed massively boosts the value of $$Z$$ from hopeless values to definitely detectable values for a wide region of gluino mass. So it definitely works for signals at the tails of signal distributions. The vertical bars are the rescaled excluded gluino masses of the full analyses (<a href="https://arxiv.org/abs/2008.04422">left</a>, <a href="https://arxiv.org/abs/2201.04206">right</a> ) which corresponds roughly to $$Z=1.645$$. Comparing this to the blue line, we are only slightly less sensitive. Keep in mind that both are full analyses that need each be built from the bottom up. One can not simply modify the analysis of the left plot slightly to get the right result. With CATHODE, one only needs to change the signal region and let the GPU handle the rest. The enormous time savings make CATHODE definitely worth the risk of missing this tiny additional exclusion.


<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/HH-HZ-ZZ-1.jpg" title="Performance HH-HZ-ZZ model" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/2000GeV_MH-1.jpg" title="Performance heavy Higgs model" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
The left plot is a model where both decays to Z-bosons and Higgs bosons occur. This time, our approach is even better than the full <a href="https://arxiv.org/abs/1712.08501">analysis</a>. Lastly, on the right, we show the results of a model where the decay results in a new heavy Higgs boson of mass $$m_H$$ that is predicted by supersymmetric models. This is completely un-searched for by classical analyses. With CATHODE, it is easily included with just a bit of computational resources.

## Concluding Remarks
I hope to have convinced you that resonant anomaly detection methods have a place in physics analyses going forward. We have covered how searching for the *special* events can be broken down to fitting a large number of parameters, only using real data. The last point is important, because simulated events have to be taken with a grain of salt. Avoiding simulations completely makes the method more robust. 

Using CATHODE, we can find anomalous events with minimal opportunity cost. Running the model repeatedly is cheap, while constructing a classical analysis is expensive and only leads to negligibly better results. Additionally, we do not even need to know what the signal model looks like. One could simply pass some large number of features $$x$$ that are most descriptive. As long as there is a difference in distribution between signal and background events in some subset of these features, the technique will work.




