### tensorflow/core/framework目录中与variable相关的接口。

- variable本质上是一个放在resource中的tensor(
无论已经弃用的'ref版本'的variable，还是当前的'resource版本'的variable，
均是如此)。'ref版本'中存放tensor的resource是LegacyVar类，
'resource版本'中存放tensor的resource是Var类。
而resource则是由resource manager来储存和管理的。

- 在tensorflow中，variable有两种实现，
分别是'ref版本'(ref-style，已弃用)，和'resource版本'(resource-style，当前版本)。
它们的主要区别之一在于，前者使用"Ref(dtype)"类型作为resource的句柄，
后者使用resource handle作为resource的句柄。

- VariableDef: 位于variable.proto。
含有variable name/初值tensor的name/initializer op的name/
snapshot name/是否是ResourceVariable/trainable等等。
variable可以是distributed，此时这个variable由多个slice构成，
每个slice也都是一个variable，它们之间需要synchronize/aggregate。

- Var: 位于resource\_var.h。继承ResourceBase。含有一个Tensor。
Var对象是variable('resource版本')在resource manager中储存的resource。
有copy-on-write(default)和copy-on-read(for sparse access)两种模式。

### tensorflow/core/ops目录中与variable相关的Op。

- resource\_variable\_ops.cc:
该文件注册了VarHandleOp/ReadVariableOp/DestroyResourceOp/
AssignVariableOp/VarIsInitializedOp/VariableShape等等Op。
这些Op是resource-style version的variable相关的Op。

- state\_ops.cc:
该文较注册了Variable(V2)/IsVariableInitialized/TemporaryVariable/
Assign/AssignAdd等等Op(注意这些Op使用了"Ref(dtype)"数据类型)。
这些Op是ref-style version的variable(已弃用)相关的Op。

### tensorflow/core/kernels目录中与variable相关的OpKernel。

- variable\_ops.h/variable\_ops.cc:
VariableOp(OpKernel)被注册到Variable/VariableV2(Op)。
IsVariableInitializedOp(OpKernel)被注册到IsVariableInitialized(Op)。
LegacyVar类继承了ResourceBase，存有一个Tensor，
LegacyVar对象是variable('ref版本')在resource manager中储存的resource。
LegacyVar/Variable/VariableOp等是所谓的ref-style version用到的类/算子，
在当前版本的python代码中调用tf.Variable时，使用的不再是这些类/算子，
而是所谓的resource-style version的Var/VarHandleOp等类/算子。

- resource\_variable\_ops.h/resource\_variable\_ops.cc:
这俩文件含有resource-style version的variable相关OpKernel。
