### tensorflow/core/framework目录中与resource相关的类。

- ResourceBase: 位于resource\_mgr.h。是所有resource的基类。
继承自RefCounted。
只定义了虚函数MemoryUsed和纯虚函数DebugString。

