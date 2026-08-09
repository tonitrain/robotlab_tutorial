---
title: "从robot_lab开始的Isaaclab教程——rsl_rl配置及参数详解"
date: 2026-04-05
categories: 
  - "学习笔记"
---

具体API详见：[https://github.com/leggedrobotics/rsl\_rl/blob/main/docs/guide/configuration.rst](https://github.com/leggedrobotics/rsl_rl/blob/main/docs/guide/configuration.rst)（注意该api是更新版本的rsl\_rl官方库的API，不能直接用于robotlab，isaaclab的rsl\_rl API是旧版的，直接使用参考下面文档的isaaclab自带的rsl\_rl的API文档）

isaaclab中文档说明：[https://isaac-sim.github.io/IsaacLab/main/\_modules/isaaclab\_rl/rsl\_rl/rl\_cfg.html](https://isaac-sim.github.io/IsaacLab/main/_modules/isaaclab_rl/rsl_rl/rl_cfg.html)

## 训练时要调整的参数

如果对于这里将的PPO算法不熟悉那么可以只调整entropy\_coef参数，而且在核心参数如网络大小等已经固定下来且效果好稳定后也不建议再调整，在训练时我们调整参数entropy\_coef的策略为，在开始时可以给大一点如0.01，使得网络在初期有较强的探索性，在训练稳定收敛之后可以调小如0.005或0.003，这样保证了策略更加收敛，同时奖励函数也可以有所上涨

## 源码部分

目前我们这里先只考虑旧版的API，以UnitreeA1RoughPPORunnerCfg为例

```
class UnitreeA1RoughPPORunnerCfg(RslRlOnPolicyRunnerCfg):
    num_steps_per_env = 24
    max_iterations = 20000
    save_interval = 100
    experiment_name = "unitree_a1_rough"
    policy = RslRlPpoActorCriticCfg(
        init_noise_std=1.0,
        actor_obs_normalization=False,
        critic_obs_normalization=False,
        actor_hidden_dims=[512, 256, 128],
        critic_hidden_dims=[512, 256, 128],
        activation="elu",
    )
    algorithm = RslRlPpoAlgorithmCfg(
        value_loss_coef=1.0,
        use_clipped_value_loss=True,
        clip_param=0.2,
        entropy_coef=0.01,
        num_learning_epochs=5,
        num_mini_batches=4,
        learning_rate=1.0e-3,
        schedule="adaptive",
        gamma=0.99,
        lam=0.95,
        desired_kl=0.01,
        max_grad_norm=1.0,
    )
```

该类继承的RslRlOnPolicyRunnerCfg是 Isaac Lab 中基于 rsl\_rl的 PPO训练器配置类。

**num\_steps\_per\_env**：每个并行环境运行T个时间步后收集一次 rollout

假设

```
num_envs=4096
num_steps_per_env=24
```

一次rollout的数据量为

4096×24\=983044096×24=98304

**max\_iterations**：表示PPO进行N轮策略更新，及N次iteration。在一次iteration中进行以下几个步骤

```
采样rollout → 计算GAE → Mini batch训练 → 更新Actor/Critic
```

**save\_interval** ：保存间隔，即在训练中个多少轮策略更新保存一次checkpoint,方便程序打断后训练的策略不丢失

**experiment\_name** ：实验名称

## policy 相关

policy由RslRlPpoActorCriticCfg类配置，用于配置actor-critic网络

**init\_noise\_std**：Actor初始动作噪声σ。

Isaac Lab中的连续动作采用高斯策略

πθ​(a∣s)\=N(μθ​(s),σ2)π\_{θ​(a∣s)}=N(μθ​(s),σ^2)

其中Actor输出：

μθ​(s)μθ​(s)

动作：

a\=μθ​(s)+σϵa=μθ​(s)+σϵ

σ反映当前策略的不确定性，通常训练后期会降低，在训练时通过tentorboard也可以观察std来判断策略有没有收敛

**actor\_obs\_normalization/critic\_obs\_normalization**：是否启用对于输入 _s_ 的归一化

**actor\_hidden\_dims/critic\_hidden\_dims**：actor-critic网络大小设置

**activation**：网络激活函数设置

## algorithm相关

以下采用代码中的最小化loss形式

首先我们明确，PPO的目标

L\=−LCLIP+c1​LV​−c2​H(π)​L=−L\_{CLIP}+c\_1​L\_V​−c\_2​H(π)​

其中各项函数

actor裁剪损失:

LCLIP(θ)\=Et​\[min(rt​(θ)At​,clip(rt​(θ),1−ϵ,1+ϵ)At​)\]L\_{CLIP}(θ)=E\_t​\[min(r\_t​(θ)A\_t​,clip(r\_t​(θ),1−ϵ,1+ϵ)A\_t​)\]

价值损失：

LV\=(Vϕ​(st​)−Vtarget​)2​L\_V=(Vϕ​(st​)−Vtarget​)^2​

熵正则项:

H(π)\=−E\[logπ(a∣s)\]H(π)=−E\[logπ(a∣s)\]

**value\_loss\_coef**：Critic损失

在PPO算法中critic网络负责预测真实的价值Vtarget​V\_{target​}

损失

LV\=(Vϕ​(st​)−Vtarget​)2​L\_V=(Vϕ​(st​)−Vtarget​)2​

总Loss中

L\=−LCLIP+c1​LV​−c2​H(π)​L=−L\_{CLIP}+c\_1​L\_V​−c\_2​H(π)​

c1就是value\_loss\_coef

**use\_clipped\_value\_loss**：是否开启价值网络裁切，开启后Lv就会变成

LV​\=max((Vϕ​−Vtarget​)2,(Vclip​−Vtarget​)2)L\_V​=max((Vϕ​−Vtarget​)^2,(Vclip​−Vtarget​)^2)

限制critic一次更新幅度，避免value function过拟合当前batch

**clip\_param**：PPO中的裁切系数，用于防止actor策略更新过猛，如果use\_clipped\_value\_loss开启了，也用于限制critic策略，也就是公式中的ϵ值,

**entropy\_coef**：策略熵系数，用于确定策略探索程度，也就是公式中的c2，

H(π)\=−E\[logπ(a∣s)\]H(π)=−E\[logπ(a∣s)\]

0.01就是表示鼓励探索，在训练基本收敛之后可以将这个值设小，以达到更收敛的状态

**gamma**：折扣因子

**lam**：偏差-方差

这两个均用于GAE优势估计

At​\=∑l\=0∞(γλ)lδt+l​​A\_t​=\\sum\_{l=0}^{∞}(γλ)^lδ\_{t+l​}​

其中δ表示TD误差

δt\=rt+γV(st+1)−V(st)δ\_t=r\_t+γV(s\_t+1)−V(s\_t)

也就是当下获得的奖励+下一次的奖励-这次的预测值=预测误差

其中折扣因子γ，表示机器人注重短期还是长期

对于λ在公式中的影响

如果λ↑，那么导致GAE低bias，高variance

如果λ↓，那么导致GAE高bias，低variance

**num\_learning\_epochs**：一次采样数据训练的轮数，用于提高资源的利用率

**num\_mini\_batches**：配置mini\_batch的数量，将训练数据拆分成多个mini\_batch降低GPU压力，如果增加mini\_batch的数量会导致增加噪声

假设：

```
num_envs=4096
steps=24
```

那么每个mini\_batch的数量为

98304 / 4\=2457698304 ~/~ 4 = 24576

**learning\_rate**：学习率

用于控制梯度下降的参数α

θ\=θ−α∇Lθ=θ−α∇L

**schedule**：学习策略，adaptive表示动态调整学习率

如果

KL\>desired KLKL>desired ~KL

那么降低learning rate，反之亦然

**desired\_kl**：KL散度用于对比新旧策略的差距，改变这个参数可以控制策略变化速度

**max\_grad\_norm**：梯度限制，用于限制梯度更新，防止策略更新过猛

比如假设我们得到的梯度

g\=\[34\]g= \\begin{bmatrix} 3\\\\ 4 \\end{bmatrix}

然后他的二范数

||g||2\=32+42\=5||g||\_2 = \\sqrt{3^2 + 4^2} = 5

可以得到

||g||\>1||g|| >1

那么我们等比缩小

gnew\=g×1||g||\=\[34\]×15\=\[0.60.8\]g\_{new} = g\\times\\frac{1}{||g||} = \\begin{bmatrix} 3\\\\ 4 \\end{bmatrix} \\times \\frac{1}{5} = \\begin{bmatrix} 0.6\\\\ 0.8 \\end{bmatrix}

可以看到梯度变化减小了，但是更新方向没有改变
