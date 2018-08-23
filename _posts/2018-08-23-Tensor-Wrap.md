---
layout: post
title: tensor-wrap
---
Hello (again), World! This is my first post in quite a while... And it's yet again about TensorFlow.

I started using TensorFlow about a year and a half ago, and while I don't claim to be an expert user of the framework,
I've had the chance to explore a good amount of the API in both personal and work-related projects.

The most tedious part of the TensorFlow learning curve for me was understanding the fundamental pillars of
TensorFlow applications - namely the creation of Tensors and Operations, and their "execution" in a Session. Only
after studying well documented examples put together by the TensorFlow development team which really encapsulated 
the data pipeline did I feel comfortable creating my own applications from scratch.

After replicating many trivial to intermediate-level examples, I finally got to design my own full-fledged
deep-learning application during my internship at B&D. Being a software developer at heart, I naturally wanted my
application to be robust and scalable, but also easy to understand for new developers who were potentially going to 
continue my work. I ended up writing a high level wrapper API around TensorFlow which would allow for the easy creation of simple,
feed-forward neural networks. The API was no Keras or TFLearn by comparison, but it still abstracted away a
large portion of the low-level op creation you typically find in bare-bones TensorFlow examples online.

As both the Python programming language and the TensorFlow framework evolve, I decided recently to start developing
a more robust version of my API, which I am calling `tensor-wrap`. My goal is to make it easier for deep-learning beginners to 
use TensorFlow without having to immediately delve into writing low-level tensor operations. I'm currently in the process of 
migrating the work I already completed during my internship, tailoring the API to Python 3.6 and TensorFlow r1.10.
Once that is done, the next steps will hopefully include making more robust derived network models that not only use the more
advanced portions of the TensorFlow framework, but also plug-and-play with the well established high-level API wrappers
like Keras.

Is `tensor-wrap` going to be the next Keras, TFLearn, or any of the other widely used TensorFlow-based libraries? 
Absolutely not - at least not for now. However, I invite deep learning enthusiasts to both
critique and contribute to `tensor-wrap` in order to better learn the ins and outs of TensorFlow! For now, I'll simply
supply the link to the current working branch, but I will hopefully get around to posting a more detailed UML digram
that encapsulates what I'm trying to throw together.

[Here](https://github.com/doneill612/tensor-wrap/tree/v1.0) is the link to the `v1.0` branch. Thanks for reading!
