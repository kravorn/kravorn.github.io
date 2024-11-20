---
title: 【记录】在软路由上部署one-api聚合及使用LLMs
layout: post
index_img: /img/2.jpg
date: 2024-11-18 00:17:12
---
![](/img/2.jpg)
## 部署one-api
[one-api](https://github.com/songquanpeng/one-api) 是 OpenAI 接口管理 & 分发系统，支持 Azure、Anthropic Claude、Google PaLM 2 & Gemini、智谱 ChatGLM、百度文心一言、讯飞星火认知、阿里通义千问、360 智脑以及腾讯混元，可用于二次分发管理 key。

### 通过Docker部署
```bash
docker run --name one-api -d --restart always \
    -p 3000:3000 \
    -e TZ=Asia/Shanghai \
    -v /home/debian/data/one-api:/data \
    justsong/one-api
```

其中，`-p 3000:3000`中的第一个`3000`是宿主机的端口，可以根据需要进行修改。

数据和日志将会保存在宿主机的`/home/debian/data/one-api`目录，请确保该目录存在且具有写入权限，或者更改为合适的目录。

## Openrouter

## Next-Chat的安装与配置