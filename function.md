### tensorflow/core/framework目录中function的相关接口。

- FunctionDefLibrary: 位于function.proto。
含有FunctionDef列表和GradientDef列表。

- FunctionDef: 位于function.proto。
含有function signature(OpDef对象，有name/arguments/return value/attrs/...)、
attributes specific、arguments、NodeDef列表、outputs等。

- GradientDef: 位于function.proto。
用于指定对某个function f进行反向传播时用到的gradient function g。
含有function f的name和gradient function g的name。

---

- InstantiateFunction: 函数，位于function.h。
实例化一个function def(注意不是实例化一个function)。
根据一个FunctionDef中的NodeDef列表，新建一个NodeDef列表。
并在新列表中设置各个node的attr values。
新建的列表储存于一个InstantiationResult对象中。

- InstantiationResult: 位于function.h。
含有NodeDef表/arg types/return types。

- FunctionLibraryDefinition: 位于function.h。继承了OpRegistryInterface。
该类通过function name在字典中检索function definition。
有函数AddFunctionDef/AddGradientDef用于添加function def/gradient def。
有函数AddLibrary用于从FunctionDefLibrary对象或FunctionLibraryDefinition对象中
批量添加function def/gradient def。

- FunctionLibraryDefinition::FunctionDefAndOpRegistration: 位于function.h。
含有FunctionDef、OpRegistrationData、StackTracesMap。

---

- CallFrameInterface: 位于function.h。
有几个纯虚函数，比如GetArg、ConsumeArg、SetRetval等。

- FunctionCallFrame: 位于function.h。继承了CallFrameInterface。
用于向function传递arguments，并获取function的return value。
(类似于kernel执行时的context?)

- FunctionLibraryRuntime: 位于function.h。
有纯虚函数Instantiate、纯虚函数Run/RunSync、
纯虚函数CreateKernel、纯虚函数Clone、纯虚函数GetFunctionBody等。

FunctionLibraryRuntime::Handle/LocalHandle: 位于function.h。
都是uint64的type alias。
用于作为从ProcessFunctionLibraryRuntime/FunctionLibraryRuntime中
查找function的实例的句柄。

- CustomKernelCreator: 位于function.h。
有纯虚函数CreateKernel。可用于创建执行某个function的OpKernel。
该类可供FunctionLibraryRuntime::CreateKernel调用。

- DistributedFunctionLibraryRuntime: 位于function.h。
有纯虚函数Instantiate/Run。

### tensorflow/core/common\_runtime目录中function的相关接口。

- FunctionLibraryRuntimeOverlay: 位于function.cc。继承FunctionLibraryRuntime。
FunctionLibraryRuntime的一个特殊实现。

- FunctionLibraryRuntimeImpl: 位于function.cc。继承FunctionLibraryRuntime。
含有device manager/device/Env/ProcessFunctionLibraryRuntime/
FunctionLibraryDefinition/GraphOptimizer等等。
含有一个字典，可以通过function handle来查找function item
(FunctionLibraryRuntimeImpl::Item对象)。
有函数FunctionDefToBody，该函数通过调用FunctionDefToBodyHelper，
从function def和attrs构建function body(function body中有function的graph)。
有函数CreateItem，该函数用GraphOptimizer对function graph进行优化，
并利用优化后的图创建item中的Executor。
有函数GetOrCreateItem，该函数根据local handle查找function item，
如果item中尚未创建executor，则调用CreateItem创建executor。
有函数Instatiate，该函数根据function name/attr等来实例化一个function item，
该函数调用FunctionDefToBody创建item中的FunctionBody，
并根据需要调用GetOrCreateItem来创建item中的executor。
有函数Run/RunSync，该函数根据handle找到function item，调用item中的executor。
有函数CreateKernel，用于根据NodeProperties创建一个kernel，
该函数可以根据需要选择是否使用CustomKernelCreator，
可以根据NodeProperties中的op来确定所创建的kernel的类型
(普通OpKernel，还是执行某个function的CallOp)。
两个不同device上的FunctionLibraryRuntimeImpl对象
可以互相Initiate/Run对方的function，
但这需要借由ProcessFunctionLibraryRuntime对象来实现，
这也是tensorflow中RemoteCall的工作方式。

