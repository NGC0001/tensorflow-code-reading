### tensorflow/core/framework目录中与tensor/types相关的类。

- TensorBuffer: 位于tensor.h。保存有mem指针。
继承了引用计数类RefCounted，使其可以在引用数为0时delete自身。
有子类SubBuffer，该子类不实际拥有指针所指向的mem。

- HostScalarTensorBuffer: 位于tensor.cc。
派生关系为TensorBuffer-\>HostScalarTensorBufferBase-\>HostScalarTensorBuffer。
针对某些host mem上的scalar类型的Tensor作了优化，buffer在栈上。
该类的operator delete被重写。

- Buffer: 位于tensor.cc。
派生关系为TensorBuffer-\>BufferBase-\>Buffer。
其父类BufferBase保存有Allocator。
该类构造时用Allocator进行allocate，析构时用Allocator进行deallocate。
这是一般的Tensor使用的TensorBuffer子类。

---

- TensorShapeRep: 位于tensor\_shape.h。
有子模板类TensorShapeBase。
是TensorShape和PartialTensorShape的内部表示。

- TensorShape: 位于tensor\_shape.h。
派生自TensorShapeBase\<TensorShape\>。

- PartialTensorShape: 位于tensor\_shape.h。
部分维数未知的TensorShape。
派生自TensorShapeBase\<PartialTensorShape\>。

---

- Tensor: 位于tensor.h。包含一个TensorShape和一个TensorBuffer指针。
析构时会把TensorBuffer的引用数减1。
有Slice/SubSlice方法，它们返回的新Tensor使用TensorBuffer的子类SubBuffer。
有一系列的方法，将自身转化为TTypes类中的类型，
比如Tensor::scalar/vec/matrix/tensor/flat方法
分别将Tensor转化为TTyptes::Scalar/Vec/Matrix/Tensor/Flat。
而TTypes类是对Eigen库的封装（位于文件tensor\_types.h中）。

---

- DataType: 位于types.proto。定义了Tensor中各数据类型。
比较特殊的类型有DT\_INVALID/DT\_STRING/DT\_VARIANT/DT\_RESOURCE。

- types.h/types.cc：定义了数据类型的一些表示和操作。
string/resource类型只能位于host mem中(to-be-verified)。

- Variant: 位于variant.h。
是一个type-erased container，用于封装满足特定条件的任意类型。
Variant类的对象可以装入DT\_VARIANT类型的Tensor。
可以使用Variant::get\<T\>获得被封装对象的指针。

- ResourceHandle: 位于resource\_handle.h。
该类的对象可以放入DT\_RESOURCE类型的Tensor。
利用该类的对象中所包含的device/container/name/dtypes\_and\_shapes等信息，
可以从resource manager中检索到相应的resource。

### 关于Tensor

- 在tensorflow中，很多被操作的对象都尽可能被直接/间接地装入tensor中。


### The placement of TensorBuffer's data (Host memory or device memory)
