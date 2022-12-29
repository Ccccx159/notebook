# Watchdog for Emby Media Server

## 修订版本

<mark><font color="red">v1.x版本后续将不再更新维护，如有需要请更新使用v2.x版本！！！</font></mark>

## 简介

借助python中的看门狗模块（“watchdog”）监视emby媒体库目录，通过电报（telegram）的bot和channel，向频道订阅者推送Emby媒体库中新增影片信息，包括电影和剧集。

## 实现说明

v2.x版本中，删除了原始版本中的xmllint依赖，仅通过python完成所有功能实现。**因此在dockerfile中将基础镜像由`ubuntu:latest`变更为`python:alpine3.17`，拉取后镜像体积由231MB减小至69.5MB，体积减少约70%**。

**watchdog_for_Emby** 对 Emby Server 自动影片刮削生成的“xxxx.nfo”文件进行监控。影片新入库后，Emby Server 自动执行刮削生成xml格式的nfo文件，~~通过xmllint可以解析到部分该影片或者剧集的信息~~通过“ElementTree”模块解析nfo文件，获取当前影片的基本信息。而影片的封面图，和剧集的详细信息，则需要通过TMDB的api进行查询获取，通过调用"requests.get()"方法完成查询。在按照电报bot的api文档对payload数据组装后，调用"requests.post()"方法推送给bot，由bot发布至对应频道。

## 依赖项

1. python3.10及以上版本（v2.x版本中，使用了match..case..语法，仅在3.10及以上版本完成支持）
2. python Module: *watchdog*, *requests* (cmd: `pip3 install watchdog requests`)，*ElementTree*
3. ~~xmllint (os: ubuntu 20.04，cmd: `sudo apt-get install libxml2-utils`)~~ v2.x版本中已去除此依赖

## 环境变量设置

| 参数 | 说明 |
| -- | -- |
| BOT_TOKEN | 电报 bot token |
| CHAT_ID | 电报频道 chat_id |
| TMDB_API | TMDB api token |
| MEDIA_PATH | Emby 媒体库路径 |
| LOG_PATH | <可选>日志文件路径，默认为`/var/tmp/overwatch.log` |

## Docker Run

~~~shell
docker run -d --name=watchdog-emby --restart=unless-stopped \
  -v "your media lib's host path":"media lib's container path" \
  -e BOT_TOKEN="your telegram bot's token" \
  -e CHAT_ID="your telegram channle's chat_id" \
  -e TMDB_API="tmdb api token" \
  -e MEDIA_PATH="media lib's container path" \
  -e LOG_PATH="log's output path" \
  b1gfac3c4t/overwatch
  
~~~

## 效果展示

电影：

![](https://user-images.githubusercontent.com/35327600/209752390-4e45180b-d8cc-4378-bd98-c489638f7cb7.png)

剧集：

![](https://user-images.githubusercontent.com/35327600/209752275-bad230b0-97a7-47e5-9a77-081afae7d6cf.png)

## 参考文档

+ tmdb api 文档：https://developers.themoviedb.org/3
+ telegram bot api 文档：https://core.telegram.org/bots/api
