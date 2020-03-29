[TOC]

####  建立目录

```shell
cd /usr/lib/
mkdir jvm
```

#### 下载软件包

```shell
cd /usr/src

wget  https://download.oracle.com/otn/java/jdk/8u211-b12/478a62b7d4e34b78b671c754eaaf38ab/jdk-8u211-linux-x64.tar.gz

tar zxvf jdk-8u211-linux-x64.tar.gz -C /usr/lib/jvm/
```

wget 失败也可以下载到本地再上传到 linux 服务器

#### 修改环境变量

```shell
vi ~/.profile
```

在文件最后加上

```shell
export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_151
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

#### 更新环境变量

```shell
source ~/.profile
```

#### 验证是否安装成功

```shell
java -version
```