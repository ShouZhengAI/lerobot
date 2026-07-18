# π₀.₅ (pi05)

本文档包含 **π₀.₅** 的 Hugging Face 移植版本，改编自 Physical Intelligence 的 [OpenPI](https://github.com/Physical-Intelligence/openpi)。
它被设计为一种**具有开放世界泛化能力的视觉-语言-动作模型**。

---

## 模型概览

| 特性                 | π₀                                                       | π₀.₅                                      |
| -------------------- | -------------------------------------------------------- | ----------------------------------------- |
| 时间条件控制         | 通过 `action_time_mlp_*` 将时间与动作拼接                   | 使用 `time_mlp_*` 进行 AdaRMS 条件控制     |
| AdaRMS               | 不使用                                                   | 在动作专家模块中使用                        |
| 分词器长度           | 48 个 token                                              | 200 个 token                              |
| 离散状态输入         | 否（使用 `state_proj` 层）                                | 是                                        |
| 参数量               | 较高（包含状态嵌入层）                                    | 较低（无状态嵌入层）                        |

---

## 相对动作

π₀.₅ 支持使用**相对动作**进行训练，即模型学习相对于当前机器人状态的偏移量，而非绝对关节位置。这借鉴了 OpenPI 中的相对动作变换（`DeltaActions`），可以提升模型性能。

### 工作原理

1. **在预处理阶段**，绝对动作被转换为相对偏移量：
   `相对值 = 动作值 - 状态值`（针对选定的关节）。
2. 相对动作使用从相对分布计算得到的统计量进行归一化。
3. **在后处理阶段**，预测的相对动作被转换回绝对动作：
   `绝对值 = 相对值 + 状态值`。

列在 `relative_exclude_joints` 中的关节（例如夹爪）保持为绝对值。

### 配置参数

| 参数                        | 类型          | 默认值         | 描述                                           |
| --------------------------- | ------------- | -------------- | ---------------------------------------------- |
| `use_relative_actions`      | `bool`        | `False`        | 启用相对动作训练                                |
| `relative_exclude_joints`   | `list[str]`   | `["gripper"]`  | 保持为绝对值的关节名称（通过子串匹配）          |
| `action_feature_names`      | `list[str]`   | `None`         | 在运行时由 `make_policy` 从数据集元数据自动填充 |

### 训练示例

```bash
python -m lerobot.scripts.lerobot_train \
  --policy.type=pi05 \
  --dataset.repo_id=your_org/your_dataset \
  --policy.use_relative_actions=true \
  --policy.relative_exclude_joints='["gripper"]'
```

当 `use_relative_actions=true` 时，训练脚本会自动：

- 从数据集中计算相对动作统计量（对采样到的片段级相对动作进行计算）
- 用相对统计量替换标准动作统计量以进行归一化
- 在分布式训练中向所有进程广播这些统计量

---

## 引用

如果您使用了本工作，请同时引用 **OpenPI** 和 π₀.₅ 论文：

```bibtex
@misc{openpi2024,
  author       = {Physical Intelligence Lab},
  title        = {OpenPI: PyTorch Implementation of π0 and π0.5 Policies},
  year         = {2024},
  publisher    = {GitHub},
  howpublished = {\url{https://github.com/Physical-Intelligence/openpi}},
  license      = {Apache-2.0}
}

@misc{intelligence2025pi05visionlanguageactionmodelopenworld,
  title        = {π₀.₅: a Vision-Language-Action Model with Open-World Generalization},
  author       = {Physical Intelligence and Kevin Black and Noah Brown and James Darpinian and Karan Dhabalia and Danny Driess and Adnan Esmail and Michael Equi and Chelsea Finn and Niccolo Fusai and Manuel Y. Galliker and Dibya Ghosh and Lachy Groom and Karol Hausman and Brian Ichter and Szymon Jakubczak and Tim Jones and Liyiming Ke and Devin LeBlanc and Sergey Levine and Adrian Li-Bell and Mohith Mothukuri and Suraj Nair and Karl Pertsch and Allen Z. Ren and Lucy Xiaoyang Shi and Laura Smith and Jost Tobias Springenberg and Kyle Stachowicz and James Tanner and Quan Vuong and Homer Walke and Anna Walling and Haohuan Wang and Lili Yu and Ury Zhilinsky},
  year         = {2025},
  eprint       = {2504.16054},
  archivePrefix= {arXiv},
  primaryClass = {cs.LG},
  url          = {https://arxiv.org/abs/2504.16054},
}
```

---

## 许可协议

本移植版本遵循 **Apache 2.0 许可协议**，与原始 [OpenPI 仓库](https://github.com/Physical-Intelligence/openpi)保持一致。
