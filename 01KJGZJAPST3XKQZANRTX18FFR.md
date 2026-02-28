---
title: 20260228-材料研发AI
created_at: 2026-02-28T01:59:47.161843+00:00
updated_at: 2026-02-28T11:04:07.528451+00:00
---

# 背景

在春节期间, 做了一个材料研发的AI, 并在春节后与苏州国家实验室交流, 获得反馈

项目地址: <https://github.com/ikarishinjieva/mat_path.ai>

commit: 090132fb3f221958db38306af8ff815c6f47d405

第一轮的输出结果: (待上传)

# 获得反馈

- 对于现在的目标指标( 击穿电压&gt; 65 kV, 闪点&gt; 110°C, 运动粘度 1.4 – 2.6 cSt, 溶胀 ≤ 纯正烷烃基准), 这些指标都是可以进行仿真计算的. 命名为 “筛选级关键指标预测器”
- 对于非常长的论文解读, 会有模型混淆知识的问题: <https://poe.com/s/xOgGWqsr4GCXgK4Rfvum>, 需要 对论文进行逻辑梳理, 以及引入事实性检查
- 非常好的一点, 是在需求分析环节, 识别出了 物理墙 (高闪点 和 低粘度 在分子原理上是冲突的)
- 但其选择初期筛选指标时, 是针对物理墙的冲突进行了选择, 但专家意见是: 需要以BDV击穿电压 作为初期筛选指标, 其在用户需求和物化本质两方面都属于 “可妥协空间很小”, 但闪点和粘度是在用户需求和物化本质两方面属于可调整/可妥协 空间比较大. 需要给模型这样的价值观, 来让模型通过一些材料研发论文, 来自己组织步骤
- AI认知误区: AI提到了关于: 常规变压油的国家标准, 认为国家标准的数值要求比较高, 因此技术难度大.  但实际上国家标准的数值要求比较低, 理论上技术难度应当不大

另外有几个参考工具, 可以作为资料来源: 

- scimaster (字节)

```
https://mail.126.com
邮箱：caizdchem@126.com
密码：CAIzhidao2025

https://scimaster.bohrium.com

先登录邮箱，然后进到scimaster的网站，选择邮箱登录，就可以在邮箱里收到验证码了
```

- google deepresearch
- openai research
- 书生的科学发现平台 MCP对接: <https://discovery.intern-ai.org.cn/home>
- <https://www.bohrium.com/>

# 对scimaster进行测试

买一个个人账号即可

其功能如下: 学术雷达/专题研报/文献检索/全景综述

## 学术雷达

三种模型: 

```
SciMaster A
极速响应
SciMaster A+
循证溯源
SciMaster S
最强大脑
```

只测试 SciMaster S:

url: <https://scimaster.bohrium.com/chat/share/882dfbef8a1240b687b5a9df7805b83c>

提示词: 

```
我要研发一种新材料, 需求如下, 需要你帮我找到调研相关技术: ``` 【主要指标】 工频击穿电压（安全性考量） - 室内场景（低压二次设备为主）：大于 65kV@2.5mm - 室外场景（一次设备为主）：大于 45 kV@2.5mm 抵抗溶胀性能（功能性考量） - 防止橡胶件、绝缘纸溶胀变形 - 通过对比橡胶件、绝缘纸在新材料与 ZDT200 样品/纯正构烷烃中浸泡后的形变、弯曲 程度，来衡量抗溶胀性能 运动黏度（清洁性考量） - 1.4 至 2.2 为最优，2.6 以下可以接受 - 测试标准遵循国标方法 闪点 - 大于 110 摄氏度（国标） 挥发性 - 不做重点考量 【配方辅料适用场景】 抗静电剂 - 在西北等干燥地区，加入微量抗静电剂，清洗后附着在物体表面，增强清洗对象绝缘性 能，减少污垢静电吸附 - 室内机房场景可不加入抗静电剂 抗凝露剂 - 在潮湿地区，加入抗凝露剂，防止用电设备开关机过程中水蒸气凝结、导致安全事故 - 室内空调房场景可不考虑加入抗凝露剂 【研发优先级】 优先面向室内机房场景，解决橡胶件溶胀问题。 ```
```

结果记录: 

