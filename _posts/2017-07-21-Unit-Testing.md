---
layout: post
title: Don't forget to unit test!
---
Unit testing is an integral part of the life cycle of any software project; this is especially true when creating complex models with TensorFlow. My previous article briefly discussed the 'summary' capabilities of [TensorBoard](https://www.tensorflow.org/get_started/summaries_and_tensorboard?lipi=urn%3Ali%3Apage%3Ad_flagship3_pulse_read%3BhDaHDgrsRBOU7QwE13JCOA%3D%3D) which allow us to visualize the evolution of variables of interest, such as training and cross-validation costs, during the training process. Before I continue with this discussion, I'd also like to point out that TensorBoard also has a computation graph visualization tool that looks something like this:

![autoencoder](/images/autoencoder.png)

This is a computation graph I was working on earlier -- it's a simple autoencoder network. The graph visualizer lets you zoom in on and expand subsections of the graph, and this can be very useful when you're trying to make sure everything is wired together correctly.

If TensorBoard isn't your cup of tea, there is a testing module built into the TensorFlow framework which allows you to hard-code unit tests for whatever you're working on ([`TestCase` class](https://www.tensorflow.org/api_docs/python/tf/test/TestCase?lipi=urn%3Ali%3Apage%3Ad_flagship3_pulse_read%3BhDaHDgrsRBOU7QwE13JCOA%3D%3D) in TensorFlow).

I highly recommend checking out the documentation in the link I provided above, but here's a very simple use-case: Imagine you've just completed your graph building module; you'd like to ensure all your ops are in the graph collection, and that all the tensors (`Variable` objects) are initializing correctly. Now would be a good time to set up a unit testing class.

```python
# graph_test.py

import tensorflow as tf
from my_modules import graph_builder

''' Inherit from the TestCase class! '''
class GraphBuildingUnitTest(tf.test.TestCase)

    def initAll(self):
        '''
        Here, you should define any variables you might﻿
        need to build your computation graph. Hyperparameters
        are a good example.﻿﻿
        '''
        self.learning_rate = 0.005
        self.batch_size = 100
        self.training_epochs = 15﻿﻿﻿﻿﻿﻿
        # ...
        # ...

    def testGraphBuild(self):
        graph = graph_builder.build(self.learning_rate,
                                    self.batch_size,
                                    self.training_epochs)

        ''' Call some unit testing functions from TestCase class'''
        self.assertIsInstance(graph, tf.Graph)
        self.assertIn('train_op', graph.get_all_collection_keys())
        self.assertIn('inputs', graph.get_all_collection_keys())﻿
        self.assertIn('labels', graph.get_all_collection_keys())
        # as many tests as you want... see documentation link

if __name__ == '__main__':
    tf.test.main() # run all unit tests﻿
```

If all goes well, this should output something like:

```
$ python graph_test.py
..
----------------------------------------------------------------------
Ran 2 tests in 0.145s

OK
```

Otherwise:

```
$ python graph_test.py
F.
======================================================================
FAIL: testBuild (__main__.GraphBuildingUnitTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "graph_test.py.py", line 31, in testBuild
    self.assertIn('global_step_var', graph.get_all_collection_keys())
AssertionError: 'global_step_var' not found in ['train_op',
 'inputs', 'labels']

----------------------------------------------------------------------
Ran 2 tests in 0.142s

FAILED (failures=1)
```

Looks like we have an error at line 31 of our unit test. Somehow our global step variable isn't getting added to the graph collection. Now we now where to go hunt this error down.

This was a very basic example, and the `TestCase` class has a lot more functionality that is very powerful for unit testing your TensorFlow models, but I hope this was a good introduction.

Best of luck!
