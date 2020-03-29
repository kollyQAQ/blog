[TOC]

#### PIcGo项目地址

- 项目地址:https://github.com/Molunerfinn/PicGo
- 下载地址:https://github.com/Molunerfinn/PicGo/releases
- 作者博客:[https://molunerfinn.com](https://molunerfinn.com/)
- 使用手册:[https://picgo.github.io/PicGo-Doc/zh/guide/config.html](https://picgo.github.io/PicGo-Doc/zh/guide/config.html)

**看一下使用手册就能知道用法，非常简单实用**

#### 遇到的问题

环境：macOS Mojave 10.14.5

PicGo版本：2.1.2

问题：剪贴板图片上传bug，选择剪切板上传文件的时候报错，提示`请检查配置和上传的文件`，打开日志文件发现报错信息为

```
2019-07-10 10:37:43 [PicGo ERROR] *** Error creating a JP2 color space: falling back to sRGB
```

解决方案：系统偏好设置-》显示器-》颜色，显示描述文件选择`普通描述文件`