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

- ContainerInfo: 位于resource\_mgr.h。
当一个node需要创建和使用resource时，ContainerInfo类帮助node决定
resource在resource manager中的container/name。
ContainerInfo还会判断resource是node私有的还是会被share。

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
该对象在析构时，会调用ResourceMgr::Delete(并忽略error)。
ResourceDeleter类的对象能装入一个Variant类型的Tensor。
ResourceDeleter类的对象能用于保证anonymous resource的销毁(是一种冗余保证措施)，
一般来说，anonymous resource由专门的destruction op负责销毁，
而这个op会接受一个封装了resource deleter的variant tensor作为input之一，
resource销毁的具体情况分为两种：情况1，如果destruction op能够正常运行，
那么op运行前，作为input的variant tensor尚未被销毁，tensor中的deleter尚未执行，
因此resource仍存在于resource manager中，而随着op的执行，resource被删除销毁，
当作为op input的variant tensor随后被销毁时，resource deleter被执行，
此时resource已经不存在，但deleter会忽略这个not found错误；
情况2，如果destruction op由于某种错误未能执行，
那么作为op input的variant tensor会被销毁，
tensor中的resource deleter被执行，从而resource被删除销毁。

- Anonymous Resource: 匿名resouce。
对于创建命名resource的op kernel，kernel不仅把resouce添加到resource manager，
还会自己保存并Ref一份指针(因而resource至少有2个引用计数)，
当kernel析构时，会Unref自己保存的指针，并视情况从resource manager中删除resouce。
而对于创建匿名resource的op kernel，kernel仅仅把resource添加到resource manager，
自己不会保存resource指针，特别是，当kernel析构时，
也不会从resource manager中删除resource。
不过，创建匿名resource的op kernel除了返回resource handle，
一般还能够返回一个封装了ResourceDeleter对象的Variant tensor，
这个ResourceDeleter对象能够从resource manager中删除相应resource。

- ResourceOpKernel: 位于resource\_op\_kernel.h。
模板类，继承了OpKernel。该类保存有一个resource指针。
该类有纯虚函数CreateResource。
该类的Compute函数会生成相应resource的ResourceHandle，
当resource指针为空时，还会首先创建resource(通过调用CreateResource)，
并对resource进行Ref。
该类析构时会对resource进行Unref，并根据需要从resource manager中删除resource。
