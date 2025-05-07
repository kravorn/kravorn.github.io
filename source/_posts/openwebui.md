---
title: 【记录】部署OpenWebUI
layout: post
index_img: ../img/openwebui/image.png
date: 2025-04-04 21:11:45
tags:
  - OpenAI
  - OpenWebUi
  - api
categories:
  - 服务器
---

拥抱OpenWebUI！


![](../img/openwebui/image.png)


## OpenWebUI相比NeatChat有哪些优劣
- 劣势：
  - 需要更多内存和cpu性能，本人测试下来至少要1G内存
  - 不原生支持历史消息截断和压缩，必须借助外部function
  - 不支持不同用户使用独立的api-key，好像开发者说这是有意为之

- 优势：
  - 自带功能丰富，支持文件全文上下文、文件RAG、知识库、联网搜索（当然其中很多功能neatchat也可以通过插件来实现，有的甚至比openwebui还要做得好，比如网页浏览）
  - 具有用户管理功能，这也是最吸引我的一点，不然只能通过http basic authentication来控制



## 安装与配置
```yaml
services:
  openwebui:
    image: ghcr.io/open-webui/open-webui:main
    volumes:
      - open-webui:/app/backend/data
    environment:
      - ENABLE_ADMIN_CHAT_ACCESS=False
      - ENABLE_COMMUNITY_SHARING=False
      - ENABLE_EVALUATION_ARENA_MODELS=False
      - ENABLE_MESSAGE_RATING=False
      - ENABLE_AUTOCOMPLETE_GENERATION=False
volumes:
  open-webui:

networks:
  default:
    external: true
    name: ngpm

```

## RAG
openwebui的RAG性能一般，只能做为尝鲜。可以在设置页面的`Document`中设置嵌入模型，推荐硅基流动或者nvidia的bge-m3。

关于重排模型，2C4G的服务器本地重排特别特别慢。但是OpenWebUI也没有提供api选项...


## 联网搜索
个人做的联网搜索再怎么样也很难比过openai和gemini的，很难应用到实际生产中去。而且用多了会出现获取不到网页信息的情况。

谷歌PSE可以参考这篇[教程](https://docs.openwebui.com/tutorials/web-search/google-pse/)，我建议在谷歌PSE控制面板里可以先设置好搜索范围，比如如果是以新闻为主，可以限定搜索网站到CNN、网易之类的。


## Token用量监控
除非是自己一个人用，不然还是加上OpenWebUI-monitor控制一下用量吧。

跟着[部署流程](https://github.com/VariantConst/OpenWebUI-Monitor)走就好。


## 历史消息截断
openwebui会带着所有的历史消息作为输入，累积下来这个token用量会非常夸张，可以使用下面这个函数：
https://openwebui.com/f/hub/context_clip_filter

## 反代
基于nginx-proxy-manager这个docker，可以参考之前的博客。