```
为满足你的研发指标并确保所有结论均可追溯到公开文献，我将先进行分目标检索：1) 绝缘油/烃类液体的工频（或交流）击穿电压在IEC 60156/ASTM D1816/D877标准下的典型范围；2) 低黏度（1.4–2.2 mm²/s@40°C）且闪点>110°C的候选基液（如合成异构烷烃/Isopar类）的数据；3) 橡胶件/绝缘纸与不同烃类液体的溶胀机制与定量比较（优先寻找Hansen溶解度参数、兼容性图谱和材料指南）；4) 抗静电添加剂在烃类体系中的应用与标准（例如航空燃料用导电性改进剂的可借鉴性）；5) 抗凝露剂在电气设备维护中的思路与可行添加剂类型。如果某些专有名词（如ZDT200）无法检索到权威来源，我会明确标注“Data unavailable”。接下来将并行进行学术与数据表的检索，以尽量减少工具使用次数。 接下来我将并行使用学术与网页检索工具，定位可验证的权威来源，并在必要时解析页面以获取关键参数与试验条件。为补足候选基液（低黏度且高闪点）的数据与中国国家标准（GB/T）条目，并核实非极性烃类液体在2.5 mm电极间的工频击穿电压可达的典型范围，我将小规模补充检索：1) 低黏度PAO（2 cSt、4 cSt）的40°C运动黏度与PMCC/COC闪点；2) 与清洁溶剂相关的高闪点异构烷烃（例如重馏分异构烷烃，是否存在1.8–2.2 cSt且闪点>110°C的牌号）；3) GB/T 265（运动黏度）与GB/T 261（闪点）等国标条目；4) 若能找到PAO或异构烷烃的工频BDV测定数据更佳。接下来将进行针对性网页与论文检索，仅在必要时解析网页。 接下来我会进行几条精确检索以补齐关键标准与候选基液数据。我将把你的需求拆解为四个维度：电气安全（工频击穿电压）；清洁性（运动黏度与闪点、测试标准）；材料兼容性（橡胶件与绝缘纸抗溶胀）；场景化添加剂（抗静电、抗凝露）。为确保可追溯性，我已检索并交叉核对IEC/ASTM/GB标准条目、变压器油工频击穿电压的真实测试数据、候选基液（异构烷烃/合成异构烷烃/GTL 和 PAO）的物性数据、橡胶溶胀的Hansen溶解度参数与实验研究、以及导电性改进剂（Stadis 450）的应用与影响。对于无法在公开文献中查到的“ZDT200”，我标注数据不可得。基于这些来源，我将给出满足指标的工程路线与实验评估方案，并指出风险点与可行替代。
## 研发目标与规范映射（概览）
- 工频击穿电压（室内≥65 kV@2.5 mm；室外≥45 kV@2.5 mm）
  - IEC 60156（蘑菇电极，2.5 mm 间隙）是通行的交流击穿测试方法。公开数据表明，合格的新矿物变压器油在该方法下的击穿电压通常在约49–61 kV范围，取决于水含量、温度与电极配置[^1] [^2] [^3] [^4] [^5]。要稳定达到或超过65 kV，需要极低水分与严格净化工艺。
  - ASTM D1816（1 mm/2 mm 蘑菇电极）与ASTM D877（盘形电极，2.54 mm）也常用于比较与监控；在D1816中，行业可接受的最低值通常为2 mm 间隙≥35 kV，1 mm 间隙≥20 kV，显示方法间阈值差异[^6]。
- 运动黏度（清洁性）：目标1.4–2.2 mm²/s@40°C（最优），≤2.6 mm²/s 可接受；测试遵循GB/T 265（等同于测定液体运动黏度的国标方法）[^7]。
- 闪点（国标）：要求>110°C；测试遵循GB/T 261（Pensky‑Martens闭口杯法）[^8]。
- 抗溶胀：优先解决室内机房橡胶件；对比在新材料与ZDT200/纯正构烷烃中浸泡后形变、弯曲程度。橡胶溶胀可用Hansen溶解度参数（HSP）与实验预测，并与材料配方（如NBR的丙烯腈含量）密切相关[^9] [^10] [^11] [^12]。
- 添加剂：抗静电剂在干燥地区可用，参考航空燃料导电性改进剂（Stadis 450）的成熟应用与剂量范围；其导电性提升与水分分离特性相互影响，需谨慎验证[^13] [^14] [^15] [^16] [^17]。
下面给出满足指标的材料路线、证据与实验方案。
## 一、工频击穿电压的可达性与控制因素
- 依据IEC 60156的2.5 mm蘑菇电极测试，公开样品的击穿电压：
  - 主油箱矿物油：59.4 kV（满足IEC 60422“良好”标准），分接开关油：36 kV（需监控）[^1]。
  - Nynas与Shell Diala‑B在2.5 mm下可达约60.0–61.35 kV；升温、增大间隙提高BDV，水含量升高显著降低BDV[^2]。
  - 另一现场样品：Shell Diala‑B约49.2 kV，超过IEC最低要求（30 kV）[^5]。
- 水分对非极性基液BDV极敏感：在PAO体系中，溶解水由0增至70 ppm时交流BDV可下降至原值的约20%（变差也急剧上升），提示需将水含量维持在低至几十ppm甚至更低以保障电强度[^18]。
- 结论与建议：
  - 要满足室内≥65 kV@2.5 mm，必须以“超干+洁净”的工艺管理为前提（真空脱水与在线滤除微粒），并严格按IEC 60156执行测试。
  - 室外≥45 kV@2.5 mm的阈值在高质量矿物油/合成烃的合格状态下具有可达性，但仍受水分与污染敏感性影响[^1] [^2] [^5]。
  - 由于不同标准与电极几何对数值影响明显，内外场景的验收建议统一采用IEC 60156（2.5 mm蘑菇电极），避免D1816/D877方法间的值差误判[^6]。
## 二、候选基液与指标匹配（黏度与闪点）
目标物性为40°C黏度1.4–2.2 mm²/s与闪点>110°C。检索结果显示：
- 传统异构烷烃（Isopar类）
  - Isopar M：40°C黏度约2.2–3.1 cSt，闪点约93.5°C（PMCC），不满足>110°C闪点要求[^19]。
  - Isopar H/L等轻馏分产品闪点更低（~53–62°C），不满足[^20] [^21]。
- 亲环/异构化去芳烃溶剂（Exxsol D系列）
  - Exxsol D80：40°C黏度1.4–1.8 cSt（满足黏度），但闪点为77–83°C（PMCC），不满足闪点要求[^22] [^23] [^24]。
  - Exxsol D130：闪点129–140°C（PMCC，满足闪点），但40°C黏度为约3.64–6 cSt（超出黏度上限）[^25] [^26] [^27]。
- 合成烃（PAO）
  - PAO 2 cSt：40°C黏度约5.1 cSt，闪点约157°C；PAO 4 cSt：40°C黏度约16.8 cSt，闪点约220–222°C；均不满足黏度目标[^28] [^29] [^30] [^31]。
- 合成GTL异构烷烃（Shell GTL isoparaffins/n‑paraffins）
  - 厂商资料显示“在较低黏度下具有更高闪点”的组合特性，相比同黏度的常规溶剂具有显著优势[^32] [^33]。
  - 具体牌号GS250闪点为110°C（刚好达线），但未在同一资料中查到其40°C黏度，需向供应商索取黏度明细或以近似产品数据做预筛选[^34]。
  - GTL n‑paraffins（C10–C13）的货运资料显示40°C黏度约1–2 mm²/s（满足黏度目标）；但该资料未同时给出闪点，需要与特定GTL等级（例如G80/G85/G100/GS系列）的闪点数据结合选型[^35] [^36] [^37]。
可行筛选策略（基于现有数据）：
- 方案A（优先）：采用GTL异构/正构烷烃体系，锁定满足40°C黏度1.4–2.2 mm²/s的C10–C13范围，同时选取闪点≥110°C的具体等级（如GS系列≥110°C），以实现“两高一低”（高闪点、低黏度、低气味）。现有资料指示其“同黏度下更高闪点”的可达性，但具体配方需供应商数据确认与实测验证[^32] [^34] [^35]。
- 方案B：若GTL单一等级无法同时满足，则考虑在同一类合成烃（GTL isoparaffin/n‑paraffin）内进行牌号复配，以在保持40°C黏度1.6–2.0 mm²/s的同时将PMCC闪点推至≥110–120°C区间。需注意闪点与黏度的非线性混配效应，必须通过GB/T 265与GB/T 261的实测确认[^7] [^8]。
- 方案C（备选）：Exxsol D80（黏度合格）与高闪点组分（如D130）复配以提高闪点。然而两者化学类别一致但波形分布差异较大，闪点提升与黏度叠加可能导致黏度超标，且目前无公开可直接外推的安全混配规则，建议作为次选并以实测为准[^22] [^25]。
风险提示：
- 公开PAO与常规Isopar在黏度目标上存在明显偏离（过高），虽电性能与化学惰性良好，但不适合作为你目标的基液主成分[^28] [^29] [^19]。
- 选型时应优先锁定“GTL异构烷烃/正构烷烃”家族，并向供应商索取完整TDS（含40°C黏度与PMCC闪点）以避免资料不对齐造成的误判[^32] [^34] [^35]。
## 三、抗溶胀：橡胶与绝缘纸的机理与评估
- 橡胶件（优先场景：室内机房）
  - 溶胀预测：Hansen溶解度参数（HSP）方法已被用于EPDM、FKM、HNBR、NBR等的溶胀预测与实验验证。EPDM/FKM的HSP通过与多种溶剂的溶胀相关分析获得；HNBR/EPDM共混的HSP随HNBR质量分数呈线性关系；溶剂的Ra（能量差）与橡胶溶胀度存在可关联性[^9] [^10] [^11]。
  - NBR的耐油性与丙烯腈（ACN）含量正相关：系统性实验显示，NBR在不同油品中的平衡溶胀显著受ACN含量影响，ACN提高通常降低溶胀[^12]。因此在材料相容性上，选择高ACN NBR配方更有利于抗烃类溶胀。
  - 溶剂类别效应：对于某些橡胶（如丁基橡胶），研究指出“溶剂化学类别”比Hansen亲和度更能决定溶胀水平，且在同类溶剂内溶胀与饱和蒸气压/摩尔体积存在关系[^13]。这提醒我们：在同属烃类体系内，异构/正构烷烃的挥发性与分子体积分布亦会影响溶胀与力学变化。
  - 设计要点：为满足“防止橡胶件溶胀变形”的优先任务，应：
    - 采用非芳香、低极性、较窄馏分的GTL异构/正构烷烃，尽量降低对非极性橡胶的溶胀驱动力（通过HSP与Ra筛选）[^32] [^35]。
    - 优先在材料侧选用高ACN NBR或FKM等耐油橡胶，并对常用EPDM配件进行对比试验（EPDM在非极性烃中亦可能有一定溶胀，需以实测为准）[^9] [^12]。
- 绝缘纸（纤维素）
  - 纤维素与矿物油/烃类液体的相容性研究多数聚焦于吸湿、老化与力学/电气性能，而非“体积溶胀”。油浸纸是经典组合，烃类液体对纤维素的溶解度极低，油浸能改善绝缘但水分会显著影响老化与气体析出[^38] [^39] [^40] [^41]。
  - 设计要点：控制纸含水率与油中微水，避免水促进的气体与劣化；新材料（GTL烃类）应验证对纸的浸润与介电损耗，同时测定尺寸与力学变化以判定“溶胀风险”在室内工况下是否可忽略[^38] [^41]。
关于“ZDT200”的对比：
- 未检索到“ZDT200”相关权威公开资料或数据（Data unavailable）。建议你提供该样品的TDS或测试记录，以便与新材料进行同条件对比。
## 四、添加剂策略（按场景）
- 抗静电剂（干燥地区与户外）
  - 航空燃料领域的导电性改进剂Stadis 450为成熟的非金属导电性添加剂，国际上获批用于喷气燃料，提高电导率以降低静电危险[^42] [^43]。常见推荐添加量约0.29–2.9 mg/L（1.1–11.1 mL/1000 gal）需按油品电导率实时校正[^44]。
  - 注意事项：SDA可能影响水分分离性能（MSEP）与过滤充电倾向，且极性污染物（如钠盐）会与SDA发生复杂交互，可能先降后升电导率；高含水时导电性亦不稳定[^15] [^16] [^17]。因此在你的新材料体系中，若用于户外干燥地区，应先做小试评估其对介电、黏度、闪点及清洁性的影响，再决定是否采用。
  - 室内机房：你的需求说明“室内可不加入”，该策略与航空燃料经验一致——在设备接地良好、空气湿度适中与流速有限条件下，可不添加SDA。
- 抗凝露剂（潮湿地区）
  - 公开资料表明，电气柜/机箱的防凝露主要依赖加热、除湿、通风与防雾/抗凝露涂层等工程措施，而非直接在清洗液中添加“抗凝露剂”来改变环境相对湿度或露点[^45] [^46] [^47] [^48] [^49] [^50]。
  - 若确需在部件表面形成防雾/抗凝露涂层，可参考抗雾涂层的材料学综述（亲水/超亲水涂层、表面能调控等），但其与电绝缘、安全性及可重复清洗的兼容性需单独论证[^51]。
  - 结论：你的场景描述符合行业实践——潮湿地区优先考虑工程控制与表面涂层；室内空调环境通常可不考虑专用“抗凝露剂”掺加。
## 五、满足“室内优先、首要解决橡胶溶胀”的研发与测试方案
- 基液选择（建议链路）：
  1) 供应商数据确认：向Shell/Renkert等供应商索取具体GTL isoparaffin/n‑paraffin牌号（如G80/G85/G100/GS系列）之完整TDS，明确40°C黏度与PMCC闪点的组合是否可同时达到1.6–2.0 mm²/s与≥110°C（部分资料显示“同黏度下更高闪点”的优势及GS250的110°C闪点）[^32] [^34] [^35]。
  2) 预筛：基于C10–C13 n‑paraffins的1–2 mm²/s黏度窗口进行黏度优选，并以闪点数据二次筛选；若单一牌号无法同时满足，采用同体系内部复配，目标黏度1.8–2.0 mm²/s、闪点≥115°C（留安全余量）[^35]。
- 电气性能验证：
  - IEC 60156（2.5 mm蘑菇电极，50/60 Hz）进行BDV测试。以新材料在“超干”（水含量https://doi.org/10.23960/jitet.v12i3.4347
[^2]: Studi Tegangan Tembus Minyak Transformator, I Nyoman Oksa Winanta; Anak Agung Ngurah Amrita; Wayan Gede Ariastina, 2019, Jurnal SPEKTRUM, https://doi.org/10.24843/spektrum.2019.v06.i03.p02
[^3]: Analisis Karakteristik Tegangan Tembus Isolasi Minyak Transformator Menggunakan Oli Sepeda Motor Shell Advance 20w-50 Dan Pertamina Mesran 20w-50, Adi Purwanto Umar; Sutiyono Hulopi; Lanto Mohamad Kamil Amali, 2023, Jurnal Informasi, Sains dan Teknologi, https://doi.org/10.55606/isaintek.v6i1.79
[^4]: The Megger guide to insulating oil dielectric breakdown testing (IEC 60156 overview), Megger datasheet (PDF), n.d., https://www.instrumart.com/assets/Megger-Oil-Testing-Datasheet.pdf
[^5]: Breakdown Voltage Characteristic Of Mineral Oil In Power Transformer 16 Mva Updl Semarang, Setyo Adi Nugroho; Setyo Aji Priambodo, 2020, Techno (Jurnal Fakultas Teknik), https://doi.org/10.30595/techno.v21i2.7957
[^6]: Transformer Dielectric Breakdown Voltage Testing – Part 6 (ASTM D1816 typical minima), SDMyers Knowledge Vault, n.d., https://www.sdmyers.com/knowledge-vault/dialectric-breakdown-voltage-testing-part-6/
[^7]: GB/T 265-1988: Petroleum products—Determination of kinematic viscosity and calculation of dynamic viscosity, National Standard (Recommended), 1988, https://www.chinesestandard.net/PDF.aspx/GBT265-1988
[^8]: GB/T 261-2021: Determination of flash point—Pensky-Martens closed cup method, National Standard, 2021, https://www.chinesestandard.net/PDF.aspx/GBT261-2021
[^9]: Elastomer swelling and Hansen solubility parameters, Tenna Brandt Nielsen; Charles M. Hansen, 2005, Polymer Testing 24(8):1057–1061, https://doi.org/10.1016/j.polymertesting.2005.05.007
[^10]: Determination of three‑dimensional solubility parameters of HNBR/EPDM blends and the transport behaviors in ester solvents, Xingwang Jiang et al., 2022, Journal of Applied Polymer Science 139(28):e52881, https://doi.org/10.1002/app.52881
[^11]: A new way to determine the three-dimensional solubility parameters of hydrogenated nitrile rubber and the predictive power, Guangyong Liu et al., 2013, Polymer Testing 32(6):1052–1058, https://doi.org/10.1016/j.polymertesting.2013.07.001
[^12]: Systematic Investigation on the Swelling Response and Oil Resistance of Nitrile Butadiene Rubber (NBR) with Various Acrylonitrile (ACN) Contents, Yang et al., 2024, Polymers 16(9):, PMC Article, https://pmc.ncbi.nlm.nih.gov/articles/PMC11478506/
[^13]: Parameters controlling the swelling of butyl rubber by solvents, Céd rick Nohilé; Patricia I. Dolez; Toan Vu‑Khanh, 2008, Journal of Applied Polymer Science 110(6):3699–3709, https://doi.org/10.1002/app.29004
[^14]: Static Dissipator Additives – Product Bulletin (Stadis 450), JOIFF, 2004, https://www.joiff.com/members/sharedlearning/topics/archive2004/documents/stadis_faq.pdf
[^15]: The effects of contaminants on the behaviour of conductivity improvers in hydrocarbons, Brian Dacre; Janice I. Hetherington, 1998, Journal of Electrostatics 44(3):195–208, https://doi.org/10.1016/s0304-3886(98)00036-9
[^16]: Conductivity and Charging Tendency of JP-8 + 100 Jet Fuel, J. T. Leonard; D. R. Hardy, 2000, Report (DTIC), https://doi.org/10.21236/ada382878
[^17]: Electrical conductivity of aviation turbine fuel (jet fuel) and its affecting factors: a case study, Kunalkanti Das et al., 2022, Petroleum Science and Technology 40(18):2167–2175, https://doi.org/10.1080/10916466.2022.2067177
[^18]: The effect of water content on the dielectric strength of polyalphaolefin (PAO) coolants, S. Gunderson et al., 2011, IEEE Transactions on Dielectrics and Electrical Insulation 18(1):257–264, https://doi.org/10.1109/tdei.2011.5704521
[^19]: Isopar M – Sales Specification/TDS, ExxonMobil Chemical, n.d., https://www.exxonmobilchemical.com/en/chemicals/webapi/dps/v1/datasheets/130000000289/1/en
[^20]: Isopar H – Univar Solutions SDS/TDS, Univar, 2025, https://www.univarsolutions.com/.../ISOPAR H
[^21]: ExxonMobil Isopar L – Datasheet, ExxonMobil Chemical, n.d., https://www.exxonmobilchemical.com/en/chemicals/webapi/dps/v1/datasheets/130000000367/1/en
[^22]: Exxsol D80 – Sales Specification/TDS, ExxonMobil Chemical, n.d., https://www.exxonmobilchemical.com/.../exxsol-d80-sales-specification.pdf
[^23]: Exxsol D80 – SDS, Univar Solutions, 2025, https://www.univarsolutions.com/.../EXXSOL D80 SDS
[^24]: Exxsol D80 – TDS (lookpolymers summary), LookPolymers, n.d., https://www.lookpolymers.com/polymer_ExxonMobil-Exxsol-D80-Dearomatized-Fluid.php
[^25]: EXXSOL D130 – Univar SDS（含40°C黏度范围与闪点范围）, Univar Solutions, 2023, https://www.univarsolutions.com/.../EXXSOL D130 SDS
[^26]: Exxsol D130 – Sales Specification, ExxonMobil Chemical, n.d., https://www.exxonmobilchemical.com/.../Exxsol D130
[^27]: Fluids at a glance（ExxonMobil烃类流体性能汇总）, ExxonMobil Chemical, n.d., https://www.exxonmobilchemical.com/.../fluids_at_a_glance_en.pdf
[^28]: Synfluid PAO 2 cSt – TDS, Chevron Phillips Chemical, 2020, https://www.cpchem.com/.../Synfluid PAO 2 cSt
[^29]: SpectraSyn 2 – TDS（COC闪点157°C）, ExxonMobil Chemical, n.d., https://www.exxonmobilchemical.com/.../SpectraSyn 2
[^30]: Synfluid PAO 4 cSt – TDS（40°C黏度16.8 cSt，COC闪点222°C）, Chevron Phillips Chemical, n.d., https://www.cpchem.com/.../Synfluid PAO 4 cSt.pdf
[^31]: SpectraSyn 4 – TDS（COC闪点220°C），ExxonMobil Chemical, n.d., https://www.exxonmobilchemical.com/.../SpectraSyn 4
[^32]: SHELL GTL ISOPARAFFINS（性能手册，强调“在相似黏度下更高闪点”），Renkert Oil，2025，https://renkertoil.com/.../Shell-GTL-0525.pdf
[^33]: Shell GTL Performance Fluids Brochure（G80/G85/G100系列，说明高闪点与低黏度特性），Shell，2024，https://www.shell.com/.../shell-gtl-performance-fluids-brochure-2024-digital.pdf
[^34]: GTL Solvent GS250 – TDS（PMCC闪点110°C），Equilex BV，2021，https://www.equilex.com/.../T-GS250-GRADE-AN-v1.pdf
[^35]: Marine Cargo Handling Sheet Shell GTL n‑Paraffins (C10‑C13)（40°C黏度1–2 mm²/s），Shell，n.d.，https://images10.newegg.com/.../shell_gtl_n_paraffins_c10_c13_revision11445515792865.pdf
[^36]: Shell GTL Fluid G100 – 性能资料（低黏度与高闪点的组合特性），Shell，n.d.，https://www.shell.com/.../shell-gtl-fluid-g100.pdf
[^37]: Shell GTL Fluids and Solvents – Brochure（低气味、低芳烃强调），Shell，n.d.，https://www.shell.com/.../gtl-fluids-and-solvents-urban-brochure-desktop-print.pdf
[^38]: Transformer Insulation Materials and Ageing（Chapter excerpt）, Wiley, n.d., https://onlinelibrary.wiley.com/doi/pdf/10.1002/9781119239970.ch1
[^39]: Degradation Mechanisms of Cellulose-Based Transformer Insulation, MDPI Insulation, 2025, 5(2):20, https://www.mdpi.com/2673-6209/5/2/20
[^40]: Thermal degradation assessment of Kraft paper in power transformers, IEEE Transactions/Elsevier abstract page, 2016, https://www.sciencedirect.com/science/article/abs/pii/S135943111630672X
[^41]: Performance Assessment of Cellulose Paper Impregnated in Nanofluid for Power Transformer Insulation Application, MDPI Energies 16(4):2002, 2023, https://www.mdpi.com/1996-1073/16/4/2002
[^42]: Stadis 450 – Product Data Sheet（导电性改进剂），Biobor Fuel Additives, 2025, https://www.biobor.com/wp-content/uploads/2025/07/Stadis450PDS.pdf
[^43]: Static Dissipators | INNOSPEC（Stadis 450全球批注用途），Innospec，n.d.，https://innospec.com/fuel-additives/refinery-pipeline/static-dissipators/
[^44]: Additional Products（Stadis 450 推荐处理量），Biobor Fuel Additives，n.d.，https://www.biobor.com/products/additional-products/
[^45]: Preventing Condensation in Electrical Enclosures, Pfannenberg USA, 2020, https://www.pfannenbergusa.com/preventing-condensation-in-electrical-enclosures/
[^46]: How to Prevent Condensation in Electrical Enclosures, AttaBox, n.d., https://attabox.com/blogs/how-prevent-condensation-electrical-enclosures
[^47]: How to prevent condensation in enclosures, Essentra Components, 2021, https://www.essentracomponents.com/.../how-to-prevent-condensation-in-enclosures
[^48]: Anti Condensation Heaters – Protect cabinets and electrical panels, anticondensationheaters.com, n.d., https://www.anticondensationheaters.com/
[^49]: Humidity Control Sheet（防凝露片材），NTT-AT，n.d.，http://www.ntt-at.com/product/docs/en-humidity_control_sheet.pdf
[^50]: Industrial Coatings – Anti-Fog, Anti-Frost & Condensation-Control, Hydromer, n.d., https://hydromer.com/industrial-coatings/
[^51]: Fundamentals of antifogging strategies, coating techniques and materials, Journal of Adhesion Science and Technology (Elsevier review), 2023, https://www.sciencedirect.com/science/article/pii/S2238785423000157
```

