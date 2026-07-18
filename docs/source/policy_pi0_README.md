# π₀ (pi0)

本仓库包含 **π₀** 的 Hugging Face 移植版本，改编自 Physical Intelligence 的 [OpenPI](https://github.com/Physical-Intelligence/openpi)。
它被设计为**用于通用机器人控制的视觉-语言-动作模型**（Vision-Language-Action model）。

---

## 模型概述（Model Overview）

| 特性                  | π₀                                                                 | π₀.₅                                                   |
| -------------------- | ------------------------------------------------------------------ | ------------------------------------------------------ |
| 时间条件（Time Conditioning） | 通过 `action_time_mlp_*` 将时间与动作拼接在一起                              | 使用 `time_mlp_*` 进行 AdaRMS 条件化                  |
| AdaRMS               | 未使用                                                              | 在动作专家网络（action expert）中使用                  |
| 分词器长度（Tokenizer Length） | 48 个 token                                                        | 200 个 token                                           |
| 离散状态输入（Discrete State Input） | 否（使用 `state_proj` 层）                                              | 是                                                     |
| 参数规模              | 更大（包含状态嵌入层）                                                  | 更小（无状态嵌入层）                                    |

---

## 相对动作（Relative Actions）

π₀ 支持使用**相对动作**（relative actions）进行训练，模型学习的是相对于当前机器人状态的偏移量，而非绝对关节位置。
这相当于 OpenPI 中的相对动作变换（`DeltaActions`），可以提升性能。

### 工作原理

1. **预处理阶段**，绝对动作被转换为相对偏移量：
   `relative = action - state`（针对选定关节）。
2. 相对动作使用从相对分布计算得到的统计量进行归一化。
3. **后处理阶段**，预测的相对动作被转换回绝对动作：
   `absolute = relative + state`。

`relative_exclude_joints`（如夹爪）中列出的关节保持为绝对动作。

### 配置参数

| 参数                        | 类型          | 默认值          | 描述                                                  |
| -------------------------- | ------------- | --------------- | ----------------------------------------------------- |
| `use_relative_actions`     | `bool`        | `False`         | 启用相对动作训练                                      |
| `relative_exclude_joints`  | `list[str]`   | `["gripper"]`   | 保持为绝对动作的关节名称（按子串匹配）                |
| `action_feature_names`     | `list[str]`   | `None`          | 运行时由 `make_policy` 根据数据集的元数据自动填充      |

### 训练示例

```bash
python -m lerobot.scripts.lerobot_train \
  --policy.type=pi0 \
  --dataset.repo_id=your_org/your_dataset \
  --policy.use_relative_actions=true \
  --policy.relative_exclude_joints='["gripper"]'
```

当 `use_relative_actions=true` 时，训练脚本会自动：

- 根据数据集计算相对动作统计量（采样的片段级相对动作，chunk-level relative actions）
- 用相对统计量替换标准动作统计量，用于归一化
- 在分布式训练中将这些统计量广播到所有进程

### 为已有数据集重新计算统计量

如果你希望离线预计算相对动作统计量，可以使用 `lerobot.datasets` 中的 `recompute_stats`：

```python
from lerobot.datasets import LeRobotDataset, recompute_stats

dataset = LeRobotDataset("your_org/your_dataset")
dataset = recompute_stats(
    dataset,
    relative_action=True,
    relative_exclude_joints=["gripper"],
)
```

---

## 引用（Citation）

如果您使用了本工作，请同时引用 **OpenPI** 和 π₀ 论文：

```bibtex
@misc{openpi2024,
  author       = {Physical Intelligence Lab},
  title        = {OpenPI: PyTorch Implementation of π0 and π0.5 Policies},
  year         = {2024},
  publisher    = {GitHub},
  howpublished = {\url{https://github.com/Physical-Intelligence/openpi}},
  license      = {Apache-2.0}
}

@misc{black2024pi0visionlanguageactionflowmodel,
  title        = {π₀: A Vision-Language-Action Flow Model for General Robot Control},
  author       = {Kevin Black and Noah Brown and Danny Driess and Adnan Esmail and Michael Equi and Chelsea Finn and Niccolo Fusai and Lachy Groom and Karol Hausman and Brian Ichter and Szymon Jakubczak and Tim Jones and Liyiming Ke and Sergey Levine and Adrian Li-Bell and Mohith Mothukuri and Suraj Nair and Karl Pertsch and Lucy Xiaoyang Shi and James Tanner and Quan Vuong and Anna Walling and Haohuan Wang and Ury Zhilinsky},
  year         = {2024},
  eprint       = {2410.24164},
  archivePrefix= {arXiv},
  primaryClass = {cs.LG},
  url          = {https://arxiv.org/abs/2410.24164},
}
```

---

## 许可证（License）

本移植版本遵循 **Apache 2.0 License**，与原始 [OpenPI 仓库](https://github.com/Physical-Intelligence/openpi) 一致。
