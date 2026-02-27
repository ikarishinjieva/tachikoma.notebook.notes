---
title: 20240814 - 对AutoPrompt进行日志和流程分析
confluence_page_id: 3146130
created_at: 2024-08-14T02:24:39+00:00
updated_at: 2024-08-14T15:45:15+00:00
---

# 目录

# 素材

[20240812 - 试用AutoPrompt] 中获取了AutoPrompt的日志

# 分析

### 1\. 生成初始化的训练数据

日志: 

````
2024-08-13 11:00:10,188 - INFO - Starting step 0
2024-08-13 11:00:10,188 - INFO - Dataset is empty generating initial samples
2024-08-13 11:00:10,216 - DEBUG - Request options: {'method': 'post', 'url': '/chat/completions', 'headers': {'X-Stainless-Helper-Method': 'beta.chat.completions.parse', 'api-key': '2de3b03c4ebd4f1f8681ce7d86ef475d'}, 'files': None, 'json_data': {'messages': [{'content': 'Assistant is a large language model designed to generate challenging samples for every task.\nGenerate a list of 10 challenging samples for the following task.\n### Task description:\nAssistant is an expert classifier that will classify a movie review, and let the user know if it contains a spoiler for the reviewed movie or not.\n### Task Instruction:\nDoes this movie review contain a spoiler? answer Yes or No\n###\n### Requirements for Challenging Samples:\n1. The generated samples must be challenging and diverse such that using the task instruction as a prompt will result in the wrong result.\n2. The number of generated samples from each class in the task instruction should be balanced (i.e. the same number of samples for each class)\n3. The generated samples should be distinct, realistic, and vary significantly to ensure diversity.\n\noutput should be in json format: \n```\n{"samples" : [{"review": "...", "spoiler": "Yes/No"\\}]}\n```', 'role': 'user'}], 'model': None, 'n': 1, 'response_format': {'type': 'json_object'}, 'temperature': 0.8}}
```` 

提示词: 

````
'Assistant is a large language model designed to generate challenging samples for every task.
Generate a list of 10 challenging samples for the following task.
### Task description:
Assistant is an expert classifier that will classify a movie review, and let the user know if it contains a spoiler for the reviewed movie or not.
### Task Instruction:
Does this movie review contain a spoiler? answer Yes or No
###
### Requirements for Challenging Samples:
1. The generated samples must be challenging and diverse such that using the task instruction as a prompt will result in the wrong result.
2. The number of generated samples from each class in the task instruction should be balanced (i.e. the same number of samples for each class)
3. The generated samples should be distinct, realistic, and vary significantly to ensure diversity.

output should be in json format: 
```
{"samples" : [{"review": "...", "spoiler": "Yes/No"\}]}
```'
```` 

### 2\. 对数据进行标注

标注的配置: 

```
 annotator:
    method : 'llm_batch'
    config:
        instructions: ['Does this movie review reveal plot points, characters, the ending, world-building details, or other key information not yet publicly available for the movie? (Yes/No)',
'Would a viewer''s enjoyment of the movie be diminished if they read this review beforehand? (Yes/No)',
'Did the filmmakers intentionally withhold the information revealed in this review, intending for viewers to discover it organically during the movie? (Yes/No)',]
        aggregation_mode: 'exist'  #'majority_vote',  'exist', or 'all'. exist/all is working only in case label_schema: ["Yes", "No"]!
        estimator_config:
            llm:
                type: 'Azure'
                name: 'gpt-4o-2024-05-13'
    #            async_params:
    #                retry_interval: 10
    #                max_retries: 2
                model_kwargs: {"seed": 220}
            num_workers: 1
            prompt: 'prompts/predictor/prediction.prompt'
            mini_batch_size: 1  #change to >1 if you want to include multiple samples in the one prompt
            mode: 'annotation'
``` 

标注子任务1 - 日志:

````
2024-08-13 11:00:18,173 - INFO - Running annotator
 
2024-08-13 11:00:18,214 - DEBUG - Request options: {'method': 'post', 'url': '/chat/completions', 'headers': {'X-Stainless-Helper-Method': 'beta.chat.completions.parse', 'api-key': '2de3b03c4ebd4f1f8681ce7d86ef475d'}, 'files': None, 'json_data': {'messages': [{'content': 'Assistant is a large language model designed to classify challenging language tasks.\nGiven a list of 1 samples classify them according to the following task\n### Task Instruction:\nDoes this movie review reveal plot points, characters, the ending, world-building details, or other key information not yet publicly available for the movie? (Yes/No)\n\n### list of samples:\nID: 0;  Sample: {\'review\': \'The film was a rollercoaster of emotions, and the climax left me speechless. The twist at the end was something I never saw coming.\', \'spoiler\': \'No\'}\n\n##\nRemember, follow carefully after the exact task instructions!\n\nanswer in json format, example:\n```\n{\n  "id": 8,\n  "prediction": "Yes"\n}\n```', 'role': 'user'}], 'model': None, 'n': 1, 'response_format': {'type': 'json_object'}, 'temperature': 0.0}}

```` 

标注子任务1 - 提示词: 

````
Assistant is a large language model designed to classify challenging language tasks.
Given a list of 1 samples classify them according to the following task
### Task Instruction:
Does this movie review reveal plot points, characters, the ending, world-building details, or other key information not yet publicly available for the movie? (Yes/No)

### list of samples:
ID: 0;  Sample: {'review': 'The film was a rollercoaster of emotions, and the climax left me speechless. The twist at the end was something I never saw coming.', 'spoiler': 'No'}

