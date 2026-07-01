# LeIsaac 项目完整部署指南：SO101 机械臂仿真抓取橘子

> **项目仓库**：https://github.com/LightwheelAI/leisaac  
> **核心功能**：在 NVIDIA Isaac Lab 仿真环境中，使用 LeRobot SO101 主臂遥操作仿真从臂完成厨房场景中的橘子抓取任务，并支持数据采集、格式转换与策略训练。  
> **适用系统**：Ubuntu 24.04 Desktop + NVIDIA GPU（显存建议 ≥ 16GB）

---

## 一、项目概述

LeIsaac 是 Lightwheel AI 开源的机器人仿真遥操作框架，打通了以下技术链路：

| 模块 | 说明 |
|------|------|
| **仿真环境** | NVIDIA Isaac Sim 5.0 + Isaac Lab 2.2.0 |
| **机械臂** | SO-101（6自由度 + 夹爪） |
| **遥操作** | 实体 SO101 Leader 主臂 → 控制 Isaac Lab 中的 SO101 Follower 仿真从臂 |
| **任务场景** | Kitchen 厨房场景，包含橘子（Orange）与盘子（Plate） |
| **数据流程** | 仿真采集 HDF5 → 转换为 LeRobot 数据集格式 → 训练策略（如 GR00T N1.5 / ACT） |

---

## 二、硬件准备清单

### 2.1 电脑配置要求

| 项目 | 最低配置 | 推荐配置 |
|------|---------|---------|
| 操作系统 | Ubuntu 22.04 | **Ubuntu 24.04 Desktop** |
| CPU | Intel i5 / AMD Ryzen 5 | Intel i5-12500TE 或更高 |
| 内存 | 32 GB | **64 GB** |
| GPU | NVIDIA RTX 3060 12GB | **NVIDIA RTX 4060 Ti 16GB** 或更高 |
| 显存 | 12 GB | **16 GB** |
| 硬盘 | 100 GB SSD | 200 GB NVMe SSD |

### 2.2 机械臂硬件（SO-101 套件）

- **Leader 主臂** × 1（遥操作用，7.4V STS3215 电机）
- **Follower 从臂** × 1（实际执行用，可选，若只做仿真可暂时不装）
- **总线伺服控制器** × 2（Waveshare 或 Feetech 控制器板）
- **5V/12V 电源**（Leader 用 5V，Follower 根据电机电压选择）
- **USB 数据线** × 2
- **3Pin 舵机线** 若干
- **摄像头**（可选，如 USB 摄像头、Realsense、手机虚拟摄像头）

### 2.3 组装与电机配置

参考 LeRobot 官方 SO-101 组装教程：  
https://github.com/huggingface/lerobot/blob/main/examples/12_use_so101.md

**Leader 臂电机减速比配置**：

| 关节 | 电机型号 | 减速比 |
|------|---------|--------|
| 基座旋转（Shoulder Pan） | STS3215 | 1:191 |
| 肩关节（Shoulder Lift） | STS3215 | 1:345 |
| 肘关节（Elbow） | STS3215 | 1:191 |
| 腕旋转（Wrist Roll） | STS3215 | 1:147 |
| 腕俯仰（Wrist Pitch） | STS3215 | 1:147 |
| 夹爪（Gripper） | STS3215 | 1:147 |

---

## 三、系统环境搭建（Ubuntu 24.04）

### 3.1 安装 NVIDIA 驱动与 CUDA

```bash
# 检查当前显卡信息
nvidia-smi

# 如果驱动未安装，建议安装最新稳定版（如 570 系列）
# 下载地址：https://www.nvidia.com/Download/index.aspx
# 或使用 Ubuntu 软件源安装
sudo apt update
sudo apt install nvidia-driver-570

# 重启后验证
nvidia-smi
# 应显示驱动版本（如 570.195.03）和 CUDA 版本（如 12.8）
```

**目标版本组合**：

| Isaac Sim | CUDA | Python | Isaac Lab |
|-----------|------|--------|-----------|
| 4.5.0 | 11.8 | 3.10 | 2.1.1 |
| **5.0.0** | **12.8** | **3.11** | **2.2.0** |

