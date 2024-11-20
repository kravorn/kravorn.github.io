---
title: 【记录】在软路由上部署one-api聚合及使用LLMs
layout: post
index_img: /img/2-1.jpg
date: 2024-11-18 00:17:12
---
![](/img/2-1.jpg)
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

其中，`-p 3000:3000`中的第一个`3000`是宿主机的端口，可以根据需要进行修改。数据和日志将会保存在宿主机的`/home/debian/data/one-api`目录，请确保该目录存在且具有写入权限，或者更改为合适的目录。

部署完成后，通过软路由的ip:3000即可访问one-api的界面。

## Openrouter
[OpenRouter](https://openrouter.ai)为大量LLMs提供了兼容OpenAI的API。在其中创建key之后，回到one-api的界面创建新的渠道，模型重定向可以参考我的设置。之后添加新的令牌，通过令牌即可访问我们在OpenRouter中的大模型。

<div style="display: flex; justify-content: space-between;">
    <img src="/img/2-2.jpg" alt="Image 1" style="width: 49%;">
    <img src="/img/2-3.jpg" alt="Image 2" style="width: 49%;">
</div>




## Next-Chat的安装与配置
- [Next-Chat](https://github.com/ChatGPTNextWeb/ChatGPT-Next-Web)

我们同样使用docker进行部署，

```bash
docker run --name chat-next-web -d \
    -p 3001:3000 \
    yidadaa/chatgpt-next-web
```
注意端口不要和one-api等冲突。

部署完成后，
- 在设置里选择`自定义接口`，接口地址填写one-api的地址，`API Key`填写one-api中的令牌，
- 在自定义模型中注意不能忘记`@`，如`-all,+claude-3.5-sonnet@OpenRouter,+claude-3.5-haiku@OpenRouter,+op-qwen-coder@OpenRouter,+qwen-2@OpenRouter`，
- `对话摘要模型`改为便宜的, 否则会默认使用最贵的模型来生成标题。

最后，打开ip:3001就可以愉快地和LLMs对话了。