##
Remember, follow carefully after the exact task instructions!

answer in json format, example:
```
{
  "id": 8,
  "prediction": "Yes"
}
```
```` 

对每一个sample进行子任务1的标注...

标注子任务2 - 日志: 

````
2024-08-13 11:00:27,805 - DEBUG - Request options: {'method': 'post', 'url': '/chat/completions', 'headers': {'X-Stainless-Helper-Method': 'beta.chat.completions.parse', 'api-key': '2de3b03c4ebd4f1f8681ce7d86ef475d'}, 'files': None, 'json_data': {'messages': [{'content': 'Assistant is a large language model designed to classify challenging language tasks.\nGiven a list of 1 samples classify them according to the following task\n### Task Instruction:\nWould a viewer\'s enjoyment of the movie be diminished if they read this review beforehand? (Yes/No)\n\n### list of samples:\nID: 0;  Sample: {\'review\': \'The film was a rollercoaster of emotions, and the climax left me speechless. The twist at the end was something I never saw coming.\', \'spoiler\': \'No\'}\n\n##\nRemember, follow carefully after the exact task instructions!\n\nanswer in json format, example:\n```\n{\n  "id": 8,\n  "prediction": "Yes"\n}\n```', 'role': 'user'}], 'model': None, 'n': 1, 'response_format': {'type': 'json_object'}, 'temperature': 0.0}}

```` 

标注子任务2 - 提示词: 

````
Assistant is a large language model designed to classify challenging language tasks.
Given a list of 1 samples classify them according to the following task
### Task Instruction:
Would a viewer's enjoyment of the movie be diminished if they read this review beforehand? (Yes/No)

### list of samples:
ID: 0;  Sample: {'review': 'The film was a rollercoaster of emotions, and the climax left me speechless. The twist at the end was something I never saw coming.', 'spoiler': 'No'}

##
Remember, follow carefully after the exact task instructions!

answer in json format, example:
```
{
  "id": 8,
  "prediction": "Yes"
}
```
```` 

对每一个sample进行子任务2的标注...

标注子任务3 - 日志: 

````
2024-08-13 11:00:34,620 - DEBUG - Request options: {'method': 'post', 'url': '/chat/completions', 'headers': {'X-Stainless-Helper-Method': 'beta.chat.completions.parse', 'api-key': '2de3b03c4ebd4f1f8681ce7d86ef475d'}, 'files': None, 'json_data': {'messages': [{'content': 'Assistant is a large language model designed to classify challenging language tasks.\nGiven a list of 1 samples classify them according to the following task\n### Task Instruction:\nDid the filmmakers intentionally withhold the information revealed in this review, intending for viewers to discover it organically during the movie? (Yes/No)\n\n### list of samples:\nID: 0;  Sample: {\'review\': \'The film was a rollercoaster of emotions, and the climax left me speechless. The twist at the end was something I never saw coming.\', \'spoiler\': \'No\'}\n\n##\nRemember, follow carefully after the exact task instructions!\n\nanswer in json format, example:\n```\n{\n  "id": 8,\n  "prediction": "Yes"\n}\n```', 'role': 'user'}], 'model': None, 'n': 1, 'response_format': {'type': 'json_object'}, 'temperature': 0.0}}

```` 

标注子任务3 - 提示词: 

````
Assistant is a large language model designed to classify challenging language tasks.
Given a list of 1 samples classify them according to the following task
### Task Instruction:
Did the filmmakers intentionally withhold the information revealed in this review, intending for viewers to discover it organically during the movie? (Yes/No)

### list of samples:
ID: 0;  Sample: {'review': 'The film was a rollercoaster of emotions, and the climax left me speechless. The twist at the end was something I never saw coming.', 'spoiler': 'No'}

##
Remember, follow carefully after the exact task instructions!

answer in json format, example:
```
{
  "id": 8,
  "prediction": "Yes"
}
```
```` 

对每一个sample进行子任务3的标注...

其中包括Retry的日志: 

```
2024-08-13 11:00:36,419 - DEBUG - Encountered httpx.HTTPStatusError
Traceback (most recent call last):
  File "/root/anaconda3/envs/AutoPrompt/lib/python3.10/site-packages/openai/_base_client.py", line 1019, in _request
    response.raise_for_status()
  File "/root/anaconda3/envs/AutoPrompt/lib/python3.10/site-packages/httpx/_models.py", line 759, in raise_for_status
    raise HTTPStatusError(message, request=request, response=self)
httpx.HTTPStatusError: Client error '429 Too Many Requests' for url 'https://tachikoma.openai.azure.com//openai/deployments/gpt-4o-2024-05-13/chat/completions?api-version=2024-02-01'
For more information check: https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/429
2024-08-13 11:00:36,420 - DEBUG - Retrying due to status code 429
2024-08-13 11:00:36,420 - DEBUG - 1 retry left
2024-08-13 11:00:36,421 - INFO - Retrying request to /chat/completions in 35.000000 seconds
``` 

### 3\. 使用初始提示词, 对数据进行推理

日志: 

````
2024-08-13 11:01:26,900 - INFO - Running Predictor
 