本指南采用 **Isaac Sim 5.0.0 + Isaac Lab 2.2.0 + CUDA 12.8 + Python 3.11** 的组合。

### 3.2 安装 Miniconda（如未安装）

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
# 安装完成后重启终端或执行
source ~/.bashrc
```

### 3.3 安装系统依赖

```bash
sudo apt-get update
sudo apt-get install -y cmake build-essential python3-dev pkg-config     libavformat-dev libavcodec-dev libavdevice-dev libavutil-dev     libswscale-dev libswresample-dev libavfilter-dev     libgl1-mesa-glx libglvnd0 libgl1 libglx0 libegl1 libxext6 libx11-6     v4l2loopback-dkms v4l-utils
```

---

## 四、Isaac Sim 与 Isaac Lab 安装

> **注意**：Isaac Sim 5.0 和 Isaac Lab 2.2 的安装较为复杂，建议预留 30GB 以上磁盘空间。以下提供 conda 安装方式。

### 4.1 创建 Isaac 环境（Python 3.11）

```bash
conda create -y -n isaaclab python=3.11
conda activate isaaclab
```

### 4.2 安装 Isaac Sim

```bash
# 通过 pip 安装 Isaac Sim 5.0
pip install isaacsim==5.0.0 --extra-index-url https://pypi.nvidia.com

# 或使用 Omniverse Launcher 安装（推荐图形化安装）
# 下载地址：https://www.nvidia.com/en-us/omniverse/
```

### 4.3 安装 Isaac Lab

```bash
# 克隆 Isaac Lab 仓库
git clone https://github.com/isaac-sim/IsaacLab.git
cd IsaacLab

# 切换到 2.2.0 版本
git checkout v2.2.0

# 安装 Isaac Lab
./isaaclab.sh --install
# 或
pip install -e source/isaaclab
```

### 4.4 验证 Isaac Lab

```bash
# 运行一个简单的测试
python -c "import isaaclab; print('Isaac Lab installed successfully')"
```

---

## 五、LeIsaac 项目安装

### 5.1 克隆仓库

```bash
# 退出到主目录
cd ~/

# 克隆 LeIsaac
git clone https://github.com/LightwheelAI/leisaac.git
cd leisaac

# 安装 LeIsaac 包
pip install -e source/leisaac

# 安装额外依赖
pip install pynput pyserial deepdiff feetech-servo-sdk
```

### 5.2 下载仿真资产（USD 文件与场景）

```bash
# 确保在 leisaac 目录下

# 下载 SO-101 Follower 的 USD 文件
wget https://github.com/LightwheelAI/leisaac/releases/download/v0.1.0/so101_follower.usd     -O assets/robots/so101_follower.usd

# 下载 Kitchen 厨房场景（含橘子）
wget https://github.com/LightwheelAI/leisaac/releases/download/v0.1.0/kitchen_with_orange.zip
unzip kitchen_with_orange.zip -d assets/scenes/
```

### 5.3 验证资产目录结构

```bash
tree assets
```

应显示如下结构：

```
assets/
├── robots/
│   └── so101_follower.usd
└── scenes/
    └── kitchen_with_orange/
        ├── scene.usd
        ├── assets/
        └── objects/
            ├── Orange001
            ├── Orange002
            ├── Orange003
            └── Plate
```

---

## 六、SO-101 硬件配置（Leader 主臂）

> **说明**：以下步骤用于配置实体 SO-101 Leader 主臂，用于遥操作。如果只做纯仿真测试（键盘控制），可跳过 6.1~6.4，直接进入第七步。

### 6.1 查找 USB 串口

将 Leader 主臂通过 USB 连接到电脑，执行：

```bash
ls /dev/ttyACM*
# 示例输出：/dev/ttyACM0 /dev/ttyACM1

# 给串口授权（Linux 需要）
sudo chmod 666 /dev/ttyACM0
```

### 6.2 配置电机 ID

Leader 主臂的 6 个电机需要分别设置 ID 为 1~6。

```bash
# 每次只连接一个电机到控制器板
# 设置 ID 为 1
python lerobot/scripts/configure_motor.py     --port /dev/ttyACM0     --brand feetech     --model sts3215     --baudrate 1000000     --ID 1

