### 下面的C++类/文件等位于tensorflow/core/framework目录。

---

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
该类可以封装入一个Variant，从而完成对DatasetBase的variant封装。

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

- DatasetIterator: 位于dataset.h。继承自DatasetIteratorBase。
表示一个与具体类型的dataset绑定的iterator。

### tensorflow/core/ops目录中与dataset相关的Op。

---

- dataset\_ops.cc/experimental\_dataset\_ops.cc:
包含了与Dataset/Iterator相关的算子的注册(REGISTER\_OP)。

### tensorflow/core/kernels/data目录中与dataset相关的OpKernel。

---

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

// comments in tensorflow/core/ops/dataset_ops.cc:
//
// The ops in this section can be composed to define an input
// pipeline. Each op produces a DT_VARIANT tensor that represents
// a DAG of "dataset" objects. An "dataset" object can be converted
// to a stateful "iterator" by passing the "dataset" to the
// "MakeIterator" op.
//
// TODO(b/123753214): DT_VARIANT tensors that represent "dataset" objects are
// not presently serializable. To avoid issues with constant folding, ensure
// that any "source dataset" ops (i.e. ops that output a dataset and do not
// take one as input) are marked "stateful".

