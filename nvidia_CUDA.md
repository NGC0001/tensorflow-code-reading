### CUDA Concept Block Grid
- https://developer.nvidia.com/blog/even-easier-introduction-cuda/
- https://leimao.github.io/blog/CUDA-Concept-Block-Grid/
- An simple example of add kernel. There are illustrations about the following code in the blog.

```cpp
__global__
void add(int n, float *x, float *y)
{
  int index = blockIdx.x * blockDim.x + threadIdx.x;
  int stride = blockDim.x * gridDim.x;
  for (int i = index; i < n; i += stride)
    y[i] = x[i] + y[i];
}
```

### CUDA Stream
- https://leimao.github.io/blog/CUDA-Stream/

