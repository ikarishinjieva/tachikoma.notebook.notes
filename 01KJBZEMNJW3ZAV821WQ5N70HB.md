---
title: 20240822 - 试用 lavague
confluence_page_id: 3146217
created_at: 2024-08-21T16:13:50+00:00
updated_at: 2024-08-22T07:14:28+00:00
---

放在10.186.62.73

```
conda create --name myenv python=3.10
conda activate myenv
ALL_PROXY=http://10.186.16.136:7890 pip install lavague
 
cd /root/lavague-test
ALL_PROXY=http://10.186.16.136:7890 python 1.py
``` 

1.py:

```
import nltk
nltk.set_proxy('http://10.186.16.136:7890')
#nltk.download()

from lavague.core import  WorldModel, ActionEngine
from lavague.core.agents import WebAgent
from lavague.drivers.selenium import SeleniumDriver

selenium_driver = SeleniumDriver(headless=False)
world_model = WorldModel()
action_engine = ActionEngine(selenium_driver)
agent = WebAgent(world_model, action_engine)
agent.get("https://huggingface.co/docs")
agent.run("Go on the quicktour of PEFT")

# Launch Gradio Agent Demo
agent.demo("Go on the quicktour of PEFT")
``` 

报错: 

```
(myenv) root@ubuntu:~/lavague-test# python 1.py
/root/anaconda3/envs/myenv/lib/python3.10/site-packages/lavague/core/__init__.py:21: UserWarning: Telemetry is turned on. To turn off telemetry, set your LAVAGUE_TELEMETRY to 'NONE'
  warnings.warn(warning_message, UserWarning)
Traceback (most recent call last):
  File "/root/lavague-test/1.py", line 9, in <module>
    selenium_driver = SeleniumDriver(headless=False)
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/lavague/drivers/selenium/base.py", line 74, in __init__
    super().__init__(url, get_selenium_driver)
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/lavague/core/base_driver.py", line 34, in __init__
    self.driver = self.init_function()
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/lavague/drivers/selenium/base.py", line 111, in default_init_code
    self.driver = webdriver.Chrome(options=chrome_options)
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/selenium/webdriver/chrome/webdriver.py", line 45, in __init__
    super().__init__(
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/selenium/webdriver/chromium/webdriver.py", line 66, in __init__
    super().__init__(command_executor=executor, options=options)
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/selenium/webdriver/remote/webdriver.py", line 212, in __init__
    self.start_session(capabilities)
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/selenium/webdriver/remote/webdriver.py", line 299, in start_session
    response = self.execute(Command.NEW_SESSION, caps)["value"]
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/selenium/webdriver/remote/webdriver.py", line 354, in execute
    self.error_handler.check_response(response)
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/selenium/webdriver/remote/errorhandler.py", line 229, in check_response
    raise exception_class(message, screen, stacktrace)
selenium.common.exceptions.SessionNotCreatedException: Message: session not created: Chrome failed to start: exited normally.
  (session not created: DevToolsActivePort file doesn't exist)
  (The process started from chrome location /root/.cache/selenium/chrome/linux64/127.0.6533.119/chrome is no longer running, so ChromeDriver is assuming that Chrome has crashed.)
Stacktrace:
#0 0x558f895bd6ca <unknown>
#1 0x558f8928e600 <unknown>
#2 0x558f892c6485 <unknown>
#3 0x558f892c22f8 <unknown>
#4 0x558f8930cce8 <unknown>
#5 0x558f89300643 <unknown>
#6 0x558f892d0d31 <unknown>
#7 0x558f892d179e <unknown>
#8 0x558f8958525b <unknown>
#9 0x558f895891f2 <unknown>
#10 0x558f89572615 <unknown>
#11 0x558f89589d82 <unknown>
#12 0x558f8955725f <unknown>
#13 0x558f895ace68 <unknown>
#14 0x558f895ad040 <unknown>
#15 0x558f895bc49c <unknown>
#16 0x7fcd8294e6db start_thread

(myenv) root@ubuntu:~/lavague-test#
``` 

手工执行chrome报错: 

