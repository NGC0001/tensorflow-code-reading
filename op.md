下面的C++类/文件等位于tensorflow/core/framework目录。

---

- OpRegistryInterface: 位于op.h。
定义有纯虚函数LookUp，根据op\_type\_name查找op\_registration\_data。

- OpRegistry: 位于op.h。继承OpRegistryInterface。
可以注册op的op\_registration\_data。
其Global静态函数提供了一个singleton。

---

- REGISTER\_OP: 位于op.h。
macro，利用OpDefBuilderWrapper类把op注册到OpRegestry::Global()中。
