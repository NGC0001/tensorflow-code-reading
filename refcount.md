### tensorflow/core/platform/refcounted.h中的引用计数类。

- RefCounted: 引用计数类。是所有需要进行引用计数的类的基类。
有虚析构方法。
用一个mutable std::atomic\_int\_fast32\_t变量储存引用计数。
有方法Ref，把引用计数加1。
有方法Unref，把引用计数减1，
如果在引用数减1后引用数为0，则调用delete this进行析构，
从而完成对象的自动销毁。

- RefCountDeleter: 该类的成员函数void operator()(RefCounted\*)，
将相应的RefCounted对象引用数减1。

- RefCountPtr: 即std::unique\_ptr\<T, RefCountDeleter\>。
析构时通过RefCountDeleter自动把指针的引用计数减1。

- ScopedUnref: 该类储存一个RefCounted对象指针。
该类超出作用域而析构时，自动把指针的引用计数减1。