```
(myenv) root@ubuntu:~/lavague-test# /root/.cache/selenium/chrome/linux64/127.0.6533.119/chrome
/root/.cache/selenium/chrome/linux64/127.0.6533.119/chrome: error while loading shared libraries: libgbm.so.1: cannot open shared object file: No such file or directory
``` 

安装libgbm1进行修复: 

```
apt-get install -y libgbm1
``` 

chrome继续报错: 

```
(myenv) root@ubuntu:~/lavague-test# /root/.cache/selenium/chrome/linux64/127.0.6533.119/chrome -no-sandbox
[20291:20291:0822/004611.818975:ERROR:ozone_platform_x11.cc(244)] Missing X server or $DISPLAY
[20291:20291:0822/004611.819050:ERROR:env.cc(258)] The platform failed to initialize.  Exiting.
``` 

在Mac上, 启动XQuatz, 使用

```
ssh -XY 10.186.62.73 -l root xterm
``` 

![image2024-8-22 1:4:46.png](/assets/01KJBZEMNJW3ZAV821WQ5N70HB/image2024-8-22%201%3A4%3A46.png)

会启动一个带GUI的ssh窗口, 执行chrome命令: 

![image2024-8-22 1:5:40.png](/assets/01KJBZEMNJW3ZAV821WQ5N70HB/image2024-8-22%201%3A5%3A40.png)

可启动chrome

执行脚本, 可正确打开chrome, 报错 "OPENAI_API_KEY is not set", 已可正常使用. 下一步要更换模型

修改脚本: 

```
import nltk
nltk.set_proxy('http://10.186.16.136:7890')
#nltk.download()

from lavague.core import  WorldModel, ActionEngine
from lavague.core.agents import WebAgent
from lavague.drivers.selenium import SeleniumDriver

from llama_index.llms.azure_openai import AzureOpenAI

llm = AzureOpenAI(
    api_key="2de3b03c4ebd4f1f8681ce7d86ef475d",
    model="gpt-4o",
    azure_endpoint="https://tachikoma.openai.azure.com/",
    deployment_name="gpt-4o-2024-05-13"
)

from llama_index.multi_modal_llms.azure_openai import AzureOpenAIMultiModal
mm_llm = AzureOpenAIMultiModal(
    api_key="2de3b03c4ebd4f1f8681ce7d86ef475d",
    model="gpt-4o",
    azure_endpoint="https://tachikoma.openai.azure.com/",
    deployment_name="gpt-4o-2024-05-13"
)

from llama_index.embeddings.azure_openai import AzureOpenAIEmbedding
embedding = AzureOpenAIEmbedding(
    api_key="2de3b03c4ebd4f1f8681ce7d86ef475d",
    model="text-embedding-3-small",
    azure_endpoint="https://tachikoma.openai.azure.com/",
    azure_deployment="text-embedding-3-small",
),

from selenium.webdriver.chrome.options import Options
chrome_options = Options()
user_agent = "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36"
chrome_options.add_argument(f"user-agent={user_agent}")
chrome_options.add_argument("--no-sandbox")
chrome_options.add_argument('--proxy-server=http://10.186.16.136:7890')
chrome_options.page_load_strategy = "normal"

selenium_driver = SeleniumDriver(headless=False, options=chrome_options)
world_model = WorldModel(mm_llm=mm_llm)
action_engine = ActionEngine(llm=llm, embedding=embedding, driver=selenium_driver)
agent = WebAgent(world_model, action_engine)
agent.get("https://huggingface.co/docs")
agent.run("Go on the quicktour of PEFT")

# Launch Gradio Agent Demo
agent.demo("Go on the quicktour of PEFT")
``` 

报错: 