# 断开，连接下一个电机，设置 ID 为 2
# ...重复直到 ID 6
```

**Leader 臂电机 ID 对应表**：

| ID | 关节名称 | 英文名 |
|----|---------|--------|
| 1 | 基座旋转 | shoulder_pan |
| 2 | 肩关节 | shoulder_lift |
| 3 | 肘关节 | elbow_flex |
| 4 | 腕俯仰 | wrist_flex |
| 5 | 腕旋转 | wrist_roll |
| 6 | 夹爪 | gripper |

### 6.3 更新 LeRobot 配置中的端口

编辑 `lerobot` 源码中的配置（如果后续需要 LeRobot 环境）：

```python
# 文件路径：lerobot/common/robots/so101.py 或类似位置
# 修改 SO101RobotConfig 中的 leader_arms port
leader_arms: dict[str, MotorsBusConfig] = field(
    default_factory=lambda: {
        "main": FeetechMotorsBusConfig(
            port="/dev/ttyACM0",  # <-- 修改为你的实际端口
            motors={
                "shoulder_pan": [1, "sts3215"],
                "shoulder_lift": [2, "sts3215"],
                "elbow_flex": [3, "sts3215"],
                "wrist_flex": [4, "sts3215"],
                "wrist_roll": [5, "sts3215"],
                "gripper": [6, "sts3215"],
            },
        ),
    }
)
```

### 6.4 校准主臂（Leader）

```bash
# 先安装 LeRobot（见第七步），然后执行校准
python lerobot/scripts/control_robot.py     --robot.type=so101     --robot.cameras='{}'     --control.type=calibrate     --control.arms='["main_leader"]'
```

按提示依次将 Leader 臂移动到以下 4 个位置：
1. **中间姿态**（Middle）
2. **零姿态**（Zero）
3. **旋转姿态**（Rotated，右侧）
4. **休息姿态**（Rest）

---

## 七、LeRobot 环境搭建（Python 3.10）

> **重要**：LeRobot 需要 Python 3.10，而 Isaac Lab 使用 Python 3.11，因此需要**单独创建环境**。

### 7.1 创建 LeRobot 环境

```bash
conda create -y -n lerobot python=3.10
conda activate lerobot

# 安装 ffmpeg
conda install ffmpeg -c conda-forge
```

### 7.2 安装 LeRobot

```bash
# 克隆 LeRobot（如未克隆）
cd ~/
git clone https://github.com/huggingface/lerobot.git
cd lerobot

# 安装 LeRobot（含 Feetech 舵机支持）
pip install -e ".[feetech]"
```

### 7.3 验证安装

```bash
python -c "import lerobot; print('LeRobot installed successfully')"
```

---

## 八、数据采集：遥操作抓取橘子

### 8.1 使用实体 Leader 主臂遥操作（推荐）

确保当前在 **Isaac 环境**（`conda activate isaaclab`）：

```bash
cd ~/leisaac

python scripts/environments/teleoperation/teleop_se3_agent.py     --task=LeIsaac-SO101-PickOrange-v0     --teleop_device=so101leader     --port=/dev/ttyACM0     --num_envs=1     --device=cuda     --enable_cameras     --record     --dataset_file=./datasets/dataset.hdf5
```

**参数说明**：

| 参数 | 说明 |
|------|------|
| `--task` | 任务名称，固定为 `LeIsaac-SO101-PickOrange-v0` |
| `--teleop_device` | 遥操作设备：`so101leader` / `keyboard` / `gamepad` |
| `--port` | Leader 主臂串口，如 `/dev/ttyACM0` |
| `--num_envs` | 并行环境数，遥操作时设为 1 |
| `--device` | 计算设备：`cuda` 或 `cpu` |
| `--enable_cameras` | 启用相机传感器，采集视觉数据 |
| `--record` | 启用数据记录，保存为 HDF5 |
| `--dataset_file` | 数据集保存路径 |

### 8.2 使用键盘遥操作（无硬件时使用）

```bash
python scripts/environments/teleoperation/teleop_se3_agent.py     --task=LeIsaac-SO101-PickOrange-v0     --teleop_device=keyboard     --num_envs=1     --device=cuda     --enable_cameras     --record     --dataset_file=./datasets/dataset.hdf5
```

### 8.3 操作说明

启动后，Isaac Lab 仿真窗口会打开，显示厨房场景和 SO101 机械臂。

| 按键 | 功能 |
|------|------|
| **`b`** | **开始遥操作同步**（实体臂与仿真臂同步） |
| **`r`** | 放弃当前回合，标记为失败，重置环境 |
| **`n`** | 标记当前回合为成功，保存并进入下一回合 |
| `Ctrl + C` | 退出程序 |

**录制建议**：
- 至少录制 **3~5 个成功回合（episode）**
- 每个回合约 30~60 秒
- 尽量让机械臂从接近橘子 → 抓取 → 放到盘子的完整流程
- 保持摄像头画面清晰，橘子始终在视野内

---

## 九、数据转换：HDF5 → LeRobot 格式

### 9.1 切换到 LeRobot 环境

```bash
conda activate lerobot
cd ~/leisaac
```

### 9.2 修改转换脚本中的文件名

编辑 `scripts/convert/isaaclab2lerobot.py`，找到第 264 行左右：

```python
# 原代码
hdf5_files = [os.path.join(hdf5_root, 'dataset.hdf5')]