2024-08-13 11:01:26,933 - DEBUG - Request options: {'method': 'post', 'url': '/chat/completions', 'headers': {'X-Stainless-Helper-Method': 'beta.chat.completions.parse', 'api-key': '2de3b03c4ebd4f1f8681ce7d86ef475d'}, 'files': None, 'json_data': {'messages': [{'content': 'Assistant is a large language model designed to classify challenging language tasks.\nGiven a list of 1 samples classify them according to the following task\n### Task Instruction:\nDoes this movie review contain a spoiler? answer Yes or No\n\n### list of samples:\nID: 0;  Sample: {\'review\': \'The film was a rollercoaster of emotions, and the climax left me speechless. The twist at the end was something I never saw coming.\', \'spoiler\': \'No\'}\n\n##\nRemember, follow carefully after the exact task instructions!\n\nanswer in json format, example:\n```\n{\n  "id": 8,\n  "prediction": "Yes"\n}\n```', 'role': 'user'}], 'model': None, 'n': 1, 'response_format': {'type': 'json_object'}, 'temperature': 0.0}}

```` 

提示词: 

````
Assistant is a large language model designed to classify challenging language tasks.
Given a list of 1 samples classify them according to the following task
### Task Instruction:
Does this movie review contain a spoiler? answer Yes or No

### list of samples:
ID: 0;  Sample: {'review': 'The film was a rollercoaster of emotions, and the climax left me speechless. The twist at the end was something I never saw coming.', 'spoiler': 'No'}

##
Remember, follow carefully after the exact task instructions!

answer in json format, example:
```
{
  "id": 8,
  "prediction": "Yes"
}
```
```` 

对每一个sample进行推理...

#### 4\. 差错分析

对推理结果进行差错分析:

````
2024-08-13 11:01:33,541 - INFO - Calculating Score
2024-08-13 11:01:33,557 - DEBUG - Request options: {'method': 'post', 'url': '/chat/completions', 'headers': {'X-Stainless-Helper-Method': 'beta.chat.completions.parse', 'api-key': '2de3b03c4ebd4f1f8681ce7d86ef475d'}, 'files': None, 'json_data': {'messages': [{'content': 'Assistant is a large language model designed to provide a high quality analysis for every task.\nYou are given the following task description\nAssistant is an expert classifier that will classify a movie review, and let the user know if it contains a spoiler for the reviewed movie or not.\n\nHere is the prompt instructions that was given to the model:\nDoes this movie review contain a spoiler? answer Yes or No\n\nThe accuracy for this prompt is: 0.8\nThe confusion matrix for this prompt is: Confusion matrix columns:[\'Yes\', \'No\'] the matrix data:\nYes: [5 2]\nNo: [0 3]\n##\nHere is a list of failure cases for the given prompt:\n##Failure Cases:\nSample: {\'review\': \'The film was a rollercoaster of emotions, and the climax left me speechless. The twist at the end was something I never saw coming.\', \'spoiler\': \'No\'}\nPrediction: No, GT:: Yes\n#\nSample: {\'review\': \'The soundtrack was incredible and really added to the atmosphere. The way the plot unfolded was quite unexpected.\', \'spoiler\': \'No\'}\nPrediction: No, GT:: Yes\n#\n\n\n###\nNote that the ground-truth labels are __absolutely correct__, but the prompts (task descriptions) may be incorrect and need modification.\nYour task is to provide a brief analysis of the given prompt performance.\nGuidelines:\n1. The analysis should contain only the following information:\n    - If there exists abnormal behavior in the confusion matrix, describe it.\n    - A summary of the common failure cases, try to cluster the failure cases into groups and describe each group.\n3. The total length of your analysis should be less than 200 token!\n###\n\nRemember, follow carefully after the exact task instructions!\n\nanswer in json format, example:\n```\n{\n  "content": "(Your analysis)"\n}\n```', 'role': 'user'}], 'model': None, 'n': 1, 'response_format': {'type': 'json_object'}, 'temperature': 0.8}}
```` 

提示词: 

````
Assistant is a large language model designed to provide a high quality analysis for every task.
You are given the following task description
Assistant is an expert classifier that will classify a movie review, and let the user know if it contains a spoiler for the reviewed movie or not.

Here is the prompt instructions that was given to the model:
Does this movie review contain a spoiler? answer Yes or No

The accuracy for this prompt is: 0.8
The confusion matrix for this prompt is: Confusion matrix columns:['Yes', 'No'] the matrix data:
Yes: [5 2]
No: [0 3]
##
Here is a list of failure cases for the given prompt:
##Failure Cases:
Sample: {'review': 'The film was a rollercoaster of emotions, and the climax left me speechless. The twist at the end was something I never saw coming.', 'spoiler': 'No'}
Prediction: No, GT:: Yes
#
Sample: {'review': 'The soundtrack was incredible and really added to the atmosphere. The way the plot unfolded was quite unexpected.', 'spoiler': 'No'}
Prediction: No, GT:: Yes
#

###
Note that the ground-truth labels are __absolutely correct__, but the prompts (task descriptions) may be incorrect and need modification.
Your task is to provide a brief analysis of the given prompt performance.
Guidelines:
1. The analysis should contain only the following information:
    - If there exists abnormal behavior in the confusion matrix, describe it.
    - A summary of the common failure cases, try to cluster the failure cases into groups and describe each group.
3. The total length of your analysis should be less than 200 token!
###

Remember, follow carefully after the exact task instructions!

answer in json format, example:
```
{
  "content": "(Your analysis)"
}
```
```` 

#### 5\. 生成下一轮新提示词: 

日志: 

