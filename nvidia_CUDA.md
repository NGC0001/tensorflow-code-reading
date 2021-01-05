### CUDA Concept 
- Block Grid
  - [Nvidia's blog](https://developer.nvidia.com/blog/even-easier-introduction-cuda/): You could think of that a grid wraps a bunch of blocks, a block wraps a bunch of threads, and a thread wraps a bunch of basic array elements.
  - [Another Blog](https://leimao.github.io/blog/CUDA-Concept-Block-Grid/) which contains figures illustrating the Block and Grid.
  - Here is an simple example of add kernel. 

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

- [CUDA Stream](https://leimao.github.io/blog/CUDA-Stream/): A stream is a sequence of commands that execute in order. Different streams execute their commands out of order with respect to one another or concurrently.

- Unified Virtual Address Space (UVA): The location of any memory on the host allocated through CUDA, or on any of the devices which use the unified address space, can be determined from the value of the pointer using `cudaPointerGetAttributes()`.
- Page-Locked Host Memory: `codaHostAlloc()` allocate page-locked host memory where `malloc()` malloc pageable host memory.
  - Benefit: Asynchronous copy, maybe eliminate the need to copy, higher bandwidth(It's pinned so there is no page fault, so device could fetch data without help from CPU)
- Mapped Memory: A block of page-locked host memory can also be mapped into the address space of the device. Such a block could have another address in device memory that can be retrieved using cudaHostGetDevicePointer() and then used to access the block from within a kernel.
