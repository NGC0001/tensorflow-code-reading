### tensorflow/core/framework目录中对dataset/iterator相关接口的定义。

- DatasetContext: 位于dataset.h。是DatasetBase的构造参数。
DatasetContext的构造方法之一通过OpKernelContext构造而来。

- DatasetBase: 位于dataset.h。
继承了引用计数类RefCounted。
仅包含有type\_string\_和node\_name\_。
有const方法MakeIterator，有纯虚函数MakeIteratorInternal。
有纯虚函数output\_shapes/output\_dtypes。
有函数Cardinality。

- DatasetVariantWrapper: 位于dataset.cc。
包含有DatasetBase指针，并对DatasetBase进行引用计数。
该类可以封装入一个Variant对象，从而使得DatasetBase能封装入Variant对象
(而Variant对象可以装入DT\_VARIANT类型的Tensor)。

- WrappedDatasetVariantWrapper/WrapDatasetVariantOp/UnwrapDatasetVariantOp:
位于dataset.cc。
WrapDatasetVariantOp把一个tensor封装为一个variant后，
放入一个variant类型scalar形状的tensor，
UnwrapDatasetVariantOp则进行反向操作。

---

- DatasetOpKernel: 位于dataset.h。继承自OpKernel。
有final方法Compute，有纯虚函数MakeDataset。
有子类UnaryDatasetOpKernel/BinaryDatasetOpKernel。

---

- IteratorContext: 位于dataset.h。是一个缩减版的OpKernelContext。
DatasetBase::MakeIterator和IteratorBase::GetNext都有该类作为参数。

- IteratorBase：位于dataset.h。有纯虚函数GetNext。
含有一个shared\_ptr\<model::Node\>。
有纯虚函数output\_shapes/output\_dtypes。

- DatasetBaseIterator: 位于dataset.h。继承自IteratorBase。
通过其嵌套类BaseParams包含DatasetBase指针，并对DatasetBase引用数加1。
有纯虚函数GetNextInternal，该函数返回类型为Status，
该函数参数有三个，分别是IteratorContext\* ctx,
std::vector\<Tensor>\* out\_tensors,
bool\* end\_of\_sequence。

- DatasetIterator: 位于dataset.h。继承自DatasetBaseIterator。
模板类，表示一个与具体类型的dataset关联的iterator。

### tensorflow/core/ops目录中与dataset/iterator相关的一些Op的注册。

- dataset\_ops.cc/experimental\_dataset\_ops.cc:
包含了一些与Dataset/Iterator相关的算子的注册(REGISTER\_OP)。

### tensorflow/core/kernels/data目录中几个DatasetOpKernel的具体实现。

- TensorDatasetOp: 位于tensor\_dataset\_op.h。继承自DatasetOpKernel。
有继承自DatasetBase的private嵌套类TensorDatasetOp::Dataset。
TensorDatasetOp有函数MakeDataset，该函数接受一个OpKernelContext对象，
用该OpKernelContext对象构造一个TensorDatasetOp::Dataset对象。

- TensorDatasetOp::Dataset: 位于tensor\_dataset\_op.cc。
该类储存有若干Tensor。
有继承自DatasetIterator\<TensorDatasetOp::Dataset\>的
private嵌套类TensorDatasetOp::Dataset::Iterator。
TensorDatasetOp::Dataset有函数MakeIteratorInternal，
该函数返回TensorDatasetOp::Dataset::Iterator对象。

- TensorDatasetOp::Dataset::Iterator: 位于tensor\_dataset\_op.cc。
其函数GetNextInternal一次性返回TensorDatasetOp::Dataset对象所储存的tensors。

---

- TFRecordDatasetOp: 位于tf\_record\_dataset\_op.h。继承自DatasetOpKernel。
该类的函数MakeDataset构造一个TFRecordDatasetOp::Dataset对象。

- TFRecordDatasetOp::Dataset: 位于tf\_record\_dataset\_op.cc。
存有一个文件名列表。

- TFRecordDatasetOp::Dataset::Iterator: 位于tf\_record\_dataset\_op.cc。
该类保存有一个RandomAccessFile类型的文件handle，
和一个io::SequentialRecordReader类型的reader。
该类的GetNextInternal函数会用reader从文件中读取一条内容(string类型)作为结果。

---

- BatchDatasetOp: 位于batch\_dataset\_op.h。
继承自UnaryDatasetOpKernel。
该类的函数MakeDataset接受一个OpKernelContext对象和
一个DatasetBase对象(上游DatasetBase对象)，
并用来构造一个BatchDatasetOp::Dataset。

- BatchDatasetOp::Dataset: 位于batch\_dataset\_op.cc。
其构造函数会Ref构造参数中的DatasetBase对象(上游DatasetBase对象)，
而其析构函数会Unref这个DatasetBase对象。

- BatchDatasetOp::Dataset::Iterator: 位于batch\_dataset\_op.cc。
该类的Initialize函数会保存上游DatasetBase的Iterator。
该类的GetNextInternal函数会从上游Iterator对象中获取一定数量的结果。

---

- MapDatasetOp: 位于map\_dataset\_op.h。继承自UnaryDatasetOpKernel。
该类在构造时，会从构造参数OpKernelConstruction对象中获取一些信息，
比如即将创建的MapDatasetOp::Dataset对象的
output\_shapes/output\_types/func\_metadata等。
该类的MakeDataset函数会创建CapturedFunction对象，
并利用该对象和其他一些信息构造一个MapDatasetOp::Dataset。

- MapDatasetOp::Dataset: 位于map\_dataset\_op.cc。
保存有一个CapturedFunction对象。

- MapDatasetOp::Dataset::Iterator: 位于map\_dataset\_op.cc。
该类的Initialize函数会保存上游Iterator，
同时该函数会把MapDatasetOp::Dataset中的CapturedFunction对象
实例化为一个InstantiatedCapturedFunction对象。
该类的GetNextInternal函数会从上游iterator获取tensors，
然后调用instantiated\_captured\_func，得到最终结果。

---

- InterleaveDatasetOp: 位于interleave\_dataset\_op.h。
继承自UnaryDatasetOpKernel。

- InterleaveDatasetOp::Dataset: 位于interleave\_dataset\_op.cc。
保存有一个CapturedFunction对象。

- InterleaveDatasetOp::Dataset::Iterator: 位于interleave\_dataset\_op.cc。
该类保存有一个InstantiatedCapturedFunction对象、
上游Iterator、一个IteratorBase列表。
该类的GetNextInternal函数从上游iterator获取元素，
再调用instantiated\_captured\_func
从所获元素(得到dataset,再从所得dataset)得到iterator，
所得iterator被放入IteratorBase列表中，
GetNextInternal函数会轮流对这些iterator调用iterator.GetNext，返回所得结果。

---

- OptimizeDatasetOp: 位于optimize\_dataset\_op.h。
继承自UnaryDatasetOpKernel。
该类比较特殊，工作过程与普通的DatasetOpKernel不同。
该类把一个dataset(包括其上游dataset)的pipeline构建成一张图，并对图进行优化。
当在python中调用iter(dataset)时，会调用dataset.\_apply\_options()。
而dataset.\_apply\_options()则可能会用到OptimizeDatasetOp。

### tensorflow/core/kernels/data目录中将Iterator作为resouce进行管理。

- IteratorResource: 位于iterator\_ops.h。继承了ResourceBase。
含有DatasetBaseIterator。储存有output types/output shapes/
function library definition/process function library runtime/
function library runtime/function handle cache/
device manager/resource manager/cancellation manager等。
有函数SetIteratorFromDataset，
该函数接受一个OpKernelContext指针、一个DatasetBase指针，
并调用DatasetBase::MakeIterator来生成iterator。
有函数GetNext，该函数调用DatasetBaseIterator::GetNext。

