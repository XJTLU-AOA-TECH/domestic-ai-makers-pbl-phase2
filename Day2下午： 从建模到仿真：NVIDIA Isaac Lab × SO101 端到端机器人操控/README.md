# SO101 远程操控橘子抓取 — Isaac Lab 实战项目

> **一句话说明**：用实体 SO101 Leader 机械臂，在电脑屏幕的 NVIDIA Isaac Lab 仿真环境中远程操控 Follower 手臂，完成「厨房台面抓橘子放到盘子里」的 Sim-to-Real 全流程。

---

## 📑 目录

- [项目简介](#项目简介)
- [硬件清单](#硬件清单)
- [软件依赖](#软件依赖)
- [环境搭建](#环境搭建)
- [快速开始：遥操作抓橘子](#快速开始遥操作抓橘子)
- [数据采集与格式转换](#数据采集与格式转换)
- [策略训练（GR00T N1.5 / N1.6）](#策略训练gr00t-n15--n16)
- [Sim-to-Real 真机部署](#sim-to-real-真机部署)
- [进阶：全自动数据生成](#进阶全自动数据生成)
- [常见问题 FAQ](#常见问题-faq)
- [参考链接](#参考链接)

---

## 项目简介

本项目基于 **NVIDIA Isaac Lab + Isaac Sim + LeRobot + GR00T**，实现 SO101 机械臂的端到端遥操作与模仿学习。核心场景为 **PickOrange（拣橘子）**：

- 在 Isaac Sim 中构建厨房台面场景，放置若干橘子与空盘子
- 通过实体 **SO101 Leader** 手臂（6-DoF + 夹爪）采集人类动作
- 仿真中的 **SO101 Follower** 实时镜像 Leader 动作，完成抓取
- 采集的数据可转换为 LeRobot Dataset，用于训练 GR00T N1.5/N1.6 策略
- 最终策略可部署到真机，实现 Sim-to-Real 迁移

### 技术栈

| 组件 | 版本 | 用途 |
|------|------|------|
| NVIDIA Isaac Sim | 5.1+ | 物理仿真与渲染 |
| NVIDIA Isaac Lab | 2.3+ | 强化学习 / 模仿学习框架 |
| LeRobot | 0.4.3+ | 机器人学习库（数据格式、训练、推理） |
| GR00T N1 | 1.5 / 1.6 | NVIDIA 多模态 VLA 策略模型 |
| SO101 | - | 开源 6-DoF 桌面机械臂（Leader/Follower） |
| ROS 2 | Jazzy | 可选，用于真机运动规划 |

---

## 硬件清单

### 必需

| 设备 | 数量 | 说明 |
|------|------|------|
| SO101 Leader 手臂 | 1 | 遥操作端（人类手持），带 6-DoF + 夹爪 |
| SO101 Follower 手臂 | 1 | 执行端（仿真或真机），与 Leader 同构 |
| 电脑（NVIDIA GPU） | 1 | RTX 3060 及以上，显存 ≥ 8GB |
| USB-C / USB 数据线 | 2 | 连接 Leader 与电脑（CH340 串口） |

### 可选（提升体验）

| 设备 | 说明 |
|------|------|
| 双目 / RGB 摄像头 | 采集第三视角视频，用于 VLA 训练 |
| 麦克风 | 语音指令输入（如 "pick the orange"） |
| 手柄 / 键盘 | 备用遥操作设备 |

### 硬件连接拓扑

```
┌─────────────────┐     USB-C/Serial      ┌─────────────────┐
│   SO101 Leader  │ ◄──────────────────►  │   电脑 (GPU)    │
│  (遥操作手柄)    │                      │  Isaac Sim/Lab  │
└─────────────────┘                      └─────────────────┘
                                                  │
                                                  │ 仿真渲染
                                                  ▼
                                         ┌─────────────────┐
                                         │  SO101 Follower │
                                         │  (仿真/真机)     │
                                         └─────────────────┘
```

---

## 软件依赖

### 系统要求

- **OS**: Ubuntu 22.04 LTS（推荐）或 Ubuntu 24.04
- **GPU**: NVIDIA RTX 3060+，驱动 ≥ 535
- **CUDA**: 12.2+
- **Docker**: 24.0+（推荐用 Docker 部署 Isaac Sim）

### Python 环境

```bash
# 创建独立环境
conda create -n so101 python=3.10
conda activate so101
```

### 核心依赖安装

#### 1. NVIDIA Isaac Sim（推荐 Docker）

```bash
# 拉取官方 Docker 镜像（已预装 Isaac Sim 5.1 + Isaac Lab 2.3）
docker pull nvcr.io/nvidia/isaac-sim:4.5.0

# 或使用 NVIDIA 官方 Workshop 的容器
git clone https://github.com/isaac-sim/Sim-to-Real-SO-101-Workshop.git
cd Sim-to-Real-SO-101-Workshop/docker
# 按 README 构建容器
```

#### 2. Isaac Lab

```bash
git clone https://github.com/isaac-sim/IsaacLab.git
cd IsaacLab
./isaaclab.sh --install  # 或 pip install -e .
```

#### 3. LeRobot

```bash
git clone https://github.com/huggingface/lerobot.git
cd lerobot
pip install -e ".[feetech]"  # 包含 SO101 的 Feetech 舵机驱动
```

#### 4. LeIsaac（本项目核心）

```bash
git clone https://github.com/LightwheelAI/leisaac.git
cd leisaac
pip install -e .
```

#### 5. GR00T N1（用于策略训练）

```bash
# 通过 NVIDIA NGC 或 pip 安装
pip install gr00t
# 或参考 https://github.com/NVIDIA/GR00T 获取模型权重
```

---

## 环境搭建

### 方式一：Docker 一键启动（推荐）

NVIDIA 官方提供了完整的 Workshop Docker，已预装所有依赖：

```bash
git clone https://github.com/isaac-sim/Sim-to-Real-SO-101-Workshop.git
cd Sim-to-Real-SO-101-Workshop

# 构建并启动容器（会自动挂载 USB 设备给 Leader 用）
docker-compose up -d

# 进入容器
docker exec -it so101-workshop bash
```

### 方式二：本地手动安装

```bash
# 1. 克隆本仓库
git clone https://github.com/LightwheelAI/leisaac.git
cd leisaac

# 2. 安装依赖
pip install -r requirements.txt

# 3. 下载 SO101 USD 模型与场景资产
mkdir -p assets/robots assets/scenes

# 下载 Follower 模型
wget https://github.com/LightwheelAI/leisaac/releases/download/v0.1.0/so101_follower.usd     -O assets/robots/so101_follower.usd

# 下载厨房场景（含橘子、盘子）
wget https://github.com/LightwheelAI/leisaac/releases/download/v0.1.0/kitchen_with_orange.zip
unzip kitchen_with_orange.zip -d assets/scenes/

# 4. 验证安装
python -c "import isaaclab; print('IsaacLab OK')"
python -c "import lerobot; print('LeRobot OK')"
```

### 方式三：ROS 2 + Isaac Sim 桥接（进阶）

如需 ROS 2 控制真机：

```bash
git clone https://github.com/legalaspro/so101-ros-physical-ai.git
cd so101-ros-physical-ai
# 按 README 配置 ROS 2 Jazzy + MoveIt 2 + ros2_control
```

---

## 快速开始：遥操作抓橘子

### 1. 连接硬件

将 SO101 Leader 通过 USB-C 线连接到电脑，确认串口：

```bash
ls /dev/ttyACM*  # 通常显示 /dev/ttyACM0
# 如果没有权限，添加用户到 dialout 组
sudo usermod -a -G dialout $USER
```

### 2. 启动遥操作

```bash
python scripts/environments/teleoperation/teleop_se3_agent.py     --task=LeIsaac-SO101-PickOrange-v0     --teleop_device=so101leader     --port=/dev/ttyACM0     --num_envs=1     --device=cpu     --enable_cameras     --record     --dataset_file=./datasets/dataset.hdf5
```

**参数说明**：

| 参数 | 说明 |
|------|------|
| `--task=LeIsaac-SO101-PickOrange-v0` | 橘子抓取任务场景 |
| `--teleop_device=so101leader` | 使用实体 SO101 Leader 遥操作 |
| `--port=/dev/ttyACM0` | Leader 串口（根据实际调整） |
| `--num_envs=1` | 单环境（可视） |
| `--device=cpu` | 使用 CPU 运行（GPU 可改 cuda） |
| `--enable_cameras` | 开启摄像头采集（用于 VLA 训练） |
| `--record` | 录制数据到 HDF5 |
| `--dataset_file` | 数据保存路径 |

### 3. 操作指南

启动后，Isaac Sim 窗口会显示厨房场景：

- **台面**：放置 3-5 颗橘子
- **目标**：空盘子
- **Leader 手臂**：手持实体 Leader，移动关节和夹爪

**按键控制**：

| 按键 | 动作 |
|------|------|
| `b` | **开始**遥操作（开始录制当前 Episode） |
| `n` | 标记 Episode **成功**（橘子放入盘子） |
| `r` | 标记 Episode **失败**并重置环境 |
| `ESC` | 退出程序 |

### 4. 遥操作技巧

1. **夹爪控制**：Leader 的夹爪开合会直接映射到 Follower
2. **姿态对齐**：抓取前确保夹爪与橘子中心对齐
3. **稳定抓取**：夹爪闭合后保持 1-2 秒再移动，避免滑落
4. **视角切换**：鼠标拖拽可调整仿真视角，建议从斜上方观察

---

## 数据采集与格式转换

### 1. 原始数据（HDF5）

遥操作录制完成后，`./datasets/dataset.hdf5` 包含：

- `observations/images`：RGB 图像序列（第三视角 + 手腕视角）
- `observations/state`：Follower 关节状态（6-DoF + 夹爪）
- `actions`：Leader 发送的目标动作
- `rewards`：稀疏奖励（成功=1，失败=0）

### 2. 转换为 LeRobot Dataset

```bash
python scripts/data_conversion/hdf5_to_lerobot.py     --input ./datasets/dataset.hdf5     --output ./datasets/lerobot_pickorange     --task_name "pick_orange_place_plate"
```

转换后目录结构：

```
datasets/lerobot_pickorange/
├── meta/
│   ├── info.json
│   ├── stats.json
│   └── tasks.jsonl
├── videos/
│   ├── observation.images.camera_0.mp4
│   └── observation.images.camera_1.mp4
└── data/
    ├── chunk-000/parquet
    └── ...
```

### 3. 数据可视化与检查

```bash
# 使用 LeRobot 可视化工具
python -m lerobot.common.datasets.visualize     --dataset_path ./datasets/lerobot_pickorange

# 或使用 Rerun（支持 3D 轨迹回放）
rerun datasets/lerobot_pickorange
```

---

## 策略训练（GR00T N1.5 / N1.6）

### 1. 准备数据

确保 LeRobot Dataset 已就绪，并上传至 Hugging Face（可选）：

```bash
huggingface-cli login
python -m lerobot.common.datasets.push_to_hub     --dataset_path ./datasets/lerobot_pickorange     --repo_id your-username/so101-pickorange
```

### 2. 微调 GR00T N1.5

```bash
# 使用 NVIDIA 官方 GR00T 训练脚本
python -m gr00t.train     --model_name gr00t-n1-5     --dataset_path ./datasets/lerobot_pickorange     --output_dir ./checkpoints/gr00t-so101-orange     --num_epochs 50     --batch_size 8     --learning_rate 1e-4     --num_gpus 1
```

### 3. 训练参数建议

| 超参数 | 推荐值 | 说明 |
|--------|--------|------|
| `num_epochs` | 50-100 | 数据量 < 100 条时适当增大 |
| `batch_size` | 8-16 | 根据显存调整 |
| `learning_rate` | 1e-4 ~ 5e-5 | 微调时建议较小 |
| `image_size` | 224x224 | GR00T 默认输入尺寸 |
| `num_diffusion_steps` | 10 | 推理加速 |

### 4. 评估策略（仿真中）

```bash
python scripts/evaluation/eval_policy.py     --task=LeIsaac-SO101-PickOrange-v0     --checkpoint ./checkpoints/gr00t-so101-orange     --num_episodes 10     --render
```

---

## Sim-to-Real 真机部署

### 1. 真机推理服务器

在搭载 GPU 的电脑或 Jetson 上启动策略推理服务：

```bash
# 方式一：使用 NVIDIA 官方 Workshop 的推理服务器
python scripts/deployment/inference_server.py     --checkpoint ./checkpoints/gr00t-so101-orange     --port 5555     --device cuda

# 方式二：使用 ROS 2 节点（推荐）
ros2 run so101_ros_physical_ai policy_server     --ros-args -p checkpoint:=./checkpoints/gr00t-so101-orange
```

### 2. 真机 Follower 连接

```bash
# 连接真机 SO101 Follower
python scripts/deployment/real_robot_follower.py     --port /dev/ttyACM1     --inference_server localhost:5555
```

### 3. 部署流程图

```
┌─────────────────┐      USB/Serial       ┌─────────────────┐
│  SO101 Leader   │ ◄────────────────────► │   电脑 (GPU)    │
│  (可选：触发)    │                        │  策略推理服务器  │
└─────────────────┘                        │  (GR00T N1.6)   │
                                           └─────────────────┘
                                                    │
                                                    │ ZMQ/gRPC
                                                    ▼
                                           ┌─────────────────┐
                                           │  SO101 Follower │
                                           │  (真机执行)      │
                                           └─────────────────┘
```

### 4. 安全注意事项

- **首次部署**：先在仿真中充分验证策略成功率 > 80%
- **速度限制**：真机执行时设置最大关节速度为仿真的 50%
- **急停**：Leader 手臂保留物理急停按钮，随时可中断
- **工作空间**：确保真机周围无易碎物品，操作半径 ≤ 0.5m

---

## 进阶：全自动数据生成

如果你不想手动遥操作，可以使用 **so101-autogen** 自动生成训练数据：

```bash
git clone https://github.com/haoran1062/so101-autogen.git
cd so101-autogen

# 自动生成 100 条橘子抓取 Episode
python scripts/generate_data.py     --task=pick_orange     --num_episodes 100     --output ./datasets/autogen_orange.pkl

# 转换为 LeRobot 格式
python scripts/convert_to_lerobot.py     --input ./datasets/autogen_orange.pkl     --output ./datasets/lerobot_autogen_orange
```

**自动生成原理**：
- 使用 IK（逆运动学）计算夹爪到橘子中心的轨迹
- 状态机控制：接近 → 对齐 → 夹取 → 提升 → 移动到盘子 → 释放
- 自动标注成功/失败，并生成多视角视频

---

## 常见问题 FAQ

### Q1: Isaac Sim 启动报错 `CUDA out of memory`

**A**: 
- 降低 `--num_envs` 为 1
- 在 Isaac Sim 设置中降低渲染分辨率（Preferences → Rendering → Resolution Scale = 0.5）
- 关闭 `--enable_cameras` 或降低摄像头分辨率

### Q2: Leader 手臂连接后无响应

**A**:
```bash
# 检查串口权限
sudo chmod 666 /dev/ttyACM0

# 检查是否被其他程序占用
lsof /dev/ttyACM0

# 确认 Leader 已上电（LED 指示灯常亮）
# 重新插拔 USB 线，等待 3 秒再启动程序
```

### Q3: 仿真中 Follower 动作与 Leader 不同步

**A**:
- 检查 Leader 和 Follower 的关节限位是否一致（SO101 Leader 和 Follower 通常为镜像配置）
- 在 `teleop_se3_agent.py` 中调整 `--teleop_scale` 参数（默认 1.0）
- 确保 Leader 已校准：启动前将 Leader 置于初始姿态（所有关节归零）

### Q4: 策略训练后仿真成功率低

**A**:
- 检查数据质量：使用可视化工具确认动作平滑、无抖动
- 增加数据量：至少 50 条成功 Episode
- 调整数据增强：在 LeRobot 配置中开启 `image_transforms`（颜色抖动、随机裁剪）
- 降低学习率，增加训练轮数

### Q5: 真机部署时夹爪力度过大/过小

**A**:
- 在 `real_robot_follower.py` 中调整 `gripper_force` 参数
- SO101 夹爪使用 Feetech STS3215 舵机，可通过 `lerobot` 的 `calibrate` 命令重新标定：
```bash
python -m lerobot.common.robots.so101.calibrate --port /dev/ttyACM1
```

### Q6: 如何添加自定义物体（如苹果、香蕉）？

**A**:
1. 在 Isaac Sim 中导入 USD 模型（支持 `.usd`, `.obj`, `.fbx`）
2. 修改任务配置文件 `source/extensions/omni.isaac.lab_tasks/omni/isaac/lab_tasks/manager_based/manipulation/pick_orange/so101_pickorange_cfg.py`
3. 在 `object_usd_path` 中添加新物体路径，并调整 `object_spawn_pose`

---

## 参考链接

| 项目 | 链接 | 说明 |
|------|------|------|
| **LeIsaac（核心）** | https://github.com/LightwheelAI/leisaac | SO101 Isaac Lab 遥操作与训练 |
| **NVIDIA 官方 Workshop** | https://github.com/isaac-sim/Sim-to-Real-SO-101-Workshop | Sim-to-Real 端到端教程 |
| **so101-autogen** | https://github.com/haoran1062/so101-autogen | 全自动数据生成 |
| **SO101-ROS-Physical-AI** | https://github.com/legalaspro/so101-ros-physical-ai | ROS 2 + MoveIt 2 真机部署 |
| **isaac_so_arm101** | https://github.com/MuammerBay/isaac_so_arm101 | Isaac Lab 外部 RL 项目 |
| **so101-playground** | https://github.com/vvrs/so101-playground | LeRobot 快速上手 |
| **Isaac Lab** | https://github.com/isaac-sim/IsaacLab | NVIDIA 官方 RL 框架 |
| **LeRobot** | https://github.com/huggingface/lerobot | Hugging Face 机器人学习库 |
| **GR00T** | https://github.com/NVIDIA/GR00T | NVIDIA 人形机器人 VLA 模型 |
| **NVIDIA 文档** | https://docs.nvidia.com/learning/physical-ai/sim-to-real-so-101/latest/ | 官方 Workshop 文档 |

---

## 许可证

本项目遵循各自子项目的开源许可证：
- LeIsaac: MIT License
- Isaac Lab: BSD-3-Clause
- LeRobot: Apache 2.0
- GR00T: NVIDIA 自定义许可（需同意 NGC 条款）

---

## 致谢

- NVIDIA Isaac Sim / Isaac Lab 团队
- Hugging Face LeRobot 社区
- LightwheelAI（LeIsaac 开源）
- Standard Bots（SO101 机械臂设计）

---

> **提示**：如遇到未列出的问题，请优先查阅 [LeIsaac Issues](https://github.com/LightwheelAI/leisaac/issues) 或 [NVIDIA 官方论坛](https://forums.developer.nvidia.com/c/omniverse/)。
