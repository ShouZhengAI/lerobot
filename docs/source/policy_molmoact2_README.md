# MolmoAct2

本文档包含 [MolmoAct2](https://allenai.org/blog/molmoact2) 的 LeRobot 策略实现，已移植到 LeRobot 中以支持训练、评估、检查点保存和数据集兼容性。

当前实现支持常规 MolmoAct2 模型的训练和评估。支持自适应深度推理的 MolmoAct2-Think 尚未包含在此 LeRobot 策略中，即将推出。

如需查看论文中报告实验所使用的原始 MolmoAct2 训练代码，请参见 [allenai/molmoact2](https://github.com/allenai/molmoact2)。

## LIBERO 评估

重要说明：我们发现 `num_steps_wait=10` 无法可靠地让 LIBERO 场景稳定下来，可能会降低测量得到的成功率。此 LeRobot 实现报告的所有 LIBERO 评估结果均使用 `num_steps_wait=50`。

## 引用

```bibtex
@misc{fang2026molmoact2actionreasoningmodels,
      title={MolmoAct2: Action Reasoning Models for Real-world Deployment},
      author={Haoquan Fang and Jiafei Duan and Donovan Clay and Sam Wang and Shuo Liu and Weikai Huang and Xiang Fan and Wei-Chuan Tsai and Shirui Chen and Yi Ru Wang and Shanli Xing and Jaemin Cho and Jae Sung Park and Ainaz Eftekhar and Peter Sushko and Karen Farley and Angad Wadhwa and Cole Harrison and Winson Han and Ying-Chun Lee and Eli VanderBilt and Rose Hendrix and Suveen Ellawela and Lucas Ngoo and Joyce Chai and Zhongzheng Ren and Ali Farhadi and Dieter Fox and Ranjay Krishna},
      year={2026},
      eprint={2605.02881},
      archivePrefix={arXiv},
      primaryClass={cs.RO},
      url={https://arxiv.org/abs/2605.02881},
}
```

## 许可协议

本模型基于 Apache 2.0 许可协议发布。它旨在用于研究和教育目的，遵循 [Ai2 的负责任使用指南](https://allenai.org/responsible-use)，与 [allenai/molmoact2](https://github.com/allenai/molmoact2) 保持一致。
