主线的 Linux 内核还不支持 RVV，但是 sifive 有个 downstream port 支持了 RVV。
所以我们可以用这个 port 来编译一个有 RVV 支持的内核。

```
git clone https://github.com/sifive/riscv-linux.git -b riscv/for-next/vector-v20 --recursive
cd riscv-linux
make menuconfig
```

然后在 `Platform Type` 里选中 `VECTOR extension support` 和 `Enable userspace Vector by default`，保存退出

然后就能开始编译了
```
make ARCH=riscv CROSS_COMPILE=riscv64-linux-gnu- -j$(nproc)
```
编译完了以后生成 `vmlinux`

要使用新编译好的内核启动的话，首先你需要一个完整的可以用 qemu-system 启动的 linux 系统（我用的是这个 [archriscv-scriptlet](https://github.com/CoelacanthusHex/archriscv-scriptlet.git)），然后可以将启动脚本改为下面的脚本
```
qemu-system-riscv64 \
	-M virt \
	-smp 12 -m 16G \
	-cpu rv64,v=true,vlen=128,vext_spec=v1.0 \ # 这里可以改 vlen 大小
	-display none -serial stdio \
	-netdev user,id=n0,hostfwd=tcp::10000-:22 -device virtio-net,netdev=n0 \
	-kernel /path/to/riscv-linux/arch/riscv/boot/Image \ # 改成之前 clone 的目录
	-device virtio-blk-device,drive=hd0 \
	-drive file=/path/to/linux-diskimg.qcow2,format=qcow2,id=hd0 \
	-append "root=/dev/vda1 rw" # 把 /dev/vda1 改成你的硬盘镜像里的根文件系统块设备，一般启动失败（kp）的时候会显示
```