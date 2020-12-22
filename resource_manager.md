### tensorflow/core/framework目录中与resource相关的类。

- tensorflow中，resource可以被认为是不直接参与计算、
但在计算的各个step之间保存信息(stateful)、占据一定mem的对象。

- ResourceBase: 位于resource\_mgr.h。是所有resource的基类。
继承自RefCounted。
只定义了虚函数MemoryUsed和纯虚函数DebugString。

- ScopedStepContainer: 位于resource\_mgr.h。
含有step id/container/cleanup function。
看起来是用来抽象resource manager中的特定container，
用于保存per step的resource。

- ResourceMgr: 位于resource\_mgr.h。用于管理resource。
ResourceMgr使用一个字典std::unordered\_map\<string,
Container\*\>储存各个Container。
其中Container也是字典
std::unordered\_map\<Key, ResourceMgr::ResourceAndName,
ResourceMgr::KeyHash, ResourceMgr::KeyEqual\>。
ResourceMgr::ResourceAndName中含有resource name和ResourceBase指针。
而Key是std::pair\<unint64, StringPiece\>，
两个元素分别用于表示resource的type index、resource name。
可以看出，resource manager使用两级字典来管理resource，
即container name-\>resource key-\>resource。

- MakeResourceHandle: 位于resource\_mgr.h。
该函数
