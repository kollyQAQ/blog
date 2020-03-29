[TOC]

#### 1.命令格式

> grep [option] pattern file

#### 2. 命令功能

用于过滤/搜索的特定字符。可使用正则表达式能多种命令配合使用，使用上十分灵活

#### 3. 命令参数

- -c    --count   #计算符合样式的列数
- -e<范本样式>  --regexp=<范本样式>   #指定字符串做为查找文件内容的样式
- -E      --extended-regexp   #将样式为延伸的普通表示法来使用
- -v   --revert-match   #显示不包含匹配文本的所有行

#### 4. 使用示例

`grep -v` 排除

```
[root@localhost test]# ll
-rw-r--r-- 1 root root  260 5月   2 19:57 data.txt
-rw-r--r-- 1 root root  221 5月   2 15:35 process.sh
-rw-r--r-- 1 root root   59 4月  26 10:50 test.sh
-rw-r--r-- 1 root root  603 5月   2 20:27 uniq.sh
-rw-r--r-- 1 root root 9551 4月  26 11:52 vim.txt
[root@localhost test]# ll | grep sh | grep -v test
-rw-r--r-- 1 root root  221 5月   2 15:35 process.sh
-rw-r--r-- 1 root root  603 5月   2 20:27 uniq.sh
```

`grep -E`  正则表达式 以下为或条件

```
[root@localhost test]# ll
-rw-r--r-- 1 root root    0 5月   3 10:45 123.txt
-rw-r--r-- 1 root root    0 5月   3 10:45 12abc.txt
-rw-r--r-- 1 root root   59 4月  26 10:50 test.sh
[root@localhost test]# ll | grep -E "12.*"
-rw-r--r-- 1 root root    0 5月   3 10:45 123.txt
-rw-r--r-- 1 root root    0 5月   3 10:45 12abc.txt

```

