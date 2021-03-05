# CMake notes

### Why "file(GLOB xxx)" is evil
- It will cause inconspicuous problem during teamwork. 
- A adds a new file and does not change CMakeLists.txt
- B does a git pull, and since there have been no modifications to the CMakeLists.txt, CMake isn't run again and B causes linker errors when compiling.
- Reference: https://stackoverflow.com/questions/32411963/why-is-cmake-file-glob-evil


### About find_package
- Check local modules: cmake --help-module-list
- Manual for a specific module: cmake --help-module FindCUDA
- find_package(CUDA 10.1 REQUIRED)
  - It is no longer necessary to use this module or call ``find_package(CUDA)`` for compiling CUDA code. Instead, list ``CUDA`` among the languages named in the top-level call to the ``project()`` command, or call the
``enable_language()`` command with ``CUDA``.
  - However, if you want to some functions defined in ``FindCUDA`` module like ``cuda_select_nvcc_arch_flags(ARCH_FLAGS)``, you need explicitly use this module.
  - The following code will automatically pass nvidia graphic's compute capability to nvcc. 
  
  ```CMake
    find_package(CUDA ${MIN_CUDA_VERSION} REQUIRED)

    set(CUDA_SEPARABLE_COMPILATION ON)
    cuda_select_nvcc_arch_flags(ARCH_FLAGS)
    string(REGEX MATCH "[0-9][0-9]" ARCH "${ARCH_FLAGS}")
    set(CMAKE_CUDA_ARCHITECTURES ${ARCH})
  ```
- find_package(CUDATookit 10.1 REQUIRED)
  - This module will define CUDAToolkit_INCLUDE_DIRS (normally /usr/local/cuda/include), CUDAToolkit_LIBRARY_DIR and etc.
- set_source_files_properties(${CUDA_SOURCE_FILES} PROPERTIES LANGUAGE CUDA)
  - This command should in the same CMakeLists.txt file with the add_library(xxx) command.
