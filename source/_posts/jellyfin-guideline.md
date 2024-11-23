---
title: 【探索】Jellyfin观影不完全指北
layout: post
index_img: /img/3-1.jpg
date: 2024-11-22 22:55:12
tags:
  - Jellyfin
  - mpv
  - 番剧
categories:
  - 探索
---

之前一直通过pc端 qbittorrent 下载种子然后使用 potplayer 播放的方式，但是这种方式存在几个缺点，1：电脑不想一直开机，也不想一直挂着 qbittorrent，但是有的种子动辄大半个月的下载时间，不长期挂着很难下完；2：通过文件夹管理的方式不方便且不美观，因此，想在软路由上整一个jellyfin 海报墙。此方法同样适用于PC本地配置。

![](/img/3-1.jpg)

## 1 Jellyfin安装与配置
依然推荐通过 docker 安装
```bash
docker run -d \
 --name jellyfin \
 -p 8096:8096 \
 --user uid:gid \
 --net=host \
 --volume jellyfin-config:/config
 --volume jellyfin-cache:/cache
 --mount type=bind,source=/path/to/media,target=/media \
 --restart=unless-stopped \
 jellyfin/jellyfin
```
初始化跟着指引一步一步来就好，**不过建议在全部配置好后再添加媒体库**。

另外，推荐完全放弃在宿主机上解码，在 用户设置 > 媒体播放 里，**仅**勾选`允许播放媒体`，让客户端来进行解码，毕竟现在手机都可以软解4K了。

### 1.1 常用插件