# 修改为你实际录制的文件名
hdf5_files = [os.path.join(hdf5_root, 'dataset.hdf5')]
```

### 9.3 执行转换

```bash
python scripts/convert/isaaclab2lerobot.py
```

### 9.4 转换结果位置

转换后的数据集默认保存在：

```bash
# 如果设置了 HF_LEROBOT_HOME
${HF_LEROBOT_HOME}/EverNorif/so101_test_orange_pick

# 否则在默认缓存目录
${HOME}/.cache/huggingface/lerobot/EverNorif/so101_test_orange_pick
```

**常见问题**：
- 如果出现 `FileExistsError`，删除或重命名 `so101_test_orange_pick` 目录后重新执行。

---

## 十、数据可视化

### 10.1 查看转换后的数据集

```bash
conda activate lerobot
cd ~/lerobot

python -m lerobot.scripts.visualize_dataset     --repo-id EverNorif/so101_test_orange_pick     --episode-index 0
```

这会打开可视化窗口，显示相机画面、关节状态、动作轨迹等。

### 10.2 使用 HTML 可视化（上传 Hugging Face 后）

```bash
python lerobot/scripts/visualize_dataset_html.py     --repo-id EverNorif/so101_test_orange_pick     --local-files-only 1
```

浏览器访问 `http://127.0.0.1:9090` 查看。

---

## 十一、策略训练（可选）

### 11.1 使用 LeRobot 训练 ACT 策略

```bash
conda activate lerobot
cd ~/lerobot

python lerobot/scripts/train.py     --dataset.repo_id=EverNorif/so101_test_orange_pick     --policy.type=act     --output_dir=outputs/train/act_so101_orange     --job_name=act_so101_orange     --policy.device=cuda     --wandb.enable=true
```

### 11.2 使用 NVIDIA GR00T N1.5 微调（高级）

参考 Seeed Studio Wiki：  
https://wiki.seeedstudio.com/control_robotic_arm_via_gr00t/

```bash
# 安装 Isaac-GR00T
git clone https://github.com/NVIDIA/Isaac-GR00T
cd Isaac-GR00T
conda create -n gr00t python=3.10
conda activate gr00t
pip install --upgrade setuptools
pip install -e .[base]
pip install --no-build-isolation flash-attn==2.7.1.post4

# 添加 modality.json 配置
cp ./getting_started/examples/so100_dualcam__modality.json     ~/.cache/huggingface/lerobot/EverNorif/so101_test_orange_pick/meta/modality.json

# 启动训练
python scripts/gr00t_finetune.py     --dataset-path ~/.cache/huggingface/lerobot/EverNorif/so101_test_orange_pick     --num-gpus 1     --output-dir ./so101-checkpoints     --max-steps 10000     --data-config so100_dualcam     --video-backend torchvision_av     --batch_size 2
```

---

## 十二、数据集回放（验证）

在 Isaac Lab 环境中回放采集的数据：