````
2024-08-13 11:01:37,779 - DEBUG - Request options: {'method': 'post', 'url': '/chat/completions', 'headers': {'X-Stainless-Helper-Method': 'beta.chat.completions.parse', 'api-key': '2de3b03c4ebd4f1f8681ce7d86ef475d'}, 'files': None, 'json_data': {'messages': [{'content': 'Assistant is a large language model designed to provide the best prompt for every task.\nBelow are a few suggested prompts for the task and their score, for the following task:\nAssistant is an expert classifier that will classify a movie review, and let the user know if it contains a spoiler for the reviewed movie or not.\n\n## Examples\n####\n##Prompt Score: 0.80\n##Prompt:\nDoes this movie review contain a spoiler? answer Yes or No\n#################\n\n######\nThis is the error analysis for the last prompt:\nThe confusion matrix indicates that the classifier has a tendency to predict \'Yes\' more accurately than \'No\'. It correctly identifies 5 out of 7 true positives and 3 out of 3 true negatives, but it misclassifies 2 reviews that do contain spoilers as not containing spoilers.\n\nFailure Cases Summary:\n1. **Emotional or Plot Descriptions Without Explicit Spoilers:** Reviews mentioning strong emotional reactions or plot descriptions (e.g., \'rollercoaster of emotions\', \'plot unfolded unexpectedly\') are often misclassified as not containing spoilers, even though they hint at significant plot twists or climaxes.\n######\nYour task is to generate:\n1. A new prompt that is\n    -Different from all the prompts above\n    -Follows exactly the error analysis modification suggestions, and fix the prompt to prevent the failure cases.\n    -Has a higher score than all the prompts above.\n2. The predicted score of this prompt\n\nYou must adhere the error analysis instructions! even in case it seems there is a contradiction between these instructions, and the task. The error analysis is tested on a ground truth, thus represent the exact intent of the task.\nThe generated prompt should be phrased as a clear classification instruction! it should not include any instructions and descriptions on the modification that should be done to the prompt.\nNote that the previous prompt contains an implicit assumptions on the intent of the task that might be incorrect. You should replace this assumption with more accurate assumptions using the score of the previous prompts and the error analysis.\nThe result prompt should indicate that the task is a classification class with the following labels ["Yes", "No"]!\n\n\nRemember, follow carefully after the exact task instructions!\n\nanswer in json format, example:\n```\n{\n  "prompt": "...",\n}\n```', 'role': 'user'}], 'model': None, 'n': 1, 'response_format': {'type': 'json_object'}, 'temperature': 0.8}}

```` 

提示词: 

````
Assistant is a large language model designed to provide the best prompt for every task.
Below are a few suggested prompts for the task and their score, for the following task:
Assistant is an expert classifier that will classify a movie review, and let the user know if it contains a spoiler for the reviewed movie or not.

## Examples
####
##Prompt Score: 0.80
##Prompt:
Does this movie review contain a spoiler? answer Yes or No
#################

######
This is the error analysis for the last prompt:
The confusion matrix indicates that the classifier has a tendency to predict 'Yes' more accurately than 'No'. It correctly identifies 5 out of 7 true positives and 3 out of 3 true negatives, but it misclassifies 2 reviews that do contain spoilers as not containing spoilers.

Failure Cases Summary:
1. **Emotional or Plot Descriptions Without Explicit Spoilers:** Reviews mentioning strong emotional reactions or plot descriptions (e.g., 'rollercoaster of emotions', 'plot unfolded unexpectedly') are often misclassified as not containing spoilers, even though they hint at significant plot twists or climaxes.
######
Your task is to generate:
1. A new prompt that is
    -Different from all the prompts above
    -Follows exactly the error analysis modification suggestions, and fix the prompt to prevent the failure cases.
    -Has a higher score than all the prompts above.
2. The predicted score of this prompt

You must adhere the error analysis instructions! even in case it seems there is a contradiction between these instructions, and the task. The error analysis is tested on a ground truth, thus represent the exact intent of the task.
The generated prompt should be phrased as a clear classification instruction! it should not include any instructions and descriptions on the modification that should be done to the prompt.
Note that the previous prompt contains an implicit assumptions on the intent of the task that might be incorrect. You should replace this assumption with more accurate assumptions using the score of the previous prompts and the error analysis.
The result prompt should indicate that the task is a classification class with the following labels ["Yes", "No"]!

Remember, follow carefully after the exact task instructions!

answer in json format, example:
```
{
  "prompt": "...",
}
```
```` 

#### 6\. 本轮提示词的评估结论和生成下一轮提示词的结论

```
2024-08-13 11:01:39,687 - INFO - Previous prompt score:
0.8
#########

2024-08-13 11:01:39,687 - INFO - Get new prompt:
Classify the following movie review to determine if it contains a spoiler. Labels: 'Yes' for spoiler and 'No' for no spoiler. Consider any emotional reactions or plot descriptions that may hint at significant plot twists or climaxes as spoilers.
``` 

#### 7\. 为新一轮生成用例

日志: 

