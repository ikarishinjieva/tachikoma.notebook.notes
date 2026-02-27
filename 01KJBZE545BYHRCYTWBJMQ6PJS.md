---
title: 20240812 - 试用AutoPrompt
confluence_page_id: 3146100
created_at: 2024-08-12T04:58:54+00:00
updated_at: 2024-08-13T15:47:18+00:00
---

# 尝试使用

```
ALL_PROXY=http://10.186.16.136:7890 git clone https://github.com/Eladlev/AutoPrompt.git
 
cd AutoPrompt
 
ALL_PROXY=http://10.186.16.136:7890 conda env create -f environment_dev.yml
 
eval "$(/root/anaconda3/bin/conda shell.bash hook)"
conda activate AutoPrompt
``` 

测试命令: 

```
python run_pipeline.py     --prompt "Does this movie review contain a spoiler? answer Yes or No"     --task_description "Assistant is an expert classifier that will classify a movie review, and let the user know if it contains a spoiler for the reviewed movie or not."
``` 

粗略步骤: 

  1. 配置 AutoPrompt/config/llm_env.yml
  2. 配置 AutoPrompt/config/config_default.yml
  3. 安装 <https://docs.argilla.io/latest/getting_started/how-to-deploy-argilla-with-docker/>
     1. (需要升级docker-compose)
     2. argilla的访问地址: <http://10.186.62.73:6900/>, 用户名argilla, 密码12345678
  4. 配置 AutoPrompt/config/config_default.yml 中的argilla配置
  5. pip install langchain_community -i <https://pypi.tuna.tsinghua.edu.cn/simple>
  6. 需要修改 AutoPrompt/estimator/estimator_argilla.py 以适配argilla最新版本
  7. 跑不通 数据生成的过程, 需要消除所有warning
  8. 使用Azure API, 会报找不到root_client, 原因是openai的client代码中, 对Azure的root_client没有初始化, 需要修改代码: 
     1. /root/anaconda3/envs/AutoPrompt/lib/python3.10/site-packages/langchain_openai/chat_models/azure.py
     2. ![image2024-8-13 9:57:42.png](/assets/01KJBZE545BYHRCYTWBJMQ6PJS/image2024-8-13%209%3A57%3A42.png)
  9. 会继续报错: "Invalid parameter: 'response_format' of type 'json_object' is not supported with this model"
     1. 原因是Azure提供的模型, 有的没有Json能力, 有Json能力的, 有会缺少ChatCompletion能力
     2. ![image2024-8-13 9:58:31.png](/assets/01KJBZE545BYHRCYTWBJMQ6PJS/image2024-8-13%209%3A58%3A31.png)
     3. 更换成4o模型, 可通过
  10. 继续报错: 主要由于argilla的API升级造成的不一致. 换用LLM进行annotator

# 使用

  1. 使用修改后的AutoPrompt代码: <https://github.com/ikarishinjieva/AutoPrompt>
  2. 需要修改 /root/anaconda3/envs/AutoPrompt/lib/python3.10/site-packages/langchain_openai/chat_models/azure.py 文件  
