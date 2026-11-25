# 2025–2026 UAV 视觉定位、文本—视觉对齐与 VLN 论文调研

本仓库收录一份围绕 **自然语言指令如何被解析、如何与图像区域/物体关系对齐，以及如何用于 UAV 视觉定位与导航** 的近期论文调研。

- 调研截止日期：2026-11-25
- 时间范围：2025–2026
- 论文数量：10 篇
- 主报告：[打开完整中文调研报告](./2025-2026_UAV-Visual-Grounding-VLN-Survey.md)

## 本次报告覆盖的十篇论文

| # | 论文 | 发表状态 | 主要研究问题 |
|---|---|---|---|
| 1 | [Hierarchical Textual-Visual Guidance for Referring Remote Sensing Segmentation](https://doi.org/10.1016/j.patcog.2026.113579) | Pattern Recognition 2026 | 层次化文本语义解析与遥感指代分割 |
| 2 | [NAVER: A Neuro-Symbolic Compositional Automaton for Visual Grounding with Explicit Logic Reasoning](https://openaccess.thecvf.com/content/ICCV2025/html/Cai_Naver_A_Neuro-Symbolic_Compositional_Automaton_for_Visual_Grounding_with_Explicit_ICCV_2025_paper.html) | ICCV 2025 | 自然语言到概率逻辑程序、视觉验证 |
| 3 | [S²CLNet: Structure-Constrained Semantic Contrastive Learning for Referring Remote Sensing Image Segmentation](https://doi.org/10.1109/TGRS.2026.3724005) | IEEE TGRS 2026 | 区域级结构约束对比学习 |
| 4 | [SceneProp: Combining Neural Network and Markov Random Field for Scene-Graph Grounding](https://doi.org/10.1109/WACV61042.2026.00595) | WACV 2026 | 查询场景图与视觉候选框的全局匹配 |
| 5 | [VIRO: Robust and Efficient Neuro-Symbolic Reasoning with Verification for Referring Expression Comprehension](https://openaccess.thecvf.com/content/CVPR2026/html/Park_VIRO_Robust_and_Efficient_Neuro-Symbolic_Reasoning_with_Verification_for_Referring_CVPR_2026_paper.html) | CVPR 2026 | 可执行推理程序与操作级验证 |
| 6 | [AerialVG: A Challenging Benchmark for Aerial Visual Grounding by Exploring Positional Relations](https://openaccess.thecvf.com/content/ICCV2025/html/Liu_AerialVG_A_Challenging_Benchmark_for_Aerial_Visual_Grounding_by_Exploring_ICCV_2025_paper.html) | ICCV 2025 | 航拍图像中的目标—参照物关系定位 |
| 7 | [FILD-Nav: Vision-and-Language Navigation with Instruction Landmark Features in Continuous Environments](https://ijcai.org/proceedings/2026/841) | IJCAI 2026 | 指令地标抽取及其与导航视觉特征对齐 |
| 8 | [GeoNav: Empowering MLLMs with Dual-Scale Geospatial Reasoning for Language-Goal Aerial Navigation](https://doi.org/10.1016/j.patcog.2026.113365) | Pattern Recognition 2026 | 面向无人机长距离导航的双尺度空间推理 |
| 9 | [DisCoCLIP: A Distributional Compositional Tensor Network Encoder for Vision-Language Understanding](https://aclanthology.org/2025.starsem-1.25/) | *SEM 2025 | CCG 句法结构驱动的组合式文本编码 |
| 10 | [GeoRefer-Bench: A Benchmark from Referring Pixels to Verifiable Geospatial Reasoning](https://arxiv.org/abs/2609.29541) | arXiv 2026 预印本 | 可执行空间逻辑、精确集合级评测 |

## 阅读建议

如果主要关心文本处理：先看 1、2、9。

如果主要关心文本—图像细粒度对齐：先看 3、6。

如果主要关心结构化关系推理和验证：先看 2、4、5、10。

如果主要关心 UAV 应用：先看 6、8、10；第 7 篇是室内连续 VLN，适合借鉴方法，不应误认为无人机实验。

> 重要说明：本报告区分论文正式发表状态、论文作者报告的实验结果、后续论文的独立对比结果，以及基于官方代码进行的实现观察。对于无法获取完整 PDF 的论文，会明确标注证据边界，不把推测写成论文事实。
