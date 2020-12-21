## Tensorflow Development Wiki

Tensorflow源码阅读笔记。
目标是了解Tensorflow的架构，及其比较重要的功能模块。
可能包含错误和表述不清的地方。

## To Do List:

- [ ] How op/kernel works from python layer to c++ layer.
How arguments are passed.

- [ ] The relationship between kernel/op/graph/node/function.
And how the graph built/placed.
And how the computation happens.

- [ ] What a dataset is and what an iterator is.
And how an iterator is generated.
And how to get from an iterator.

- [ ] resource handle
- [ ] resource manager
- [ ] device manager
- [ ] cancellation manager
- [ ] function and function library
- [ ] node and graph
- [ ] placer
- [ ] rendezvous
- [ ] grappler
- [ ] xla/jit

## Notes:

- Tensorflow的C++代码主要由各个类构成。
某一任务的完成往往需要一条或多条派生链，其中每个类的功能非常细化具体。
