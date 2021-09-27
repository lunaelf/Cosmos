# Cosmos

把下面的代码复制，然后插入到 /boot/grub/grub.cfg 文件中，然后把 HelloOS.bin 文件复制到 /boot/ 目录下，最后重启计算机就可以看到 HelloOS 启动选项了。

```
menuentry 'HelloOS' {
    insmod part_gpt
    insmod ext2
    set root='hd0,gpt2' # 注意 boot 目录挂载的分区，这是我机器上的情况
    multiboot2 /HelloOS.bin
    boot
}
```

## 参考资料

[操作系统实战：实验环境搭建、热门问题解答](https://blog.csdn.net/damiaomiao666/article/details/116914684)
