---
title: convert azw3 to mobi
date: 2017-12-12 18:45:05
categories:
- Tip
- Kindle
tags:
- Kindle
- mobi
- azw3
---

亚马逊购买的图书一般下载下来是 azw3 格式，将 azw3 格式转换为 mobi 可以方便发送到邮箱来同步到 kindle。

<!--more-->

## 下载完整版图书
首先进入[管理我的内容和设备](https://www.amazon.cn/gp/digital/fiona/manage)，点击操作，通过电脑下载 USB 传输，然后选择一个 Kindle 
设备进行下载。

{% asset_img download.webp download %}

因为有的 azw3 书籍文字和图片是分离的，从亚马逊网页下载的文件是包含图片的。

## 去除 DRM
使用 [DeDRM_tools](https://github.com/apprenticeharper/DeDRM_tools) 去除 DRM，这里用 calibre 的插件来实现，这个工具也有命令行版本。

{% asset_img deDrm.webp deDrm %}

使用该工具之前需要设置刚才下载图书选择的设备的序列号，可以通过刚才页面里面我的设备查看，也可以直接在 Kindle 中查看。

{% asset_img id.webp id %}

设置好之后直接将下载的 azw3 文件拖入到 calibre 就可以了。可是双击试试能否打开，不能打开说明 DeDRM 失败了。

## 解压 azw3 为 epub
使用 [KindleUnpack](https://github.com/kevinhendricks/KindleUnpack) 来将去除了 DRM 的 azw3 文件转换为 epub，这里还是使
用 calibre 插件来实现。

{% asset_img unpack.webp unpack %}

解压后的 epub 文件和 azw3 文件在同一目录，在 calibre 里面点击书籍下方的路径：点击打开，就可以找到。

## 将 epub 转为 mobi
使用亚马逊官方的 [Kindle Previewer](https://www.amazon.com/gp/feature.html?docId=1000765261) 来将 epub 制作成 mobi。

{% asset_img makeMobi.webp makeMobi %}

直接将上一步做好的 epub 拖入 Kindle Previewer 即可生成 mobi 文件，会在同级目录下生成一个文件夹。
文件夹里的 mobi 就是完全保留原书格式的 mobi 版本文件。