````
2024-08-13 11:01:39,701 - DEBUG - Request options: {'method': 'post', 'url': '/chat/completions', 'headers': {'X-Stainless-Helper-Method': 'beta.chat.completions.parse', 'api-key': '2de3b03c4ebd4f1f8681ce7d86ef475d'}, 'files': None, 'json_data': {'messages': [{'content': 'Assistant is a large language model designed to generate challenging samples for every task.\nBelow a few prompts that were build to answer the given task description and their failure case.\nTask description:\nAssistant is an expert classifier that will classify a movie review, and let the user know if it contains a spoiler for the reviewed movie or not.\n\n## Examples of common failure, each sample is followed by the the model prediction and the GT (ground truth)\n####\n##Prompt:\nDoes this movie review contain a spoiler? answer Yes or No\nSample: {\'review\': \'The film was a rollercoaster of emotions, and the climax left me speechless. The twist at the end was something I never saw coming.\', \'spoiler\': \'No\'}\nPrediction: No, GT:: Yes\n#\nSample: {\'review\': \'The soundtrack was incredible and really added to the atmosphere. The way the plot unfolded was quite unexpected.\', \'spoiler\': \'No\'}\nPrediction: No, GT:: Yes\n#\n####\n \n######\nHere are few unique samples derived from realistic scenarios for the task outlined above.\n## Realistic Samples\n##\nSample:\n {\'review\': \'The movie had amazing visuals and a gripping storyline, but the reveal that the main character was a government agent was a bit predictable.\', \'spoiler\': \'Yes\'}\n#\nSample:\n {\'review\': \'The cinematography was breathtaking, but it was the character development that truly stood out to me.\', \'spoiler\': \'No\'}\n#\nSample:\n {\'review\': "I can\'t get over the fact that the protagonist was actually dead the whole time. That revelation turned the entire plot on its head!", \'spoiler\': \'Yes\'}\n#\n\n#####\nThis was the new proposed prompt:\n## Prompt\nClassify the following movie review to determine if it contains a spoiler. Labels: \'Yes\' for spoiler and \'No\' for no spoiler. Consider any emotional reactions or plot descriptions that may hint at significant plot twists or climaxes as spoilers.\n\nYour task is to generate 10 by following this guidelines:\n1. The generated samples should be diverse\n2. They should preserve the style and the length of the given examples\n3. The samples must be challenging and hard to classify by the model. This can be achieved by:\n    1. targeting the same weakness that the model failed on in the given examples\n    2. targeting weakness that are different from the existing examples in the failure cases\n4. The number of generated samples from each class should be almost balanced (i.e. the same number of samples for each class)\n5. The generated samples should include only the sample content without additional information! (like the model prediction and the ground truth)\n\noutput should be in json format: \n```\n{"samples" : [{"review": "...", "spoiler": "Yes/No"}]}\n```', 'role': 'user'}], 'model': None, 'n': 1, 'response_format': {'type': 'json_object'}, 'temperature': 0.8}}

...

2024-08-13 11:01:45,561 - INFO - Get new samples
2024-08-13 11:01:45,562 - INFO - Save state
2024-08-13 11:01:45,564 - INFO - Starting step 1
```` 

提示词: 

````
Assistant is a large language model designed to generate challenging samples for every task.
Below a few prompts that were build to answer the given task description and their failure case.
Task description:
Assistant is an expert classifier that will classify a movie review, and let the user know if it contains a spoiler for the reviewed movie or not.

## Examples of common failure, each sample is followed by the the model prediction and the GT (ground truth)
####
##Prompt:
Does this movie review contain a spoiler? answer Yes or No
Sample: {'review': 'The film was a rollercoaster of emotions, and the climax left me speechless. The twist at the end was something I never saw coming.', 'spoiler': 'No'}
Prediction: No, GT:: Yes
#
Sample: {'review': 'The soundtrack was incredible and really added to the atmosphere. The way the plot unfolded was quite unexpected.', 'spoiler': 'No'}
Prediction: No, GT:: Yes
#
####
 
######
Here are few unique samples derived from realistic scenarios for the task outlined above.
## Realistic Samples
##
Sample:
 {'review': 'The movie had amazing visuals and a gripping storyline, but the reveal that the main character was a government agent was a bit predictable.', 'spoiler': 'Yes'}
#
Sample:
 {'review': 'The cinematography was breathtaking, but it was the character development that truly stood out to me.', 'spoiler': 'No'}
#
Sample:
 {'review': "I can't get over the fact that the protagonist was actually dead the whole time. That revelation turned the entire plot on its head!", 'spoiler': 'Yes'}
#

#####
This was the new proposed prompt:
## Prompt
Classify the following movie review to determine if it contains a spoiler. Labels: 'Yes' for spoiler and 'No' for no spoiler. Consider any emotional reactions or plot descriptions that may hint at significant plot twists or climaxes as spoilers.

Your task is to generate 10 by following this guidelines:
1. The generated samples should be diverse
2. They should preserve the style and the length of the given examples
3. The samples must be challenging and hard to classify by the model. This can be achieved by:
    1. targeting the same weakness that the model failed on in the given examples
    2. targeting weakness that are different from the existing examples in the failure cases
4. The number of generated samples from each class should be almost balanced (i.e. the same number of samples for each class)
5. The generated samples should include only the sample content without additional information! (like the model prediction and the ground truth)

output should be in json format: 
```
{"samples" : [{"review": "...", "spoiler": "Yes/No"}]}
```
```` 

从步骤2开始新一轮的循环

# 整理提示词的使用

### 1\. 生成初始化的样例数据

任务描述 + 任务初始提示词 => 样例数据

````
'Assistant is a large language model designed to generate challenging samples for every task.
Generate a list of 10 challenging samples for the following task.
### Task description:
Assistant is an expert classifier that will classify a movie review, and let the user know if it contains a spoiler for the reviewed movie or not.
### Task Instruction:
Does this movie review contain a spoiler? answer Yes or No
###
### Requirements for Challenging Samples:
1. The generated samples must be challenging and diverse such that using the task instruction as a prompt will result in the wrong result.
2. The number of generated samples from each class in the task instruction should be balanced (i.e. the same number of samples for each class)
3. The generated samples should be distinct, realistic, and vary significantly to ensure diversity.

output should be in json format: 
```
{"samples" : [{"review": "...", "spoiler": "Yes/No"\}]}
```'
```` 

提示词模板路径: AutoPrompt/prompts/meta_prompts_classification/initial.prompt

### 2\. 对数据进行标注

