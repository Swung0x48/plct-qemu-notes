先 clone QEMU 源码

```
./configure \
    --target-list=riscv64-linux-user,x86_64-linux-user,riscv64-softmmu \ # 只编译需要的 target
    --enable-debug \ # 启用 debug
    --prefix=/root/plct-qemu/build \ # 设置 prefix（要把编译产物放到哪里？）
    --disable-gtk --disable-sdl --disable-bpf # 禁用一些不需要的依赖

make -j$(nproc)
```

然后去 configure 时设置的 prefix 目录找编译产物