#### 1.1.1 二次元
对于一般使用来说，自带的TMDB插件已经足够，不过对于看番来说，还需要一个[Bangumi](https://github.com/kookxiang/jellyfin-plugin-bangumi)，

- 进入 Jellyfin 控制台 > 插件目录/存储库 > 设置，点击添加
- 输入存储库名称：Bangumi
- 输入存储库 URL：https://jellyfin-plugin-bangumi.pages.dev/repository.json
- 在插件目录下找到 MetaTube，点击安装
- 重启 Jellyfin
- 对番剧选择`节目`作为媒体库类型，对剧场版则选择`电影`
- 勾选 Bangumi 作为元数据下载器与图片获取器。

#### 1.1.2 ???
推荐 [MetaTube](https://github.com/metatube-community/jellyfin-plugin-metatube)，先通过 docker 部署后端
```bash
docker run -d -p 8080:8080 \ 
 -v $PWD/config:/config \
 --name metatube metatube/metatube-server:latest \
 -dsn /config/metatube.db
```
然后，
- 进入 Jellyfin 控制台 > 插件目录/存储库 > 设置，点击添加
- 输入存储库名称：MetaTube
- 输入存储库 URL：https://raw.githubusercontent.com/metatube-community/jellyfin-plugin-metatube/dist/manifest.json
- 在插件目录下找到 MetaTube，点击安装
- 进入 MetaTube 插件所在的配置页面。
- 输入之前配置好的后端地址 URL 以及需要的后端密钥 Token
- 重启 Jellyfin
- 进入需要使用插件的媒体库：
  - 务必选择电影作为媒体库类型
  - **仅**勾选 MetaTube 作为元数据下载器与图片获取器。

### 1.2 命名规范
想要刮削到正确的信息，对文件夹和文件的命名必须规范。
#### 1.2.1 番剧
对于番剧我建议每一季都单独建立一个文件夹，如果将每一季作为`子`文件夹放在`主`文件夹里，经常出现刮削效果不好的情况。番外等内容可以全部放入`extras`文件夹内。如果是外挂字幕，一定要将字幕和视频文件放在一起，且文件名必须一致。

这里介绍一下我的命名方式：

- 首先去[番组计划](https://bgm.tv/)找到番剧对应的名字，如`租借女友 第二季`
- 将每一个视频文件命名为类似于`S02E01-原视频文件名.mkv`这样的名字

最终的文件结构应该是
```tree
动画
├── 租借女友
│   ├── S01E01-原视频文件名.mkv
│   ├── S01E01-原视频文件名.srt
│   └── extras
│       └── sp1.mkv
└── 租借女友 第二季
    ├── S02E01-原视频文件名.mkv
    ├── S02E01-原视频文件名.srt
    └── extras
        └── sp1.mkv
```
如果番剧数量特别多的话，可以使用[ReNamer](https://www.den4b.com/downloads/renamer)来批量重命名。

#### 1.2.2 ???
- 字母-数字，可以一部一个文件夹，也可以把所有视频文件都放在一个文件夹内。
```tree
???[第一种方法]
├── xx-xx1.mp4
└── xx-xx2.mp4

???[第二种方法]
├── xx-xx1
│   └── xx-xx1.mp4
└── xx-xx2
    └── xx-xx2.mp4
```

### 1.3 播放器

#### 1.3.1 电脑端
推荐使用[jellyfin-mpv-shim](https://github.com/jellyfin/jellyfin-mpv-shim)调用外部配置好的mpv播放器，启动jellyfin-mpv-shim后，在浏览器中打开`ip:8096`，就可以选择其作为默认播放方法。

jellyfin-mpv-shim中需要修改的conf.json设置如下：

```json
"enable_gui": false,
"enable_osc": false,
"fullscreen": false,
"mpv_ext": true,
"mpv_ext_ipc": null,
"mpv_ext_no_ovr": true,
"mpv_ext_path": "D:\\mpv-lazy\\mpv.exe",
"thumbnail_enable": false,
"thumbnail_osc_builtin": false,
```

#### 1.3.2 iPhone/iPad
- 推荐[Swiftfin](https://github.com/jellyfin/Swiftfin)。


### 1.4 其他
- 想要追求全自动化，可以使用NASTool之类的工具，这里不做介绍。
- 如果想在重命名、移动文件的情况下保种，必须借助硬链接，可以使用[hlink](https://github.com/likun7981/hlink)，这里不做介绍。


## 2 BT下载
推荐使用[增强版qbittorrent](https://github.com/c0re100/qBittorrent-Enhanced-Edition)，同时搭配[PeerBanHelper](https://github.com/PBH-BTN/PeerBanHelper)，防止国内流氓软件吸血。

- qbittorrent安装

```bash
docker run \
  --name=qbittorrent \
  -e QB_WEBUI_PORT=8989 \
  -e QB_EE_BIN=false \
  -e UID=1000 \
  -e GID=1000 \
  -e UMASK=022 \
  -p 6881:6881 \
  -p 6881:6881/udp \
  -p 8989:8989 \
  -v /配置文件位置:/config \
  -v /下载位置:/Downloads \
  --restart unless-stopped \
  johngong/qbittorrent:latest
```

- PeerBanHelper安装：

```bash
docker run -d --name peerbanhelper --stop-timeout 15 -p 9898:9898 -v ${PWD}/:/app/data/ ghostchu/peerbanhelper:latest
```

其使用可以参考这本[手册](https://docs.pbh-btn.com/docs/intro)。

## 3 mpv
- 推荐[mpv-lazy](https://github.com/hooke007/MPV_lazy)，可以大幅优化初次使用的体验。

## 4 potplayer
- 推荐potplayer+lavfilters+madvr以获得最佳观影体验，可以参考[此教程](https://vcb-s.com/archives/7228/comment-page-10)，如果想深入调教，可以参考这篇[文章](https://lysandria1985.blogspot.com/2013/01/3-madvr.html);
- 如果不喜欢折腾，可以参考这一套potplayer的[配置](https://hooke007.github.io/DirectShow+/mpc.html);
- 在jellyfin里调用potplayer可以使用[embyToLocalPlayer](https://github.com/kjtsune/embyToLocalPlayer)。