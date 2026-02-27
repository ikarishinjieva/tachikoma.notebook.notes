---
title: 20230721 - p-tuning进行微调
confluence_page_id: 2588919
created_at: 2023-07-23T01:16:07+00:00
updated_at: 2023-07-23T03:00:20+00:00
---

尝试使用p-tuning进行微调

使用与lora相同的参数 (梯度和epoch), 效果很差: 

```
Input: 判断SQL是否符合 我们定义的SQL规则: SQL: <select * from b limit 9>, 规则: <select 语句<不能使用>order by>
ChatGLM-6B: 我们需要判断SQL是否符合我们定义的SQL规则，其中SQL规则为<select * from b limit 9>，规则为<select 语句<不能使用>order by>。

根据题目中的规则，我们可以得出答案为B。

解题步骤：
1. 根据题目中的规则，我们可以判断SQL是否符合<select * from b limit 9>和<select 语句<不能使用>order by>的规则。

2. 根据题目中的规则，我们可以得出答案为B。

3. 因此，根据题目中的规则，我们可以得出答案为B。

Input: 判断SQL是否符合 我们定义的SQL规则: SQL: <select * from a group by b having 1=1>, 规则: <select 语句<不能使用>Having子句>
ChatGLM-6B: 根据给出的SQL语句，我们可以发现其中存在语法错误，因此答案为错。

SQL语句中，如果存在语法错误，则需要进行修正，而如果存在语法正确，则无需修改。
``` 

loss很高: 

第一轮

```
{'loss': 2.3267, 'learning_rate': 9.842699304829007e-07, 'epoch': 7.29}
{'loss': 2.2346, 'learning_rate': 9.134078046140248e-07, 'epoch': 7.31}
{'loss': 1.5957, 'learning_rate': 8.451452853123265e-07, 'epoch': 7.34}
{'loss': 2.3486, 'learning_rate': 7.7948973998338e-07, 'epoch': 7.37}
{'loss': 2.3677, 'learning_rate': 7.164482546684642e-07, 'epoch': 7.39}
{'loss': 1.6863, 'learning_rate': 6.560276332798155e-07, 'epoch': 7.42}
{'loss': 2.2364, 'learning_rate': 5.98234396866279e-07, 'epoch': 7.45}
{'loss': 2.1145, 'learning_rate': 5.430747829095001e-07, 'epoch': 7.47}
{'loss': 1.5021, 'learning_rate': 4.905547446507536e-07, 'epoch': 7.5}
{'loss': 2.2672, 'learning_rate': 4.406799504484055e-07, 'epoch': 7.52}
{'loss': 2.0133, 'learning_rate': 3.9345578316614396e-07, 'epoch': 7.55}
{'loss': 1.7146, 'learning_rate': 3.488873395920217e-07, 'epoch': 7.58}
{'loss': 2.2374, 'learning_rate': 3.0697942988838214e-07, 'epoch': 7.6}
{'loss': 2.7424, 'learning_rate': 2.677365770726886e-07, 'epoch': 7.63}
{'loss': 2.3502, 'learning_rate': 2.3116301652938987e-07, 'epoch': 7.66}
{'loss': 2.2722, 'learning_rate': 1.9726269555278566e-07, 'epoch': 7.68}
{'loss': 2.5192, 'learning_rate': 1.6603927292102305e-07, 'epoch': 7.71}
{'loss': 2.0589, 'learning_rate': 1.374961185011958e-07, 'epoch': 7.74}
{'loss': 2.2854, 'learning_rate': 1.116363128856518e-07, 'epoch': 7.76}
{'loss': 2.3213, 'learning_rate': 8.846264705952289e-08, 'epoch': 7.79}
{'loss': 1.7008, 'learning_rate': 6.797762209947434e-08, 'epoch': 7.82}
{'loss': 2.3382, 'learning_rate': 5.018344890379833e-08, 'epoch': 7.84}
{'loss': 2.2155, 'learning_rate': 3.508204795377445e-08, 'epoch': 7.87}
{'loss': 2.1176, 'learning_rate': 2.267504910641316e-08, 'epoch': 7.89}
{'loss': 2.1744, 'learning_rate': 1.2963791418538207e-08, 'epoch': 7.92}
{'loss': 1.9414, 'learning_rate': 5.94932300227169e-09, 'epoch': 7.95}
{'loss': 1.9271, 'learning_rate': 1.6324009119161877e-09, 'epoch': 7.97}
``` 

di'er'lu

