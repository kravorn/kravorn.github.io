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

抛弃NeatChat，拥抱OpenWebUI！


![](../img/openwebui/image.png)


## OpenWebUI相比NeatChat有哪些优劣
- 劣势：
  - 需要更多内存和cpu性能，本人测试下来至少要2G内存
  - 不原生支持历史消息截断和压缩，必须借助外部function
  - 不支持不同用户使用独立的api-key，好像开发者说这是有意为之

- 优势：
  - 自带功能丰富，支持文件全文上下文、文件RAG、知识库、联网搜索（当然其中很多neatchat也可以通过插件来实现，有的甚至比openwebui还要做得好，比如网页浏览）
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
openwebui的RAG性能一般，只能做为尝鲜
可以调用本地/外部的嵌入模型和本地的重排模型，但是本地重排特别特别慢。


## 联网搜索
个人做的联网搜索再怎么样也很难比过openai和gemini的，很难应用到实际生产中去


## Token用量监控
除非是自己一个人用，不然还是加上这个控制一下用量吧


## 历史消息截断
openwebui会带着所有的历史消息作为输入，累积下来这个token用量过于夸张

## 反代
```yaml
services:
  nginx-proxy-manager:
    image: 'docker.io/jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt

networks:
  default:
    external: true
    name: ngpm
```

https://www.cnblogs.com/Hollow2333/p/18694512