- FunctionLibraryRuntimeImpl::Item: 位于function.cc。
该类保存一个function实例化产生的结果。
function item中含有graph/function body/executor等等，
其中executor能够执行该function的graph。

CallOp: 位于function.cc。继承AsyncOpKernel。
其构造函数接受一个function handle。
其ComputeAsync函数通过调用FunctionLibraryRuntime::Run来执行一个特定function。

- FunctionBody: 位于function\_body.h。
含有FunctionDef/function的graph/arg types/return types/
arg nodes/return nodes/control return nodes等。
 
- FunctionDefToBodyHelper: 位于function\_def\_utils.h。
该函数调用InstantiateFunction将function def实例化，
然后调用ConvertNodeDefsToGraph构建出function的graph，
最后生成一个function body。

---

- ProcessFunctionLibraryRuntime: 位于process\_function\_library\_runtime.h。
该类保存所有的FunctionLibraryRuntime对象(每个device一个FunctionLibraryRuntime)。
含有Env/FunctionLibraryDefinition/device manager/DeviceSet/
DistributedFunctionLibraryRuntime(parent)/rendezvous factory/
Handle-FunctionData table/Handle-MultiDeviceFunctionData table/
string-Handle table/Device-FunctionLibraryRuntime table等等。
有函数GetFLR，用于返回指定device上的FunctionLibraryRuntime。
有函数Instantiate/InstantiateRemote/InstantiateMultiDevice，
用于实例化一个function，根据function所分布的device的不同，
调用一个或多个device上的FunctionLibraryRuntime::Instantiate，
或者调用DistributedFunctionLibraryRuntime::Instantiate(remote function)，
其中函数InstantiateMultiDevice涉及到图的
placement(Placer)/optimization/partition(PartitionFunctionGraph)等。
有函数Run/RunInternal/RunMultiDevice，用于执行一个function，
根据function所分布的device的不同，
调用一个或多个device上的FunctionLibraryRuntime::Run，
或者调用DistributedFunctionLibraryRuntime::Run(remote function)。
有函数Clone/GetFunctionLibraryDefinition/
AddHandle/GetHandle/GetHandleOnDevice/RunSync/
IsMultiDevice/AddMultiDeviceHandle等等。

- ProcessFunctionLibraryRuntime::ComponentFunctionData:
位于process\_function\_library\_runtime.h。
一个component function是一个multi-device function在某个特定device上的组成部分。
该类含有handle/arg indices/ret indices/allocator attributes等。

- ProcessFunctionLibraryRuntime::MultiDeviceFunctionData:
位于process\_function\_library\_runtime.h。
该类表示一个实例化后的multi-device function的相关信息。
该类含有function name/function key(canonicalized name)/return types等，
并含有一个字典，保存分布在各个device上的component function data。

- ProcessFunctionLibraryRuntime::FunctionData:
位于process\_function\_library\_runtime.h。
该类表示一个实例化后的remote function的相关信息。
含有target device/local handle/function key等。

### tensorflow/python/framework目录中function相关的python代码。

- Tensor: 位于ops.py。该类即tf.Tensor。
该类用于图模式下，在构建一个function的graph时用得到。
该类可以看作是图构建过程中tensor的占位符，它不拥有数据，
它并不是C++中Tensor类的python对应物。
该类保存有tensor的dtype、产生这个tensor的operation、
这个tensor在这个op的outputs中的index。

- Operation: 位于ops.py。该类即tf.Operation。
该类保存有算子的node\_def/parent graph/
inputs(ops.Tensor对象列表)/output types等等。
注意，构建一个Operation需要一个Graph对象，
而Operation在构建时会调用Graph.\_add\_op把自己插入到Graph对象中。

- RegisterGradient: 位于ops.py。该类即tf.RegisterGradient。
该类用于为op注册梯度。

