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

- [ ] when node attrs are actually turned into computational values.
- [ ] Executor/graph-algorithm/stream-executor/graph-runner
- [ ] the interface between python and c++ in tensorflow. (and raw\_ops)
- [ ] python arg-captured function
- [ ] Graph rewriting in dataset optimization (optimize\_dataset\_op).
- [ ] ProcessFunctionLibraryRuntime::SendTensor/RescTensor. Will it merge multiple tensors?

- [ ] variables & optimizers
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

## Notes:

- Tensorflow的C++代码主要由各个类构成。
某一任务的完成往往需要一条或多条派生链，其中每个类的功能非常细化具体。

- Tensorflow的C++代码中有大量的指针，
管理这些指针的生命周期是代码的重要任务，也影响着tensorflow的一些设计逻辑。
