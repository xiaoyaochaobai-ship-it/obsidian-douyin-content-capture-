# 抖音内容提取 · Obsidian 插件


将抖音分享链接或完整分享文案直接导入 Obsidian。


本插件通过 **Obsidian 插件 + 本地 Python 后端** 完成本地化的抖音内容提取：


- 视频：解析抖音链接 → 下载无水印视频 → FFmpeg 提取音频 → 本地 Faster-Whisper 转写 → 转换为简体中文
- 图文：解析作品信息 → 下载全部图片 → 提取抖音 `desc` 文案
- 自动创建 Obsidian Markdown 笔记
- 视频、图片和文案均保存在本地
- 不依赖付费语音 API
- 不需要登录抖音 Cookie
- 支持本地 Whisper 模型


> **重要：** 本插件需要同时运行本地 Python 后端服务。


---


## 一、项目组成


本项目由两个部分组成：


### 1. Obsidian 插件


负责：


- 接收抖音链接
- 调用本地后端
- 获取提取结果
- 将视频、图片复制到 Obsidian Vault
- 自动创建 Markdown 笔记
- 管理插件配置
- 检查本地后端连接状态


### 2. 本地 Python 后端


负责：


- 抖音分享链接解析
- 抖音视频 / 图文识别
- 无水印视频下载
- 图片下载
- FFmpeg 音频提取
- Faster-Whisper 本地语音识别
- 繁体中文转换为简体中文
- 向 Obsidian 插件提供 HTTP API


整体架构：


