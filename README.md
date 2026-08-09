# robotlab_tutorial

基于 [robot_lab](https://github.com/fan-ziqi/robot_lab) 的 Isaac Lab 中文入门教程，面向希望了解机器人强化学习项目结构、接入自定义机器人，并进一步配置训练环境与 PPO 参数的读者。

[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04-E95420?logo=ubuntu&logoColor=white)](https://ubuntu.com/)
[![Isaac Lab](https://img.shields.io/badge/Isaac%20Lab-2.3.2-76B900?logo=nvidia&logoColor=white)](https://isaac-sim.github.io/IsaacLab/)
[![robot_lab](https://img.shields.io/badge/based%20on-robot__lab-0969da)](https://github.com/fan-ziqi/robot_lab)

> 本仓库保存的是教程与学习笔记，不包含 `robot_lab` 的完整训练源码。实际运行前，请先安装 Isaac Lab 并获取上游 `robot_lab` 项目。

![机器人强化学习框架整体架构](./isaaclab入门教程/images/ChatGPT-Image-2026年8月8日-22_39_24-1024x819.png)

## 教程内容

建议按以下顺序阅读：

| 顺序 | 主题 | 主要内容 | 仓库文档 | 在线文章 |
| --- | --- | --- | --- | --- |
| 1 | 总体介绍 | 认识 `robot_lab`、Isaac Sim、Isaac Lab 与训练器之间的关系，以及项目目录结构 | [阅读文档](./isaaclab入门教程/index.md) | [阅读原文](http://tonightrain.top/2026/08/05/isaaclab%e5%85%a5%e9%97%a8%e6%95%99%e7%a8%8b/) |
| 2 | 快速开始 | 以 Unitree A1 为例，配置 URDF、关节初始状态、执行器和训练任务注册 | [阅读文档](./isaaclab中导入自己的模型进行训练/index.md) | [阅读原文](http://tonightrain.top/2026/04/05/isaaclab%e4%b8%ad%e5%af%bc%e5%85%a5%e8%87%aa%e5%b7%b1%e7%9a%84%e6%a8%a1%e5%9e%8b%e8%bf%9b%e8%a1%8c%e8%ae%ad%e7%bb%83/) |
| 3 | 环境配置 | 讲解 Scene、Reward、Command、Event、Terrain 与 Termination 等配置 | [阅读文档](./robot_lab学习笔记-env_cfg/index.md) | [阅读原文](http://tonightrain.top/2026/03/30/robot_lab%e5%ad%a6%e4%b9%a0%e7%ac%94%e8%ae%b0-env_cfg/) |
| 4 | `rsl_rl` 配置 | 理解 Actor-Critic 网络与 PPO 训练参数，包括熵系数、GAE、学习率和梯度裁剪 | [阅读文档](./rsl_rl强化学习框架在isaaclab中的配置/index.md) | [阅读原文](http://tonightrain.top/2026/04/06/rsl_rl%e5%bc%ba%e5%8c%96%e5%ad%a6%e4%b9%a0%e6%a1%86%e6%9e%b6%e5%9c%a8isaaclab%e4%b8%ad%e7%9a%84%e9%85%8d%e7%bd%ae/) |

## 你将学到什么

- 理解 Isaac Sim、Isaac Lab 和 `rsl_rl` 在机器人强化学习中的分工。
- 找到 `robot_lab` 中脚本、机器人资源、环境配置与训练配置所在的位置。
- 使用 `ArticulationCfg` 接入自定义 URDF，并配置关节初始状态和执行器参数。
- 注册新的机器人训练任务，并将环境配置与训练器配置关联起来。
- 配置观测、动作、指令、奖励、随机事件、地形、终止条件和课程学习。
- 理解常见 PPO 参数对探索、收敛速度与训练稳定性的影响。

## 运行环境

教程内容基于以下环境编写：

- Ubuntu 22.04
- Isaac Lab 2.3.2
- [fan-ziqi/robot_lab](https://github.com/fan-ziqi/robot_lab)
- `rsl_rl`（通过 Isaac Lab / `robot_lab` 使用）

Isaac Lab 更新频繁，不同版本的目录结构、类名和配置字段可能发生变化。遇到 API 差异时，请优先核对你所安装版本的 [Isaac Lab 官方文档](https://isaac-sim.github.io/IsaacLab/) 与源码。

## 快速开始

本仓库无需安装，直接阅读各目录下的 `index.md` 即可。若要运行教程中的训练配置，请先按照上游文档安装 Isaac Lab 和 `robot_lab`，然后在 `robot_lab` 根目录执行：

```bash
python -m pip install -e source/robot_lab
```

接入自定义机器人时，推荐按以下流程进行：

1. 在 `robot_lab/source/robot_lab/robot_lab/assets` 中添加机器人资源与配置。
2. 根据 URDF 修改 `asset_path`、初始关节角、关节限制和执行器参数。
3. 参考现有机器人创建对应的环境配置类。
4. 通过 `gym.register(...)` 关联环境配置与 `rsl_rl` 训练配置。
5. 先使用检查脚本验证环境，再开始训练并通过播放脚本查看策略效果。

具体代码与参数说明请参阅[快速开始](./isaaclab中导入自己的模型进行训练/index.md)。

## 仓库结构

```text
robotlab_tutorial/
├── isaaclab入门教程/
│   ├── images/
│   └── index.md
├── isaaclab中导入自己的模型进行训练/
│   ├── images/
│   └── index.md
├── robot_lab学习笔记-env_cfg/
│   └── index.md
├── rsl_rl强化学习框架在isaaclab中的配置/
│   └── index.md
└── README.md
```

## 相关链接

- [个人网站：下夜雨 tonightrain](http://tonightrain.top/)
- [robot_lab](https://github.com/fan-ziqi/robot_lab)
- [Isaac Lab 官方文档](https://isaac-sim.github.io/IsaacLab/)
- [rsl_rl](https://github.com/leggedrobotics/rsl_rl)

如发现内容错误、版本差异或描述不清，欢迎提交 Issue 或 Pull Request。
