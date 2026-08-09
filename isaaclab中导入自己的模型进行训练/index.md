---
title: "从robot_lab开始的Isaaclab教程——快速开始"
date: 2026-04-05
categories: 
  - "学习笔记"
---

在[总体介绍](http://tonightrain.top/2026/08/05/isaaclab%e5%85%a5%e9%97%a8%e6%95%99%e7%a8%8b/)中我们了解了robotlab的项目结构，通过他我们可以了解到如果只是希望将自己的机器人加到这个训练框架里，只要将机器人配置文件配置即可，该框架下的机器人都放在了robot\_lab\\source\\robot\_lab\\robot\_lab\\assets目录下，所以可以仿照现有的配置文件，只将urdf配置替换，即将ArticulationCfg下的asset\_path替换

这里以A1为例，具体参考哪个机器人按照自己的自由度机器人需求：

```

UNITREE_A1_CFG = ArticulationCfg(
    spawn=sim_utils.UrdfFileCfg(
        fix_base=False,
        merge_fixed_joints=True,
        replace_cylinders_with_capsules=False,
        asset_path=f"{ISAACLAB_ASSETS_DATA_DIR}/Robots/unitree/a1_description/urdf/a1.urdf",         # 替换这个
        activate_contact_sensors=True,
        rigid_props=sim_utils.RigidBodyPropertiesCfg(
            disable_gravity=False,
            retain_accelerations=False,
            linear_damping=0.0,
            angular_damping=0.0,
            max_linear_velocity=1000.0,
            max_angular_velocity=1000.0,
            max_depenetration_velocity=1.0,
        ),
        articulation_props=sim_utils.ArticulationRootPropertiesCfg(
            enabled_self_collisions=False, solver_position_iteration_count=4, solver_velocity_iteration_count=0
        ),
        joint_drive=sim_utils.UrdfConverterCfg.JointDriveCfg(
            gains=sim_utils.UrdfConverterCfg.JointDriveCfg.PDGainsCfg(stiffness=0, damping=0)
        ),
    ),
    init_state=ArticulationCfg.InitialStateCfg(
        pos=(0.0, 0.0, 0.38),
        joint_pos={
            ".*L_hip_joint": 0.0,
            ".*R_hip_joint": -0.0,
            "F.*_thigh_joint": 0.8,
            "R.*_thigh_joint": 0.8,
            ".*_calf_joint": -1.5,
        },
        joint_vel={".*": 0.0},
    ),
    soft_joint_pos_limit_factor=0.9,
    actuators={
        "legs": DCMotorCfg(
            joint_names_expr=[".*_joint"],
            effort_limit=33.5,
            saturation_effort=33.5,
            velocity_limit=21.0,
            stiffness=20.0,
            damping=0.5,
            friction=0.0,
        ),
    },
)
```

根据实际情况对部分参数也可以做实际微调，比如说`InitialStateCfg`的关节初始角，`soft_joint_pos_limit_factor`仿真器关节角度软约束，以及`actuators`配置中的`DCMotorCfg`电机类进行修改比如修改stiffness和damping，这两个参数对应实际电机的PD控制器的Kp和Kd，或者观察实机关节相应延迟,将关节类换成`DelayedPDActuator`，可以加入时间步为单位个延时具体配置可以参考文档

DelayedPDActuator可配置参数

[`cfg`](https://isaac-sim.github.io/IsaacLab/main/source/api/lab/isaaclab.actuators.html#isaaclab.actuators.DelayedPDActuator.cfg)  
The configuration for the actuator model.  
[`is_implicit_model`](https://isaac-sim.github.io/IsaacLab/main/source/api/lab/isaaclab.actuators.html#isaaclab.actuators.DelayedPDActuator.is_implicit_model)  
Flag indicating if the actuator is an implicit or explicit actuator model.  
[`joint_indices`](https://isaac-sim.github.io/IsaacLab/main/source/api/lab/isaaclab.actuators.html#isaaclab.actuators.DelayedPDActuator.joint_indices)  
Articulation's joint indices that are part of the group.  
[`joint_names`](https://isaac-sim.github.io/IsaacLab/main/source/api/lab/isaaclab.actuators.html#isaaclab.actuators.DelayedPDActuator.joint_names)  
Articulation's joint names that are part of the group.  
[`num_joints`](https://isaac-sim.github.io/IsaacLab/main/source/api/lab/isaaclab.actuators.html#isaaclab.actuators.DelayedPDActuator.num_joints)  
Number of actuators in the group.  
[`computed_effort`](https://isaac-sim.github.io/IsaacLab/main/source/api/lab/isaaclab.actuators.html#isaaclab.actuators.DelayedPDActuator.computed_effort)  
The computed effort for the actuator group.  
[`applied_effort`](https://isaac-sim.github.io/IsaacLab/main/source/api/lab/isaaclab.actuators.html#isaaclab.actuators.DelayedPDActuator.applied_effort)  
The applied effort for the actuator group.  
[`effort_limit`](https://isaac-sim.github.io/IsaacLab/main/source/api/lab/isaaclab.actuators.html#isaaclab.actuators.DelayedPDActuator.effort_limit)  
The effort limit for the actuator group.  
[`effort_limit_sim`](https://isaac-sim.github.io/IsaacLab/main/source/api/lab/isaaclab.actuators.html#isaaclab.actuators.DelayedPDActuator.effort_limit_sim)  
The effort limit for the actuator group in the simulation.  
[`velocity_limit`](https://isaac-sim.github.io/IsaacLab/main/source/api/lab/isaaclab.actuators.html#isaaclab.actuators.DelayedPDActuator.velocity_limit)  
The velocity limit for the actuator group.  
[`velocity_limit_sim`](https://isaac-sim.github.io/IsaacLab/main/source/api/lab/isaaclab.actuators.html#isaaclab.actuators.DelayedPDActuator.velocity_limit_sim)  
The velocity limit for the actuator group in the simulation.  
[`stiffness`](https://isaac-sim.github.io/IsaacLab/main/source/api/lab/isaaclab.actuators.html#isaaclab.actuators.DelayedPDActuator.stiffness)  
The stiffness (P gain) of the PD controller.  
[`damping`](https://isaac-sim.github.io/IsaacLab/main/source/api/lab/isaaclab.actuators.html#isaaclab.actuators.DelayedPDActuator.damping)  
The damping (D gain) of the PD controller.  
[`armature`](https://isaac-sim.github.io/IsaacLab/main/source/api/lab/isaaclab.actuators.html#isaaclab.actuators.DelayedPDActuator.armature)  
The armature of the actuator joints.  
[`friction`](https://isaac-sim.github.io/IsaacLab/main/source/api/lab/isaaclab.actuators.html#isaaclab.actuators.DelayedPDActuator.friction)  
The joint static friction of the actuator joints.  
[`dynamic_friction`](https://isaac-sim.github.io/IsaacLab/main/source/api/lab/isaaclab.actuators.html#isaaclab.actuators.DelayedPDActuator.dynamic_friction)  
The joint dynamic friction of the actuator joints.  
[`viscous_friction`](https://isaac-sim.github.io/IsaacLab/main/source/api/lab/isaaclab.actuators.html#isaaclab.actuators.DelayedPDActuator.viscous_friction)  
The joint viscous friction of the actuator joints.

将机器人配置的CFG配置好后就可以仿照现有的环境配置替换掉环境设置中的机器人配置了

依旧按照A1机器人为例

```
# 内容来自于robot_lab\source\robot_lab\robot_lab\tasks\manager_based\locomotion\velocity\config\quadruped\unitree_a1\rough_env_cfg.py

@configclass
class UnitreeA1RoughEnvCfg(LocomotionVelocityRoughEnvCfg):                        # 名字不要忘记改了
    base_link_name = "base"
    foot_link_name = ".*_foot"
    # fmt: off
    joint_names = [
        "FR_hip_joint", "FR_thigh_joint", "FR_calf_joint",
        "FL_hip_joint", "FL_thigh_joint", "FL_calf_joint",
        "RR_hip_joint", "RR_thigh_joint", "RR_calf_joint",
        "RL_hip_joint", "RL_thigh_joint", "RL_calf_joint",
    ]
    # fmt: on

    def __post_init__(self):
        # post init of parent
        super().__post_init__()

        # ------------------------------Sence------------------------------
        self.scene.robot = UNITREE_A1_CFG.replace(prim_path="{ENV_REGEX_NS}/Robot")           #将这里UNITREE_A1_CFG替换
        self.scene.height_scanner.prim_path = "{ENV_REGEX_NS}/Robot/" + self.base_link_name
        self.scene.height_scanner_base.prim_path = "{ENV_REGEX_NS}/Robot/" + self.base_link_name
```

最后注册任务

```
gym.register(
    id="RobotLab-Isaac-Velocity-Rough-Unitree-A1-v0",
    entry_point="isaaclab.envs:ManagerBasedRLEnv",
    disable_env_checker=True,
    kwargs={
        "env_cfg_entry_point": f"{__name__}.rough_env_cfg:UnitreeA1RoughEnvCfg",                  # 替换掉这个
        "rsl_rl_cfg_entry_point": f"{agents.__name__}.rsl_rl_ppo_cfg:UnitreeA1RoughPPORunnerCfg", # 这个如设置也可以替换
        "cusrl_cfg_entry_point": f"{agents.__name__}.cusrl_ppo_cfg:UnitreeA1RoughTrainerCfg",
    },
)
```

最后记得install一下，方便训练脚本可以导入库

```
python -m pip install -e source/robot_lab
```

这样就可以开始训练了
