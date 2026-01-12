“可即插即用模块清单 + 接入位置 + 适配器代码”，直接帮你把 dream_code_v3 里的注意力/时序/SSM 模块缝到 DSTA-SLR 的 GCN 主干里。

一、推荐可插模块（按功能）

仅通道注意力（低风险，几乎即插即用）
1_1_SENet.py（SE/Squeeze-Excite）
1_6_ECA-Net.py（ECA，高效通道注意）
2_24_SimamAttention.py（轻量通道增强）
1_2_SKNets.py（SK 可选核，通道重标定）
通道+空间/图混合注意（适合放在空间GCN前后）
1_3_CBAM.py、1_3_3_CBAM_Sequence.py（CBAM）
1_5_CoordAttention.py（坐标注意，长程位置信息）
1_4_TripletAttention.py（跨维交互，稳健）
1_7_DANet.py（双非局部注意）
2_27_EGA.py、2_28_RGSA.py、2_29_STB.py（关系/图风格注意，适合骨架）
纯时序/时序卷积（替换或增强 TCN）
3_5_Temporal_conv.py（TCN 基元）
3_12_ModernTCN.py、3_4_M_GTU.py、3_15_TBA.py、3_14_TSLANet.py、3_9_MSDR.py、3_8_STAR.py、3_7_Deformable_Attention.py（按 T 维做可变形/门控/金字塔时序）
3_10_TimesNet.py（较重，强时序）
SSM/Mamba 系列（替换长序列建模，按 T 或 T×V 做 token）
Mamba_file/5_1_Mamba.py、5_10_PyramidSSM.py、5_6_CSMamba.py、5_8_ShuffleMamba.py（推荐优先：PyramidSSM 或基础 Mamba）
多尺度/融合块（用于多模态 joint/bone/motion 融合或层间融合）
6_1_Gated_Fusion.py、6_2_MSFblock.py、6_3_MSTF.py、6_5_SAFM.py、6_7_MSAA.py、6_9_MSTC.py、6_8_TFF.py
二、接入位点（DSTA-SLR 的 N,C,T,V 约定）

空间-GCN单元前后：插 CBAM/Coord/Triplet/ECA/SE（N,C,T,V → N,C,T,V，零改接口）
时序栈之间：插时序模块（ModernTCN/M_GTU/Deformable_Attn），按每个关节做 1D(T) 或把 T×V 展平成序列
头部分类前：做一次全局注意/SSM（Mamba/TimesNet），聚合全时空再接分类器
多模态融合处（joint/bone/motion）：用 Gated_Fusion/MSFblock 做通道或 token 级融合
三、轻量适配器（把 2D/序列模块适配到 N,C,T,V）
新建一个适配器文件，统一把 T 当 H、V 当 W 喂给 2D 注意力；或者按每个关节在 T 上做 1D；或把 T×V 展平给 Transformer/Mamba。
