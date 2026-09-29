# MorePlay

MorePlay 是一个面向 Docker 与 NAS 场景的影视源多协议桥接服务。

它可以加载 CatPawOpen 猫源，并接入兼容的影视仓 / TVBox HTTP 数据源，将统一的资源目录、详情与播放能力转换为 Forward / Rex、Emby、STRM 和 T4 等不同使用方式。

> MorePlay 本身不提供影视内容，也不提供播放器。资源目录、详情与播放结果均来自用户自行配置的上游数据源。

## 主要功能

- 支持 CatPawOpen 猫源加载、校验、更新与独立配置。
- 支持接入兼容的影视仓 / TVBox HTTP 数据源。
- 支持多个输入源统一管理，并为不同协议提供独立的站点范围与运行设置。
- 提供 Forward / Rex 模块，可直接生成订阅地址。
- 提供 Emby 兼容接口，可用于媒体浏览、搜索、线路选择与播放。
- 支持生成 STRM、NFO 与相关媒体信息，供媒体库应用扫描使用。
- 提供 T4 配置与 VOD 接口，兼容常见 TVBox / CatVod 使用方式。
- 支持媒体直连（302）与 NAS 转发两种播放链路。
- 支持海报与图片代理、来源网络代理及相关兼容处理。
- 提供 Web 管理台，用于管理输入源、媒体服务、播放设置、日志与运行状态。
- 支持 `amd64` 与 `arm64` Docker 环境。

## 快速部署

### Docker 镜像

```text
mobai1231/catpawopen-fw:latest
```

如需固定版本，可将 `latest` 替换为对应的已发布版本标签。

### NAS 最简部署

如果 NAS 的容器管理器支持 Docker Compose，可直接使用下面的配置：

```yaml
services:
  catpaw-open-bridge:
    image: mobai1231/catpawopen-fw:latest
    container_name: catpaw-open-bridge
    restart: unless-stopped

    ports:
      - "2333:2333"

    environment:
      CATPAW_SOURCE_URL: "请填写你的猫源地址"

    volumes:
      - catpaw-data:/data
      - ./strm-library:/strm

volumes:
  catpaw-data:
```

部署完成后访问：

```text
http://<NAS地址>:2333/bridge
```

其中：

- `CATPAW_SOURCE_URL`：填写自己的猫源地址。
- `catpaw-data:/data`：保存 MorePlay 的配置和运行数据。
- `./strm-library:/strm`：将 STRM 媒体目录保存在 Compose 项目目录下的 `strm-library` 文件夹中。
- `2333:2333`：MorePlay 的 Web 管理台及协议服务端口。

### 创建数据卷

```bash
docker volume create moreplay-data
docker volume create moreplay-source
docker volume create moreplay-strm
```

### 启动容器

```bash
docker run -d \
  --name moreplay \
  --restart unless-stopped \
  -p 2333:2333 \
  -e CATPAW_SETTINGS_PASSWORD=请替换为自己的管理密码 \
  -v moreplay-data:/data \
  -v moreplay-source:/source \
  -v moreplay-strm:/strm \
  mobai1231/catpawopen-fw:latest
```

启动完成后访问：

```text
http://<服务器地址>:2333/bridge
```

首次进入管理台后，可继续配置猫源、影视仓 / TVBox 输入源以及各媒体服务。

## 从源码构建 Docker 镜像

```bash
git clone https://github.com/xdbiaoge/moreplay.git
cd moreplay

docker build -t moreplay .
```

构建完成后可使用：

```bash
docker run -d \
  --name moreplay \
  --restart unless-stopped \
  -p 2333:2333 \
  -e CATPAW_SETTINGS_PASSWORD=请替换为自己的管理密码 \
  -v moreplay-data:/data \
  -v moreplay-source:/source \
  -v moreplay-strm:/strm \
  moreplay
```

## 数据目录

MorePlay 将运行数据与 STRM 媒体目录分开保存。

| 容器目录 | 用途 |
| --- | --- |
| `/data` | 管理设置、Profile、协议配置、缓存、索引及其它持久化运行数据 |
| `/source` | 本地猫源文件目录；仅使用在线猫源时可以保持为空 |
| `/strm` | STRM、NFO、图片及下载后的媒体文件 |

升级或迁移时，建议至少备份 `/data` 和 `/strm`。

如果使用本地猫源，也应同时保留 `/source`。

## 常用地址

假设服务地址为：

```text
http://<服务器地址>:2333
```

常用入口如下：

| 功能 | 地址 |
| --- | --- |
| 管理台 | `/bridge` |
| 媒体服务 | `/bridge/services` |
| 站源管理 | `/bridge/sites` |
| STRM 管理 | `/bridge/strm` |
| Emby 管理 | `/bridge/emby` |
| T4 管理 | `/bridge/t4` |
| Forward / Rex 管理 | `/bridge/fw` |
| Forward / Rex 模块 | `/bridge/widget.js` |
| 播放诊断 | `/bridge/playback-diagnostics` |
| 日志 | `/bridge/logs` |
| 系统设置 | `/bridge/system` |
| T4 配置 | `/t4/config.json` |
| T4 VOD API | `/t4/vod` |
| 健康检查 | `/check` |

### Emby

Emby 兼容服务直接使用 MorePlay 根地址：

```text
http://<服务器地址>:2333
```

