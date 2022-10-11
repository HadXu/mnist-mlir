# mnist mlir demo


## 1. 将模型转为onnx [TORCH.ONNX](https://pytorch.org/docs/stable/onnx.html)

## 2. 将onnx转为.mlir
```
python onnx-mlir.py -O3 --EmitMLIR mnist-12.onnx
或者使用编译好的onnx-mlir工具
```

## 3. 对.mlir进行分析并生成代码
```
python code-gen.py
```

## 4. 运行
```
mlir-opt mnist-mlir-runner.mlir --lower-affine -convert-scf-to-cf -convert-linalg-to-llvm -convert-memref-to-llvm -convert-func-to-llvm -reconcile-unrealized-casts | mlir-cpu-runner -O3 -e entry -entry-point-result=void -shared-libs=/home/lay/work/llvm-study/llvm-project/build/lib/libmlir_runner_utils.so
```


## Reference
```
alias onnx-mlir-opt=/home/lay/soft/mlir/onnx-mlir/build/Debug/bin/onnx-mlir-opt
alias onnx-mlir=/home/lay/soft/mlir/onnx-mlir/build/Debug/bin/onnx-mlir

--convert-krnl-to-affine
--convert-krnl-to-llvm  

mlir-opt mnist.mlir --lower-affine -convert-scf-to-cf -convert-linalg-to-llvm -convert-memref-to-llvm -convert-func-to-llvm -reconcile-unrealized-casts | mlir-cpu-runner -O3 -e entry -entry-point-result=void -shared-libs=/home/lay/work/llvm-study/llvm-project/build/lib/libmlir_runner_utils.so

mlir-opt mnist.mlir --lower-affine -convert-scf-to-cf -convert-linalg-to-llvm -convert-memref-to-llvm -llvm-request-c-wrappers -convert-func-to-llvm  -reconcile-unrealized-casts | mlir-translate --mlir-to-llvmir | llc -o mnist.o -filetype=obj
```