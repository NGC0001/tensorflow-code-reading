下面的类/文件等位于tensorflow/core/framework目录。

---

- TensorBuffer: 位于tensor.h。保有mem指针。
继承了引用计数类RefCounted，使其可以在引用数为0时析构自身。
有不同子类，比如针对某些host mem上的Scalar类型的Tensor作了优化，此时buffer在栈上。
有子类SubBuffer，该子类不实际拥有指针所指向的mem。

- Buffer: 位于tensor.cc。
派生关系为TensorBuffer-\>BufferBase-\>Buffer。
其父类BufferBase保存有Allocator。
该类构造时用Allocator进行allocate，析构时用Allocator进行deallocate。
这是一般的Tensor使用的TensorBuffer子类。

---

- TensorShape: 位于tensor\_shape.h。

- PartialTensorShape: 部分维数未知的TensorShape。

---

- Tensor: 位于tensor.h。包含一个TensorShape和一个TensorBuffer指针。
析构时会把TensorBuffer的引用数减1。
有Slice/SubSlice方法，它们返回的新Tensor使用TensorBuffer的子类SubBuffer。
有一些列的方法，将自身转化为TTypes中的类型，
而TTypes类是对Eigen库的封装（位于文件tensor\_types.h中）。

---

- types.h/types.cc：定义了Tensor中数据的类型的一些表示和操作。
有三个比较特殊的类型DT\_STRING/DT\_VARIANT/DT\_RESOURCE，其它为数值类型。
string/resource类型只能位于host mem中。

---

- IteratorContext: 位于dataset.h。包含很多组件。

- IteratorBase：位于dataset.h。有纯虚函数GetNext。
含有一个shared\_ptr\<model::Node\>。

- DatasetContext: 位于dataset.h。

- DatasetBase: 位于dataset.h。
继承了引用计数类RefCounted。
有const方法MakeIterator。
仅包含有type\_string\_和node\_name\_。

- DatasetBaseIterator: 位于dataset.h。继承自IteratorBase。
通过其嵌套类BaseParams包含DatasetBase指针，并对DatasetBase引用数加1。

- DatasetIterator: 位于dataset.h。继承自DatasetIteratorBase。

