## Tensorflow Development Wiki

- Tensorflow源码阅读笔记。
目标是了解Tensorflow的架构，及其比较重要的功能模块。
可能包含错误和表述不清的地方。

- 当前代码版本的时间节点是:
Pre-Release TensorFlow 2.4.0-rc4 已经发布。
Release TensorFlow 2.4.0 即将发布。

- 笔记中提及"类A含有类B"的时候 ，不区分类A的对象是含有类B对象的实体，
还是含有指向类B对象的指针。

## To Do List:

- [ ] Executor/graph-algorithm/stream-executor/graph-runner
- [ ] How pybind11 works.
- [ ] python arg-captured function
- [ ] Graph rewriting in dataset optimization (optimize\_dataset\_op).
- [ ] ProcessFunctionLibraryRuntime::SendTensor/RescTensor.
Will it merge multiple tensors?
Does it use rendezvous?

- [ ] optimizers
- [ ] device manager
- [ ] thread pool
- [ ] eager mode
- [ ] Nccl/Collective
- [ ] C++ TensorHandle vs python EagerTensor.
What hapens when a Tensor returned from a function
is converted into EagerTensor in eager mode.
- [ ] async op
- [ ] AutoDiff and GradientTape (in graph and eager)
- [ ] AutoGraph
- [ ] GraphOptimizer/grappler/xla/jit/compiler
- [ ] cancellation manager
- [ ] rendezvous
