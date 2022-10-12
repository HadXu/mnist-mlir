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

#### 对于产生的weights进行分析
```
weights = "16E911BCDD9772BE234202BFF73884BD2037113EC08B17BF9B58F3BE48214ABDC7A9443FE9E4863E6BC8FBBE56CF633D6E6D823F18010E3FD621E2BEAA5D23BE0ABB0E3F4B93173F58E896BE5AF91CBFF8B01D3D9B71673E39CD5FBED052F4BE5A5A95BEAC5B23BDA1D85F3E2759FF3E82EBD83E0CDA463D5E38CFBD14C78DBEB52590BBC2F4063FB38BCE3E40751ABEE744A7BE28CD22BE4ECEB73E17CABE3E425311BFA07CC5BE44DD31BE9F82693E145EAA3ED5F5C2BEB3FC6ABEAFA887BD42906EBCFFDF913E8C07C2BCE606D63D6DDB813E1F3AB93EEBE2303F2E7AAB3EFC80D53E3C08C53E70A8A83EE9D8783E092AF0BE55909BBD5BFE13BE65BA31BE70B17CBEA30179BFB33237BF4FC809BF49E51DBF709BE5BE26C24BBE9FADAABE29BAB2BE678A15BE8A3F82BD253897BE6432AABEE8EEA7BEBD37A2BDF9C463BE431AF4BEFDC557BE94A36BBD2AAB273E9BDC253E4C90C8BE5DA0C0BC0773FC3DA4AB2D3DB285693E729EC13DA67BBE3E0EA7603E6D8137BC3D86393EA2B0A73E86C5B33E311AEE3C5F3E563E7272AE3E0CB05BBE3EF4F9BD50021CBB2B2867BE470825BECC5EB63D35E5513EBC1920BED21167BEADE50FBF88B876BD1A24D23EF85C6DBC5829F3BEF06510BFEAABECBDD781113EB6B08D3E3F7C2E3DB2393DBC0DDD1E3C52057C3E3128203F62EA2E3F79BC713EEB072ABEBDE9AABE78C843BEDD205B3DFCF8103E90423B3E104F0CBF4DB83BBFB29816BF03B9C4BE09500A3F8E37B13E9D2C81BE066DB1BE26FA57BE296F613EAFAE283FFEC4BA3E8E7A043EBE60E0BD85B0763D8F0F9B3E78AC723EB034F03EEFA6643E47A5C83D643A0F3F16945BBEA954EABEE4E858BEEC93BA3E4560F93E56BBD0BE69C0DBBE017C7DBE8C71D43E589B993E0171EABED09A9BBE5960B7BD345D8E3EDD5AEA3D177330BE250FB4BE71AD1BBEF884B23E88413E3E142D913D9365093E3C2A3DBD26EDB4BC482C07BEF8C5BB3D562A6D3EB0F2AD3EC102293E71C2C03EEEB5473EFA5DA13E2230113F57ACF1BD7AAF483E91BC073EFDBF433DD1D9083EF975E83C8E7D0EBEFC59F7BDE4454ABEDE9500BF71BB12BD4B6242BE1AE6DABE88CA91BE19EB42BE"
x = binascii.unhexlify(weights)
print(x)

x = np.frombuffer(x, dtype=np.float32).reshape(8, 1, 5 ,5)

print(x)
```

## 4. 运行
```
mlir-opt mnist-mlir-runner.mlir --lower-affine -convert-scf-to-cf -convert-linalg-to-llvm -convert-memref-to-llvm -convert-func-to-llvm -reconcile-unrealized-casts | mlir-cpu-runner -O3 -e entry -entry-point-result=void -shared-libs=/home/lay/work/llvm-study/llvm-project/build/lib/libmlir_runner_utils.so
```

## 5. 另一种方式运行
```
mlir-opt mnist-12.onnx-global.mlir --lower-affine -convert-scf-to-cf -convert-linalg-to-llvm -convert-memref-to-llvm -llvm-request-c-wrappers -convert-func-to-llvm  -reconcile-unrealized-casts | mlir-translate --mlir-to-llvmir | clang -x ir - -fPIC -c -o mnist-global.o
c++ mnist.cpp Container.cpp mnist-global.o
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
