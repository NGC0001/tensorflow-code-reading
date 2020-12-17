下面的类/文件等位于tensorflow/core/framework目录。

---

- Variant: 位于variant.h。
是一个type-erased container，用于封装满足特定条件的类型。
Variant类的目的是让被封装的对象可以装入DT\_VARIANT类型的Tensor。
可以使用Variant::get\<T\>获得被封装对象的指针。

---

- DatasetContext: 位于dataset.h。

- DatasetBase: 位于dataset.h。
继承了引用计数类RefCounted。
仅包含有type\_string\_和node\_name\_。
有const方法MakeIterator，有纯虚函数MakeIteratorInternal。
有纯虚函数output\_shapes/output\_dtypes。

- DatasetVariantWrapper: 位于dataset.cc。
包含有DatasetBase指针，并对DatasetBase进行引用计数。
该类可以封装入一个Variant，从而完成对DatasetBase的variant封装。

- WrappedDatasetVariantWrapper/WrapDatasetVariantOp/UnwrapDatasetVariantOp:
位于dataset.cc。
可以把一个tensor封装为一个variant后，放入一个类型为variant形状为scalar的tensor，
也可以进行反向操作。

- DatasetOpKernel: 位于dataset.h。继承自OpKernel。
有final方法Compute，有纯虚函数MakeDataset。
该类针对不同Dataset类型派生不同的子类。

---

- [] IteratorContext: 位于dataset.h。是一个缩减版的OpKernelContext。
包含很多组件。

- IteratorBase：位于dataset.h。有纯虚函数GetNext。
含有一个shared\_ptr\<model::Node\>。
有纯虚函数output\_shapes/output\_dtypes。

- DatasetBaseIterator: 位于dataset.h。继承自IteratorBase。
通过其嵌套类BaseParams包含DatasetBase指针，并对DatasetBase引用数加1。
有纯虚函数GetNextInternal。

- DatasetIterator: 位于dataset.h。继承自DatasetIteratorBase。
表示一个与具体类型的dataset绑定的iterator。

