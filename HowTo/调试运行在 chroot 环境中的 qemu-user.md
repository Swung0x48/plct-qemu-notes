1. 准备工作

Host：运行 chroot 环境的地方（通常指的是 x86_64 的环境，或者任意的可以跑 qemu user static 的物理机器）

chroot 环境：chroot/pychroot/systemd-nspawn 等工具起来的容器环境内部，通过 qemu user riscv64 static 来模拟 rv64 的环境。（如果还没有环境的话可以用 https://github.com/felixonmars/archriscv-packages/wiki/Setup-Arch-Linux-RISC-V-Development-Environment 起一个）

1. 安装 gdb-multiarch

首先需要安装 gdb-multiarch，archlinux 有个 aur https://aur.archlinux.org/packages/gdb-multiarch

1. 查看 binfmt 状态

`$ cat /proc/sys/fs/binfmt_misc/qemu-riscv64`

```bash
enabled
interpreter /usr/bin/qemu-riscv64-static
flags: POCF
offset 0
magic 7f454c460201010000000000000000000200f300
mask ffffffffffffff00fffffffffffffffffeffffff
```

类似如上输出则说明 binfmt 已经配置好了

此时，假定 host 上的 qemu static 的路径为 `$QEMU` （本例中为 `/usr/bin/qemu-riscv64-static`，上一步中 `interpreter` 栏的值）, chroot 环境的 rootfs 为 `$rootfs` （如果按照上文肥猫的教程的话应该是 `./archriscv` ）

1. 把 QEMU static 拷贝进 rootfs

`(host)$ cp $QEMU $rootfs/path/to/qemu-riscv64-static`

1. 用刚才拷贝的 qemu-user-static 启动要调试的任意的 userprogram，并开启 gdbstub

`(chroot)$ /path/to/qemu-riscv64-static -g 1234 /path/to/userprogram` 

特别地，在这边是用于 debug 正在开发中的 qemu-user

`(chroot)$ /path/to/qemu-riscv64-static -g 1234 /path/to/plct-qemu/build/qemu-x86_64 -cpu Haswell-v1 ./avx` 

1. 在 host 侧启动 `gdb-multiarch`，并连接 gdbstub

```bash
(host)$ sudo gdb-multiarch
(gdb) file /path/to/plct-qemu/build/qemu-x86_64
(gdb) target remote localhost:1234
# 此时已经开始 remote debug 了
```