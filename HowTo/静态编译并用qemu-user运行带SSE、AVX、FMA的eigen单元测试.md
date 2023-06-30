要用 SSE/AVX 来测试 qemu 后端 V 扩展的实现正确性，可以借用 eigen 的单元测试。

参考 [Tests - eigen](https://eigen.tuxfamily.org/index.php?title=Tests)

编译
```
git clone <eigen repo.git>
mkdir build && cd build
cmake .. \
    -DCMAKE_FIND_LIBRARY_SUFFIXES=".a" \
    -DBUILD_SHARED_LIBS=OFF \ -DCMAKE_EXE_LINKER_FLAGS="-static-libgcc \
    -static-libstdc++ -static" \ # 静态编译
     \ # 如果你的编译机 CPU 不支持 AVX/AVX2，记得在这里指定 CPU family，比如 -march=alderlake
    -DCMAKE_CXX_FLAGS="-march=native" \
    -DEIGEN_TEST_AVX=ON \
    -DEIGEN_TEST_AVX2=ON \
    -DEIGEN_TEST_SSE2=ON \
    -DEIGEN_TEST_SSE3=ON \
    -DEIGEN_TEST_SSSE3=ON \
    -DEIGEN_TEST_SSE4_1=ON \
    -DEIGEN_TEST_SSE4_2=ON \
    -DEIGEN_TEST_FMA=ON # 对 SSE/AVX/FMA 跑测试

make check -j$(nproc) # 编译并且跑一遍测试，确保在 x86 物理机上编译的测试没问题
```

检查并运行一个 test
```
ldd basicstuff_2
# 应该输出类似 not a dynamic executable.

# 用 qemu-user-x86_64 运行测试
/path/to/qemu-x86_64 -cpu EPYC basicstuff_2
```

（还不知道怎么批量运行测试，现在是很脏的直接一个 shell 把能跑的都跑一下，但感觉应该是改一下 cmake 就可以了，搜 `add_test()` 函数然后魔改一下命令用 qemu 跑，但我没测试过，等后人踩坑吧