# tips

## GDB

info symbol $pc 查看符号文件路径

## ssh

``` shell
ssh -CqTnN -L 10081:172.16.9.1:10081 ngsrm.com
```

其中 `-C` 为压缩数据，`-q` 安静模式，`-T` 禁止远程分配终端，`-n` 关闭标准输入，`-N` 不执行远程命令。此外视需要还可以增加 `-f` 参数，把 ssh 放到后台运行。