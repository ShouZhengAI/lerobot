# 实时分块（Real-Time Chunking，RTC）

该模块包含 **实时分块（RTC）** 的 LeRobot 实现，这是一种用于基于流匹配（flow-matching）策略的推理时增强技术。

**注意**：RTC 本身并非一种策略，而是与基于流匹配的策略（包括 [π₀](../pi0/)、[π₀.₅](../pi05/) 和 [SmolVLA](../smolvla/)）配合使用的推理增强方法。

---

## 引用（Citation）

如果您在工作中使用了实时分块，请引用：

```bibtex
@misc{openpi2024,
  author       = {Physical Intelligence Lab},
  title        = {OpenPI: PyTorch Implementation of π0 and π0.5 Policies},
  year         = {2024},
  publisher    = {GitHub},
  howpublished = {\url{https://github.com/Physical-Intelligence/openpi}},
  license      = {Apache-2.0}
}

@misc{black2025realtimeexecutionactionchunking,
      title={Real-Time Execution of Action Chunking Flow Policies},
      author={Kevin Black and Manuel Y. Galliker and Sergey Levine},
      year={2025},
      eprint={2506.07339},
      archivePrefix={arXiv},
      primaryClass={cs.RO},
      url={https://arxiv.org/abs/2506.07339},
}
```

---

## 许可证（License）

本实现遵循 **Apache 2.0 License**，与 LeRobot 项目一致。