```
2024-08-22 01:31:02,371 - INFO - Screenshot folder cleared
2024-08-22 01:31:26,904 - ERROR - Error while running the agent: Error code: 404 - {'error': {'code': '404', 'message': 'Resource not found'}}
Traceback (most recent call last):
  File "/root/lavague-test/1.py", line 48, in <module>
    agent.run("Go on the quicktour of PEFT")
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/lavague/core/agents.py", line 521, in run
    raise e
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/lavague/core/agents.py", line 506, in run
    result = self.run_step(objective)
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/lavague/core/agents.py", line 453, in run_step
    world_model_output = self.world_model.get_instruction(
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/lavague/core/world_model.py", line 433, in get_instruction
    mm_llm_output = mm_llm.complete(prompt, image_documents=image_documents).text
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/llama_index/core/instrumentation/dispatcher.py", line 230, in wrapper
    result = func(*args, **kwargs)
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/llama_index/core/llms/callbacks.py", line 429, in wrapped_llm_predict
    f_return_val = f(_self, *args, **kwargs)
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/llama_index/multi_modal_llms/openai/base.py", line 337, in complete
    return self._complete(prompt, image_documents, **kwargs)
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/llama_index/core/llms/callbacks.py", line 429, in wrapped_llm_predict
    f_return_val = f(_self, *args, **kwargs)
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/llama_index/multi_modal_llms/openai/base.py", line 219, in _complete
    response = self._client.chat.completions.create(
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/openai/_utils/_utils.py", line 274, in wrapper
    return func(*args, **kwargs)
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/openai/resources/chat/completions.py", line 668, in create
    return self._post(
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/openai/_base_client.py", line 1260, in post
    return cast(ResponseT, self.request(cast_to, opts, stream=stream, stream_cls=stream_cls))
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/openai/_base_client.py", line 937, in request
    return self._request(
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/openai/_base_client.py", line 1041, in _request
    raise self._make_status_error_from_response(err.response) from None
openai.NotFoundError: Error code: 404 - {'error': {'code': '404', 'message': 'Resource not found'}}

``` 

修改代码增加debug日志, 可看到具体报错: 

![image2024-8-22 1:54:47.png](/assets/01KJBZEMNJW3ZAV821WQ5N70HB/image2024-8-22%201%3A54%3A47.png)

修改代码, 增加api_version:

```
import logging
logging.basicConfig(level=logging.DEBUG)

import nltk
nltk.set_proxy('http://10.186.16.136:7890')
#nltk.download()

from lavague.core import  WorldModel, ActionEngine
from lavague.core.agents import WebAgent
from lavague.drivers.selenium import SeleniumDriver

from llama_index.llms.azure_openai import AzureOpenAI

llm = AzureOpenAI(
    api_key="2de3b03c4ebd4f1f8681ce7d86ef475d",
    model="gpt-4o",
    azure_endpoint="https://tachikoma.openai.azure.com/",
    deployment_name="gpt-4o-2024-05-13",
    api_version="2024-02-01"
)

from llama_index.multi_modal_llms.azure_openai import AzureOpenAIMultiModal
mm_llm = AzureOpenAIMultiModal(
    api_key="2de3b03c4ebd4f1f8681ce7d86ef475d",
    model="gpt-4o-mini",
    azure_endpoint="https://tachikoma.openai.azure.com/",
    deployment_name="gpt-4o-mini-20240718",
    api_version="2024-02-01"
)

from llama_index.embeddings.azure_openai import AzureOpenAIEmbedding
embedding = AzureOpenAIEmbedding(
    api_key="2de3b03c4ebd4f1f8681ce7d86ef475d",
    model="text-embedding-3-small",
    azure_endpoint="https://tachikoma.openai.azure.com/",
    azure_deployment="text-embedding-3-small",
    api_version="2024-02-01"
),

from selenium.webdriver.chrome.options import Options
chrome_options = Options()
user_agent = "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36"
chrome_options.add_argument(f"user-agent={user_agent}")
chrome_options.add_argument("--no-sandbox")
chrome_options.add_argument('--proxy-server=http://10.186.16.136:7890')
chrome_options.page_load_strategy = "normal"

selenium_driver = SeleniumDriver(headless=False, options=chrome_options)
world_model = WorldModel(mm_llm=mm_llm)
action_engine = ActionEngine(llm=llm, embedding=embedding, driver=selenium_driver)
agent = WebAgent(world_model, action_engine)
agent.get("https://huggingface.co/docs")
agent.run("Go on the quicktour of PEFT")

# Launch Gradio Agent Demo
agent.demo("Go on the quicktour of PEFT")
``` 

报错: 

