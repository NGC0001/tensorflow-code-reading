### tensorflow/core/framework目录中node/graph相关的接口。

- NodeDef: 位于node\_def.proto。
含有name(图内唯一)、op name(以\_开头，internal use)、
inputs(以字符串形式记录input来源，src\_node:src\_output)、
device(device限制条件,optional)、
attr(对应于OpDef中的attr)等。

- NodeDefBuilder: 位于node\_def\_builder.h。
根据OpDef建立一个NodeDef，
并在NodeDef中记录该node的输入来自其他哪些node。

- NodeProperties: 位于node\_properties.h。
含有OpDef/NodeDef/input types/output types。
在为node创建OpKernel实例时，
该类用于为OpKernelConstruction对象提供关于node的信息。

---

- GraphDef: 位于graph.proto。
含有nodes/FunctionDefLibrary/graph version。

- GraphToFunctionDef: 位于graph\_to\_functiondef.h。
该函数把一个Graph转变成一个FunctionDef。

### tensorflow/core/graph目录中node/graph相关的接口。

- NodeBuilder: 位于node\_builder.h。
用于创建一个Node，并把它添加到Graph中。
该类用NodeDefBuilder创建NodeDef，
该类会设置Node的AssignedDevice，
并且会在Graph中把所建Node与该Node的各input Node连接起来。
