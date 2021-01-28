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

- TTypes: 位于tensor_types.h。是Eigen::TensorMap类

### tensorflow/python/framework目录中tensor相关的python代码。

- tensorflow.python.framework.ops.Tensor类用于图模式，
它并不是C++中Tensor类的对应物。

- EagerTensor: 位于ops.py。
在python环境下，eager模式下的tensor即为EagerTensor类型。
ops.py文件中定义了\_EagerTensorBase类，
而后调用C++代码在\_EagerTensorBase类基础上创建了EagerTensor类(
即EagerTensor类并不是由python代码直接定义出来的)。
当前版本，\_EagerTensorBase类继承了ops.Tensor类。
需要进一步弄清楚该类如何与C++中的Tensor类对应。

- convert\_to\_tensor\_v2: 位于ops.py。该函数即tf.convert\_to\_tensor。
该函数调用ops.convert\_to\_tensor。

- convert\_to\_tensor: 位于ops.py。
如果传入的值的类型是ops.EagerTensor并且运行在非eager模式下(正在创建function
的graph)，则调用graph.capture。
如果传入的值的类型是ops.Tensor，则检查类型并直接返回原值。
如果传入的值的类型是ops.EagerTensor(注意当前版本的EagerTensor继承了Tensor
)并且运行在eager模式下，则检查类型并直接返回原值。
其他情况另有处理方法(会根据需要调用constant\_op.constant)。

- constant\_op.py:constant: 函数，该函数即tf.constant。
该函数和ops.convert\_to\_tensor没有事实上的区别。
在eager模式下，会创建一个EagerTensor。
在非eager模式下，该函数在graph中添加Const op。

### 关于Tensor

- 在tensorflow中，很多被操作的对象都尽可能被直接/间接地装入tensor中。

- 需要弄清楚C++代码对eager模式下tensor的一些特殊处理。

### The placement of TensorBuffer's data (Host memory or device memory)
