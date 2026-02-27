---
title: 20250527 - 尝试用AI工具对tiktok视频进行分析
confluence_page_id: 3997758
created_at: 2025-05-27T10:19:11+00:00
updated_at: 2025-06-06T11:45:34+00:00
---

# 尝试1

  - 榜单: <https://www.shoplus.net/discovery/videos> (低粉爆款视频)
  - 下载视频
    -   
  

  - 给gemini进行分析

```
这是一段tiktok视频, 帮我列出其内容, 尽量详细
 
---
 
好的，这是一段 TikTok 视频的内容，尽可能详细：

视频展示了两位穿着白色实验服、戴着护目镜的年轻女性在实验室里进行实验。一位深棕色头发的女性正在使用滴定管将液体滴入锥形瓶中。锥形瓶中最初装有透明液体，随着滴定管中液体的滴入，锥形瓶中的液体逐渐变成鲜艳的粉红色，并且产生了一些沉淀物。这位女性面带微笑，似乎对实验结果感到满意。

在她旁边，另一位金发女性也戴着护目镜，正用手捂住鼻子和嘴巴，似乎对实验产生的气味或反应有所反应。

在她们身后，可以看到一位男性也在进行实验操作。他穿着白色实验服，背对着镜头。

视频中还叠加了文字“Oopsie！”，暗示实验结果可能与预期不符，或者发生了某种意外，但两位女性似乎并不介意。

整个视频的氛围轻松愉快，背景音乐欢快。视频的目的是展示一个有趣的化学实验现象，并以轻松幽默的方式呈现。
```

# 使用google Veo API生成视频

说明: [https://cloud.google.com/vertex-ai/generative-ai/docs/video/generate-videos?hl=zh-cn&_gl=1*k9sqp*_ga*MTg0OTMzNDY4NC4xNzE2NDY1MDc5*_ga_WH2QY8WWF5*czE3NDgzOTk0NTAkbzE3JGcwJHQxNzQ4Mzk5NDUwJGo2MCRsMCRoMCRkcXVadHpzLWxSWXFpM2xNWHJMWkRLeEp2UGtGS1hrOVgxdw..#rest](<https://cloud.google.com/vertex-ai/generative-ai/docs/video/generate-videos?hl=zh-cn&_gl=1*k9sqp*_ga*MTg0OTMzNDY4NC4xNzE2NDY1MDc5*_ga_WH2QY8WWF5*czE3NDgzOTk0NTAkbzE3JGcwJHQxNzQ4Mzk5NDUwJGo2MCRsMCRoMCRkcXVadHpzLWxSWXFpM2xNWHJMWkRLeEp2UGtGS1hrOVgxdw..#rest>)

配置google的service account, 下载key的json文件:

![image2025-5-28 11:39:35.png](/assets/01KJBZT45Z2MD7YR2QNH7HTJ4W/image2025-5-28%2011%3A39%3A35.png)

创建存储桶 tachikoma-veo2

文生视频的调用demo: [veo.ipynb](/assets/01KJBZT45Z2MD7YR2QNH7HTJ4W/veo.ipynb)

图生视频的demo: [veo.2.ipynb](/assets/01KJBZT45Z2MD7YR2QNH7HTJ4W/veo.2.ipynb)

即梦提供了对口型的功能

# 额外: 对MoneyPrinterTurbo的分析

项目: <https://github.com/harry0703/MoneyPrinterTurbo>

主要配置: 

![image](https://github.com/harry0703/MoneyPrinterTurbo/raw/main/docs/webui.jpg)

主要问题: 

  1. 文本生成和视频的关系
     1. 文本生成提示词: 

```
"""
# Role: Video Script Generator

## Goals:
Generate a script for a video, depending on the subject of the video.

## Constrains:
1. the script is to be returned as a string with the specified number of paragraphs.
2. do not under any circumstance reference this prompt in your response.
3. get straight to the point, don't start with unnecessary things like, "welcome to this video".
4. you must not include any type of markdown or formatting in the script, never use a title.
5. only return the raw content of the script.
6. do not include "voiceover", "narrator" or similar indicators of what should be spoken at the beginning of each paragraph or line.
7. you must not mention the prompt, or anything about the script itself. also, never talk about the amount of paragraphs or lines. just write the script.
8. respond in the same language as the video subject.

# Initialization:
- video subject: {video_subject}
- number of paragraphs: {paragraph_number}
"""
```

```
# Role: Video Search Terms Generator

## Goals:
Generate {amount} search terms for stock videos, depending on the subject of a video.

## Constrains:
1. the search terms are to be returned as a json-array of strings.
2. each search term should consist of 1-3 words, always add the main subject of the video.
3. you must only return the json-array of strings. you must not return anything else. you must not return the script.
4. the search terms must be related to the subject of the video.
5. reply with english search terms only.

## Output Example:
["search term 1", "search term 2", "search term 3","search term 4","search term 5"]

## Context:
### Video Subject
{video_subject}

### Video Script
{video_script}

Please note that you must use English for generating video search terms; Chinese is not accepted.

```

     2. 文本只用于生成音频
  2. 视频处理如何进行
     1. 使用moviepy库, 对视频进行拼接+专场
  3. 字幕如何处理
     1. 用文本生成音频, 对音频进行识别, 生成字幕文件, 然后用文本去校准字幕文件, 获得新的字幕文件

# 使用google Veo API生成视频-2

脚本: [veo.20250530.ipynb](/assets/01KJBZT45Z2MD7YR2QNH7HTJ4W/veo.20250530.ipynb)

主要思路:

  - 将视频截图生成视频, 以脱去其上的文字等信息, 并截图(不在脚本中)
  - 用图片模型, 将其中道具换掉 (不在脚本中)
  - 用图片生成视频
  - 将视频最后一帧的图片, 再次生成视频
  - 将多段视频拼接
  - 使用"对口型"模型(不在脚本中)

在google Veo-2碰到的问题: 

  - 视频在3秒位置, 会出现突然放大视角的情况
    - 即使加了negative prompt, 也没有改善
  - 会出现东南亚字幕

# 脚本算法-1

先生成分镜和分镜点的图片, 然后使用首尾图片来生成视频, 再将视频拼接

  - 解决的问题: 如果先生成视频, 再截取最后一帧, 这一帧可能是模糊的(带有动作印记), 用这一帧生成下一个视频就会走形
  - 带来的问题: 分镜点的图片生成质量不高, 跟原图片差异可能过大或过小

脚本: [aliyun.ipynb](/assets/01KJBZT45Z2MD7YR2QNH7HTJ4W/aliyun.ipynb)
