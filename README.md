<p align="center">
  <img src="docs/logo-moreplay.png" alt="MorePlay Logo" width="180">
</p>

# MorePlay

MorePlay 是一个面向 Docker 与 NAS 场景的影视源多协议桥接服务。

它可以加载 CatPawOpen 猫源，并接入兼容的影视仓 / TVBox HTTP 数据源，将统一的资源目录、详情与播放能力转换为 Forward / Rex、Emby、STRM 和 T4 等不同使用方式。

> MorePlay 本身不提供影视内容，也不提供播放器。资源目录、详情与播放结果均来自用户自行配置的上游数据源。

本项目为闭源软件。本仓库用于提供使用文档、界面预览和用户更新说明，程序通过 Docker 镜像分发。

本文按正式版本 **2.7.1** 介绍。可先阅读 [快速部署](#快速部署)、[首次使用](#首次使用) 和 [常见问题](#常见问题)；各版本更新日志见 [Releases](https://github.com/xdbiaoge/moreplay/releases)。

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

## 界面预览

<p align="center">
  <img src="docs/screenshots/dashboard-overview.webp" alt="MorePlay 管理台概览" width="100%">
</p>

<p align="center">
  <sub>管理台概览</sub>
</p>

<table>
  <tr>
    <td width="50%" align="center">
      <img src="docs/screenshots/sites-management.webp" alt="MorePlay 站源管理" width="100%">
      <br>
      <sub>站源管理</sub>
    </td>
    <td width="50%" align="center">
      <img src="docs/screenshots/services-management.webp" alt="MorePlay 媒体服务" width="100%">
      <br>
      <sub>媒体服务</sub>
    </td>
  </tr>
  <tr>
    <td width="50%" align="center">
      <img src="docs/screenshots/emby-management.webp" alt="MorePlay Emby 管理" width="100%">
      <br>
      <sub>Emby 兼容服务</sub>
    </td>
    <td width="50%" align="center">
      <img src="docs/screenshots/strm-management.webp" alt="MorePlay STRM 媒体库" width="100%">
      <br>
      <sub>STRM 媒体库</sub>
    </td>
  </tr>
</table>

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
      CATPAW_SOURCE_URL: ""
      CATPAW_SETTINGS_PASSWORD: "请替换为至少8位的管理密码"

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

- `CATPAW_SOURCE_URL`：可填写自己的猫源地址；留空时先进入管理台，再添加猫源或影视仓 / TVBox 输入源。
- `CATPAW_SETTINGS_PASSWORD`：管理台登录密码，部署前请替换。通过环境变量指定的密码需要修改部署配置并重新创建容器，网页不能覆盖它。
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

如果全新数据卷没有设置管理密码，初始密码是 `88888888`；首次登录后请修改。已有数据卷继续使用原来保存的密码。管理台密码与 Emby 播放账号分别配置。

## 首次使用

1. 打开 `/bridge`，使用部署时设置的管理密码登录。
2. 在“站源管理”添加信任的猫源或兼容的影视仓 / TVBox 输入源。需要网盘账号的站源，请在对应猫源的设置页完成账号配置。
3. 检查站点是否可用，再分别配置需要使用的 Forward/Rex、Emby、STRM 或 T4。各功能的站点范围独立保存，不要只设置一个功能就认为其它功能也已配置。
4. 使用 Emby 时，在 Emby 页面配置账号和站点范围。第一次使用建议先等预热完成；预热是在后台提前读取分类首页，减少首次浏览等待，不是下载整部视频。
5. 如需 Emby 跨服聚合，先在“STRM → 资料与订阅”配置 TMDB，并在 Emby 搜索站点里选择需要参与匹配的站源。匹配结果取决于作品资料和上游返回内容。
6. 使用 STRM 时，配置播放器或媒体服务器能够访问的 MorePlay 对外地址，再生成媒体文件。将 `/strm` 对应的宿主机目录挂载给媒体库应用，并让它扫描该目录。

猫源是可执行程序，四文件及在线地址都应来自信任的作者。新增、切换、启停猫源或修改猫源代理等操作可能触发 MorePlay 自动重启；期间管理页或播放会短暂断开，等待启动后再刷新。

建议先添加少量稳定站源，确认能浏览和播放后再逐步增加。站源越多，搜索、扫描和预热的等待及资源占用通常越多。

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

#### 推荐猫源

在“站源管理”中添加以下在线地址，并确认信任对应作者：

| 猫源 | 地址 |
| --- | --- |
| 牛二源 | [https://9280.kstore.vip/cat/index.js.md5](https://9280.kstore.vip/cat/index.js.md5) |
| 豆源 | [https://woleigedouer:woleigedouer@catpaw.douer.me/index.js.md5](https://woleigedouer:woleigedouer@catpaw.douer.me/index.js.md5) |

这些 MD5 地址可以直接填入猫源地址栏，程序会识别对应目录并下载、校验四文件。豆源地址中的访问认证部分需要完整保留。源站内容和可用性由对应作者维护。

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

普通协议的播放方式在“系统设置 → 播放链路”调整；STRM 在“STRM → 播放设置”独立配置，提供“自动”和“始终由 NAS 转发”。自动模式会优先使用满足直连条件的媒体，其余由 NAS 转发。

百度、夸克等网盘常需要特定请求头。Forward/Rex 模块支持的请求头直连，不代表所有 Emby 播放器也能使用；客户端不支持时应使用 NAS 转发。直连可能需要将网盘登录凭据交给播放器，更适合本人使用。

STRM 文件是指向 MorePlay 播放入口的小文件，生成 STRM 不会把整部视频下载下来。主动下载视频时由容器后台取流，消耗服务器网络和磁盘空间，不受播放器的直连选择影响。MorePlay 和原站源需要持续可用，已有 STRM 才能解析在线播放地址。

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

普通浏览或使用源站资料时可以不配置 TMDB；如需 Emby 跨服聚合或 TMDB 资料匹配，需在 STRM 页面配置。TMDB 的网络代理与猫源、TVBox 输入源代理分别设置，按各自的网络需求配置。

## 常用环境变量

常用配置模板见 [`.env.example`](.env.example)。填写后需要通过 Compose 的 `environment` / `env_file` 或 `docker run --env-file` 加载，单纯编辑模板文件不会自动改变容器配置。

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

如果希望暂时回到旧版本，可改用已发布的固定镜像标签，继续挂载原数据卷；先保留备份，避免新旧版本的数据格式差异影响恢复。不要通过删除数据卷来完成普通升级。

## 常见问题

**可以只用 TVBox 输入，不添加猫源吗？** 可以，兼容的普通 HTTP 站点可以独立使用。需要 Spider 的站点仍取决于额外运行环境，不是所有影视仓配置都能直接播放。

**为什么有些封面、搜索结果或线路会失败？** 结果来自上游站源，可能遇到网络不通、接口变化、登录失效、链接过期或播放器不支持。先查看播放诊断和日志，再确认对应猫源账号、输入源代理和客户端设置。

**为什么切换设备后直连不成功？** 设备需要能访问最终媒体地址，并支持这条线路要求的请求头。服务器能访问不等于手机也能访问；无法满足时使用 NAS 转发。

**为什么生成的 STRM 在别的媒体服务器上不能播放？** 检查 STRM 中配置的 MorePlay 对外地址是否能从那台服务器访问，以及目录是否正确挂载。只复制 STRM 文件不能替代 MorePlay 服务。

**重建容器会丢配置吗？** 正常重建并保留原 `/data`、`/strm` 挂载即可保留配置及媒体文件；本地猫源还应保留对应文件。备份和迁移请同时核对宿主机目录或数据卷名称。

## 版本更新说明

各版本的详细更新日志和 Docker 镜像标签见 [Releases](https://github.com/xdbiaoge/moreplay/releases)。升级前请阅读对应版本说明，并保留原有数据卷。

## 安全建议

- 为管理台设置独立的强密码。
- 不要将管理端口直接暴露到不受信任的网络。
- 不要把 Token、Cookie、API Key 等内容提交到公开仓库。
- 对外访问时建议通过受控的反向代理和 HTTPS 提供服务。
- 仅添加自己信任的猫源、影视仓和其它外部数据源。

## 项目说明

MorePlay 不附带或提供影视资源。程序提供数据源接入、协议转换、目录整理、播放桥接和用户主动下载能力；配置、缓存、封面及下载的视频保存在用户自己的存储目录。

上游数据源、接口、媒体文件及相关服务均由其各自提供方负责。使用者应自行确认所使用数据源和内容符合所在地法律法规、服务条款及授权要求。

本项目按现状提供，不对第三方数据源的可用性、稳定性、内容准确性或持续兼容性作保证。
