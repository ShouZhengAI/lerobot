<p align="center">
  <img alt="LeRobot —— Hugging Face 机器人学习库" src="./media/readme/lerobot-logo-thumbnail.png" width="100%">
</p>

<div align="center">

[![Tests](https://github.com/huggingface/lerobot/actions/workflows/latest_deps_tests.yml/badge.svg?branch=main)](https://github.com/huggingface/lerobot/actions/workflows/latest_deps_tests.yml?query=branch%3Amain)
[![Tests](https://github.com/huggingface/lerobot/actions/workflows/docker_publish.yml/badge.svg?branch=main)](https://github.com/huggingface/lerobot/actions/workflows/docker_publish.yml?query=branch%3Amain)
[![Python versions](https://img.shields.io/pypi/pyversions/lerobot)](https://www.python.org/downloads/)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://github.com/huggingface/lerobot/blob/main/LICENSE)
[![Status](https://img.shields.io/pypi/status/lerobot)](https://pypi.org/project/lerobot/)
[![Version](https://img.shields.io/pypi/v/lerobot)](https://pypi.org/project/lerobot/)
[![Contributor Covenant](https://img.shields.io/badge/Contributor%20Covenant-v2.1-ff69b4.svg)](https://github.com/huggingface/lerobot/blob/main/CODE_OF_CONDUCT.md)
[![Discord](https://img.shields.io/badge/Discord-Join_Us-5865F2?style=flat&logo=discord&logoColor=white)](https://discord.gg/q8Dzzpym3f)

</div>

**LeRobot** 致力于在 PyTorch 中为真实世界机器人技术提供模型、数据集和工具。目标是降低入门门槛，让每个人都能为共享数据集和预训练模型做出贡献并从中受益。

🤗 一套兼容多硬件的纯 Python 原生接口，标准化从低成本机械臂（SO-100）到人形机器人等各种平台的操控。

🤗 一种标准化、可扩展的 LeRobotDataset 格式（Parquet + MP4 或图像），托管在 Hugging Face Hub 上，实现海量机器人数据集的高效存储、流式传输和可视化。

🤗 经过真实场景验证的最先进策略（Policy），可直接用于训练和部署。

🤗 对开源生态系统的全面支持，推动实体 AI（Physical AI）的普及化。

## 快速开始

LeRobot 可以直接从 PyPI 安装。

```bash
pip install lerobot
lerobot-info
```

> [!IMPORTANT]
> 关于详细的安装指南，请参考[安装文档](https://huggingface.co/docs/lerobot/installation)。

## 机器人硬件与控制

<div align="center">
  <img src="./media/readme/robots_control_video.webp" width="640px" alt="Reachy 2 演示">
</div>

LeRobot 提供统一的 `Robot` 类接口，将控制逻辑与硬件细节解耦。它支持多种机器人及遥操作设备。

```python
from lerobot.robots.myrobot import MyRobot

# 连接机器人
robot = MyRobot(config=...)
robot.connect()

# 读取观测并发送动作指令
obs = robot.get_observation()
action = model.select_action(obs)
robot.send_action(action)
```

**支持的硬件：** SO100、LeKiwi、Koch、HopeJR、OMX、EarthRover、Reachy2、游戏手柄、键盘、手机、OpenARM、宇树 G1（Unitree G1）、reBot B601。

这些设备已原生集成到 LeRobot 代码库中，同时该库的设计具有良好的可扩展性。你可以轻松实现 Robot 接口，从而为自己的自定义机器人使用 LeRobot 的数据采集、训练和可视化工具。

有关详细的硬件设置指南，请参见[硬件文档](https://huggingface.co/docs/lerobot/integrate_hardware)。

## LeRobot 数据集

为解决机器人领域的数据碎片化问题，我们采用了 **LeRobotDataset** 格式。

- **结构：** 视觉数据使用同步的 MP4 视频（或图像），状态/动作数据使用 Parquet 文件。
- **HF Hub 集成：** 在 [Hugging Face Hub](https://huggingface.co/lerobot) 上探索数千个机器人数据集。
- **工具：** 无缝删除片段、按索引/比例拆分、添加/删除特征、合并多个数据集。

```python
from lerobot.datasets.lerobot_dataset import LeRobotDataset

# 从 Hub 加载数据集
dataset = LeRobotDataset("lerobot/aloha_mobile_cabinet")

# 访问数据（自动处理视频解码）
episode_index=0
print(f"{dataset[episode_index]['action'].shape=}\n")
```

了解更多：[LeRobotDataset 文档](https://huggingface.co/docs/lerobot/lerobot-dataset-v3)

## 最先进模型（SoTA Models）

LeRobot 用纯 PyTorch 实现了多种最先进的策略模型，涵盖模仿学习（Imitation Learning）、强化学习（Reinforcement Learning）、视觉-语言-动作模型（VLA）、世界模型（World Model）和奖励模型（Reward Model），更多功能即将推出。同时，它还为你提供了检测和分析训练过程的工具。

<p align="center">
  <img alt="Gr00t 架构" src="./media/readme/VLA_architecture.jpg" width="640px">
</p>

训练一个策略只需运行一条脚本配置：

```bash
lerobot-train \
  --policy.type=act \
  --dataset.repo_id=lerobot/aloha_mobile_cabinet
```

| 类别                       | 模型                                                                                                                                                                                                                                                                                                                                                                                     |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **模仿学习（Imitation Learning）**     | [ACT](./docs/source/policy_act_README.md)、[Diffusion](./docs/source/policy_diffusion_README.md)、[VQ-BeT](./docs/source/policy_vqbet_README.md)、[Multitask DiT Policy](./docs/source/policy_multi_task_dit_README.md)                                                                                                                                                                    |
| **强化学习（Reinforcement Learning）** | [HIL-SERL](./docs/source/hilserl.mdx)、[TDMPC](./docs/source/policy_tdmpc_README.md) 和 QC-FQL（即将推出）                                                                                                                                                                                                                                                                                |
| **VLA 模型（Vision-Language-Action）**            | [Pi0](./docs/source/pi0.mdx)、[Pi0Fast](./docs/source/pi0fast.mdx)、[Pi0.5](./docs/source/pi05.mdx)、[GR00T N1.7](./docs/source/policy_groot_README.md)、[SmolVLA](./docs/source/policy_smolvla_README.md)、[XVLA](./docs/source/xvla.mdx)、[EO-1](./docs/source/eo1.mdx)、[MolmoAct2](./docs/source/molmoact2.mdx)、[WALL-OSS](./docs/source/walloss.mdx)、[EVO1](./docs/source/evo1.mdx) |
| **世界模型（World Model）**           | [VLA-JEPA](./docs/source/vla_jepa.mdx)、[LingBot-VA](./docs/source/lingbot_va.mdx)、[FastWAM](./docs/source/fastwam.mdx)                                                                                                                                                                                                                                                                   |
| **奖励模型（Reward Model）**          | [SARM](./docs/source/sarm.mdx)、[TOPReward](./docs/source/topreward.mdx)、[Robometer](./docs/source/robometer.mdx)                                                                                                                                                                                                                                                                         |

与硬件类似，你可以轻松实现自己的策略，并利用 LeRobot 的数据采集、训练和可视化工具，然后将模型分享到 HF Hub。

有关详细的策略设置指南，请参见[策略文档](https://huggingface.co/docs/lerobot/bring_your_own_policies)。关于各策略的 GPU/内存需求和预期训练时间，请参见[计算硬件指南](https://huggingface.co/docs/lerobot/hardware_guide)。

## 推理与评估（Inference & Evaluation）

使用统一的评估脚本，在仿真环境或真实硬件上评估你的策略。LeRobot 支持 **LIBERO**、**MetaWorld** 等标准基准测试，更多功能即将推出。

```bash
# 在 LIBERO 基准上评估策略
lerobot-eval \
  --policy.path=lerobot/pi0_libero_finetuned \
  --env.type=libero \
  --env.task=libero_object \
  --eval.n_episodes=10
```

了解如何实现自己的仿真环境或基准测试，并通过 HF Hub 分发：请参阅 [EnvHub 文档](https://huggingface.co/docs/lerobot/envhub)

## 资源

- **[文档中心](https://huggingface.co/docs/lerobot/index)：** 完整的教程与 API 指南。
- **[中文教程：LeRobot+SO-ARM101 中文教程——同济子豪兄](https://zihao-ai.feishu.cn/wiki/space/7589642043471924447)** 涵盖组装、遥操作、数据集、训练、部署的详细文档。由 Seeed Studio 及 5 位全球黑客松玩家验证。
- **[Discord](https://discord.gg/q8Dzzpym3f)：** 加入 LeRobot 服务器，与社区交流讨论。
- **[X（推特）](https://x.com/LeRobotHF)：** 关注我们的 X 账号，获取最新动态。
- **[机器人学习教程](https://huggingface.co/spaces/lerobot/robot-learning-tutorial)：** 一个免费的动手实践课程，带你使用 LeRobot 学习机器人技术。
- **[叠 T 恤实验](https://huggingface.co/spaces/lerobot/robot-folding)：** 使用 LeRobot 折叠 T 恤的端到端演示。
- **[LeLab](https://github.com/huggingface/leLab)：** LeRobot 的 Web 界面——无需命令行，从浏览器即可完成遥操作、标定、录制数据集、回放和训练你的 SO 机械臂。

## 引用

如果在你的项目中使用了 LeRobot，请引用 GitHub 仓库，以鸣谢持续的开发和贡献者：

```bibtex
@misc{cadene2024lerobot,
    author = {Cadene, Remi and Alibert, Simon and Soare, Alexander and Gallouedec, Quentin and Zouitine, Adil and Palma, Steven and Kooijmans, Pepijn and Aractingi, Michel and Shukor, Mustafa and Aubakirova, Dana and Russi, Martino and Capuano, Francesco and Pascal, Caroline and Choghari, Jade and Meftah, Khalil and Ellerbach, Maxime and Moss, Jess and Wolf, Thomas},
    title = {LeRobot: State-of-the-art Machine Learning for Real-World Robotics in Pytorch},
    howpublished = "\url{https://github.com/huggingface/lerobot}",
    year = {2024}
}
```

如果你在引用我们的研究或学术论文，请同时引用我们的 ICLR 论文：

<details>
<summary><b>ICLR 2026 论文</b></summary>

```bibtex
@inproceedings{cadenelerobot,
  title={LeRobot: An Open-Source Library for End-to-End Robot Learning},
  author={Cadene, Remi and Alibert, Simon and Capuano, Francesco and Aractingi, Michel and Zouitine, Adil and Kooijmans, Pepijn and Choghari, Jade and Russi, Martino and Pascal, Caroline and Palma, Steven and Shukor, Mustafa and Moss, Jess and Soare, Alexander and Aubakirova, Dana and Lhoest, Quentin and Gallou\'edec, Quentin and Wolf, Thomas},
  booktitle={The Fourteenth International Conference on Learning Representations},
  year={2026},
  url={https://arxiv.org/abs/2602.22818}
}
```

</details>

## 贡献

我们欢迎来自社区每一位成员的贡献！要开始贡献，请阅读我们的 [CONTRIBUTING.md](https://github.com/huggingface/lerobot/blob/main/CONTRIBUTING.md) 指南。无论你是添加新功能、改进文档还是修复 Bug，你的帮助和反馈都弥足珍贵。我们对开源机器人技术的未来充满期待，迫不及待想要与你一起创造下一个精彩——感谢你的支持！

<p align="center">
  <img alt="SO101 演示视频" src="./media/readme/so100_video.webp" width="640px">
</p>

<div align="center">
<sub>由 <a href="https://huggingface.co">Hugging Face</a> <a href="https://huggingface.co/lerobot">LeRobot</a> 团队 ❤️ 打造</sub>
</div>
