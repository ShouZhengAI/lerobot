## 研究论文

GR00T N1 技术报告（涵盖 GR00T N1.x 系列，包括 N1.7）：https://arxiv.org/abs/2503.14734

GR00T N1.7 模型卡片：https://huggingface.co/nvidia/GR00T-N1.7-3B

GR00T N1.5 研究页面（早期版本）：https://research.nvidia.com/labs/gear/gr00t-n1_5/

> LeRobot 中已移除 GR00T N1.5 的支持；最后一个支持它的版本是 `lerobot==0.5.1`。
> 当前版本仅支持 GR00T N1.7。

## 代码仓库

代码：https://github.com/NVIDIA/Isaac-GR00T

## 引用

```bibtex
@inproceedings{gr00tn1_2025,
  archivePrefix = {arxiv},
  eprint     = {2503.14734},
  title      = {{GR00T} {N1}: An Open Foundation Model for Generalist Humanoid Robots},
  author     = {NVIDIA and Johan Bjorck andFernando Castañeda, Nikita Cherniadev and Xingye Da and Runyu Ding and Linxi "Jim" Fan and Yu Fang and Dieter Fox and Fengyuan Hu and Spencer Huang and Joel Jang and Zhenyu Jiang and Jan Kautz and Kaushil Kundalia and Lawrence Lao and Zhiqi Li and Zongyu Lin and Kevin Lin and Guilin Liu and Edith Llontop and Loic Magne and Ajay Mandlekar and Avnish Narayan and Soroush Nasiriany and Scott Reed and You Liang Tan and Guanzhi Wang and Zu Wang and Jing Wang and Qi Wang and Jiannan Xiang and Yuqi Xie and Yinzhen Xu and Zhenjia Xu and Seonghyeon Ye and Zhiding Yu and Ao Zhang and Hao Zhang and Yizhou Zhao and Ruijie Zheng and Yuke Zhu},
  month      = {March},
  year       = {2025},
  booktitle  = {ArXiv Preprint},
}
```

## 其他资源

博客：https://developer.nvidia.com/isaac/gr00t

Hugging Face 模型：

- GR00T N1.7：https://huggingface.co/nvidia/GR00T-N1.7-3B
- GR00T N1.7 LIBERO 检查点：https://huggingface.co/nvidia/GR00T-N1.7-LIBERO

<details>
<summary><b>原始实现与 LeRobot 实现的一致性测试</b></summary>

## 原始实现与 LeRobot 实现的一致性测试

`tests/policies/groot/test_groot_vs_original.py` 验证 LeRobot 对 GR00T N1.7（Qwen3-VL 主干网络 + flow-matching 动作头）的重新实现与 NVIDIA 原始 `gr00t` 包的一致性，通过两个对比维度进行验证，每个维度覆盖检查点中存在的主体标签：

1. **模型一致性** —— 给定字节级相同的预处理输入和相同的 flow-matching 种子（记录在每个工件中），两个实现必须产生**相同的原始模型输出**（`get_action(...)["action_pred"]`，即归一化后的 flow-matching 预测值）。输出形状必须完全一致；任何动作视界或动作维度不匹配都将导致测试失败。
2. **预处理一致性** —— 给定完全相同的原始观察（每个相机的画面帧、状态向量、语言指令），LeRobot 自身的预处理管线（真实的 Qwen3-VL 对话模板/分词器/图像打包 + 基于检查点的状态归一化，不使用任何模拟组件）必须产生与原始包处理器相同的**整理后模型输入**（`input_ids`、`attention_mask`、`pixel_values`、`image_grid_thw`、`state`、`embodiment_id`）。

### 为什么需要两个环境

原始的 `gr00t` 包固定使用 `transformers==4.57.3`（Python 3.10）；而本集成需要 `transformers>=5.x`（Qwen3-VL）。在 5.x 版本下，`PretrainedConfig` 本身是一个带有默认值的 dataclass，导致原始配置 dataclass 无法导入（`non-default argument follows default argument`）。因此，两个实现**无法在同一个 Python 进程中导入**。