其中有探索步骤, 使用了两个agent, 做学术搜索和web搜索: 

```
science_navigator

{
  "query": "IEC 60156 breakdown voltage transformer oil typical values 2.5 mm gap mushroom electrodes AC breakdown",
  "num": 5,
  "page_size": 30
}

web_search

{
  "query": "IEC 60156 breakdown voltage 2.5 mm transformer oil typical kV site:iec.ch OR site:prominent transformer oil datasheet"
}

web_search

{
  "query": "ASTM D1816 breakdown voltage insulating oil 2 mm 1 mm typical values transformer oil"
}

web_search

{
  "query": "ASTM D877 dielectric breakdown voltage test 2.5 mm electrodes insulating oil typical values"
}

web_search

{
  "query": "Isopar M datasheet kinematic viscosity 40C flash point breakdown voltage electrical insulating properties"
}

web_search

{
  "query": "isoparaffinic solvent data sheet viscosity flash point isopar L M H ExxonMobil datasheet"
}

science_navigator

{
  "query": "Dielectric breakdown strength of n-hexane or decane between electrodes AC or impulse",
  "num": 5,
  "page_size": 30
}

web_search

{
  "query": "rubber swelling isoparaffins EPDM NBR compatibility Hansen solubility parameter hydrocarbons"
}

science_navigator

{
  "query": "Hansen solubility parameters rubber swelling hydrocarbons EPDM NBR cellulose petroleum hydrocarbons",
  "num": 5,
  "page_size": 30
}

web_search

{
  "query": "cellulose insulating paper swelling in hydrocarbon oils transformer oil cellulose compatibility"
}

web_search

{
  "query": "ZDT200 溶剂 橡胶 溶胀 清洗 液"
}

web_search

{
  "query": "Stadis 450 conductivity improver additive hydrocarbon solvent antistatic datasheet"
}

science_navigator

{
  "query": "conductivity improver additive for hydrocarbon fuels static dissipation Stadis 450 electrical conductivity jet fuel",
  "num": 5,
  "page_size": 30
}

web_search

{
  "query": "anti-condensation additive electrical cabinet anti-fogging agent moisture condensation prevention industry"
}
```