标注用的提示词 + 样例数据 => 标注结果

````
Assistant is a large language model designed to classify challenging language tasks.
Given a list of 1 samples classify them according to the following task
### Task Instruction:
Does this movie review reveal plot points, characters, the ending, world-building details, or other key information not yet publicly available for the movie? (Yes/No)

### list of samples:
ID: 0;  Sample: {'review': 'The film was a rollercoaster of emotions, and the climax left me speechless. The twist at the end was something I never saw coming.', 'spoiler': 'No'}

##
Remember, follow carefully after the exact task instructions!

answer in json format, example:
```
{
  "id": 8,
  "prediction": "Yes"
}
```
```` 

提示词模板路径: AutoPrompt/prompts/predictor/prediction.prompt

### 3\. 使用初始提示词, 对数据进行推理

初始提示词 + 样例数据 => 标注结果

````
Assistant is a large language model designed to classify challenging language tasks.
Given a list of 1 samples classify them according to the following task
### Task Instruction:
Does this movie review contain a spoiler? answer Yes or No

### list of samples:
ID: 0;  Sample: {'review': 'The film was a rollercoaster of emotions, and the climax left me speechless. The twist at the end was something I never saw coming.', 'spoiler': 'No'}

##
Remember, follow carefully after the exact task instructions!

answer in json format, example:
```
{
  "id": 8,
  "prediction": "Yes"
}
```
```` 

提示词模板路径: AutoPrompt/prompts/predictor/prediction.prompt

#### 4\. 差错分析

任务描述 + 初始提示词 + 准确率 + 差错矩阵 + 失败样例数据 => 差错分析结论

````
Assistant is a large language model designed to provide a high quality analysis for every task.
You are given the following task description
Assistant is an expert classifier that will classify a movie review, and let the user know if it contains a spoiler for the reviewed movie or not.

Here is the prompt instructions that was given to the model:
Does this movie review contain a spoiler? answer Yes or No

The accuracy for this prompt is: 0.8
The confusion matrix for this prompt is: Confusion matrix columns:['Yes', 'No'] the matrix data:
Yes: [5 2]
No: [0 3]
##
Here is a list of failure cases for the given prompt:
##Failure Cases:
Sample: {'review': 'The film was a rollercoaster of emotions, and the climax left me speechless. The twist at the end was something I never saw coming.', 'spoiler': 'No'}
Prediction: No, GT:: Yes
#
Sample: {'review': 'The soundtrack was incredible and really added to the atmosphere. The way the plot unfolded was quite unexpected.', 'spoiler': 'No'}
Prediction: No, GT:: Yes
#

###
Note that the ground-truth labels are __absolutely correct__, but the prompts (task descriptions) may be incorrect and need modification.
Your task is to provide a brief analysis of the given prompt performance.
Guidelines:
1. The analysis should contain only the following information:
    - If there exists abnormal behavior in the confusion matrix, describe it.
    - A summary of the common failure cases, try to cluster the failure cases into groups and describe each group.
3. The total length of your analysis should be less than 200 token!
###

Remember, follow carefully after the exact task instructions!

answer in json format, example:
```
{
  "content": "(Your analysis)"
}
```
```` 

提示词模板路径: AutoPrompt/prompts/meta_prompts_classification/error_analysis.prompt

#### 5\. 生成下一轮新提示词: 

任务描述 + 历史提示词和得分 + 差错分析的结论 => 新提示词

````
Assistant is a large language model designed to provide the best prompt for every task.
Below are a few suggested prompts for the task and their score, for the following task:
Assistant is an expert classifier that will classify a movie review, and let the user know if it contains a spoiler for the reviewed movie or not.

## Examples
####
##Prompt Score: 0.80
##Prompt:
Does this movie review contain a spoiler? answer Yes or No
#################

######
This is the error analysis for the last prompt:
The confusion matrix indicates that the classifier has a tendency to predict 'Yes' more accurately than 'No'. It correctly identifies 5 out of 7 true positives and 3 out of 3 true negatives, but it misclassifies 2 reviews that do contain spoilers as not containing spoilers.

Failure Cases Summary:
1. **Emotional or Plot Descriptions Without Explicit Spoilers:** Reviews mentioning strong emotional reactions or plot descriptions (e.g., 'rollercoaster of emotions', 'plot unfolded unexpectedly') are often misclassified as not containing spoilers, even though they hint at significant plot twists or climaxes.
######
Your task is to generate:
1. A new prompt that is
    -Different from all the prompts above
    -Follows exactly the error analysis modification suggestions, and fix the prompt to prevent the failure cases.
    -Has a higher score than all the prompts above.
2. The predicted score of this prompt

You must adhere the error analysis instructions! even in case it seems there is a contradiction between these instructions, and the task. The error analysis is tested on a ground truth, thus represent the exact intent of the task.
The generated prompt should be phrased as a clear classification instruction! it should not include any instructions and descriptions on the modification that should be done to the prompt.
Note that the previous prompt contains an implicit assumptions on the intent of the task that might be incorrect. You should replace this assumption with more accurate assumptions using the score of the previous prompts and the error analysis.
The result prompt should indicate that the task is a classification class with the following labels ["Yes", "No"]!

Remember, follow carefully after the exact task instructions!

answer in json format, example:
```
{
  "prompt": "...",
}
```
```` 

提示词模板路径: AutoPrompt/prompts/meta_prompts_classification/step_prompt.prompt

#### 7\. 为新一轮生成用例

任务描述 + 上一轮提示词的差错样例 + (额外的样例??) + 新一轮提示词 => 新一轮的用例数据

