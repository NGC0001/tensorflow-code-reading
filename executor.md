## graph mode:

- executors are private to their graphs

- executor/executor args/local executor param/propagate/immutable state

- graph view

## eager mode:

- TFE\_Py\_FastPathExucute:
位于tensorflow/python/tfe\_wrapper.cc。该函数是pybind11接口。
eager模式下运行一个算子时，会优先调用TFE\_Py\_FastPathExucute，
它是对TFE\_Py\_FastPathExucute\_C的封装。

- TFE\_Py\_FastPathExucute\_C:
位于tensorflow/python/eager/pywrap\_tfe\_src.cc。
该函数从PyObject中获取信息，调用GetOp获取TFE\_Op，
并调用TFE\_Execute来执行TFE\_Op。

- TFE\_Execute: 位于tensorflow/c/eager/c\_api.cc。
该函数仅仅是把TFE\_Op\* unwrap为ImmediateExecutionOperation\*，
随后调用ImmediateExecutionOperation.Execute。

- TFE\_Op: 位于tensorflow/c/eager/immediate\_execution\_operation.h。
该类只有声明没有定义，不是一个实体类型。

- tensorflow/c/eager/tfe\_op\_internal.h:
该文件中，利用宏DEFINE\_CONVERSION\_FUNCTIONS，
定义了TFE\_Op和ImmediateExecutionOperation之间的指针转换，
比如ImmediateExecutionOperation\* unwrap(TFE\_Op\*)、
TFE\_Op\* wrap(ImmediateExecutionOperation\*)等，
这使得成为ImmediateExecutionOperation成为TFE\_Op的具体实现。

- AbstractOperation:
位于tensorflow/c/eager/abstract\_operation.h。
定义了Execute等几个纯虚函数。

- ImmediateExecutionOperation:
位于tensorflow/c/eager/immediate\_execution\_operator.h。
继承自AbstractOperation。定义了几个纯虚函数。

- EagerOperation:
位于tensorflow/core/common\_runtime/eager/eager\_operation.h。
继承了ImmediateExecutionOperation，是ImmediateExecutionOperation目前唯一的实现。

- EagerOperation::Execute:
位于tensorflow/core/common\_runtime/eager/core.cc（而非eager\_operation.cc）。
该方法会根据情况调整op的device（eager placement logic），
然后调用EagerExecute(位于tensorflow/core/common\_runtime/eager/execute.h)。
