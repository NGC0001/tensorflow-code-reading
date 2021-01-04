## 编译tensorflow2.3的可调试GPU版本

- TF官网源码编译手册 https://www.tensorflow.org/install/source

- bazel官网使用说明 https://docs.bazel.build/versions/master/user-manual.html

### 已有环境：

- docker镜像为由tensorflow/tensorflow:2.3.0-gpu得到的子镜像

- ubuntu18.04

- 含基本组件(不完整)的Cuda10.1

- gcc7.5.0

- python3.6.9

### 编译过程/遇到的问题：

- 首先尝试tf2.3.1版本源码，并尝试采用bazel的"-c dbg"编译选项。

- Nvidia官网的deb包安装cudnn7.6.5及其dev包/doc包

- 镜像内cuda不完整，需要安装部分cuda组件(tf源码configure提示缺少cublas\_api.h)。
安装Nvidia官网的cuda-repo的deb包（cuda网络安装版deb包）。
apt update后，可以从apt源找到各种cuda组件包，但cublas缺少10.1版本。
apt安装cuda-libraries-dev-10-1,cuda-libraries-10-1，会附带安装cublas10.2版本，
将cuda10.2目录下的cublas相关文件拷贝到cuda10.1目录中。

- tf源码目录中configure时选择支持cuda，其他的如tensorrt等不选择。

- 使用bazelisk，编译命令为
bazel build --config=cuda --strip=never -c dbg --verbose\_failures --keep\_going
//tensorflow/tools/pip\_package:build\_pip\_package

- 有些文件bazel无法下载(llvm,aws-sdk)，可以手动下载，
再把bazel的下载链接替换为 file:///path/to/downloaded/file (bazel使用curl，支持file://)

- aws-checksum在dbg模式下汇编块编译错误，修改third\_party/aws/aws-checksums.bazel，添加gdb模式下的DEBUG\_BUILD。
```
29a30,35
>     defines = select({
>         "@org\_tensorflow//tensorflow:debug": [
>             "DEBUG\_BUILD"
>         ],
>         "//conditions:default": [],
>     }),
```
https://github.com/tensorflow/tensorflow/issues/37498 ,
https://github.com/tensorflow/tensorflow/pull/42743/files

- nenv虚拟环境下安装生成的包出错： invalid command bdist\_wheel，需要 pip3 install wheel 。

- wheel包安装后，在python中import tensorflow时出错，
提示\_pywrap\_tensorflow\_internal.so文件中有未定义的符号
\_ZN10tensorflow4data12experimental19SnapshotDatasetV2Op11kReaderFuncE，
即tensorflow::data::experimental::SnapshotDatasetV2Op::kReaderFunc。

---

- 切换到tf2.3.0版本源码，重新尝试编译，仍然在import tensorflow时出现同样的未定义符号错误。

- 继续使用2.3.0版本源码，但放弃'-c dgb'编译模式，采用'**-c opt**'，调整编译命令为 
bazel build --config=cuda -c opt --copt -g --strip=never --keep\_going --verbose\_failures
//tensorflow/tools/pip\_package:build\_pip\_package ，
同时删除aws-checksums的DEBUG\_BUILD。
编译、安装、运行均成功。
