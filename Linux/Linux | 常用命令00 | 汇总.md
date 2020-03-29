### 系统命令 
| 命令           | 功能             |
| -------------- | ---------------- |
| man            | 查看帮助         |
| uname -a       | 显示操作系统信息 |
| hostname       | 显示主机名称     |
| whoami         | 查看当前登录用户 |
| lscpu          | 查看 cpu 信息    |
| reboot         | 重启系统         |
| shutdown -r | 关机重启         |
| shutdown -h  | 关机不重启       |
| shutdown now  | 立刻重启         |

### 文件 & 目录
| 命令             | 功能                                |
| ---------------- | ----------------------------------- |
| mkdir  -p {path} | 创建目录，若无父目录，则创建        |
| wc {filename}    | 显示操作系统信息                    |
| tree             | 树形结构显示目录，需要安装 tree命令 |

### 压缩 & 解压缩
| 命令           | 功能             |
| -------------- | ---------------- |
| tar  -cvf    aaa.tar *          |     打包     |
| tar  -czvf  aaa.tar.gz *       | 打包&压缩 |
| tar  -xvf    aaa.tar       |   解压未压缩包   |
| tar  -xzvf  aaa.tat.gz         | 解压压缩包  |
| tar  -tvf    aaa.tar          |   查阅  |
| zip aaa.zip * | 压缩当前目录所有文件到 aaa.zip |
| zip -r bbb * | 压缩当前目录所有文件和文件夹到 bbb.zip |
| unzip aaa.zip | 解压缩 aaa.zip |
| unzip -v bbb.zip | 我不想解压缩，只想看看它里面有什么 |
| unzip -j bbb.zip | 把文件和文件夹里的文件都下载到第一级目录，而不是一层一层建目录 |

> tar 命令参数
> -c：建立一个压缩档案的参数指令(create 的意思)； 
> -x：解开一个压缩档案的参数指令 
> -t：查看 tarfile 里面的档案 
> -z：使用gzip压缩 
> -v：压缩的过程中显示档案(后台执行不要用) 
> -f：使用档名（f必须放在参数最后） 

### 系统资源查看
| 命令           | 功能             |
| -------------- | ---------------- |
| top            |  |
| df -h       |  |
| du -h       |      |
| du --max-depth=1 -h         |  |
| free -h          |     |
| iostat          |     |

### 网端口查看
| 命令           | 功能             |
| -------------- | ---------------- |
| netstat -anp \| grep 12091 |             |
| lsof -i:12091 |  |

