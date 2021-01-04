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
