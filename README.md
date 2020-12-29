## Tensorflow Development Wiki

- Tensorflow源码阅读笔记。
目标是了解Tensorflow的架构，及其比较重要的功能模块。
可能包含错误和表述不清的地方。

- 当前代码版本的时间节点是:
Pre-Release TensorFlow 2.4.0-rc4 已经发布。
Release TensorFlow 2.4.0 即将发布。

## To Do List:

- [ ] multi-device iterator / generator dataset

- [ ] function and instantiation and function library
- [ ] kernel/op/graph/node/function. how a graph built/placed/runned.
- [ ] the interface between python and c++ in tensorflow.

- [ ] device manager
- [ ] placer
- [ ] eager mode
- [ ] Tensor vs EagerTensor:
What hapens when a Tensor returned from a function
is converted into EagerTensor in eager mode.
- [ ] async op
- [ ] function retracing
- [ ] AutoDiff and GradientTape (in graph and eager)
- [ ] AutoGraph
- [ ] cancellation manager
- [ ] rendezvous
- [ ] grappler
- [ ] xla/compiler/GraphOptimization
- [ ] stream-executor

## Notes:

- Tensorflow的C++代码主要由各个类构成。
某一任务的完成往往需要一条或多条派生链，其中每个类的功能非常细化具体。
