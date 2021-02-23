### tensorflow中的resource。

- tensorflow中，resource可以被认为是不直接参与计算、
但在计算的各个step之间保存信息(stateful)、占据一定(0或更多)mem的对象。
比如获取data的iterator就是resource。

- 各resource放入resource manager中进行管理。
每个device上都有各自的resource manager。

- ResourceHandle对象是从resource manager中获取resource的句柄，
利用一个ResourceHandle对象中储存的信息，
可以从resource manager中检索到相应的resource。
而ResourceHandle对象能够被放入Tensor中，
这也就间接使得resource可以通过Tensor进行传递。

### tensorflow/core/framework目录中与resource相关的类。

- ResourceBase: 位于resource\_mgr.h。是所有resource的基类。
继承自RefCounted。
只定义了虚函数MemoryUsed和纯虚函数DebugString。

- ResourceMgr: 位于resource\_mgr.h。用于管理resource的resource manager。
ResourceMgr使用一个字典std::unordered\_map\<string,
Container\*\>储存各个Container。
其中Container也是字典
std::unordered\_map\<Key, ResourceMgr::ResourceAndName,
ResourceMgr::KeyHash, ResourceMgr::KeyEqual\>。
ResourceMgr::ResourceAndName中含有resource name和RefCountPtr\<ResourceBase\>智能指针。
而Key是std::pair\<unint64, StringPiece\>，
两个元素分别用于表示resource的type index、resource name。
可以看出，resource manager使用两级字典来管理resource(resource以智能指针的形式存放)，
即container name-\>resource key-\>resource。
ResourceMgr类有函数Lookup，该函数根据参数找到指定的resource后，
会返回resource的指针prsc，并且会调用prsc-\>Ref() (RefCounted::Ref)，
使得这个resource的引用数加1，
因而当Lookup的caller不再使用这个resource的时候，
应当调用prsc-\>Unref() (RefCounted::Unref)，使得这个resource的引用数减1。
ResourceMgr类有函数Create，该函数把一个resource的指针存入两级字典内，
但该函数并不改变这个resource的引用数(RefCounted对象在创建时已经自带1个引用数)。
ResourceMgr类有函数LookupOrCreate，
该函数将resource的引用数加1(不论最终是lookup还是create)。
ResourceMgr类有函数Delete，该函数把一个resource的指针从两级字典内删除，
并通过ResourceMgr::ResourceAndName里RefCountPtr对象的析构
来把这个resource的引用数减1。

- ScopedStepContainer: 位于resource\_mgr.h。
含有step id/container/cleanup function。
看起来是用来抽象resource manager中的特定container，
用于保存per step的resource。

- ResourceHandle: 位于resource\_handle.h。
该类的对象可以放入DT\_RESOURCE类型的Tensor。
包含有device/container/name/dtypes\_and\_shapes等信息，
通过这些信息可以从resource manager中检索到相应的resource。
一般地，如果某个OpKernel需要使用一个resource，
那么该OpKernel的Compute函数的input tensors中，
会有一个含有ResourceHandle的DT\_RESOURCE类型的Tensor，
利用该ResourceHandle可以从resource manager中检索到相应的resource。

- MakeResourceHandle: 位于resource\_mgr.h。
该函数用container name/resource name/device等信息生成一个ResourceHandle。

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
该类含有一个ResourceDeleter::Helper对象。
ResourceDeleter::Helper对象含有一个ResourceHandle和一个ResourceMgr\*，
该对象在析构时，会调用ResourceMgr::Delete。
ResourceDeleter类的对象装入一个Variant类型的Tensor后，
可以传递给resource deleter op，以便保证anonymous resource的销毁。

- ResourceOpKernel: 位于resource\_op\_kernel.h。
模板类，继承了OpKernel。该类保存有一个resource指针。
该类有纯虚函数CreateResource。
该类的Compute函数会生成相应resource的ResourceHandle，
当resource指针为空时，还会首先创建resource(通过调用CreateResource)，
并对resource进行Ref。
该类析构时会对resource进行Unref，并根据需要从resource manager中删除resource。
