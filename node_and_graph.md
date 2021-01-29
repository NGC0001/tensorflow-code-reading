### tensorflow/core/framework目录中node/graph相关的接口。

- NodeDef: 位于node\_def.proto。
含有name(在一张图内是唯一的)、op name(以\_开头，internal use)、
inputs(以字符串形式记录该结点的input来自哪个结点的第几个输出，
格式为src\_node:src\_output)、
device(requested device for this node, optional)、attr等。

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
一个图中有唯一的源结点(source node, node id=0)，
所有那些没有输入边的结点(源结点自身除外)需要连接源结点作为它们的输入边，
一个图中有唯一的汇结点(sink node, node id=1)，
所有那些没有输出边的结点(汇结点自身除外)需要连接汇结点作为它们的输出边，
源结点和汇结点的op都是NoOp，
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

---

- Partition: 函数，位于graph\_partition.h。
用于把图分成不同device上的子图。
该函数的重要工作之一是添加一些在device之间Send/Receive数据的nodes。

### tensorflow/core/common\_runtime目录中node/graph相关的接口。

- GraphConstructor: 位于graph\_constructor.h。
有static函数Construct，该函数用于从NodeDef列表/GraphDef构建一张图。

- ConvertNodeDefsToGraph: 函数，位于graph\_constructor.h。
该函数利用GraphConstructor::Construct从NodeDef列表构建一张图。

- ConvertGraphDefToGraph: 函数，位于graph\_constructor.h。
该函数利用GraphConstructor::Construct从GraphDef构建一张图。

---

- Placer: 位于placer.h。用于把一张图的各个node分配到DeviceSet中的各个device上。
目前placer是独立于各个图的，但将来tensorflow也许会定义统一的placer接口，
并让各个图可以拥有各自专属的placer。
placement遵从以下限制条件(来自placer.h中的代码注释)：
(1)已经assigned的node不再更改device；
(2)node中的device specification会被满足；
(3)referece/resource edge连接的两个nodes会在同一device上；
(4)同一个colocation group中的nodes会在同一个device上。
Placer类有函数Run，用来执行placement，
该函数主要依靠ColocationGraph类来帮助满足各限制条件，该函数执行了一些优化策略。

- ColocationGraph: 位于colocation\_graph.h。
用到了[incremental connectivity (disjoint-set forest / union-find algorithm)](
http://web.stanford.edu/class/archive/cs/cs166/cs166.1166/lectures/16/Small16.pdf
)算法。该类配合Placer完成node的placement。
该类根据各限制条件和各优化措施来把各个node划分到不同的coloation groups，
其中同一个colocation group中的nodes需要被放到同一个device上。
同时该类为每个colocation group都维护着一个可以放置的device列表，
这个列表随着placement的进行是不断变化的，
比如当一个colocation group中的某node被放置到device a后，
则这个colocation group中其他的所有node都只能被放置到device a上。

- Placer和ColocationGraph的工作流程可以参考
腾讯云blog [TF中Placement的最后一道防线——Placer](
https://cloud.tencent.com/developer/article/1685158)。
大致来说分为以下几步：
    * ColocationGraph初始化，这一步会处理
      显式指定的assignment、resource/ref edge的colocation、
      host-only data的限制、显式指定的colocation等。
    * Placer::Run遍历所有nodes。
      如果有assigned device，则为node分配这个device；
      如果node是generator node(生产数据的node)，则把node放入待定列表；
      如果node是metadata node(不操作数据的node，比如reshape)，且该node可以和
              它的input node共同放置，则把该node放在它的input node的device上；
      最后的默认行为是把node放在按优先级排列的可行device列表中的第一个device上。
    * Placer::Run遍历待定列表中的nodes。
      如果node是generator node，且该node的consumer nodes在同一个device上，
              且该node也能放在这个device上，则把该node放在这个device上；
      最后的默认行为是把node放在按优先级排列的可行device列表中的第一个device上。

- Member: 位于colocation\_graph.h。
用于在ColocationGraph对象中表示一个node。
有函数SetParentAndSupportedDevices，
该函数会调用(tensorflow/core/framework/op\_kernel.h中的
)SetParentAndSupportedDevices函数(依据注册的kernel为node筛选可行的device)。

- PartitionFunctionGraph: 函数，位于partitioning\_utils.h。
用于把place过后的图分成不同device上的子图。
核心工作是通过函数Partition来完成。
