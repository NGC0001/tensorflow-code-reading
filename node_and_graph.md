### tensorflow/core/framework目录中node/graph相关的接口。

- NodeDef: 位于node\_def.proto。
含有name(在一张图内是唯一的)、op name(以\_开头，internal use)、
inputs(以字符串形式记录该结点的input来自哪个结点的第几个输出，
格式为src\_node:src\_output)、
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
含有NodeDef列表/FunctionDefLibrary/graph version。

- GraphToFunctionDef: 位于graph\_to\_functiondef.h。
该函数把一个Graph转变成一个FunctionDef。

### tensorflow/core/graph目录中node/graph相关的接口。

- NodeBuilder: 位于node\_builder.h。
该类通过调用Graph::AddNode，创建一个Node并把它添加到Graph中。
该类用NodeDefBuilder创建NodeDef。
该类会设置Node的AssignedDevice。
该类会在Graph中把所建Node与该Node的各input Node连接起来(利用Graph::AddEdge)。

- Node: 位于graph.h。
含有node id(in a graph)/node type(NodeClass)/NodeProperties/
assigned device/in edges/out edges等。
结点有不同类型，比如while循环的控制结点等等。
有static函数GetNodeClassForOp，该函数根据结点的Op的名称来确定结点类型。

- Node::NodeClass: 位于graph.h。enum。定义了各种结点类型。
结点中除了可以是一个Op(普通Op和特殊Op)，也可以是一个fucntion。

- Edge: 位于graph.h。
含有edge id/source node/dest node等。
图中有些边是数据流动的边，有些边是控制边(control edge)没有数据流动。

- Graph: 位于graph.h。
可以用OpRegistryInterface或FunctionLibraryDefinition作为构造参数。
一个图中，有唯一的源结点(source node, node id=0)，
有唯一的汇结点(sink node, node id=1)，
这两个结点的op都是NoOp，
一个图刚被构造时，仅仅有这两个结点，仅有一条从source到sink的control edge。
Graph类含有FunctionLibraryDefinition/nodes/num\_nodes/
edges/num\_edges/assigned device names等等。
有函数AddNode，该函数allocate一个Node并做一些设置。
有函数AddEdge。
有函数ToGraphDef。

---

- GraphDefBuilder: 位于graph\_def\_builder.h。
该类可以构建起一个Graph(该类有一个私有的Graph)。
有函数ToGraphDef把Graph转变为GraphDef(调用Graph::ToGraphDef)。

- GraphDefBuilder::Options: 位于graph\_def\_builder.h。
是GraphDefBuilder的辅助类。该类调用了NodeBuilder。

### tensorflow/core/common\_runtime目录中node/graph相关的接口。

- GraphConstructor: 位于graph\_constructor.h。
有static函数Construct，该函数用于从NodeDef列表/GraphDef构建一张图。

- ConvertNodeDefsToGraph: 函数，位于graph\_constructor.h。
该函数利用GraphConstructor::Construct从NodeDef列表构建一张图。

- ConvertGraphDefToGraph: 函数，位于graph\_constructor.h。
该函数利用GraphConstructor::Construct从GraphDef构建一张图。

