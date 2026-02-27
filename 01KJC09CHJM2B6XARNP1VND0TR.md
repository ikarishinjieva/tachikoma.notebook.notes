---
title: 20251216 - 尝试YOLO
confluence_page_id: 4359457
created_at: 2025-12-16T14:26:08+00:00
updated_at: 2025-12-16T14:26:08+00:00
---

# 使用roboflow进行标记

建立项目, 上传10个图片, 标注 pipette top 和 pipette neck, 用训练集验证, 只有一张图片的neck识别有误

使用如下代码进行测试: 

先 pip install inference-sdk

```

from inference_sdk import InferenceHTTPClient

client = InferenceHTTPClient(
    api_url="https://serverless.roboflow.com",
    api_key="LG58D7QWjX6SvUID8c1a"
)

result = client.run_workflow(
    workspace_name="pipette",
    workflow_id="find-pipette-tops-and-pipette-necks",
    images={
        "image": "val.jpg"
    },
    use_cache=True # cache workflow definition for 15 minutes
)

print(result)

# ==========================================
# 本地过滤与重新可视化
# ==========================================
import cv2
import numpy as np

# 设置您的置信度阈值
CONFIDENCE_THRESHOLD = 0.5

try:
    # 1. 读取本地原始图片
    image_path = "val.jpg"
    image = cv2.imread(image_path)
    
    if image is None:
        print(f"错误: 无法在本地找到或读取图片 '{image_path}'，无法进行重新绘制。")
    else:
        # 2. 解析预测结果
        # 根据之前的输出，结构是 result[0]["predictions"]["predictions"]
        predictions_data = []
        
        # 尝试适配 Roboflow 返回的结构
        res_item = result[0]
        if "predictions" in res_item:
            sub_pred = res_item["predictions"]
            if isinstance(sub_pred, dict) and "predictions" in sub_pred:
                predictions_data = sub_pred["predictions"]
            elif isinstance(sub_pred, list):
                predictions_data = sub_pred
        
        print(f"\n原始检测数量: {len(predictions_data)}")
        
        # 3. 过滤并绘制
        count = 0
        for pred in predictions_data:
            conf = pred.get("confidence", 0)
            
            # 过滤逻辑
            if conf < CONFIDENCE_THRESHOLD:
                continue
            
            count += 1
            
            # 获取坐标 (Roboflow 返回的是中心点 x,y 和宽高 width,height)
            x, y, w, h = pred["x"], pred["y"], pred["width"], pred["height"]
            class_name = pred.get("class", "object")
            
            # 转换为 OpenCV 需要的左上角和右下角坐标
            x1 = int(x - w / 2)
            y1 = int(y - h / 2)
            x2 = int(x + w / 2)
            y2 = int(y + h / 2)
            
            # 绘制边界框 (绿色, 线宽 2)
            cv2.rectangle(image, (x1, y1), (x2, y2), (0, 255, 0), 2)
            
            # 绘制标签背景和文字
            label = f"{class_name}: {conf:.2f}"
            (label_w, label_h), baseline = cv2.getTextSize(label, cv2.FONT_HERSHEY_SIMPLEX, 0.5, 1)
            cv2.rectangle(image, (x1, y1 - label_h - 5), (x1 + label_w, y1), (0, 255, 0), -1)
            cv2.putText(image, label, (x1, y1 - 5), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 0, 0), 1)

        print(f"过滤后剩余数量 (Confidence >= {CONFIDENCE_THRESHOLD}): {count}")

        # 4. 保存新图片
        output_filename = "visualization_filtered.jpg"
        cv2.imwrite(output_filename, image)
        print(f"已保存高置信度结果图到: {output_filename}")

except Exception as e:
    print(f"本地处理出错: {e}")

``` 

用陌生图片进行测试, 效果不错: 

![image2025-12-16 22:25:58.png](/assets/01KJC09CHJM2B6XARNP1VND0TR/image2025-12-16%2022%3A25%3A58.png)