- IteratorHandleOp: 位于iterator\_ops.h。继承了OpKernel。
该类含有一个IteratorResource的指针。
该类的函数Compute会生成相应IteratorResource的ResourceHandle，
如果IteratorResource指针为空，
则还会先调用ResourceMgr::LookupOrCreate获得并Ref指针。
该类析构时，会在IteratorResource指针上进行Unref，
并根据情况调用ResourceMgr::Delete。

- AnonymousResourceOp: 位于dataset\_utils.h。模板类，继承了OpKernel。
有纯虚函数CreateResource。
该类的函数Compute会调用CreateResource创建resource，
然后生成该resource的ResourceHandle，
并根据需要生成该resource的ResourceDeleter。

- AnonymousIteratorHandleOp: 位于iterator\_ops.h。
继承了AnonymousResourceOp\<IteratorResource\>。
该类的函数CreateResource生成一个IteratorResource。
该类作用类似IteratorHandleOp，但生成的resource handle不会被share，
该类不会保存iterator resource指针(当然也不会Ref)。

- HybridAsyncOpKernel: 位于iterator\_ops.h。继承AsyncOpKernel。

- MakeIteratorOp: 位于iterator\_ops.h。继承HybridAsyncOpKernel。
该类的DoCompute函数从OpKernelContext中获取DatasetBase/IteratorResource，
而后调用IteratorResouce::SetIteratorFromDataset。

- IteratorGetNextOp: 位于iterator\_ops.h。继承HybridAsyncOpKernel。
该类的DoCompute函数从OpKernelContext中获取IteratorResource，
而后调用IteratorResouce::GetNext。

- IteratorToStringHandleOp: 位于iterator\_ops.h。继承OpKernel。
该类的Compute函数从OpKernelContext中获取一个resource handle，
并验证这个resource handle的确对应一个iterator resource，
随后调用ResourceHandle::SerializeAsString将resource handle序列化，
并装入一个string类型的Tensor。

- iterator\_ops.cc: 含有一些OpKernel的注册。
kernel IteratorHandleOp被注册到Op Iterator和Op IteratorV2。
AnonymousIteratorHandleOp被注册到AnonymousIterator/AnonymousIteratorV2。
MakeIteratorOp被注册到MakeIterator。
IteratorGetNextOp被注册到IteratorGetNext。
......

### tensorflow中dataset的工作模式。

- tensorflow的C++类中，所有派生自DatasetBase的类只表示某个dataset的类型、
该dataset的元素(Tensor)的类型、该dataset的元素的形状、该dataset的必要信息等，
真正能从dataset中"看到"一个个元素的，
是从该dataset获得、派生自DatasetBaseIterator的iterator。
iterator会一次次迭代出元素，dataset不参与迭代。
在元素的各次迭代之间，iterator是有状态的，而dataset可以是无状态的。

- 按照tensorflow的代码风格，
不同类型的dataset(DatasetBase的派生类)
由不同类型的datasetOp(DatasetOpKernel的派生类)得到。
某个datasetOp返回的dataset一般是该datasetOp的嵌套类。
从不同类型的dataset得到的iterator(DatasetBaseIterator的派生类)也不同。
某个dataset得到的iterator一般是该dataset的嵌套类。

- DatasetBase和DatasetBaseIterator做了足够的抽象。
某种类型的dataset可以是从其他dataset构建而来，
这样就可以建立一条dataset链，下游的dataset有上游dataset的指针，
下游dataset的iterator也会调用上游dataset的iterator。
反映在Python代码上，就是input pipeline，比如:
dataset.interleave(...).map(...).batch(...).prefetch(...)，
从上游到下游，分别是original dataset、
interleave dataset(由InterleaveDatasetOp构建而来)、
map dataset(由MapDatasetOp构建而来)、
batch dataset(由BatchDatasetOp构建而来)、
prefetch dataset(由PrefetchDatasetOp构建而来)。

