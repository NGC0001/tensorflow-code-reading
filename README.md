## Tensorflow Development Wiki

Tensorflow源码阅读笔记。
目标是了解Tensorflow的架构，及其比较重要的功能模块。
可能包含错误和表述不清的地方。

## To Do List:

- [ ] How op/kernel works in python/c++.
How arguments are passed.

- [ ] kernel/op/graph/node/function.
how the graph built/placed.

- [ ] async op
- [ ] device manager
- [ ] cancellation manager
- [ ] function and function library
- [ ] placer
- [ ] rendezvous
- [ ] eager
- [ ] grappler
- [ ] xla/compiler
- [ ] stream-executor

## Notes:

- Tensorflow的C++代码主要由各个类构成。
某一任务的完成往往需要一条或多条派生链，其中每个类的功能非常细化具体。