```text
┌──────────────────────┐
│       Obsidian       │
│   抖音内容提取插件    │
└──────────┬───────────┘
           │
           │ HTTP
           │ 127.0.0.1:5050
           ▼
┌──────────────────────┐
│     本地 Python 后端   │
│        Flask          │
├──────────────────────┤
│ 抖音链接解析           │
│ 视频 / 图文识别         │
│ 无水印视频下载          │
│ 图片下载               │
│ FFmpeg 音频处理        │
│ Faster-Whisper 转写    │
│ 繁体 → 简体            │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│      本地文件系统      │
│      Obsidian Vault   │
├──────────────────────┤
│ Markdown 笔记          │
│ 视频                   │
│ 图片                   │
│ 文案                   │
└──────────────────────┘
二、主要功能
功能	说明
抖音短链	支持 v.douyin.com
抖音视频链接	支持 www.douyin.com/video/...
抖音图文链接	支持 www.douyin.com/note/...
分享文案	支持直接粘贴完整抖音分享文本
视频下载	下载无水印视频
图文下载	下载作品中的全部图片
文案提取	提取视频语音或图文 desc
Whisper	本地 Faster-Whisper
简体转换	自动转换为简体中文
Obsidian 笔记	自动创建 Markdown
本地处理	数据主要在本机处理
后端检测	插件自动检测本地服务状态
三、支持的抖音链接

支持以下形式：

https://v.douyin.com/xxxxx/


https://www.douyin.com/video/xxxxxxxxxxxx


https://www.douyin.com/note/xxxxxxxxxxxx


https://www.iesdouyin.com/share/video/xxxxxxxxxxxx

也支持完整分享文案，例如：

7.48 复制打开抖音，看看这个视频
https://v.douyin.com/xxxxx/

插件会自动从分享文本中识别抖音链接。

四、视频内容处理流程

对于视频作品：

抖音分享链接
      ↓
解析作品信息
      ↓
获取无水印视频地址
      ↓
下载 video.mp4
      ↓
FFmpeg 提取音频
      ↓
Faster-Whisper 本地转写
      ↓
繁体中文 → 简体中文
      ↓
生成 transcript.txt
      ↓
写入 Obsidian Markdown

整个语音识别过程使用本地 Whisper 模型。

不需要：

OpenAI Whisper API
硅基流动等第三方语音 API
其他付费语音识别服务
五、图文内容处理流程

对于抖音图文 / Note：

抖音 Note 链接
      ↓
解析作品信息
      ↓
读取 desc 文案
      ↓
获取图片地址
      ↓
下载图片
      ↓
生成 Obsidian Markdown

图文内容不会调用 Whisper。

当前版本也不进行图片 OCR。

如果图片本身包含文字，插件只保存图片，不会自动识别图片中的文字。

六、本地后端

本项目需要运行本地 Python 后端。

后端默认地址：

http://127.0.0.1:5050

健康检查：

GET /api/health

正常情况下返回类似：

{
  "api_key_configured": true,
  "engine": "local",
  "models": [
    "tiny",
    "base",
    "small",
    "medium",
    "large-v2",
    "large-v3"
  ],
  "success": true,
  "whisper": "faster-whisper"
}
七、后端安装

进入后端项目：

cd ~/Downloads/obsidian-content-capture-backend-main

激活虚拟环境：

source .venv/bin/activate

如果还没有安装依赖：

pip install -r requirements.txt

安装 FFmpeg：

brew install ffmpeg

启动后端：

python web/app.py

或者：

./run-web.sh

默认监听：

http://127.0.0.1:5050

浏览器打开：

http://127.0.0.1:5050

如果看到后端页面，说明服务已经启动。

八、检查后端是否正常

可以在终端执行：

curl http://127.0.0.1:5050/api/health

如果返回：

{
  "success": true
}

说明后端正常。

九、Obsidian 插件安装
方法一：手动安装

下载本项目中的：

main.js
manifest.json
styles.css

复制到：

你的 Obsidian Vault/
└── .obsidian/
    └── plugins/
        └── douyin-content-capture/
            ├── main.js
            ├── manifest.json
            └── styles.css

然后：

Obsidian
→ 设置
→ 社区插件
→ 关闭安全模式
→ 启用抖音内容提取
十、插件配置

进入：

设置
→ 社区插件
→ 抖音内容提取

主要配置：

配置	默认值	说明
后端地址	http://127.0.0.1:5050	本地 Python 服务
Whisper 模型	small	视频语音识别模型
笔记目录	Douyin	生成 Markdown 的目录
附件目录	attachments/douyin	视频和图片保存位置
嵌入视频	开启	是否在笔记中直接嵌入视频
创建后打开笔记	开启	创建完成后自动打开

插件设置页面会显示后端连接状态。

例如：

✓ 后端已连接

或者：

✕ 后端未连接
十一、Whisper 模型

支持：

tiny
base
small
medium
large-v2
large-v3

模型越大：

识别准确率通常越高
CPU 占用越高
内存占用越高
转写速度越慢

推荐：

tiny

适合：

快速测试
短视频
对准确率要求不高
base

适合：

日常快速提取
普通短视频
small

推荐作为默认模型。

适合：

中文内容
OOTD
口播
视频文案提取
medium / large

适合：

对转写准确率要求较高
长视频
对性能要求不敏感
十二、插件使用
方法一：Ribbon 图标

点击 Obsidian 左侧插件图标。

输入：

抖音分享链接

然后选择：

提取文案

插件会开始处理。

方法二：命令面板

打开：

Cmd + P

搜索：

抖音

可以使用相关命令。

十三、两种提取模式
1. 提取文案

完整处理：

视频
解析
↓
下载视频
↓
提取音频
↓
Whisper 转写
↓
简体中文
↓
生成 Obsidian 笔记
图文
解析
↓
下载图片
↓
读取 desc
↓
生成 Obsidian 笔记
2. 仅提取视频

只下载视频：

解析
↓
下载无水印视频
↓
写入 Obsidian

不会执行：

FFmpeg
Whisper
语音识别

如果只是想保存视频，建议使用这个模式。

十四、生成的 Obsidian 笔记

默认目录：

Douyin/

例如：

Douyin/
└── 2026-08-19-74282908446-电车变装.md

笔记通常包含：

---
type: douyin
content_type: video
douyin_id: 7666394421184810259
author: 74282908446
source: https://www.douyin.com/video/...
tags:
  - douyin
---


# 电车变装


#电车变装 #地铁穿搭 #ootd


![[video.mp4]]


## Caption


这里是视频转写后的中文文案。
十五、本地文件结构

后端处理结果会保存到：

output/

视频作品：

output/
└── {作品ID}_{标题}/
    ├── video.mp4
    ├── audio.wav
    ├── download_url.txt
    ├── transcript.txt
    ├── transcript_segments.json
    └── meta.json

图文作品：

output/
└── {作品ID}_{标题}/
    ├── images/
    │   ├── 01.jpg
    │   ├── 02.jpg
    │   └── ...
    ├── image_urls.txt
    ├── transcript.txt
    └── meta.json
十六、本地 API

Obsidian 插件主要通过 HTTP 调用本地后端。

健康检查
GET /api/health
获取作品信息
POST /api/video/info

请求：

{
  "url": "https://www.douyin.com/video/xxxxxxxx"
}
提取内容
POST /api/video/extract

请求：

{
  "url": "https://www.douyin.com/video/xxxxxxxx",
  "model": "small",
  "skip_transcribe": false
}

仅下载视频：

{
  "url": "https://www.douyin.com/video/xxxxxxxx",
  "model": "small",
  "skip_transcribe": true
}
下载视频
GET /api/video/download
访问处理结果
GET /files/<path>
十七、隐私

本项目设计为本地内容处理工具。

主要处理流程：

抖音
 ↓
本地 Python 后端
 ↓
本地 Whisper
 ↓
本地文件
 ↓
Obsidian Vault

语音识别不依赖云端语音 API。

Whisper 模型首次使用时需要联网下载模型。

抖音视频和图片下载本身需要访问抖音相关资源。

十八、常见问题
1. 插件显示后端未连接

检查：

curl http://127.0.0.1:5050/api/health

如果无法访问，启动：

cd ~/Downloads/obsidian-content-capture-backend-main


source .venv/bin/activate


python web/app.py
2. 视频提取很慢

这是正常现象。

视频需要：

下载
→ FFmpeg
→ Whisper
→ 中文转换

如果电脑性能有限，可以选择：

tiny

或者：

base
3. 只想下载视频

使用：

仅提取视频

不会执行 Whisper。

4. 图片中的文字没有被提取

当前版本没有 OCR。

图片中的文字会保留在图片中，但不会自动转换成 Markdown 文本。

5. 修改插件代码后 Obsidian 没有变化

重新构建：

npm run build

然后复制新的：

main.js
manifest.json
styles.css

到：

.obsidian/plugins/douyin-content-capture/

然后在 Obsidian 中：

Cmd + P
→ Reload app without saving
十九、插件开发

进入插件目录：

cd ~/Downloads/obsidian-douyin-capture-master

安装依赖：

npm install

开发模式：

npm run dev

生产构建：

npm run build

构建完成后会生成：

main.js
二十、项目结构
obsidian-douyin-capture/
│
├── src/
│   ├── main.ts
│   ├── modal.ts
│   ├── settings.ts
│   ├── settingTab.ts
│   ├── backend.ts
│   └── vaultWriter.ts
│
├── docs/
│   └── obsidian-plugin-contract.md
│
├── manifest.json
├── styles.css
├── main.js
├── package.json
├── package-lock.json
├── tsconfig.json
└── esbuild.config.mjs
二十一、后端项目

本插件对应的本地后端项目：

obsidian-content-capture-backend

后端主要负责：

抖音解析
视频下载
图片下载
FFmpeg
Whisper
中文转换
HTTP API

插件负责：

Obsidian UI
调用 API
媒体复制
Markdown 创建
Vault 管理
二十二、版本说明

当前项目基于原有 Douyin Capture 项目进行功能调整和本地化改造。

当前版本重点是：

本地 Python 后端
本地 Faster-Whisper
抖音视频提取
抖音图文提取
Obsidian 本地内容管理
本地媒体保存
简体中文转换

后续将继续完善：

内容结构化
AI 内容分析
自动标签
自动分类
内容摘要
视频内容理解
Obsidian 知识库管理
二十三、版权与免责声明

本项目仅用于个人学习、研究以及内容管理。

请遵守：

抖音平台规则
内容版权相关法律法规
Obsidian 使用规则
所在国家和地区的相关法律

请勿将本工具用于侵犯他人版权、绕过平台限制或其他违法用途。

使用本工具产生的相关责任由使用者自行承担。