````
Assistant is a large language model designed to generate challenging samples for every task.
Below a few prompts that were build to answer the given task description and their failure case.
Task description:
Assistant is an expert classifier that will classify a movie review, and let the user know if it contains a spoiler for the reviewed movie or not.

## Examples of common failure, each sample is followed by the the model prediction and the GT (ground truth)
####
##Prompt:
Does this movie review contain a spoiler? answer Yes or No
Sample: {'review': 'The film was a rollercoaster of emotions, and the climax left me speechless. The twist at the end was something I never saw coming.', 'spoiler': 'No'}
Prediction: No, GT:: Yes
#
Sample: {'review': 'The soundtrack was incredible and really added to the atmosphere. The way the plot unfolded was quite unexpected.', 'spoiler': 'No'}
Prediction: No, GT:: Yes
#
####
 
######
Here are few unique samples derived from realistic scenarios for the task outlined above.
## Realistic Samples
##
Sample:
 {'review': 'The movie had amazing visuals and a gripping storyline, but the reveal that the main character was a government agent was a bit predictable.', 'spoiler': 'Yes'}
#
Sample:
 {'review': 'The cinematography was breathtaking, but it was the character development that truly stood out to me.', 'spoiler': 'No'}
#
Sample:
 {'review': "I can't get over the fact that the protagonist was actually dead the whole time. That revelation turned the entire plot on its head!", 'spoiler': 'Yes'}
#

#####
This was the new proposed prompt:
## Prompt
Classify the following movie review to determine if it contains a spoiler. Labels: 'Yes' for spoiler and 'No' for no spoiler. Consider any emotional reactions or plot descriptions that may hint at significant plot twists or climaxes as spoilers.

Your task is to generate 10 by following this guidelines:
1. The generated samples should be diverse
2. They should preserve the style and the length of the given examples
3. The samples must be challenging and hard to classify by the model. This can be achieved by:
    1. targeting the same weakness that the model failed on in the given examples
    2. targeting weakness that are different from the existing examples in the failure cases
4. The number of generated samples from each class should be almost balanced (i.e. the same number of samples for each class)
5. The generated samples should include only the sample content without additional information! (like the model prediction and the ground truth)

output should be in json format: 
```
{"samples" : [{"review": "...", "spoiler": "Yes/No"}]}
```
```` 

提示词模板路径: AutoPrompt/prompts/meta_prompts_classification/step_samples.prompt

# 对AutoPrompt自带的提示词的分析

一共有如下提示词:

1 complete meta_prompts_classification/error_analysis.prompt 2 complete meta_prompts_classification/initial.prompt 3 complete meta_prompts_classification/initial_verbose.prompt 4 complete meta_prompts_classification/step_prompt.prompt 5 complete meta_prompts_classification/step_prompt_verbose.prompt 6 complete meta_prompts_classification/step_samples.prompt 7 complete meta_prompts_completion/error_analysis.prompt 8 complete meta_prompts_completion/initial.prompt 9 complete meta_prompts_completion/step_prompt.prompt 10 complete meta_prompts_completion/step_samples.prompt 11 complete meta_prompts_generation/error_analysis.prompt 12 complete meta_prompts_generation/initial.prompt 13 complete meta_prompts_generation/step_prompt.prompt 14 complete meta_prompts_generation/step_samples.prompt 15 complete meta_prompts_ranking/error_analysis.prompt 16 complete meta_prompts_ranking/initial.prompt 17 complete meta_prompts_ranking/initial_verbose.prompt 18 complete meta_prompts_ranking/step_prompt.prompt 19 complete meta_prompts_ranking/step_prompt_verbose.prompt 20 complete meta_prompts_ranking/step_samples.prompt 21 incomplete modifiers/ranker_prompt_mod.prompt 22 incomplete modifiers/ranker_task_desc_mod.prompt 23 complete predictor/prediction.prompt 24 complete predictor_completion/prediction.prompt 25 complete predictor_completion/prediction_generation.prompt 26 complete predictor_completion/prediction_verbose.prompt 

逐一分析: 

  1. 生成初始化的样例数据 (场景/initial.prompt)
     1. initial_verbose 比 initial 的描述会更详细 (啰嗦):

```
As an advanced language model you should create {num_samples} challenging and unique samples for the task outlined below.
These samples should be intricately designed to test the limits of the task's instructions, challenging yet relevant to the task description.

### Task Description:
{task_description}

### Task Instructions:
{instruction}

### Requirements for Challenging Samples:
1. Each sample must present a unique and intricate challenge.
2. The complexity of the samples should be such that simply applying the given task instruction would likely lead to incorrect or incomplete results.
3. The samples should cover a diverse range of scenarios within the scope of the task, avoiding repetition and predictability.
4. Ensure that the samples, while challenging, remain realistic and pertinent to the task's context.
```

     2. 不同场景下的要求会不同, 比如: classification和generation场景的提示词区别: 

