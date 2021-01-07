# Nvidia HashTable Kernel Builder Implementation Details 
- How to save Tensor object to Nvidia HashTable and how to get scalar or vector from Op's Tensor input?
    - Here is an example to get element number of keys and get matrix from key and value.
        ```C++
        const int64 num_elements = (key.dims() == 0) ? 1 : key.dim_size(0);
        const int64 value_size = value_shape_.num_elements();
        const int64 key_size = key_shape_.num_elements();
        const auto key_matrix = key.shaped<K, 2>({num_elements, key_size});
        auto value_matrix = value.shaped<V, 2>({num_elements, value_size});

        Status DoInsert(bool clear, const Tensor& keys, const Tensor& values) {
            const auto key_values = keys.flat<K>();
            const auto value_values = values.flat_inner_dims<V, 2>();
            int64 value_dim = value_shape_.dim_size(0);

            mutex_lock l(mu_);

            for (int64 i = 0; i < key_values.size(); ++i) {
                ValueArray value_vec;
                for (int64 j = 0; j < value_dim; j++) {
                    V value = value_values(i, j);
                    value_vec.push_back(value);
                }
                gtl::InsertOrUpdate(&table_, SubtleMustCopyIfIntegral(key_values(i)),
                                    value_vec);
            }
            return Status::OK();
        }
        ```
        ```C++
        template <typename T>
        typename TTypes<T>::Flat flat() {
            return shaped<T, 1>({NumElements()});
        }
        template <typename T, size_t NDIMS>
        typename TTypes<T, NDIMS>::Tensor Tensor::shaped(
                gtl::ArraySlice<int64> new_sizes) {
            CheckTypeAndIsAligned(DataTypeToEnum<T>::v());
            Eigen::array<Eigen::DenseIndex, NDIMS> dims;
            FillDimsAndValidateCompatibleShape(new_sizes, &dims);
            return typename TTypes<T, NDIMS>::Tensor(base<T>(), dims);
        }
        ```
- Does it always need to transfer the TensorBuffer's data_ to device memory?
- TODO: Finished the SetShapeFn function of HashTable.
- Add more function related to Nvidia multi gpu HashTable
    - insert, set, get, update
    - insert_from_cpu, dump_to_cpu, dump_to_gpu, 
    - get_size, clear, remove

- Need understand LookupTableOp::Compute (core/kernels/lookup_table_op.h)
- struct void operator()
- core/util/gpu_kernel_helper.h for example of computing block count


# Structure of TensorFlow HashTable source code
### core/kernels/lookup_table_op.h
- class LookupTableOp: This class inherits from the OpKernel class and supports different table implementations specified by the 'Container' template. Container must be derived from LookupInterface. Container example: MutableDenseHashTable, MutableHashTableOfTensors and etc. 

- class HashTable: This class inherits from InitializableLookupTable class and wraps an flat_hash_map, where the key and value data type is specified. Once the table is marked as initialized it becomes read-only.


### core/kernels/lookup_table_op.cc
- LookupTableOpKernel: This class inherits from the OpKernel. This is a base class for kernels which take a LookupTable handle as the 0th input. This class will be inherited by operations like Remove, Insert, Find, Size, Export, Import and etc. For example:

    ```c++
    class LookupTableOpKernel : public OpKernel;
    class LookupTableFindOp : public LookupTableOpKernel;
    REGISTER_KERNEL_BUILDER(Name("LookupTableFindV2").LookupTableFindOp);
    ```
- Here are some examples of hashtable kernel builder:
    ```c++
    REGISTER_KERNEL_BUILDER(Name("HashTableV2"), LookupTableOp<lookup::HashTable<key_dtype, value_dtype>, key_dtype, value_dtype>);
    REGISTER_KERNEL_BUILDER(Name("MutableHashTableV2"), LookupTableOp<lookup::MutableHashTableOfScalars<key_dtype, value_dtype>, key_dtype, value_dtype>);
    REGISTER_KERNEL_BUILDER(Name("MutableHashTableOfTensorsV2"), LookupTableOp<lookup::MutableHashTableOfTensors<key_dtype, value_dtype>, key_dtype, value_dtype>);
    ```


### core/ops/lookup_ops.cc
```c++
  REGISTER_OP("MutableHashTableOfTensorsV2")
    .Output("table_handle: resource")
    .Attr("container: string = ''")
    .Attr("shared_name: string = ''")
    .Attr("use_node_name_sharing: bool = false")
    .Attr("key_dtype: type")
    .Attr("value_dtype: type")
    .Attr("value_shape: shape = {}")
    .SetIsStateful()
    .SetShapeFn([](InferenceContext* c) {
      PartialTensorShape value_p;
      TF_RETURN_IF_ERROR(c->GetAttr("value_shape", &value_p));
      ShapeHandle value_s;
      TF_RETURN_IF_ERROR(c->MakeShapeFromPartialTensorShape(value_p, &value_s));
      return MutableHashTableShape(c, /*key=*/c->Scalar(), /*value=*/value_s);
    });

  REGISTER_OP("MutableHashTableV2")

  REGISTER_OP("LookupTableInsertV2")
    .Input("table_handle: resource")
    .Input("keys: Tin")
    .Input("values: Tout")
    .Attr("Tin: type")
    .Attr("Tout: type")
    .SetShapeFn([](InferenceContext* c) {
      ShapeHandle handle;
      TF_RETURN_IF_ERROR(c->WithRank(c->input(0), 0, &handle));

      // TODO: Validate keys and values shape.
      return Status::OK();
    });
  REGISTER_OP("LookupTableSizeV2")
  REGISTER_OP("LookupTableFindV2")
  REGISTER_OP("LookupTableRemoveV2")
  REGISTER_OP("LookupTableExportV2")
  REGISTER_OP("LookupTableImportV2")
  ```

### core/framework/lookup_interface.h
- class LookupInterface: public ResourceBase
- Function: performs batch lookups


### Multi_GPU Single_Thread_Example.cu
- Defination:     
    using Table_ = nv::MultiGpuHashTable<key_type, val_type, ModPolicy<key_type>>;
    auto table = new Table_(num_gpu * len / load_factor, gpu_id, num_gpu); 
- Methods: 
    - Example: void insert(const KeyType* d_keys, const ValType* d_vals, size_t len, Stream_event_resource& s_e_resource);
    - insert, set, get, update
    - insert_from_cpu, dump_to_cpu, dump_to_gpu, 
    - get_size, clear, remove
    - stream_event_recource_create, stream_event_resource_destroy

- Methods Argument:
    KeyType* keys
    ValType* vals
    size_t* len
    Stream_event_resource& s_e_resource

### Nvidia Muti-GPU Hashtable Class Design
- class name: MutableNvidiaHashTableOfScalars
- inherit: LookupInterface or Opkernal or Hashtable

