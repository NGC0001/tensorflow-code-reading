## tensorflow中的算子

- https://zhuanlan.zhihu.com/p/34168765

- 算子在tensorflow中包含两部分，
面向图的接口定义Op，和执行计算过程的具体实现OpKernel。

- 很多op没有注册int32类型在GPU上的kernel，
int32 tensor的相关计算往往会被放到CPU上。

## tensorflow/core/framework目录中对op/kernel相关接口的定义。

- AttrValue: 位于attr\_value.proto。
用于保存OpDef、NodeDef等等的attr的值。

---

- OpDef: 位于op\_def.proto。
记录一个op的name/input/output/attr/optimizationConstraints等。

- InferenceContext: 位于shape\_inference.h。
该类配合op的shape inference function来完成op的output的shape的自动推断。

- OpRegistrationData: 位于op\_def\_builder.h。
包含了op的OpDef、shape\_inference\_fn等。这个类的对象由全局注册器Registry负责分配，作用简单来说就是保存OpDef和OpShapeInferenceFn函数，前者保存有Op的各种具体信息，会由OpDefBuilder在最后的解析参数时（成员函数Finalize）放进来，后者在SetShapeFn传进来（由Wrapper转发），所谓注册就是将op name和OpRegistrationData关联起来，具体来说放进hashmap。

    ```c++
    mutable std::unordered_map<string, const OpRegistrationData*> registry_;
    ```

- OpDefBuilder: 位于op\_def\_builder.h。
用于构建出一个OpRegistrationData。OpDefBuilder会负责接收Op的各种属性和参数定义（就是REGISTER_OP时指定的），最后统一解析（注意只是解析并不保证合法性之类的）并转给OpRegistrationData这个类（包括ShapeFn）。

---

- OpRegistryInterface: 位于op.h。
定义有纯虚函数LookUp，根据op\_type\_name查找OpRegistrationData。

- OpRegistry: 位于op.h。继承OpRegistryInterface。
可以注册op的OpRegistrationData。
其Global静态函数返回该类的一个static local实例。

- REGISTER\_OP: 位于op.h。macro。
利用OpDefBuilderWrapper类和其他一些操作来构建op并注册到OpRegestry::Global()中。

- OpDefBuilderWrapper: 位于op.h。
是对OpDefBuilder类的封装，方便REGISTER\_OP使用OpDefBuilder。

---

- OpKernel: 位于op\_kernel.h。
其构造函数接受一个OpKernelConstruction对象，
在OpKernel构造函数执行时(即OpKernel对象实例化期间)，
可以利用OpKernelConstruction::GetAttr来获得opkernel对应node的各个attr。
从tensorflow代码来看，在图模式下，
OpKernel对象是在graph执行时，由executor负责创建的，
而最终调用的则可能是(op\_kernel.h中的)CreateOpKernel函数。
OpKernel类有thread-safe纯虚函数Compute，该函数执行kernel的计算工作，
它仅接受一个OpKernelContext对象作为参数，
OpKernelContext提供了Compute所需的输入，
Compute的计算结果也写入OpKernelContext中。
OpKernel包含有NodeProperties。

- AsyncOpKernel: 位于op\_kernel.h。继承自OpKernel。
有纯虚函数ComputeAsync，该函数接受一个OpKernelContext对象和一个回调函数。
AsyncOpKernel的Compute函数会调用ComputeAsync，
并使用了Notification对象创建ComputeAsync的回调函数。

- OpKernelConstruction: 位于op\_kernel.h。
含有一个OpKernel对象实例化时所需的信息，
如device/allocator/function library/resource manager/node properties等。
有方法allocate\_temp/allocate\_persistent用于OpKernel自身创建临时/持久的tensor。
有方法GetAttr，该方法通过(node\_def\_util.h文件中的)GetNodeAttr函数
来获取OpKernel对象所对应的node的attr。

- OpKernelContext: 位于op\_kernel.h。
是OpKernel执行Compute时的参数，
为OpKernel的计算提供input及上下文信息，并接受计算出的output。
含有step/device/allocator/resource manager/rendezvous/
collective executor/call frame/control flow/
session/cancellation manager/input/function library等等。
有一系列方法帮助OpKernel分配mem/设置output等。

---

- KernelDef: 位于kernel\_def.proto。
含有op name/device type/attribute constraints/host mem args/priority等。

- KernelDefBuilder: 位于kernel\_def\_builder.h。
用于构建出一个KernelDef。

---

- KernelRegistration: 位于op\_kernel.cc。
包含kernel的KernelDef、kernel\_class\_name、
实例化该OpKernel的函数(该函数被封装到OpKernelFactory类)。

- KernelRegistry: 位于op\_kernel.cc。
该类的一个全局对象用于注册kernel的KernelRegistration。

- REGISTER\_KERNEL\_BUILDER: 位于op\_kernel.h。macro。
构建将OpKernel实例化的函数，用Name类来构建KernelDef，
并使用OpKernelRegistrar类将OpKernel注册到一个全局的KernelRegistry对象中。

- Name: 位于op\_kernel.h。
该类封装KernelDefBuilder类，是REGISTER\_KERNEL\_BUILDER的辅助类。

- CreateOpKernel: 位于op\_kernel.h。
该函数根据device/node等信息从KernelRegistry中寻找合适的KernelRegistration。
找到后，创建OpKernelConstruction对象。
随后，调用KernelRegistration中的实例化函数，创建OpKernel对象。

---

- SupportedDeviceTypesForNode: 函数，位于op\_kernel.h。
该函数从一个DeviceType列表中筛选出指定的node所支持的DeviceType(
'node支持device'是指有为该node的device/op组合注册的kernel)，
返回依照优先级(优先级是kernel注册信息中的优先级)排序的筛选后的列表。