![image](http://8.134.54.170:8330/download/attachments/3146100/image2024-8-13%209%3A57%3A42.png?version=1&modificationDate=1723514262823&api=v2)

测试命令:

```
python run_pipeline.py     --prompt "Does this movie review contain a spoiler? answer Yes or No"     --task_description "Assistant is an expert classifier that will classify a movie review, and let the user know if it contains a spoiler for the reviewed movie or not."

``` 

标准流输出样例: 

```
Get new prompt:
Classify the following movie review to determine if it contains a spoiler. Labels: ['Yes' for spoiler and 'No' for no spoiler]. Consider explicit plot descriptions, significant plot twists, specific details about major plot points, endings, or significant events, and vague terms that suggest major developments or plot twists as spoilers. Be particularly attentive to reviews that mention the ending or plot development in a mysterious or ambiguous manner, as these can imply significant plot revelations. Avoid classifying general emotional reactions or atmospheric descriptions as spoilers unless they clearly imply critical plot details. Also, avoid misclassifying reviews that highlight plot-related elements like character disappearances or flashbacks without revealing explicit details.
Starting step 10
Processing samples: 0it [00:00, ?it/s]
Processing samples: 0it [00:00, ?it/s]
Processing samples: 0it [00:00, ?it/s]
Processing samples: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 50/50 [02:45<00:00,  3.32s/it]
Previous prompt score:
0.92
#########

Get new prompt:
Classify the following movie review to determine if it contains a spoiler. Labels: ['Yes' for spoiler and 'No' for no spoiler]. Consider explicit plot descriptions, significant plot twists, specific details about major plot points, endings, or significant events, and vague terms that suggest major developments or plot twists as spoilers. Pay close attention to reviews that ambiguously mention the ending or plot developments, as these can imply significant plot revelations. Additionally, be cautious with reviews that discuss the overall plot or storytelling techniques (like flashbacks) in a way that implies major developments without explicit details. Avoid counting general emotional reactions or atmospheric descriptions as spoilers unless they clearly imply critical plot details.
 

Starting step 11
Processing samples: 0it [00:00, ?it/s]
Processing samples: 0it [00:00, ?it/s]
Processing samples: 0it [00:00, ?it/s]
Processing samples: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 50/50 [02:19<00:00,  2.79s/it]
Previous prompt score:
0.9
#########

Get new prompt:
Classify the following movie review to determine if it contains a spoiler. Labels: ['Yes' for spoiler and 'No' for no spoiler]. Consider explicit plot descriptions, significant plot twists, specific details about major plot points, endings, or significant events, and vague terms that suggest major developments or plot twists as spoilers. Pay close attention to reviews that ambiguously mention the ending or plot developments, as these can imply significant plot revelations. Be particularly cautious with reviews that describe the overall plot or storytelling techniques (like flashbacks) in a way that implies major developments without explicit details. Avoid classifying general emotional reactions or atmospheric descriptions as spoilers unless they clearly imply critical plot details. Also, ensure reviews that highlight plot-related elements like character disappearances or flashbacks without revealing explicit details are not misclassified.
 

Starting step 12
Processing samples: 0it [00:00, ?it/s]
Processing samples: 0it [00:00, ?it/s]
Processing samples: 0it [00:00, ?it/s]
Processing samples: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 50/50 [02:24<00:00,  2.89s/it]
Previous prompt score:
0.9
``` 

日志文件和数据文件都在dump文件夹中, 样例: 

  - [info.log.zip](/assets/01KJBZE545BYHRCYTWBJMQ6PJS/info.log.zip): 对API的调用日志
  - [history.pkl](/assets/01KJBZE545BYHRCYTWBJMQ6PJS/history.pkl): state的pickle文件, 需解析打印
  - [dataset.csv](/assets/01KJBZE545BYHRCYTWBJMQ6PJS/dataset.csv): 挑战数据  
  

# 转换pickle文件history.pkl

代码: 

```
import argparse
import pickle
import json
import pandas as pd
import numpy as np

def convert_to_json_serializable(data):
    """递归地将数据转换为 JSON 可序列化的格式。

    Args:
        data: 要转换的数据。

    Returns:
        转换后的数据。
    """

    if isinstance(data, pd.DataFrame):
        return data.to_dict(orient='records')
    elif isinstance(data, np.ndarray):
        return data.tolist()  # 将 ndarray 转换为列表
    elif isinstance(data, list):
        return [convert_to_json_serializable(item) for item in data]
    elif isinstance(data, dict):
        return {key: convert_to_json_serializable(value) for key, value in data.items()}
    else:
        return data

def print_pickle_as_json(file_path):
    """解析 pickle 文件并以 JSON 格式打印其内容。

    Args:
        file_path (str): pickle 文件的路径。
    """

    try:
        with open(file_path, 'rb') as f:
            loaded_data = pickle.load(f)

            # 递归地将数据转换为 JSON 可序列化的格式
            json_serializable_data = convert_to_json_serializable(loaded_data)

            json_data = json.dumps(json_serializable_data, indent=4)
            print(json_data)
    except FileNotFoundError:
        print(f"错误：文件 '{file_path}' 不存在。")
    except pickle.PickleError:
        print(f"错误：无法解析文件 '{file_path}'。请确保它是一个有效的 pickle 文件。")

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description='解析 pickle 文件并以 JSON 格式打印其内容。')
    parser.add_argument('file_path', type=str, help='pickle 文件的路径')
    args = parser.parse_args()

    print_pickle_as_json(args.file_path) 
``` 

结果: [history.pkl.txt](/assets/01KJBZE545BYHRCYTWBJMQ6PJS/history.pkl.txt)

history文件中, 记录了每一轮使用的prompt, 测试出的错误用例, 以及对错误的分析 (用作下一轮的prompt生成建议): 举例:

```
 {
            "prompt": "Classify the following movie review to determine if it contains a spoiler. Labels: ['Yes' for spoiler and 'No' for no spoiler]. Consider any emotional reactions, plot descriptions, or subtle hints and implications about the ending or significant events as spoilers.",
            "score": 0.9333333333333333,
            "errors": [
                {
                    "id": 5,
                    "text": {
                        "review": "The soundtrack was incredible and really added to the atmosphere. The way the plot unfolded was quite unexpected.",
                        "spoiler": "No"
                    },
                    "prediction": "No",
                    "annotation": "Yes",
                    "metadata": NaN,
                    "score": false,
                    "batch_id": 0
                },
                {
                    "id": 11,
                    "text": {
                        "review": "The dialogue was sharp and witty, and the ending left so much to the imagination without revealing too much.",
                        "spoiler": "No"
                    },
                    "prediction": "No",
                    "annotation": "Yes",
                    "metadata": NaN,
                    "score": false,
                    "batch_id": 1
                }
            ],
            "confusion_matrix": [
                [
                    17,
                    2
                ],
                [
                    0,
                    11
                ]
            ],
            "analysis": "The confusion matrix indicates a generally high performance with 17 correct 'Yes' predictions and 11 correct 'No' predictions. However, there are 2 false negatives where the model failed to identify spoilers. Common failure cases include reviews that contain subtle hints or implications regarding the plot or ending. These cases often involve nuanced descriptions or emotional reactions that imply significant events without explicitly stating them. The model struggles to catch these subtleties, leading to incorrect classifications."
        }
``` 

# TODO

分析日志, 梳理出主要流程和控制参数
