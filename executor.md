## graph mode:

- executors are private to their graphs

- executor/executor args/local executor param/propagate/immutable state

- graph view

## eager mode:

- python/eager/pywrap\_tfe\_src.cc: TFE\_Py\_FastPathExucute\_C

- TFE\_Execute: TFE\_Op.Execute

- c/eager/tfe\_op\_internal.h: DEFINE\_CONVERSION\_FUNCTIONS macro.
ImmediateExecutionOperation\* = unwrap(TFE\_Op\*)
TFE\_Op\* = wrap(ImmediateExecutionOperation\*)

- c/eager/immediate\_execution\_operator.h: ImmediateExecutionOperation

- class EagerOperation : public ImmediateExecutionOperation.
core/common\_runtime/eager/eager\_operation.h
core/common\_runtime/eager/core.cc