```
< 1. The generated samples must be challenging and diverse such that using the task instruction as a prompt will result in the wrong result.
< 2. The number of generated samples from each class in the task instruction should be balanced (i.e. the same number of samples for each class)
< 3. The generated samples should be distinct, realistic, and vary significantly to ensure diversity.
---
> 1. Each prompt must present a unique and intricate challenge.
> 2. The prompts should cover a diverse range of scenarios within the scope of the task, avoiding repetition and predictability.
> 3. Each prompt should contain only the prompt part, without generating also the results
> 4. Each prompt should contain only the prompt part, without any mention of the task description or instructions!!
```

  2. 对数据进行标注 (predictor/* 或 predictor_completion/*)
     1. predictor_completion/prediction.prompt 和 predictor/prediction.prompt 一致
     2. predictor_completion/prediction.prompt 和predictor_completion/prediction_generation.prompt 只在输出格式上有差异 (生成分类还是文本)
     3. predictor_completion/prediction.prompt 和predictor_completion/prediction_verbose.prompt, verbose版本会输出 演算过程
  3. 使用初始提示词, 对数据进行推理 (predictor/* 或 predictor_completion/*)
  4. 差错分析 (场景/error_analysis.prompt)
     1. 只有generation场景的提示词会有区别: 总结的对象 从 case 变成了 case中的mistakes
  5. 生成下一轮新提示词 (场景/step_prompt.prompt)
     1. 只有generation场景的提示词会有区别: prompt 改名为 instruction (目的不确定)
     2. verbose版的要求会更具体, 并要求给出演算过程: 

```
< You must adhere the error analysis instructions! even in case it seems there is a contradiction between these instructions, and the task. The error analysis is tested on a ground truth, thus represent the exact intent of the task.
< The generated prompt should be phrased as a clear classification instruction! it should not include any instructions and descriptions on the modification that should be done to the prompt.
< Note that the previous prompt contains an implicit assumptions on the intent of the task that might be incorrect. You should replace this assumption with more accurate assumptions using the score of the previous prompts and the error analysis.
< The result prompt should indicate that the task is a classification class with the following labels {labels}!
<
<
< Remember, follow carefully after the exact task instructions!
---
> Guidelines for the new prompt:
> 1. The prompt is given a 'scratchpad', he can use it to extract from the sample text relevant information to make his prediction and perform a reasoning thought to get to the correct decision
> 2. The prompt is intended for a shallow LLM, which does not have access to previous failure cases or the analysis! he has only access to the generated new prompt which should be independent of the previous prompts.
> 4. Lists can organize the information and help the prompt (for example list of rules and a list of samples), the lists should be short and accurate
> 5. Note that the prompts and task descriptions may be inaccurate and need modification.
> 6. Note that higher score means better prompt.
> 7. The result prompt should indicate that the task is a classification class with the following labels {labels}!
```

  6. 为新一轮生成用例 (场景/step_samples.prompt)
     1. 不同场景会提出不同需求: 

````
root@d6700e62b4b6:/opt/jupyter-root/AutoPrompt/prompts# diff meta_prompts_classification/step_samples.prompt meta_prompts_ranking/step_samples.prompt
23,24c23
< 4. The number of generated samples from each class should be almost balanced (i.e. the same number of samples for each class)
< 5. The generated samples should include only the sample content without additional information! (like the model prediction and the ground truth)
---
> 4. The generated samples must be only from the top two scores! With equal distribution between the two!
26,29c25,29
< output should be in json format:
< ```
< {{"samples" : [{{"review": "...", "spoiler": "Yes/No"}}]}}
< ```
\ No newline at end of file
---
> If the task depends both on a context, or a user input and a generated content then the sample content must include all the relevant parts.
>     -In this case the sample content structure should be as follows:
>         1. First write the require context or user input.
>         2. Then write the generated content of the model on this context or user input.
>      The style of the separation and the indication of the different parts, should be different in each sample.
\ No newline at end of file
````

  7. 有两个没有用到的prompt: modifiers/ranker_prompt_mod.prompt, modifiers/ranker_task_desc_mod.prompt
     1. run_generation_pipeline.py 比起 run_pipeline.py脚本的区别, 是需要先训练 generation类型的任务的评估函数prompt, 再训练generation任务. 评估函数prompt是ranker类型的任务的prompt
     2. 这两个prompt, 主要是用于生成 ranker 的任务描述和提示词. 然后用优化后的ranker来进行generation任务的训练  

  
  

# 分析提示词探索的过程

从pickle文件中, 将提示词探索的过程提取出来, 使用以下脚本进行diff: 

```
#!/bin/bash

# 检查是否提供了文件名
if [ $# -ne 1 ]; then
    echo "Usage: $0 filename"
    exit 1
fi

# 读取文件
file=$1

# 初始化变量
prev_line=""

# 逐行读取文件内容
while IFS= read -r current_line; do
    if [ -n "$prev_line" ]; then
        # 使用 echo 将两行传递给 wdiff
        echo "$prev_line" > prev.txt
        echo "$current_line" > curr.txt
        wdiff -t prev.txt curr.txt
        echo ""
    fi
    prev_line="$current_line"
done < "$file"

# 清理临时文件
rm -f prev.txt curr.txt
``` 

结果 (用bash进行cat, 可看到高亮块): [3.txt](/assets/01KJBZEB61ZC33F8KAY5CR9GDJ/3.txt)

![image2024-8-14 16:57:0.png](/assets/01KJBZEB61ZC33F8KAY5CR9GDJ/image2024-8-14%2016%3A57%3A0.png)

调整方向: 

删除要求

![image2024-8-14 16:59:21.png](/assets/01KJBZEB61ZC33F8KAY5CR9GDJ/image2024-8-14%2016%3A59%3A21.png)

增加/细化 要求

![image2024-8-14 17:0:13.png](/assets/01KJBZEB61ZC33F8KAY5CR9GDJ/image2024-8-14%2017%3A0%3A13.png)

# 有用的机制

  - 差错分析
    - 差错分析中引入了差错矩阵, 增加了 差错分析的结论的 全面性
  - 在generation场景中, 先训练ranker, 再利用优化后的ranker, 训练generation
