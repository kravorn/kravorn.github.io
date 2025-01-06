---
title: 【探索】避免BT的流量经过mihomo内核
layout: post
index_img: /img/5-1.jpg
date: 2025-01-05 13:32:22
tags:
  - mihomo
  - qbittorrent
  - docker
categories:
  - 探索
---

之前将istoreos作为家庭服务器的系统，在上面装了mihomo和qbittorrent，但是长时间挂着qbittorrent会导致会有上万条连接经过mihomo的内核，进而吃掉大量内存，因此，这里提出一种通过docker和openclash黑名单来实现BT流量绕过mihomo内核的方法