```
learning_rate=5e-05,
{'loss': 2.1485, 'learning_rate': 4.995438885558294e-05, 'epoch': 0.06}
{'loss': 2.2529, 'learning_rate': 4.97969343613113e-05, 'epoch': 0.13}
{'loss': 2.0775, 'learning_rate': 4.955969343539162e-05, 'epoch': 0.19}
{'loss': 2.0525, 'learning_rate': 4.9191036297534454e-05, 'epoch': 0.26}
{'loss': 2.0858, 'learning_rate': 4.8713411048678635e-05, 'epoch': 0.32}
{'loss': 2.223, 'learning_rate': 4.812896914362309e-05, 'epoch': 0.38}
{'loss': 2.0647, 'learning_rate': 4.744034319097535e-05, 'epoch': 0.45}
{'loss': 1.9029, 'learning_rate': 4.665063509461097e-05, 'epoch': 0.51}
{'loss': 2.1314, 'learning_rate': 4.5763402081200294e-05, 'epoch': 0.58}
{'loss': 1.9723, 'learning_rate': 4.478264067674155e-05, 'epoch': 0.64}
{'loss': 1.8607, 'learning_rate': 4.371276870427753e-05, 'epoch': 0.7}
{'loss': 1.9257, 'learning_rate': 4.255860538388694e-05, 'epoch': 0.77}
{'loss': 1.8939, 'learning_rate': 4.132534962458962e-05, 'epoch': 0.83}
{'loss': 2.0467, 'learning_rate': 4.001855660594948e-05, 'epoch': 0.89}
{'loss': 1.8578, 'learning_rate': 3.8644112754862614e-05, 'epoch': 0.96}
{'loss': 1.874, 'learning_rate': 3.720820923024778e-05, 'epoch': 1.02}
{'loss': 1.838, 'learning_rate': 3.5717314035076355e-05, 'epoch': 1.09}
{'loss': 1.862, 'learning_rate': 3.417814288136319e-05, 'epoch': 1.15}
{'loss': 1.8746, 'learning_rate': 3.2597628939356175e-05, 'epoch': 1.21}
{'loss': 1.8579, 'learning_rate': 3.098289160718895e-05, 'epoch': 1.28}
{'loss': 1.7338, 'learning_rate': 2.9341204441673266e-05, 'epoch': 1.34}
{'loss': 1.8855, 'learning_rate': 2.7679962394686198e-05, 'epoch': 1.41}
{'loss': 1.8268, 'learning_rate': 2.600664850273538e-05, 'epoch': 1.47}
{'loss': 1.842, 'learning_rate': 2.4328800179748475e-05, 'epoch': 1.53}
{'loss': 1.839, 'learning_rate': 2.265397526492052e-05, 'epoch': 1.6}
{'loss': 1.7056, 'learning_rate': 2.098971797855599e-05, 'epoch': 1.66}
{'loss': 1.7817, 'learning_rate': 1.934352493925695e-05, 'epoch': 1.73}
{'loss': 1.7441, 'learning_rate': 1.7722811395532178e-05, 'epoch': 1.79}
{'loss': 1.6961, 'learning_rate': 1.613487782393661e-05, 'epoch': 1.85}
{'loss': 1.6854, 'learning_rate': 1.4586877044199016e-05, 'epoch': 1.92}
{'loss': 1.7162, 'learning_rate': 1.3085781999467303e-05, 'epoch': 1.98}
{'loss': 1.7535, 'learning_rate': 1.1638354346804971e-05, 'epoch': 2.04}
{'loss': 1.7786, 'learning_rate': 1.0251113999421935e-05, 'epoch': 2.11}
{'loss': 1.8015, 'learning_rate': 8.930309757836517e-06, 'epoch': 2.17}
{'loss': 1.7663, 'learning_rate': 7.681891162260015e-06, 'epoch': 2.24}
{'loss': 1.7478, 'learning_rate': 6.511481692994076e-06, 'epoch': 2.3}
{'loss': 1.6711, 'learning_rate': 5.424353439559446e-06, 'epoch': 2.36}
{'loss': 1.669, 'learning_rate': 4.425403352658591e-06, 'epoch': 2.43}
{'loss': 1.7691, 'learning_rate': 3.5191311859445796e-06, 'epoch': 2.49}
{'loss': 1.6385, 'learning_rate': 2.70961922695743e-06, 'epoch': 2.56}
{'loss': 1.8101, 'learning_rate': 2.0005139085293945e-06, 'epoch': 2.62}
{'loss': 1.7738, 'learning_rate': 1.3950093834903866e-06, 'epoch': 2.68}
{'loss': 1.6722, 'learning_rate': 8.958331366609423e-07, 'epoch': 2.75}
{'loss': 1.7718, 'learning_rate': 5.052336989433082e-07, 'epoch': 2.81}
{'loss': 1.6748, 'learning_rate': 2.2497051885228827e-07, 'epoch': 2.88}
{'loss': 1.7361, 'learning_rate': 5.6306037109371544e-08, 'epoch': 2.94}
``` 

learning_rate 降得很快, 但loss徘徊

将初始LR调整到 2e-2, loss效果很好

lora的训练结果 ([20230723 - chatglm 微调, 对sql进行规则审核 [3]]) 会比p-tuning ([20230717 - chatglm 微调, 对sql进行规则审核 [2]]) 好一点