```
DEBUG:urllib3.connectionpool:https://telemetrylavague.mithrilsecurity.io:443 "POST /telemetry_new HTTP/11" 200 2
Traceback (most recent call last):
  File "/root/lavague-test/1.py", line 54, in <module>
    agent.run("Go on the quicktour of PEFT")
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/lavague/core/agents.py", line 521, in run
    raise e
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/lavague/core/agents.py", line 506, in run
    result = self.run_step(objective)
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/lavague/core/agents.py", line 470, in run_step
    action_result = self.action_engine.dispatch_instruction(
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/lavague/core/action_engine.py", line 238, in dispatch_instruction
    return next_engine.execute_instruction(instruction)
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/lavague/core/navigation.py", line 417, in execute_instruction
    source_nodes = self.get_nodes(instruction)
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/lavague/core/navigation.py", line 155, in get_nodes
    source_nodes = self.retriever.retrieve(
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/lavague/core/retrievers.py", line 50, in retrieve
    html_nodes = retriever.retrieve(query, html_nodes, viewport_only)
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/lavague/core/retrievers.py", line 492, in retrieve
    index = VectorStoreIndex(nodes=nodes, embed_model=self.embedding)
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/llama_index/core/indices/vector_store/base.py", line 69, in __init__
    resolve_embed_model(embed_model, callback_manager=callback_manager)
  File "/root/anaconda3/envs/myenv/lib/python3.10/site-packages/llama_index/core/embeddings/utils.py", line 136, in resolve_embed_model
    embed_model.callback_manager = callback_manager or Settings.callback_manager
AttributeError: 'tuple' object has no attribute 'callback_manager'
``` 
    
    
    AzureOpenAIEmbedding对象定义最后多了一个逗号.

调整代码: 

```
import logging
logging.basicConfig(level=logging.DEBUG)

import os
os.environ["no_proxy"] = "localhost,127.0.0.1,::1"

import nltk
nltk.set_proxy('http://10.186.16.136:7890')
#nltk.download()

from lavague.core import  WorldModel, ActionEngine
from lavague.core.agents import WebAgent
from lavague.drivers.selenium import SeleniumDriver

from llama_index.llms.azure_openai import AzureOpenAI

llm = AzureOpenAI(
    api_key="2de3b03c4ebd4f1f8681ce7d86ef475d",
    model="gpt-4o",
    azure_endpoint="https://tachikoma.openai.azure.com/",
    deployment_name="gpt-4o-2024-05-13",
    api_version="2024-02-01"
)

from llama_index.multi_modal_llms.azure_openai import AzureOpenAIMultiModal
mm_llm = AzureOpenAIMultiModal(
    api_key="2de3b03c4ebd4f1f8681ce7d86ef475d",
    model="gpt-4o-mini",
    azure_endpoint="https://tachikoma.openai.azure.com/",
    deployment_name="gpt-4o-mini-20240718",
    api_version="2024-02-01"
)

from llama_index.embeddings.azure_openai import AzureOpenAIEmbedding
embedding = AzureOpenAIEmbedding(
    api_key="2de3b03c4ebd4f1f8681ce7d86ef475d",
    model="text-embedding-3-small",
    azure_endpoint="https://tachikoma.openai.azure.com/",
    azure_deployment="text-embedding-3-small",
    api_version="2024-02-01"
)

from selenium.webdriver.chrome.options import Options
chrome_options = Options()
user_agent = "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36"
chrome_options.add_argument(f"user-agent={user_agent}")
chrome_options.add_argument("--no-sandbox")
chrome_options.add_argument('--proxy-server=http://10.186.16.136:7890')
chrome_options.page_load_strategy = "normal"

selenium_driver = SeleniumDriver(headless=False, options=chrome_options)
world_model = WorldModel(mm_llm=mm_llm)
action_engine = ActionEngine(llm=llm, embedding=embedding, driver=selenium_driver)
agent = WebAgent(world_model, action_engine)
agent.get("https://huggingface.co/docs")
#agent.run("Go on the quicktour of PEFT")

# Launch Gradio Agent Demo
agent.demo("Go on the quicktour of PEFT")
 
``` 

可正确运行

  - 需要修改 /root/anaconda3/envs/myenv/lib/python3.10/site-packages/lavague/gradio/base.py, 修改最后一行以暴露gradio端口: 

```
demo.launch(server_name="0.0.0.0", server_port=server_port, share=False, debug=True)
```