```bash
conda activate isaaclab
cd ~/leisaac

python scripts/environments/teleoperation/replay.py     --task=LeIsaac-SO101-PickOrange-v0     --num_envs=1     --device=cuda     --enable_cameras     --dataset_file=./datasets/dataset.hdf5     --episode_index=0
```

---

## 十三、自动数据生成（进阶，无需遥操作）

如果不想手动遥操作，可使用 **so101-autogen** 自动生成抓取数据：

```bash
git clone https://github.com/haoran1062/so101-autogen.git
cd so101-autogen
```

该项目通过 IK 控制和状态机自动生成抓取橘子的轨迹，无需人工干预。

---

## 十四、常见问题排查

### Q1: Isaac Lab 启动报错 `CUDA out of memory`

**解决**：
```bash
# 减少并行环境数
--num_envs=1

# 或使用 CPU 渲染（较慢）
--device=cpu
```

### Q2: 串口权限不足（Linux）

```bash
sudo usermod -aG dialout $USER
# 重新登录后生效
sudo chmod 666 /dev/ttyACM0
```

### Q3: LeRobot 安装报错 `libsvtav1` 不支持

```bash
conda install ffmpeg=7.1.1 -c conda-forge
```

### Q4: 数据转换时提示 `FileExistsError`

```bash
rm -rf ~/.cache/huggingface/lerobot/EverNorif/so101_test_orange_pick
# 然后重新执行转换脚本
```

### Q5: 仿真中机械臂抖动或穿模

- 检查 Isaac Sim 版本是否与 Isaac Lab 版本匹配
- 降低仿真步长或调整物理参数
- 确保 `so101_follower.usd` 文件正确下载

### Q6: 键盘按键在录制时无响应

```bash
# 确保 DISPLAY 环境变量已设置
echo $DISPLAY
# 如果为空，执行
export DISPLAY=:1
```

---

## 十五、相关资源链接

| 资源 | 链接 |
|------|------|
| **LeIsaac 主仓库** | https://github.com/LightwheelAI/leisaac |
| **LeRobot 官方仓库** | https://github.com/huggingface/lerobot |
| **SO-101 组装教程** | https://github.com/huggingface/lerobot/blob/main/examples/12_use_so101.md |
| **NVIDIA Isaac Lab** | https://github.com/isaac-sim/IsaacLab |
| **NVIDIA Sim-to-Real Workshop** | https://github.com/isaac-sim/Sim-to-Real-SO-101-Workshop |
| **腾讯云 LeIsaac 教程** | https://cloud.tencent.com/developer/article/2578981 |
| **Seeed Studio Wiki** | https://wiki.seeedstudio.com/simulate_soarm101_by_leisaac/ |
| **自动数据生成** | https://github.com/haoran1062/so101-autogen |

---

## 十六、快速命令速查表

```bash
# ===== 环境切换 =====
conda activate isaaclab    # Isaac Lab 环境（Python 3.11）
conda activate lerobot     # LeRobot 环境（Python 3.10）

# ===== 数据采集 =====
cd ~/leisaac
python scripts/environments/teleoperation/teleop_se3_agent.py     --task=LeIsaac-SO101-PickOrange-v0     --teleop_device=so101leader     --port=/dev/ttyACM0     --num_envs=1 --device=cuda --enable_cameras --record     --dataset_file=./datasets/dataset.hdf5

# ===== 数据转换 =====
conda activate lerobot
cd ~/leisaac
python scripts/convert/isaaclab2lerobot.py

# ===== 可视化 =====
conda activate lerobot
cd ~/lerobot
python -m lerobot.scripts.visualize_dataset     --repo-id EverNorif/so101_test_orange_pick --episode-index 0

# ===== 训练 ACT =====
python lerobot/scripts/train.py     --dataset.repo_id=EverNorif/so101_test_orange_pick     --policy.type=act --output_dir=outputs/train/act_orange     --job_name=act_orange --policy.device=cuda
```

---

> **作者备注**：本指南基于 LeIsaac v0.4.0、Isaac Lab 2.2.0、LeRobot 最新主分支整理。如遇版本更新导致命令变化，请以官方仓库 README 为准。建议在部署前确认各仓库的最新版本说明。
