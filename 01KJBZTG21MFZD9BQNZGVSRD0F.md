---
title: 20250616 - 保险数字人
confluence_page_id: 4161546
created_at: 2025-06-16T14:58:10+00:00
updated_at: 2025-06-16T17:04:34+00:00
---

# 探索技术

<https://fal.ai/models/fal-ai/hunyuan-portrait/playground> : 对视频换脸

<https://fal.ai/models/fal-ai/hunyuan-avatar>: 使用声音和图片, 生成视频

<https://fal.ai/models/fal-ai/wan-vace-1-3b>: 使用原视频+遮罩视频+图片, 将图片替换到视频的遮罩中并生成动作

<https://fal.ai/models/fal-ai/magi/extend-video>: 延长视频时间 (续写视频)

<https://fal.ai/models/fal-ai/instant-character>: 上传一个任务形象, 可以将人物形象变换动作

<https://fal.ai/models/fal-ai/vidu/reference-to-video>: 多张参考图片, 生成视频

# 图生视频测试

图片: ![微信图片_20250616235911_3.jpg](/assets/01KJBZTG21MFZD9BQNZGVSRD0F/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20250616235911_3.jpg)

模型: <https://fal.ai/models/fal-ai/wan-pro/image-to-video>

  - 提示词: Realistic, High-quality. A young man is standing confidently, delivering a training session to a group of attentive participants. He is gesturing with his hands, explaining concepts clearly, with a professional and engaging demeanor. The setting is a modern training room with a whiteboard or presentation screen in the background.
  - 效果: (效果不错)

模型: <https://fal.ai/models/fal-ai/hunyuan-custom>

  - 提示词如上
  - 效果: (效果不好)

模型: <https://fal.ai/models/fal-ai/hunyuan-portrait>

  - 根据参考视频, 进行换脸
  - 效果: (效果还可以)

模型: <https://fal.ai/models/fal-ai/kling-video/v2.1/standard/image-to-video>

  - 提示词如上
  - 效果: (生成10秒视频, 效果不好, 增加了一个转场, 并且到最后有一个物理错误)

模型: <https://fal.ai/models/fal-ai/veo2/image-to-video>

  - 提示词如上
  - 效果: (效果还可以)

模型: <https://fal.ai/models/fal-ai/wan-i2v>

  - 提示词如上
  - 效果: (效果还可以)

模型: <https://fal.ai/models/fal-ai/hunyuan-avatar>

  - 根据参考音频, 生成动作和罪行
  - 效果: (生成时间长, 效果一般)

模型: <https://fal.ai/models/fal-ai/kling-video/v2.1/master/image-to-video>

  - 提示词如上
  - 效果: 

模型: 可灵官网, 可灵2.1大师版, 生成10s视频

  - 提示词如上
  - 效果: (效果不错, 有一次转场)

模型: 

  - 续写视频: <https://fal.ai/models/fal-ai/magi/extend-video>
  - 效果: (续写的部分动作僵硬)

模型: <https://fal.ai/models/fal-ai/flux-pro/kontext/max>

  - 更新图片中人物的姿势
  - ![oJIpjE5KRUVACaHMZd601_8912e4fe90b84f6bbf9fad65620922db.jpg](/assets/01KJBZTG21MFZD9BQNZGVSRD0F/oJIpjE5KRUVACaHMZd601_8912e4fe90b84f6bbf9fad65620922db.jpg)