## 专题研报

需要10-30分钟, 结果待上传

(面向电气设备高绝缘与防溶胀性能的清洗剂材料体系研究：技术分析、配方设计与应用策略.pdf)

## 文献检索

检索到三篇: 

```
[
  {
    "title": "Synthesis, Characterization, and Evaluation of Castor Oil-Based Acylated Derivatives as Potential Lubricant Base Stocks",
    "arxiv_id": "",
    "url": "https://doi.org/10.1021/acs.iecr.6b01550",
    "bohrium_url": "https://www.bohrium.com/paper-details/synthesis-characterization-and-evaluation-of-castor-oil-based-acylated-derivatives-as-potential-lubricant-base-stocks/811164076452347904-4160",
    "reason": "Conclusion: - 该论文聚焦于生物基润滑油基础油的合成与性能评估，核心目标为工业润滑应用，而非电气绝缘介质。\n- 论文未提供任何关于介电强度（击穿电压）的数据或测试结果，无法验证其是否满足65kV/2.5mm（室内）或45kV/2.5mm（室外）的要求。\n\nConstraint checks: \nC1:材料必须具有功率频率击穿电压 >65kV/2.5mm（室内）和/或 >45kV/2.5mm（室外），Score:1,证据:\"论文未提及任何介电性能测试或击穿电压数据\" vs \"要求击穿电压指标\",给出理由\n论文研究对象为润滑油，非绝缘液体，无相关电学性能评估，完全不满足。\nC2:材料必须抵抗膨胀：应防止橡胶部件和绝缘纸在浸入后发生形变，需与ZDT200或纯正构烷烃对比，Score:2,证据:\"论文提到‘emulsion stability’ and ‘load carrying capacity’ but no swelling test or comparison with ZDT200/paraffin\" vs \"抗膨胀性能需通过橡胶/纸张形变与参考物比较\",给出理由\n虽涉及材料稳定性，但未进行橡胶或绝缘纸的浸渍形变实验，仅提“emulsion stability”属乳化性能，非抗膨胀性，属于同领域但不匹配。\nC3:材料应具有1.4–2.2 cSt最优、低于2.6 cSt可接受的运动粘度，Score:4,证据:\"论文报告‘kinematic viscosity’ in range of 1.4–2.2 cSt (e.g., hexanoylated ester: 1.8 cSt)\" vs \"1.4–2.2 cSt最优，<2.6可接受\",给出理由\n明确给出具体数值且符合范围，有直接证据支持。\nC4:材料闪点需>110°C，Score:5,证据:\"flash point (174–280 °C)\" vs \">110°C\",给出理由\n所有样品均远超110°C，有明确数据支持。\nC5:材料可含添加剂策略：干燥区加抗静电剂，潮湿区加防凝剂，Score:1,证据:\"论文未提及任何添加剂如抗静电剂或防凝剂\" vs \"添加策略针对环境条件设计\",给出理由\n全文未讨论添加剂功能，也未涉及使用场景差异，完全无关。\n\nIrrelevance/risk points/gaps: \n- 论文研究主题为生物基润滑油，非电气绝缘液，核心应用场景不符。\n- 缺失关键约束：击穿电压、抗膨胀性能测试、添加剂策略，均为用户核心需求，但论文未覆盖。",
    "abstract": "Synthesis, characterization, and evaluation of a series of novel acylated derivatives of castor oil as biolubricant base stocks are described. The acylated derivatives of castor oil, castor oil fatty acid methyl and 2-ethylhexyl esters were synthesized using different anhydrides (C1–C6) in about 90–95% yield. All the products were structurally characterized using NMR and IR spectral data. The acylated products were evaluated for their physicochemical and lubricant properties. Although these products belong to group V, based on viscosity index (130–156), acylated derivatives of castor fatty acid alkyl esters belong to group III, the category of base fluids as per API classification. The acylated products exhibited excellent pour point (−21 to −39 °C) and flash point (174–280 °C). The hexanoylated and butanoylated esters of castor oil exhibited excellent flash points of 280 and 272 °C, respectively. The air release value was found to be excellent in the range of 0.38–0.99 min, and NOACK volatilities in the range of 3.25–3.92%. The other lubrication properties such as load carrying capacity, and emulsion stability were found to be good. Therefore, these derivatives will have utility in hydraulic and metal working fluids and other industrial fluids with their wide range of properties.",
    "score": 52.95718211012363,
    "authors": "Gorla Geethanjali, Korlipara V. Padmaja, Rachapudi B. N. Prasad",
    "journal": "Industrial & Engineering Chemistry Research",
    "year": "2016-08-22"
  },
  {
    "title": " Compatibility Testing of New Insulating Fluids and Materials in Distribution Transformer ",
    "arxiv_id": "",
    "url": "https://doi.org/10.3390/en18071831",
    "bohrium_url": "https://www.bohrium.com/paper-details/compatibility-testing-of-new-insulating-fluids-and-materials-in-distribution-transformer/1183522624915898370-0",
    "reason": "Conclusion: - 该论文聚焦于新型绝缘液体与变压器内部材料的相容性测试，未直接评估目标材料在特定电场强度下的击穿电压性能。\n- 论文虽提及部分电气性能（如介电常数、局部放电起始电压），但未提供2.5mm间隙下功率频率击穿电压数据，无法验证是否满足65kV或45kV标准。\n\nConstraint checks: \nC1:材料必须具有功率频率击穿电压 >65kV/2.5mm（室内）和/或 >45kV/2.5mm（室外）,Score:1,证据:\"论文中提到局部放电起始电压及预击穿活动差异\" vs \"需明确2.5mm间隙下功率频率击穿电压≥65kV（室内）或≥45kV（室外）\",给出理由:论文未报告任何击穿电压数据，仅讨论局部放电行为，无法支持该关键安全指标；\nC2:材料必须抵抗膨胀：防止橡胶件和绝缘纸变形，通过与ZDT200/纯石蜡对比浸渍后的形变和弯曲度,Score:3,证据:\"论文指出天然酯油对密封圈硬度影响较小（Shore 21–22.5 vs 矿物油35.25），且电缆扎带在天然酯中拉伸强度上升\" vs \"需比较橡胶与绝缘纸在新液与ZDT200/正构烷烃中的形变与弯曲度\",给出理由:虽涉及材料老化与硬度变化，但未使用ZDT200或纯正构烷烃作为对照，也未测量橡胶/纸张的形变与弯曲度，仅以硬度和强度间接推断，存在推理偏差；\nC3:材料应具有1.4–2.2 cSt最优，低于2.6 cSt可接受，测试符合国家标准,Score:4,证据:\"论文报告了油品粘度变化，并在120°C和140°C下进行老化测试\" vs \"需明确原始粘度范围在1.4–2.2 cSt，且测试方法符合国标\",给出理由:虽提及粘度测试，但未说明原始粘度值是否落在目标区间，也未明确引用国家标准方法；\nC4:材料闪点需大于110°C,Score:4,证据:\"论文提到高温老化实验（120°C、140°C）并关注热稳定性\" vs \"需明确闪点>110°C\",给出理由:虽未直接报告闪点数值，但高温老化实验暗示其具备较高热稳定性，可合理推断闪点达标；\nC5:材料可含添加剂策略：干区加抗静电剂，湿区加防凝剂,Score:2,证据:\"论文未提及任何添加剂应用，亦无关于抗静电或防凝剂的讨论\" vs \"需考虑添加剂在不同环境下的适用性\",给出理由:完全缺失添加剂相关研究，仅关注基础油与材料相容性，不涉及功能化添加策略。\n\nIrrelevance/risk points/gaps: \n- 论文未提供击穿电压数据，无法验证核心安全指标C1；\n- 未使用ZDT200或纯正构烷烃作为对照，无法评估抗膨胀性能；\n- 缺乏对添加剂（抗静电/防凝）的应用研究，偏离R&D优先方向。",
    "abstract": "The introduction of a new insulating oil or, for instance, a new type of insulation or sealing into a transformer necessitates tests for material compatibility. Compatibility tests of liquids with the structural internal materials of transformers are conducted to prevent undesired interactions between insulating fluids and the formation of products that could lead to the generation of undesirable ions, sediments, or chemical compounds that result in a reduction in the dielectric property performance of the fluid. This includes chemical reactions (hydrolysis, hydrogenation, oxidation, formation of sulfates or sulfides, etc.) and degradation, the formation of conductive suspensions, the generation of undesirable condensation, and alterations in other fluid properties, such as interfacial tension between oil and water, viscosity, flashpoint, etc. Changes must also not occur in the strength and hardness of gasket material, which could result in undesirable fluid leakage. This paper describes the novel methodology and results of several proposed tests, including the impact on oil viscosity, material hardness, FT-IR analysis of oils, partial discharges in different oils, dielectric properties, and more, conducted during compatibility and aging tests at 120 °C and 140 °C performed on materials used in particular distribution transformers being prepared for natural ester use. The results show notable differences in the behavior of insulating fluids and aged submerged materials. While mineral oils exhibit lower dissipation factors compared to natural esters, the latter demonstrate slower and less severe hardening effects on gaskets during high-temperature aging (e.g., Shore 35.25 in mineral oil vs. 21–22.5 in natural esters). The tensile strength of the tested cable ties decreased significantly (from 260 to approx. 60 N) in mineral oil but increased in natural ester (320 N/120 °C exposition). This study also highlights a novel insight into partial discharge mechanisms, where differences in viscosity, conductivity phenomena, and dielectric constants result in presented differences in inception voltages and prebreakdown activity.",
    "score": 52.391437735369344,
    "authors": "Pavel Trnka, Jaroslav Hornak, Magdalena Trnkova, Ondrej Michal, Zdislava Mokra, Pavel Slama, Jiri Kopriva, Pavel Prosr, Jan Leffler, Zdenek Frana",
    "journal": "Energies",
    "year": "2025-04-04"
  },
  {
    "title": "Fluids and Elastomers for Low-Temperature Heat Transfer and Hydraulic Systems",
    "arxiv_id": "",
    "url": "https://doi.org/10.1080/05698196608972122",
    "bohrium_url": "https://www.bohrium.com/paper-details/fluids-and-elastomers-for-low-temperature-heat-transfer-and-hydraulic-systems/811981208043913216-26081",
    "reason": "Conclusion: - 该论文聚焦于低温液压系统中流体与弹性体的性能评估，未涉及电绝缘性能（如介电强度）或材料在电气设备中的应用。\n- 论文未提供任何关于功率频率击穿电压的数据或测试结果，无法满足用户对室内/室外设备绝缘安全性的核心要求。\n\nConstraint checks: \nC1:材料必须具有介电性能：室内场景（低电压二次设备）>65kV@2.5mm，户外场景（主设备）>45kV@2.5mm,Score:1,证据:\"论文未提及任何介电强度或击穿电压测试\" vs \"功率频率击穿电压 >65kV/2.5mm\",理由：完全缺失相关电学性能数据，且研究对象为液压流体而非绝缘材料；\nC2:材料必须抗膨胀：防止橡胶件和绝缘纸变形，以ZDT200或正烷烃为参照进行对比测试,Score:3,证据:\"论文测试了氟化聚醚与苯基硅胶的相容性及静态密封泄漏情况\" vs \"抗膨胀性能需通过橡胶与绝缘纸在新物质与ZDT200/正烷烃中浸泡后的形变与弯曲度比较\",理由：虽涉及橡胶与流体相容性，但未使用ZDT200或正烷烃作为对照，也未测量形变或弯曲度，仅报告泄漏，属部分相关但证据不足；\nC3:材料应具有1.4–2.2 cSt最优、低于2.6 cSt可接受的运动粘度,Score:4,证据:\"论文列出五种流体的粘度值并用于系统设计\" vs \"1.4–2.2 cSt最优，低于2.6 cSt可接受\",理由：明确提供了粘度数据，且其系统设计依赖粘度参数，符合标准范围推断；\nC4:材料闪点需>110°C,Score:4,证据:\"论文提到各流体的闪点，并指出氟化聚醚为最佳选择\" vs \"闪点>110°C\",理由：虽未直接给出数值，但“氟化聚醚”通常具备高闪点，结合其被选为最适流体，可合理推断其闪点高于110°C；\nC5:材料可添加抗静电剂（干燥区）或防凝剂（潮湿区），Score:2,证据:\"论文未讨论添加剂功能或环境适应性策略\" vs \"添加抗静电剂/防凝剂以应对特定环境\",理由：未提及任何添加剂应用，也无针对干/湿环境的配方调整，仅关注基础流体与弹性体匹配。\n\nIrrelevance/risk points/gaps: \n- 研究主题为低温液压系统用流体与密封材料，非电气绝缘材料，与电力设备绝缘安全无关；\n- 缺失关键电学性能指标（击穿电压）、未按用户指定参考物（ZDT200/正烷烃）进行抗膨胀测试，且未涉及添加剂在特定环境下的应用逻辑。",
    "abstract": "Five fluids (a dimethyl silicone, a silicate ester, a branched chain chloro-fluorocarbon, a cyclic fluorinated ether and a fluorinated polyether) were evaluated for use as the fluid in a single pass, gas actuated hydraulic system capable of operation from −150 to 275 F. Important physical and chemical properties, including thermal stability, lubricity, viscosity, pour, low-temperature characteristics and flash point, were determined. Specific heat, thermal conductivity and density values were compiled from manufacturer&#x27;s data. The fluorinated polyether was chosen as the most suitable fluid for the low-temperature system. The phenyl silicones were determined to be the only elastomers suitable for use in dynamic and static seals of the system. Several typical phenyl silicone compounds were tested for compatibility with the five low-temperature fluids. Leakage tests were run in a static seal fixture with several of the fluids. A proprietory phenyl silicone compound was chosen as the best material for use with the fluorinated polyether. Fluid and elastomer performance in the low-temperature system are covered briefly. Presented as an American Society of Lubrication Engineers paper at the Lubrication Conference held in San Francisco, Calif., October 18–20, 1965. Presented as an American Society of Lubrication Engineers paper at the Lubrication Conference held in San Francisco, Calif., October 18–20, 1965.",
    "score": 52.39078300200272,
    "authors": "C. S. Armstrong",
    "journal": "A S L E Transactions",
    "year": "1966-01-01"
  }
]
```

## 全景综述

需加入其 共创计划 才能获取: <https://dptechnology.feishu.cn/wiki/PdCywnCLtiSzppkkHUSc62Ohnqb>. 提示词记录: 

```
我希望生成一篇关于 电气设备绝缘清洗新材料研发 的综述，要求涵盖 低粘度高绝缘性溶剂配方、抗溶胀机理及材料兼容性、环境适应性添加剂（抗静电/抗凝露） 等具体研究方向，重点讨论 如何在保证工频击穿电压（室内>65kV@2.5mm）和高闪点（>110℃）的同时，将运动黏度控制在1.4-2.2之间并显著降低对橡胶件及绝缘纸的溶胀作用 这一主要问题或挑战，目标是 总结现有绝缘清洗剂的缺陷，提出针对室内机房场景的低溶胀、高安全性的新型配方设计思路。
```

未测试