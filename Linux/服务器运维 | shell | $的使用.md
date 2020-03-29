> $$ 是Shell本身的PID（ProcessID）
> $0 是脚本本身的名字
> $1 是传递给该shell脚本的第一个参数
> $2 是传递给该shell脚本的第二个参数
> $@ 是传给脚本的所有参数的列表
> $# 是传给脚本的参数个数

`tesh.sh` 内容如下

```
echo $$
echo $0
echo $1
echo $2
echo $@
echo $#
```

运行 `tesh.sh`输出结果如下

```
[root@localhost ~]# sh test.sh aaa bbb ccc ddd
3431
test.sh
aaa
bbb
aaa bbb ccc ddd
4
```