所以测试采用**生产者/消费者**模式，在两个虚拟环境之间进行：

1. **生产者** —— `tests/policies/groot/utils/dump_original_n1_7.py`，在_原始_ gr00t 虚拟环境中运行。对于每个主体，它根据检查点元数据（状态维度来自 `statistics.json`；相机/语言键来自处理器模态配置）泛化地构建虚拟输入，运行原始模型，并为每个标签保存为一个 `.npz` 文件：原始观察（`raw::` 键）、精确的整理后输入（`in::` 键）、种子以及原始 `action_pred`。
2. **消费者** —— 上述 pytest，在 _LeRobot_ 虚拟环境中运行。它发现所有 `.npz` 文件；模型一致性测试将字节级相同的整理后输入通过 LeRobot 模型以记录的种子重新运行，并断言输出匹配；预处理一致性测试将原始观察通过 LeRobot 的完整预处理管线运行，并断言整理后的张量匹配。

> 由旧版本转存脚本生成的工件不包含 `raw::` 字段；此时预处理一致性测试将**跳过**并提示重新生成。重新运行生产者以刷新这些工件。

### 公平性控制

- **相同的预处理输入（模型一致性）** —— 原始处理器的 `input_ids`、`pixel_values`、`image_grid_thw`、`attention_mask`、`state`、`embodiment_id` 被逐字输入 LeRobot 模型（不重新分词/重新归一化），因此模型比较可以隔离模型本身。LeRobot 自己的分词/图像打包由预处理一致性测试单独覆盖，该测试将其输出与来自相同原始观察的那些整理后张量进行比较。
- **相同的精度 + 注意力核** —— 两侧都运行 **fp32 + SDPA**。原始版本默认使用 `use_flash_attention=True`（flash_attention_2 + bf16）；生产者强制使用 SDPA + fp32。（使用默认设置时差距约为 3e-2 —— 这纯粹是核函数/舍入噪声，而非实现差异。）
- **相同的 flow-matching 种子** —— 在两侧采样前固定；生产者将其记录在每个工件中（`--seed`，默认为 42），消费者回放记录的值。

### 如何运行

```bash
# 解析本地检查点（GR00T-N1.7-LIBERO / libero_10）
CKPT=$(python - <<'PY'
import os
from huggingface_hub import snapshot_download
print(os.path.join(snapshot_download("nvidia/GR00T-N1.7-LIBERO",
      allow_patterns=["libero_10/*"]), "libero_10"))
PY
)

# 1) 为所有主体生成原始端工件（原始 gr00t 虚拟环境，CUDA）
CUDA_VISIBLE_DEVICES=0 /path/to/Isaac-GR00T/.venv-original/bin/python \
    tests/policies/groot/utils/dump_original_n1_7.py \
    --ckpt "$CKPT" --out-dir tests/policies/groot/artifacts --device cuda --seed 42

# 2) 运行一致性测试（LeRobot 虚拟环境）——每个主体一个参数化用例
CUDA_VISIBLE_DEVICES=0 GROOT_PARITY_DEVICE=cuda \
    uv run pytest tests/policies/groot/test_groot_vs_original.py -v -s
```

`.npz` 工件仅在本地存在（已加入 gitignore，每个大约 6–10 MB），由生产者重新生成；它们永远不会提交。在 CI 上或当检查点/工件不存在时，测试会**跳过**（不会失败）。

#### 环境变量（均为可选）

| 变量                                        | 默认值                            | 用途                       |
| ------------------------------------------ | -------------------------------- | -------------------------- |
| `GROOT_N1_7_PARITY_DIR`                    | `tests/policies/groot/artifacts` | 每个标签的 `.npz` 工件目录 |
| `GROOT_N1_7_LIBERO_CKPT`                   | 自动（HF 缓存）                   | 覆盖检查点目录              |
| `GROOT_PARITY_DEVICE`                      | `cuda`（如果可用）                 | `cpu` 或 `cuda`            |
| `GROOT_PARITY_ATOL` / `GROOT_PARITY_RTOL` | `1e-3`                           | 比较容差                    |

</details>