- Graph: 位于ops.py。该类即tf.Graph。
该类代表tf.function中的计算图。
该类含有图中的各个operation以及它们的name/id。
有函数as\_graph\_def，可以把图serialize为GraphDef。
有函数\_create\_op\_internal，可以根据op type/inputs等向图中插入operation，
并处理operation的一些与图相关的attr。

- EagerTensor: 位于ops.py。
在python环境下，eager模式下的tensor即为EagerTensor类型。
ops.py文件中定义了\_EagerTensorBase类，
而后调用C++代码在\_EagerTensorBase类基础上创建了EagerTensor类(
即EagerTensor类并不是由python代码直接定义出来的)。
当前版本，\_EagerTensorBase类继承了Tensor类。
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
在非eager模式下，该函数在图中添加Const op。

- FuncGraph: 位于func\_graph.py。继承ops.Graph。
该类是"Graph representing a function body"。
该类记录一个function的inputs/outputs/
variables/trainable\_variables/captures等等。
该类的对象可以当作一个ops.Graph对象使用，可以放进default graph stack中。
该类有方法capture，用于捕获一个EagerTensor(会调用constant\_op.constant)，
或者为graph外的某个value创立一个placeholder(
调用eager.graph\_only\_ops.graph\_placeholder)并捕获这个placeholder。

- func\_graph\_from\_py\_func: 函数，位于func\_graph.py。
该函数从一个python函数及input signature构建一个FuncGraph对象，
构建过程中会根据需要调用tensorflow.python.autograph模块。
该函数会把正在创建中的FuncGraph对象设置为defautl graph，
处理inputs(通过调用函数\_get\_defun\_inputs进行处理，比如，
如果一个input是TensorSpec对象/ops.EagerTensor对象，
则调用eager.graph\_only\_ops.graph\_placeholder为它创建placeholder，
从而转换为ops.Tensor对象)，再以非eager模式执行所传入的python函数。

- \_apply\_op\_helper: 函数，位于op\_def\_library.py。
该函数在由python代码构建function graph的过程中起作用。
非eager模式下，调用某个算子(比如Op Add)对应的python函数(比如tf.add)时，
python函数并不会真的执行算子，
而是会调用\_apply\_op\_helper，把算子插入到图中。
该函数根据算子名称查找相应的op\_def，
处理算子的inputs/attrs等(例如调用ops.convert\_to\_tensor对inputs做必要的转换)，
在图中插入算子(通过调用FuncGraph/Graph.\_create\_op\_internal)，
返回算子、算子的输出等。
该函数中算子的输入、输出等，
并不会使用C++中管理数据的Tensor类，而是使用python中的ops.Tensor类，
ops.Tensor类只是一个占位符。

### tensorflow/python/eager目录中function相关的python代码。

- graph\_placeholder: 函数，位于graph\_only\_ops.py。
用于为一个graph的外部输入创建placeholder。
该函数在graph中添加Placeholder op，并返回该op的输出(ops.Tensor类型)。

- def\_function.py中定义了function函数(也即tf.function)、
Function类(tf.function返回的对象)等，
它们最终依赖于function.py中定义的defun函数、Function类等。

- function.py中的defun函数、Function类、ConcreteFunction类:
defun函数最终会生成一个Function对象。
Function类有\_create\_graph\_function方法，
该方法会调用(python/framework/func\_graph.py中的)
func\_graph\_from\_py\_func函数
生成一个(python/framework/func\_graph.py中的)FuncGraph对象，
并用这个FuncGraph对象生成一个ConcreteFunction对象。
Function类有\_maybe\_define\_function方法，
该方法会处理function的inputs(对input args/input kwargs进行canonicalize)，
根据input signature查找是否有缓存的对应的ConcreteFunction对象，
如果有则直接返回找到的对象，
如果没有则调用\_create\_graph\_function方法
新建一个ConcreteFunction对象并把这个对象缓存起来。
Function类有\_\_call\_\_方法、get\_concrete\_function方法，
它们会调用\_maybe\_define\_function方法。
Function.\_\_call\_\_会调用ConcreteFunction.\_filtered\_call。

- ConcreteFunction: 位于function.py。
该类用于封装function definition以及它的gradient。

- function.py中有一些类和函数用于构建function的gradient。