- 在python中构建input pipeline时，对应的C++代码是创建了一条DatasetBase链。
当python中iter(dataset)时，C++代码主要做了两件事情。
一是调用XxxIteratorXxx算子(Op)，
在ResourceMgr中建立IteratorResource并获得它的ResourceHandle，
这一步是为了把iterator作为resource交由resource manager来管理，
但此时iterator resource并没有和dataset关联起来。
二是调用MakeIterator算子(Op，对应的kernel是MakeIterator)，
该算子的input包含一个ResourceHandle和一个DatasetBase，
该算子根据ResourceHandle获取IteratorResource，
并调用IteratorResource::SetIteratorFromDataset，
这将导致DatasetBase::MakeIterator被调用(
直接被调用的是dataset链上最下游的DatasetBase)，
得到的iterator被放入iterator resource中。

- 在python中对iterator进行迭代时，C++代码调用的是IteratorGetNext算子。

- 在tensorflow的python代码中，对dataset和iterator进行抽象的类分别位于
tensorflow/python/data/ops/dataset\_ops.py和
tensorflow/python/data/ops/iterator\_ops.py。
这两个文件中的代码最终会依赖于相应的C++算子来实现其功能，
而这些dataset/iterator相关的C++算子的调用接口则位于
tensorflow/python/ops/gen\_dataset\_ops.py，
因此dataset\_ops.py和iterator\_ops.py会大量调用gen\_dataset\_ops.py。

### MultiDeviceIterator。

- tensorflow/core/kernels/data/multi\_device\_iterator\_ops.cc。
类MultiDeviceIterator，该类继承了ResourceBase，
该类通过其嵌套类MultiDeviceBuffer把一路数据(输入)轮流分发到多路(输出)，
该类提供了函数GetNextFromShard(...,int shard\_num,...)
来获取第shard\_num路的下一份数据。
类MultiDeviceIterator::MultiDeviceBuffer，
该类含有一个host iterator(IteratorBase)
以及一个buffer列表(每个buffer是一个
MultiDeviceIterator::MultiDeviceBuffer::HostBuffer对象)，
buffer列表中的每个buffer代表一路数据(输出)，
该类异步地从host iterator中获取(输入)数据并轮流放入各个buffer中。
MultiDeviceIterator::MultiDeviceBuffer::HostBuffer，
该类使用一个队列std::deque<HostBufferElement>来缓存多份数据，
队列里每个HostBufferElement对象中存有一份数据。
MultiDeviceIteratorHandleOp、AnonymousMultiDeviceIteratorOp、
MultiDeviceIteratorInitOp、MultiDeviceIteratorGetNextFromShardOp、
DeleteMultiDeviceIteratorOp ...:
这些类是围绕MultiDeviceIterator类的一些操作和封装，
其中MultiDeviceIteratorHandleOp是OpKernel，注册到了Op MultiDeviceIterator。

- tensorflow/python/data/ops/multi\_device\_iterator\_ops.py。
类MultiDeviceIterator，是一个把数据prefetch到多个device上的类，
该类通过调用C++ Op MultiDeviceIterator
(调用接口为tensorflow.python.ops.gen\_dataset\_ops.multi\_device\_iterator，
也即tensorflow.raw\_ops.MultiDeviceIterator)，
在host上生成多路buffer(每路对应一个device)，
然后通过GeneratorDataset/remote\_call在各个device上生成各自的dataset，
再在各device上进行prefetch。

- tensorflow/python/distribute/input\_lib.py:
该文件调用了tensorflow/python/data/ops/multi\_device\_iterator\_ops.py，
用于为一些tensorflow.distribute.Strategy提供数据输入
(比如MirroredStrategy，先用multi device iterator在各device上生成各自的一份数据，
然后把数据封装为per replica value)。
