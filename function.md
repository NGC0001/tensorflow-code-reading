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
根据一个FunctionDef中的NodeDef列表，新建一个NodeDef列表。
并在新列表中设置各个node的attr values。
新建的列表储存于一个InstantiationResult对象中。
(use?)

- FunctionLibraryDefinition: 位于function.h。继承了OpRegistryInterface。
该类通过function name在字典中检索function definition。

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
纯虚函数CreateKernel、纯虚函数Clone等。

- CustomKernelCreator: 位于function.h。
有纯虚函数CreateKernel。可用于创建执行某个function的OpKernel。
该类可供FunctionLibraryRuntime::CreateKernel调用。

### tensorflow/core/common\_runtime目录中function的相关接口。

- FunctionLibraryRuntimeOverlay: 位于function.cc。继承FunctionLibraryRuntime。
FunctionLibraryRuntime的一个特殊实现。

- FunctionLibraryRuntimeImpl: 位于function.cc。继承FunctionLibraryRuntime。
含有device manager/device/Env/ProcessFunctionLibraryRuntime/
FunctionLibraryDefinition/GraphOptimizer等等。
含有一个字典，可以通过function handle(FunctionLibraryRuntime::Handle对象)
来查找function item
(FunctionLibraryRuntimeImpl::Item对象，
function item中含有graph/function body/executor等等，
其中executor能够执行该function的graph)。
有函数Instatiate，该函数根据function name/attr等来实例化一个function item。
有函数Run/RunSync，该函数根据handle找到function item，调用item中的executor。
有函数CreateKernel，用于根据NodeProperties创建一个kernel，
该函数可以根据需要选择是否使用CustomKernelCreator，
可以根据NodeProperties中的op来确定所创建的kernel的类型
(普通OpKernel，还是执行function的CallOp)。

CallOp: 位于function.cc。继承AsyncOpKernel。
其构造函数接受一个function handle。
其ComputeAsync函数通过调用FunctionLibraryRuntime::Run来执行一个特定function。

- FunctionBody:
