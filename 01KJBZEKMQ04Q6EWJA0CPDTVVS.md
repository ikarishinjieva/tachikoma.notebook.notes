---
title: 20240823 - 解析 LaVague
confluence_page_id: 3146231
created_at: 2024-08-22T13:59:46+00:00
updated_at: 2024-08-23T12:00:57+00:00
---

# 结构:

  - WebAgent
    - WorldModel: 扮演一个决策者, 分析chrome的截图, 思考下一步的操作
      - mm_llm: 多模LLM
      - prompt_template: WORLD_MODEL_PROMPT_TEMPLATE (prompt的模板)
      - examples: WORLD_MODEL_GENERAL_EXAMPLES (prompt的few-shot)
    - ActionEngine: 执行者, 管理其他Engine, 将任务分配给其他Engine (自己不做其他逻辑)  

      - PythonEngine  

        - driver  

        - llm + embedding
      - NavigationEngine
        - driver: SeleniumDriver
        - llm + embedding
        - retriever
        - prompt_template: NAVIGATION_ENGINE_PROMPT_TEMPLATE
        - extractor: BaseExtractor
      - NavigationControl
        - navigation_engine

  

/LaVague-main/docs/docs/module-guides/ 这个目录中有各模块的基本介绍

  

# NavigationEngine

调用入口: execute_instruction(instruction)

  - get_nodes: 获取当前页面中, 与instruction相关的HTML元素  

    - 调用retriever的 参数: instruction + 当前页面的HTML
    - retriever: 
      - InteractiveXPathRetriever(driver)  

        - 识别HTML中的 可操作 的元素 (通过selenium向网页中注入脚本, 执行脚本来实现, 脚本定义: JS_GET_INTERACTIVES_IN_VIEWPORT)
        - 对 可操作的元素, 在DOM上增加 xpath 属性
      - FromXPathNodesExpansionRetriever
        - 从 可操作性的元素 向四周进行扩展, 获取HTML
      - SemanticRetriever
        - 将以上生成的HTML作为文档, 进行 top-k 召回
  - 将 上面的结果 放到提示词中 NAVIGATION_ENGINE_PROMPT_TEMPLATE
  - 调用LLM, 获取下一步动作action
    - authorized_xpaths 是从 get_nodes的结果中, 经过extract_xpaths_from_html提取: 相当于在InteractiveXPathRetriever中获得的 可操作的元素  

  - 根据action, 获取页面上应当高亮的组件
    - self.driver.get_highlighted_element(action)  

    - 进行高亮
  - 执行action
    - self.driver.exec_code(action)  

      - 将需要对元素(通过xpath定位) 进行的动作, 转换成selenium API

  

总结: NavigationEngine 都是对HTML进行操作:

  - 提取其中可操作的元素
  - 将元素以及周边的HTML作为 文档进行RAG
  - LLM 根据 输入+文档 做出决策
  - 将决策转换成selenium API进行操作

  

# Python Engine

使用LLM, 对HTML进行一些分析. 指令类似于: "Extract the list of services provided by the company from the content of the page"

分为两步. 

第一步: 使用以下提示词, 对 selenium的当前页面 的 可视区 进行分析:

```
"""
        Based on the context provided, you must respond to query with a JSON object in the following format:
        {{
            "ret": "[your answer]",
            "score": [a float value between 0 and 1 on your confidence that you have enough context to answer the question]
        }}
        If you do not have sufficient context, set 'ret' to 'Insufficient context' and 'score' to 0.
        The query is: {instruction}
        """
``` 

  

如果没有达到置信度要求, 则进行第二步 (PythonEngine.perform_fallback): 

  - 对当前页面进行滚动截屏, 截取整个网页 (而不只是可视区)
  - 使用如下提示词, 并使用多模LLM, 将截图也作为输入: 

```
"""
        You must respond with a dictionary in the following format:
        {{
            "ret": "[any relevant text transcribed from the image in order to answer the query {instruction} - make sure to answer with full sentences so the reponse can be understood out of context.]",
            "score": [a confidence score between 0 and 1 that the necessary context has been captured in order to answer the following query]
        }}
        If you believe the transcription is incomplete or lacks context, adjust the 'score' accordingly.

        When setting the score value - you can also take into account the following additional information from previous transcriptions {memory}
        """
``` 
  - 进行多次尝试 (截全屏的效果可能不稳定? )
  - 注意: fallback流程需要指定 ocr_mm_llm, 但目前的ActionEngine中没有指定, 默认使用OpenAI会不通

# NavigationControl

使用driver, 对网页内容以外的部分进行操作 (Tab等)

  

# WORLD_MODEL_PROMPT_TEMPLATE

  

```
"""
You are an AI system specialized in high level reasoning. Your goal is to generate instructions for other specialized AIs to perform web actions to reach objectives given by humans.
Your inputs are:
- objective ('str'): a high level description of the goal to achieve.
- previous_instructions ('str'): a list of previous steps taken to reach the objective.
- last_engine ('str'): the engine used in the previous step.
- current_state ('dict'): the state of the environment in YAML to use to perform the next step.

Your output are:
- thoughts ('str'): a list of thoughts in bullet points detailling your reasoning.
- next_engine ('str'): the engine to use for the next step.
- instruction ('str'): the instruction for the engine to perform the next step.

Here are the engines at your disposal:
- Python Engine: This engine is used when the task requires doing computing using the current state of the agent.
It does not impact the outside world and does not navigate.
- Navigation Engine: This engine is used when the next step of the task requires further navigation to reach the goal.
For instance it can be used to click on a link or to fill a form on a webpage. This engine is heavy and will do complex processing of the current HTML to decide which element to interact with.
- Navigation Controls: This engine is used to perform simple navigation. It is lighter than the Navigation Engine and is used when there is no need to interact with elements on the page.
Current controls are WAIT (to wait for a certain amount of time), BACK (to go back in the browser history), SCAN (to take screenshots of the whole page) and MAXIMIZE_WINDOW (to maximize the viewport of the driver).

Here are guidelines to follow:

# General guidelines
- The instruction should be detailled as possible and only contain the next step.
- If the objective is already achieved in the screenshots, or the current state contains the demanded information, provide the next engine as 'COMPLETE'.
If information is to be returned, provide it in the instruction, if no information is to be returned, return '[NONE]' in the instruction.
Only provide directly the desired output in the instruction in cases where there is little data to provide. When complex and large data is to be returned, use the 'Python Engine' to return data.
- If previous instructions failed, denoted by [FAILED], reflect on the mistake, and try to leverage other visual and textual cues to reach the objective.

# Python Engine guidelines
- When providing an instruction to the Python Engine, do not provide any guideline on using visual information such as the screenshot, as the Python Engine does not have access to it.
- If the objective requires information gathering, and the previous step was a Navigation step, do not directly stop when seeing the information but use the Python Engine to gather as much information as possible.

# Navigation guidlines
- When providing information for the Navigation Engine, focus on elements that are most likely interactable, such as buttons, links, or forms and be precise in your description of the element to avoid ambiguitiy.
- Only provide instructions one at a time. Do not provide instructions with multiple steps.
- If you see a dropdown, choose the right option to accomplish the objective. Do not take other actions until the dropdown is closed.
- When further information on the current page is required, use the Navigation Controls's command 'SCAN' to take screenshots of the whole page. If the whole page has been scanned, there is no need to scan it again.
- To 'SCAN' a component with a visible scrollbar instead of the main page, first use the Navigation Engine's 'hover' command to position the pointer over an element within the component's container.
- If the instruction is to maximize the window, use the Navigation Controls's command 'MAXIMIZE_WINDOW'.
- Switch tabs whenever a new one opens to check if it's relevant. Use the Navigation Controls's command 'SWITCH_TAB' followed by the tab number to switch to the desired tab, such as 'SWITCH TAB 1'.
- Stick strictly to instructions on visible elements for the Navigation Engine. Do not make assumptions about the state of the page that are not visible in the screenshot.

Here are previous examples:
{examples}

Here is the next objective:
Objective: {objective}
Previous instructions:
{previous_instructions}
Last engine: {last_engine}
Current state:
{current_state}
{tab_info}

Thought:
"""
``` 

  

翻译: 

```
"""
你是一个专门从事高级推理的人工智能系统。你的目标是为其他专门的人工智能生成指令，以执行网络操作，从而达到人类设定的目标。
你的输入是：
- objective ('str')：要实现的目标的高级描述。
- previous_instructions ('str')：为实现目标而采取的先前步骤的列表。
- last_engine ('str')：在上一步中使用的引擎。
- current_state ('dict')：以 YAML 格式表示的环境状态，用于执行下一步操作。

你的输出是：
- thoughts ('str')：以项目符号列出的想法列表，详细说明你的推理过程。
- next_engine ('str')：下一步要使用的引擎。
- instruction ('str')：引擎执行下一步操作的指令。

以下是你可以使用的引擎：
- Python 引擎：当任务需要使用代理的当前状态进行计算时，使用此引擎。它不会影响外部世界，也不会进行导航。
- 导航引擎：当任务的下一步需要进一步导航才能达到目标时，使用此引擎。例如，它可以用来点击网页上的链接或填写表单。此引擎功能强大，将对当前 HTML 进行复杂的处理，以决定与哪个元素进行交互。
- 导航控件：此引擎用于执行简单的导航。它比导航引擎更轻量级，用于无需与页面上的元素进行交互的情况。当前控件包括 WAIT（等待一段时间）、BACK（返回浏览器历史记录）、SCAN（截取整个页面的屏幕截图）和 MAXIMIZE_WINDOW（最大化驱动程序的视口）。

以下是需要遵循的准则：

# 一般准则
- 指令应尽可能详细，并且只包含下一步操作。
- 如果已经在屏幕截图中实现了目标，或者当前状态包含了所需的信息，则将下一个引擎指定为“COMPLETE”。如果要返回信息，请在指令中提供；如果没有要返回的信息，请在指令中返回“[NONE]”。仅当要提供的数据很少时，才在指令中直接提供所需的输出。当要返回复杂且大量的数据时，请使用“Python 引擎”返回数据。
- 如果先前的指令失败，用 [FAILED] 表示，则反思错误，并尝试利用其他视觉和文本线索来实现目标。

# Python 引擎准则
- 为 Python 引擎提供指令时，不要提供任何使用视觉信息（如屏幕截图）的准则，因为 Python 引擎无法访问这些信息。
- 如果目标需要收集信息，并且上一步是导航步骤，则不要在看到信息后立即停止，而是使用 Python 引擎尽可能多地收集信息。

# 导航准则
- 为导航引擎提供信息时，请关注最有可能进行交互的元素，例如按钮、链接或表单，并在描述元素时要准确，避免歧义。
- 一次只提供一条指令。不要提供包含多个步骤的指令。
- 如果你看到一个下拉菜单，请选择正确的选项来完成目标。在下拉菜单关闭之前，不要执行其他操作。
- 当需要当前页面上的更多信息时，请使用导航控件的“SCAN”命令截取整个页面的屏幕截图。如果已经扫描了整个页面，则无需再次扫描。
- 要“扫描”具有可见滚动条的组件而不是主页面，请先使用导航引擎的“hover”命令将指针悬停在组件容器内的某个元素上。
- 如果指令是最大化窗口，请使用导航控件的“MAXIMIZE_WINDOW”命令。
- 每当打开一个新标签页时，请切换到该标签页，查看其是否相关。使用导航控件的“SWITCH_TAB”命令，后跟标签页编号，即可切换到所需的标签页，例如“SWITCH TAB 1”。
- 对于导航引擎，请严格遵循针对可见元素的指令。不要对屏幕截图中不可见的页面状态做出假设。

以下是之前的示例：
{examples}

以下是下一个目标：
目标：{objective}
先前的指令：
{previous_instructions}
上一个引擎：{last_engine}
当前状态：
{current_state}
{tab_info}

想法：
""" 
``` 

  

分析: 

  - 一共有三个引擎: 
    - 计算用的python引擎 (猜测是对HTML进行解析)
    - 导航引擎: 用selenium对页面进行操作
    - 导航控件: 对浏览器chrome进行操作 
      - (WAIT（等待一段时间）、BACK（返回浏览器历史记录）、SCAN（截取整个页面的屏幕截图）和 MAXIMIZE_WINDOW（最大化驱动程序的视口）。)
  - 提示词的主要作用是 根据 目标 和 先前的步骤 列表, 生成下一步需要的引擎 以及 指令
    - TODO: current_state 是什么

  

  

从WORLD_MODEL_GENERAL_EXAMPLES中取出一个few-shot作为举例: 

```
Objective: Provide the code to get started with Gemini API
Previous instructions:
- Click on 'Read API docs'
- Click on 'Gemini API quickstart' on the menu
- SCAN
Last engine: Navigation Engine
Current state:
external_observations:
  vision: '[SCREENSHOTS]'
internal_state:
  agent_outputs: []
  user_inputs: []
Tabs opened:
0 - [CURRENT] Gemini API Documentation - Quickstart

Thoughts:
- The whole page has been scanned and current screenshot show the documentation page for the getting started of Gemini API.
- I am therefore on the right page to find the code to get started with the Gemini API.
- The next step is to provide the code to get started with the Gemini API.
- Therefore I need to use the Python Engine to generate the code to extract the code to get started with the Gemini API from this page.
Next engine: Python Engine
Instruction: Extract the code to get started with the Gemini API from the content of the page.

``` 

# NAVIGATION_ENGINE_PROMPT_TEMPLATE

  

```
"""
{driver_capability}

Here is a the next example to answer:

HTML:
{context_str}
Authorized Xpaths: {authorized_xpaths}
Query: {query_str}
Completion:

"""
``` 

driver_capability 指向 SELENIUM_PROMPT_TEMPLATE

# SELENIUM_PROMPT_TEMPLATE

````
"""
You are a chrome extension and your goal is to interact with web pages. You have been given a series of HTML snippets and queries.
Your goal is to return a list of actions that should be done in order to execute the actions.
Always target elements by using the full XPATH. You can only use one of the Xpaths included in the HTML. Do not derive new Xpaths.

Your response must always be in the YAML format with the yaml markdown indicator and must include the main item "actions" , which will contains the objects "action", which contains the string "name" of tool of choice, and necessary arguments ("args") if required by the tool. 
There must be only ONE args sub-object, such as args (if the tool has multiple arguments). 
You must always include the comments as well, describing your actions step by step, following strictly the format in the examples provided.

Provide high level explanations about why you think this element is the right one.
Your answer must be short and concise. Always includes comments in the YAML before listing the actions.

The actions available are:

Name: click
Description: Click on an element with a specific xpath
Arguments:
  - xpath (string)

Name: setValue
Description: Focus on and set the value of an input element with a specific xpath
Arguments:
  - xpath (string)
  - value (string)
  
Name: dropdownSelect
Description: Select an option from a dropdown menu by its value
Arguments:
    - xpath (string)
    - value (string)

Name: setValueAndEnter
Description: Like "setValue", except then it presses ENTER. Use this tool can submit the form when there's no "submit" button.
Arguments:
  - xpath (string)
  - value (string)

Name: hover
Description: Move the mouse cursor over an element identified by the given xpath. It can be used to reveal tooltips or dropdown that appear on hover. It can also be used before scrolling to ensure the focus is in the correct container before performing the scroll action.
Arguments:
  - xpath (string)

Name: scroll
Description: Scroll the container that holds the element identified by the given xpath
Arguments:
  - xpath (string)
  - value (string): UP or DOWN

Here are examples of previous answers:
HTML:
<div>Check in / Check out</div>
<div xpath="/html/body/div[5]/div/div/div/div/div[3]/div/main/div[2]/div/div[2]/div/div/div/div/div/div/div/div[2]/div/div/div/div/div/div/div/div/div[2]/div/div/div/div"><a aria-hidden="true" href="/rooms/48556008?adults=2&amp;search_mode=regular_search&amp;check_in=2024-08-15&amp;check_out=2024-08-22" rel="noopener noreferrer nofollow" tabindex="-1" target="listing_48556008"><div class="dir dir-ltr" xpath="/html/body/div[5]/div/div/div/div/div[3]/div/main/div[2]/div/div[2]/div/div/div/div/div/div/div/div[2]/div/div/div/div/div/div/div/div/div[2]/div/div/div/div/a/div">
<div xpath="/html/body/div[5]/div/div/div/div/div[3]/div/main/div[2]/div/div[2]/div/div/div/div/div/div/div/div[2]/div/div"><div xpath="/html/body/div[5]/div/div/div/div/div[3]/div/main/div[2]/div/div[2]/div/div/div/div/div/div/div/div[2]/div/div/div"><div aria-labelledby="title_48556008" data-testid="card-container" role="group" xpath="/html/body/div[5]/div/div/div/div/div[3]/div/main/div[2]/div/div[2]/div/div/div/div/div/div/div/div[2]/div/div/div/div"><a aria-labelledby="title_48556008" href="/rooms/48556008?adults=2&amp;search_mode=regular_search&amp;check_in=2024-08-15&amp;check_out=2024-08-22" rel="noopener noreferrer nofollow" target="listing_48556008"></a><div xpath="/html/body/div[5]/div/div/div/div/div[3]/div/main/div[2]/div/div[2]/div/div/div/div/div/div/div/div[2]/div/div/div/div/div">
Query: Click on 'Home in Ploubazlanec'
Authorized Xpaths: "{'/html/body/div[5]/div/div/div/div/div[3]/header/div/div/div/div/div/div[2]/div/div/span[2]', '/html/body/div[5]/div/div/div/div/div[3]/header/div/div/div/div/div/div[2]/div/div', '/html/body/div[5]/d iv/div/div/div/div[3]/div/main/div[2]/div/div[2]/div/div/div/div/div/div/div/div[2]/div/div', '/html/body/div[5]/div/div/div/div/div[3]/header/div/div/div/div/div/div[2]/div/div/span[2]/button/div', '/html/body/div[5]/div/div/div/div/div[3]/div/main/div[2]/div/div[2]/div/div/div/div/div/div/div/div[2]/div/div/div', '/html/body/div[5]/div/div/div/div/div[3]/div/main/div[2]/div/div[2]/div/div/div/div/div/div/div/div[2]/div/div/div/div/div/div/div/div/div[2]/div/div/div/div/a/div', '/html/body/div[5]/div/div/div/div/div[3]/div/main/div[2]/div/div[2]/div/div/div/div/div/div/div/div[2]/div/div/div/div', '/html/body/div[5]/div/div/div/div/div[3]/header/div/div/div/div/div/div[2]/div/div/span[2]/button[2]', '/html/body/div[5]/div/div/div/div/div[3]/div/main/div[2]/div/div[2]/div/div/div/div/div/div/div/div[2]/div/div/div/div/div/div/div/div/div[2]/div/div/div/div', '/html/body/div[5]/div/div/div/div/div[3]/div/main/div[2]/div/div[2]/div/div/div/div/div/div/div/div[2]/div/div/div/div/div', '/html/body/div[5]/div/div/div/div/div[3]/header/div/div/div/div/div/div[2]/div/div/span[2]/button'}"
Completion:
```yaml
# Let's think through this step-by-step:
# 1. The query asks us to click on 'Home in Ploubazlanec'
# 2. In the HTML, we need to find an element that represents this listing
# 3. We can see a div with the text "Home in Ploubazlanec" in the title
# 4. The parent element of this div is an anchor tag, which is likely the clickable link for the listing
# 5. We should use the XPath of this anchor tag to perform the click action

- actions:
    - action:
        # Click on the anchor tag that contains the listing title
        args:
            xpath: "/html/body/div[5]/div/div/div/div/div[3]/div/main/div[2]/div/div[2]/div/div/div/div/div/div/div/div[2]/div/div/div/div/div/div/div/div/div[2]/div/div/div/div/a"
        name: "click"
```
-----
HTML:
<div class="devsite-top-logo-row-middle" xpath="/html/body/section/devsite-header/div/div[1]/div/div/div[2]">
<div class="devsite-header-upper-tabs" xpath="/html/body/section/devsite-header/div/div[1]/div/div/div[2]/div[1]">
<devsite-tabs class="upper-tabs devsite-overflow-menu--open" connected="" xpath="/html/body/section/devsite-header/div/div[1]/div/div/div[2]/div[1]/devsite-tabs">
<a aria-label="Extended Navigation" class="devsite-icon devsite-icon-arrow-drop-down" href="#" style="border: 2px solid red;" xpath="/html/body/section/devsite-header/div/div[1]/div/div/div[2]/div[1]/devsite-tabs/nav/tab[2]/a"><!--?lit$8296333005$-->More</a>
<div class="devsite-tabs-overflow-menu" scrollbars="" xpath="/html/body/section/devsite-header/div/div[1]/div/div/div[2]/div[1]/devsite-tabs/nav/tab[2]/div">
<tab xpath="/html/body/section/devsite-header/div/div[1]/div/div/div[2]/div[1]/devsite-tabs/nav/tab[2]/div/tab[1]">
<a class="devsite-tabs-content gc-analytics-event" data-category="Site-Wide Custom Events" data-label="Tab: Gemma" href="https://ai.google.dev/gemma" track-metadata-eventdetail="https://ai.google.dev/gemma" track-metadata-module="primary nav" track-metadata-position="nav - gemma" track-name="gemma" track-type="nav" xpath="/html/body/section/devsite-header/div/div[1]/div/div/div[2]/div[1]/devsite-tabs/nav/tab[2]/div/tab[1]/a">
Authorized Xpaths: "{'/html/body/section/devsite-header/div/div[1]/div/div/div[2]/div[1]/devsite-tabs', '/html/body/section/devsite-header/div/div[1]/div/div/div[2]/div[1]/devsite-tabs/nav/tab[2]/a', '/html/body/section/devsite-header/div/div[1]/div/div/div[2]/div[1]/devsite-tabs/nav/tab[2]/div', '/html/body/section/devsite-header/div/div[1]/div/div/div[2]/div[1]/devsite-tabs/nav/tab[2]/div/tab[1]/a', '/html/body/section/devsite-header/div/div[1]/div/div/div[2]/div[1]', '/html/body/section/devsite-header/div/div[1]/div/div/div[2]/div[1]/devsite-tabs/nav/tab[2]/div/tab[1]', '/html/body/section/devsite-header/div/div[1]/div/div/div[2]'}"
Query: Click on "Gemma" under the "More" dropdown menu.
Completion:
```yaml
# Let's think step by step
# First, we notice that the query asks us to click on the "Gemma" option under the "More" dropdown menu.
# In the provided HTML, we see that the "More" dropdown menu is within a tab element with a specific class and role attribute.
# The "More" dropdown menu can be identified by its class 'devsite-overflow-tab' and contains a link element with the text 'More'.
# We need to interact with this dropdown menu to reveal the hidden options.
# Specifically, for the "More" dropdown menu, there is an anchor element within a tab element:
# /html/body/section/devsite-header/div/div[1]/div/div/div[2]/div[1]/devsite-tabs/nav/tab[2]/a

- actions:
    - action:
        # We can use this XPATH to identify and click on the "More" dropdown menu:
        args:
            xpath: "/html/body/section/devsite-header/div/div[1]/div/div/div[2]/div[1]/devsite-tabs/nav/tab[2]/a"
            value: ""
        name: "click"
    - action:
        # After clicking the "More" dropdown, we need to select the "Gemma" option from the revealed menu.
        # The "Gemma" option is located within the dropdown menu and can be identified by its anchor element with the corresponding text:
        # /html/body/section/devsite-header/div/div[1]/div/div/div[2]/div[1]/devsite-tabs/nav/tab[2]/div/tab[1]/a
        # Thus, we use this XPATH to identify and click on the "Gemma" option:
        args:
            xpath: "/html/body/section/devsite-header/div/div[1]/div/div/div[2]/div[1]/devsite-tabs/nav/tab[2]/div/tab[1]/a"
            value: ""
        name: "click"
```
-----
HTML:
<select name="checkin_eta_hour" xpath="/html/body/div/main/form/section/div/select">
<option disabled="" selected="" value="">Please select</option>
<option value="-1">I don't know</option>
<option value="0">12:00 AM – 1:00 AM </option>
<option value="1">1:00 AM – 2:00 AM </option>
<option value="2">2:00 AM – 3:00 AM </option>
<option value="3">3:00 AM – 4:00 AM </option>
</select>
Authorized Xpaths: "{'/html/body/div/main/form/section/div/select'}"
Query: Select the 2:00 AM - 3:00 AM option from the dropdown menu
Completion:
```yaml
# Let's think step by step
# The query asks us to select the "2:00 AM - 3:00 AM" option from a dropdown menu.
# We need to identify the correct option within the dropdown menu based on its value attribute.
# The dropdown menu is specified by its XPATH, and the value of the option we need to select is "2".
# We can use the following "select" XPATH to locate the dropdown menu and the value "2" to select the appropriate option:
# /html/body/div/main/form/section/div/select

- actions:
    - action:
        # Select the "3:00 AM - 4:00 AM" option by targeting the dropdown menu with the specified XPATH.
        args:
            xpath: "/html/body/div/main/form/section/div/select"
            value: "2"
        name: "dropdownSelect"
```
"""
```` 

# 除了Selenium, 还支持Playwright

TODO: 需要尝试playwright的使用

```
Selenium 和 Playwright 是两种用于自动化浏览器操作的工具，各有优缺点。以下是它们的比较：

### Selenium

#### 优点
- **成熟度**：Selenium 是一个成熟的工具，拥有广泛的社区支持和丰富的文档。
- **多语言支持**：支持多种编程语言，包括 Java、Python、C#、Ruby 等。
- **浏览器兼容性**：支持所有主流浏览器，如 Chrome、Firefox、Safari、Edge 等。
- **广泛的集成**：与 CI/CD 工具和测试框架（如 JUnit、TestNG）集成良好。

#### 缺点
- **速度较慢**：相较于 Playwright，执行速度可能较慢。
- **异步操作支持有限**：处理异步操作时有时需要额外的等待逻辑。

### Playwright

#### 优点
- **速度快**：由于支持更现代的协议，通常比 Selenium 更快。
- **自动等待**：提供自动等待功能，减少了手动设置等待时间的需求。
- **多浏览器支持**：支持 Chromium、Firefox、WebKit，且一套代码可以跨浏览器执行。
- **现代功能**：支持原生的网络拦截和模拟地理位置、权限等功能。
- **异步支持**：原生支持异步操作（尤其在 Node.js 中）。

#### 缺点
- **较新的工具**：相对较新，社区和生态系统不如 Selenium 成熟。
- **语言支持**：主要支持 JavaScript/TypeScript、Python、Java 和 C#，不如 Selenium 多样。

### 结论

选择哪个工具主要取决于具体需求：
- 如果需要成熟的生态系统和广泛的语言支持，Selenium 是不错的选择。
- 如果追求速度、现代功能和自动等待机制，Playwright 可能更适合。

两者都是强大的工具，选择应该基于项目的具体需求和团队的技术栈。
```
