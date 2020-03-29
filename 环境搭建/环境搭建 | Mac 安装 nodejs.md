[TOC]

#### 通过homebrew安装node

在终端中输入命令如下

```
brew install node
```

#### 验证node是否安装成功

```
node -v
npm -v
```

能正确显示版本号即表示node安装成功

#### 安装cnpm

npm由于源服务器在国外下载node包经常超时，cnpm使用国内镜像，通过以下命令安装cnpm

```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

#### 安装 npm 依赖

```
npm install
```

#### bulid

项目根目录，执行

```
npm run build
```