账号、站点范围及相关设置可在：

```text
/bridge/emby
```

中管理。

### Forward / Rex

模块订阅地址：

```text
http://<服务器地址>:2333/bridge/widget.js
```

相关设置：

```text
http://<服务器地址>:2333/bridge/fw
```

### STRM

STRM 管理入口：

```text
http://<服务器地址>:2333/bridge/strm
```

生成的 STRM、NFO、图片及媒体文件写入 `/strm`。

### T4

配置地址：

```text
http://<服务器地址>:2333/t4/config.json
```

VOD 接口：

```text
http://<服务器地址>:2333/t4/vod
```

## 输入源

### CatPawOpen 猫源

MorePlay 支持在线猫源和本地猫源。

在线模式可通过管理台配置，也可以通过环境变量指定：

```text
CATPAW_SOURCE_URL
```

启用完整性校验时：

```text
REQUIRE_MD5=true
```

校验失败的源码不会被加载。

### 影视仓 / TVBox

可在管理台添加兼容的影视仓配置或单站 HTTP 地址。

不同输入源可分别维护自己的配置、网络代理和运行状态，并统一提供给 Forward / Rex、Emby、STRM 与 T4 使用。

部分需要额外 Spider 运行环境的站点，可通过以下变量接入对应运行服务：

```text
CATPAW_TVBOX_RUNTIME_URL
CATPAW_TVBOX_RUNTIME_TOKEN
```

普通 HTTP 数据源无需配置该项。

## 播放链路

MorePlay 支持两种主要播放方式。

### 直连模式

服务端完成资源解析后，通过 HTTP 302 将播放器引导到最终媒体地址。

适合播放器能够直接访问最终媒体地址的环境，可减少 NAS 的媒体转发流量。

### NAS 转发

媒体流量经过 MorePlay 所在设备转发。

适合最终媒体需要服务端网络环境、请求头、Cookie、代理或其它运行状态的场景。

播放方式可在管理台的媒体服务设置中调整。

## 图片与海报

MorePlay 会根据图片来源和输入源配置决定是否直接返回原始图片地址或通过服务端代理。

如某些图片域名需要强制经过图片代理，可以使用：

```text
CATPAW_POSTER_PROXY_HOSTS
```

多个域名使用逗号分隔。

## TMDB

STRM 的媒体信息匹配可选用 TMDB。

可以在管理台设置，也可以使用环境变量：

```text
CATPAW_TMDB_API_KEY
CATPAW_TMDB_READ_TOKEN
CATPAW_TMDB_PROXY
```

不使用 TMDB 时可保持为空。

## 常用环境变量

完整可选项可参考仓库中的 `.env.example`。

| 变量 | 说明 |
| --- | --- |
| `CATPAW_SETTINGS_PASSWORD` | 管理台密码 |
| `CATPAW_SOURCE_URL` | 在线猫源地址 |
| `REQUIRE_MD5` | 是否要求猫源文件通过 MD5 校验 |
| `CATPAW_SOURCE_TIMEOUT_MS` | 猫源下载超时时间 |
| `CATPAW_SEARCH_TIMEOUT_MS` | 搜索总等待时间 |
| `CATPAW_POSTER_TIMEOUT_MS` | 图片请求超时时间 |
| `CATPAW_POSTER_PROXY_HOSTS` | 强制使用图片代理的域名列表 |
| `CATPAW_TVBOX_TIMEOUT_MS` | 影视仓 / TVBox HTTP 请求超时时间 |
| `CATPAW_TVBOX_RUNTIME_URL` | 可选 Spider 运行服务地址 |
| `CATPAW_TVBOX_RUNTIME_TOKEN` | 可选 Spider 运行服务鉴权 Token |
| `CATPAW_TMDB_API_KEY` | TMDB API Key |
| `CATPAW_TMDB_READ_TOKEN` | TMDB Read Token |
| `CATPAW_TMDB_PROXY` | TMDB 网络代理 |
| `EMBY_CATALOG_CACHE_TTL_MS` | Emby 目录缓存有效时间 |
| `EMBY_LIST_CACHE_TTL_MS` | Emby 列表缓存有效时间 |

没有明确需要时，建议保持默认值。

## 更新镜像

先拉取最新镜像：

```bash
docker pull mobai1231/catpawopen-fw:latest
```

随后使用原有端口、环境变量和数据卷重新创建容器即可。

只要继续挂载原来的 `/data` 和 `/strm`，已保存的配置和 STRM 数据不会因为容器重建而丢失。

升级前建议备份重要数据。

## 安全建议

- 为管理台设置独立的强密码。
- 不要将管理端口直接暴露到不受信任的网络。
- 不要把 Token、Cookie、API Key 等内容提交到公开仓库。
- 对外访问时建议通过受控的反向代理和 HTTPS 提供服务。
- 仅添加自己信任的猫源、影视仓和其它外部数据源。

## 项目说明

MorePlay 只提供数据源接入、协议转换、目录整理和播放桥接能力，不存储或提供第三方影视内容。

上游数据源、接口、媒体文件及相关服务均由其各自提供方负责。使用者应自行确认所使用数据源和内容符合所在地法律法规、服务条款及授权要求。

本项目按现状提供，不对第三方数据源的可用性、稳定性、内容准确性或持续兼容性作保证。
