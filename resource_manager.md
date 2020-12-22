### tensorflow/core/framework目录中与resource相关的类。

- tensorflow中，resource可以被认为是不直接参与计算、
但在计算的各个step之间保存信息(stateful)、占据一定(0或更多)mem的对象。
比如获取data的iterator。

- ResourceBase: 位于resource\_mgr.h。是所有resource的基类。
继承自RefCounted。
只定义了虚函数MemoryUsed和纯虚函数DebugString。

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

- ScopedStepContainer: 位于resource\_mgr.h。
含有step id/container/cleanup function。
看起来是用来抽象resource manager中的特定container，
用于保存per step的resource。

- MakeResourceHandle: 位于resource\_mgr.h。
该函数用container name/resource name/device等信息生成一个ResourceHandle。
(ResourceHandle:通过一个ResourceHandle中储存的信息可以从
resource manager中找到相应的resource。)

- CreateResource/LookupResource/DeleteResource...:
位于resource\_mgr.h。
这些函数根据一个OpKernelContext和一个ResourceHandle来
创建/查找/删除...相应的resource。
其工作过程实际上是从OpKernelContext中获取resource manager，
由resource manager根据ResourceHandle中的信息找到对应的
resource并进行创建/查找/删除...。

- ResourceHandleOp: 位于resource\_mgr.h。
继承自OpKernel。用于生成ResourceHandle。

- ResourceDeleter: 位于resource\_mgr.h。
可以装入一个Variant中。
该类的对象装入一个Variant类型的Tensor后，
传递给resource deleter op，以便保证anonymous resource的销毁。
