# Text-to-CAD 全链路实战 — 教学大纲

## 课程定位

两天密集实战课程，面向已有 Python/Node.js 基础、希望掌握 AI 辅助参数化建模与制造闭环的学生与工程师。课程基于真实开源项目 earthtojake/text-to-cad，覆盖从自然语言描述到 3D 打印实物、再到机器人仿真的完整链路。

## 课程目标

完成本课程后，学员能够：
1. 理解 Text-to-CAD 三种技术路线（序列生成/代码生成/Agent 编排），并能在工程中正确选型
2. 熟练使用 earthtojake/text-to-cad 的 11 项 Skill，完成 CAD→Viewer→STL→G-code→打印的全流程
3. 编写工业级 Prompt，将自然语言精确转化为可制造的参数化模型（误差 ≤ 2%）
4. 独立完成制造链路：切片参数优化、Bambu Lab 局域网控制、DXF 激光切割对接
5. （进阶）生成机器人 URDF/SDF 描述文件，并在 Gazebo 中验证运动学
6. （进阶）使用 Implicit CAD 进行有机形态设计与实时渲染

## 评估方式

| 项目 | 占比 | 说明 |
|---|---|---|
| Day 1 上午 | 10% | 环境搭建 + 第一个零件生成与预览 |
| Day 1 下午 | 25% | 3 个 Benchmark 复现 + Prompt 模板库 + 尺寸验证报告 |
| Day 2 上午 | 25% | 制造链路：G-code 切片 + Bambu dry-run + DXF 生成 |
| Day 2 下午 | 20% | URDF 机械臂生成 + 仿真加载 + Implicit CAD 实验 |
| 综合大作业 | 15% | 桌面夹具系统：GitHub 仓库 + 5 分钟演示视频 + 设计报告 |
| 课堂参与 | 5% | 提问、互助、代码 Review |

## Day 1 详细安排

### 09:00-10:30 模块1：环境搭建与项目架构

**目标**：跑通第一个零件，理解 Text-to-CAD 为什么重要

**理论（45分钟）**
- CAD vs Mesh：为什么参数化模型不可替代（ISO 10303 STEP 标准）
- Text-to-CAD 技术谱系：
  - 序列生成：Text2CAD (NeurIPS'24) — Transformer 输出草图+拉伸序列
  - 代码生成：Text-to-CadQuery — LLM 写 Python 脚本
  - Agent 编排：earthtojake/text-to-cad — Skill-based 多工具协作
  - 多模态：CAD-MLLM — 图文混合输入
- earthtojake 项目架构：CAD / Viewer / step.parts / DXF / URDF / G-code / Bambu / Implicit

**实战（45分钟）**
- 安装 Skills CLI：`npm install -g @anthropic-ai/skills`
- 安装 text-to-cad：`npx skills install earthtojake/text-to-cad`
- 生成 20mm 立方体：`npx skills run cad --prompt "Create a 20 mm cube" --output ./cube.step`
- 导出 STL：`npx skills run cad --input ./cube.step --export stl --output ./cube.stl`
- 启动 Viewer：`npx skills run cad-viewer --file ./cube.step`
- 标准件检索：`npx skills run step.parts --query "M6 hex socket head screw" --standard ISO`

**交付物**：`cube.step` + `cube.stl` + 环境截图（Viewer 中显示）

### 10:45-12:00 模块2：Prompt 工程基础

**目标**：掌握从模糊描述到精确工程语言的转换

**理论（30分钟）**
- Prompt 反模式分析：'大一点的圆盘' vs '直径 80mm、厚度 10mm 的圆盘'
- 五大原则：精确性、方向性、层级性、约束性、可验证性
- 迭代策略：粗生成 → 测量验证 → 差异反馈 → 精修 Prompt

**实战（45分钟）**
- Lab 1：矩形校准块（100×60×20mm，4×φ8 通孔，顶部 2mm 倒角）
- Lab 2：圆形法兰盘（φ80×10mm，φ30 中心孔，6×φ6 螺栓孔均布 φ60）
- Lab 3：L 型支架（底板+立板+三角筋+多向孔+过渡圆角）
- 用 FreeCAD 测量关键尺寸，记录误差

**交付物**：`lab01_block.step` + `lab02_flange.step` + `lab03_lbracket.step` + 尺寸测量表

### 13:30-15:00 模块3：10 个 Benchmark 挑战

**目标**：覆盖从简单到复杂的全品类零件，建立 Prompt 模板库

**Benchmark 列表**
1. 矩形校准块 — 验证尺寸精度与孔阵列
2. 圆形法兰盘 — 验证圆周阵列与配合
3. L 型支架 — 验证多方向特征与加强筋
4. 阶梯轴 — 验证旋转体、键槽、端部倒角
5. 电子外壳 — 验证薄壁、空心、内部支柱
6. 航空叉形支架 — 验证复杂轮廓与减重孔
7. 径向发动机气缸 — 验证散热片阵列与曲面
8. 离心叶轮 — 验证旋转阵列与叶片扭曲
9. 螺旋楼梯 — 验证扫掠/放样与 helical 路径
10. 行星齿轮组 — 验证装配体与干涉检查（进阶）

**实战（90分钟）**
- 自选 3 个 Benchmark 完成（建议覆盖 1 简单 + 1 中等 + 1 困难）
- 记录每个零件的 Prompt、生成时间、FreeCAD 测量结果、误差分析
- 编写 Prompt 模板库：板类 / 轴类 / 壳体 / 支架 四大类

**交付物**：`benchmark_report.md` + `prompt_templates/`（Jinja2 格式）

### 15:15-17:00 模块4：尺寸验证与误差分析

**目标**：建立从"生成"到"验收"的工程化流程

**理论（30分钟）**
- 误差来源：LLM 理解偏差、单位遗漏、方向歧义、特征冲突
- 验收标准：关键尺寸误差 ≤ 2%，形位公差目视合格，壁厚 ≥ 1.2mm（打印约束）
- 自动化验证：Python + FreeCAD API 批量测量 STEP 文件

**实战（75分钟）**
- 用 FreeCAD 的 Part → Measure 工具测量所有生成零件
- 编写 `measure_step.py` 脚本，自动提取边界框、孔径、板厚
- 对比设计值 vs 实测值，生成误差分析表
- 根据误差反推 Prompt 优化方向

**交付物**：`measure_step.py` + `error_analysis.md` + 优化后的 Prompt 模板

## Day 2 详细安排

### 09:00-10:30 模块5：制造链路 — 从 STEP 到 G-code

**目标**：理解制造端的完整数据流

**理论（45分钟）**
- 格式转换链：STEP（设计）→ STL（打印）→ 3MF（多材料）→ G-code（机器指令）
- 工艺检查：SendCutSend 钣金可行性（壁厚、折弯半径、孔边距）
- 2D 工程图：DXF 生成策略（投影方向、比例、线型、标注）
- 切片原理：层高、壁厚、填充率、支撑、回抽、温度塔

**实战（45分钟）**
- 将 Day 1 的 3 个零件导出 STL
- 用 G-code Skill 切片：`npx skills run gcode --input ./flange.stl --profile "Bambu Lab A1 0.4mm" --material PLA --infill 20% --supports auto --output ./flange.gcode`
- 用 CAD Viewer 预览 G-code 路径
- 为 L 型支架生成 DXF 切割图并做 SendCutSend 验证

**交付物**：`stl_files/` + `gcode_files/` + `dxf_files/` + 切片参数记录

### 10:45-12:00 模块6：Bambu Lab 深度集成

**目标**：掌握局域网打印控制与异常处理

**理论（30分钟）**
- Bambu Lab 开放协议：局域网 API、MQTT 状态、FTP 文件传输
- 安全机制：dry-run → 上传 → 人工确认 → 启动
- AMS 多材料：料槽映射、换料塔、材料配置文件
- 常见故障： spaghetti（堵头/回抽不足）、翘曲（底板温度/ adhesion）、层间分离（温度/速度）

**实战（45分钟）**
- dry-run 验证：`npx skills run bambu-labs --dry-run ./flange.gcode`
- 上传文件：`npx skills run bambu-labs --upload ./flange.gcode --printer "Bambu A1"`
- 启动打印（如实验室有设备）：观察首层、调整 Z-offset
- 记录打印时间、材料用量、实际质量

**交付物**：`bambu_log.md` + `print_profiles.json`（PLA/PETG/ABS 三套参数）

### 13:30-15:00 模块7：机器人描述与仿真

**目标**：将 CAD 能力扩展到机器人领域

**理论（45分钟）**
- URDF：links、joints、limits、inertials、collision meshes
- SRDF：MoveIt 规划组、末端执行器、预设位姿（home/ready/pick）
- SDF：Gazebo 仿真世界、物理引擎、传感器插件、灯光
- 从 CAD 到 URDF：STEP → 简化 mesh → 链接到 link → 定义 joint 轴

**实战（45分钟）**
- 生成 6 轴机械臂 URDF：
  ```bash
  npx skills run urdf --prompt "Create a 6-DOF articulated arm with 600mm reach,
    base rotation ±180°, shoulder ±90°, elbow ±135°. End-effector: parallel gripper
    with 50mm stroke." --output ./robot_arm.urdf
  ```
- 生成 SRDF 配置：`npx skills run srdf --urdf ./robot_arm.urdf --group "manipulator" --end-effector "gripper"`
- 生成 SDF 仿真世界：`npx skills run sdf --world "empty_warehouse" --add-robot ./robot_arm.urdf --pose "0 0 0.5 0 0 0"`
- 在 Gazebo 中加载，验证关节运动范围

**交付物**：`urdf_robot/`（urdf + srdf + sdf）+ Gazebo 加载截图

### 15:15-16:30 模块8：Implicit CAD 与 Agent 编排

**目标**：了解前沿方向与多技能协作

**理论（30分钟）**
- Implicit CAD：GLSL SDF + 光线 marching，适合有机形态、拓扑优化、轻量化结构
- Agent 编排：多 Skill 流水线 — CAD → step.parts → DXF → SendCutSend → G-code → Bambu
- 场景：'设计一个机器人底盘，输出切割图和 3D 打印件，并生成 URDF'

**实战（45分钟）**
- 实验 Implicit CAD：`npx skills run implicit-cad --prompt "Create a Voronoi-style cell structure enclosure with 2mm wall thickness and 50mm diameter" --output ./voronoi.glsl`
- 在浏览器预览：`npx skills run cad-viewer --file ./voronoi.glsl --mode implicit`
- 导出 STL：`npx skills run implicit-cad --input ./voronoi.glsl --resolution 0.1mm --export stl`
- 设计一个 Agent 编排场景：用自然语言描述一个多零件任务，观察 Skill 自动调用链

**交付物**：`implicit_cad/` + `agent_scenario.md`

### 16:30-17:30 模块9：综合大作业答辩

**答辩要求**
- 5 分钟 Demo 视频或现场演示
- 必须展示：文本 → CAD → 切片/仿真 的完整链路
- 讲解设计决策：为什么这样写 Prompt、为什么选这些参数、遇到了什么问题

**评分标准**
- 功能完整性（30%）：零件是否满足设计需求
- 制造可行性（25%）：壁厚、悬垂、支撑、材料选择是否合理
- Prompt 质量（20%）：描述是否精确、可复现、迭代记录是否清晰
- 文档与视频（15%）：README 是否完整，视频是否展示端到端链路
- 创新性（10%）：是否使用 URDF/Implicit CAD 等进阶功能

## 前置知识

- Python 3.10+，熟悉 requests / asyncio / subprocess
- Node.js 18+，熟悉 npm / npx
- 基础 CAD 概念：草图、拉伸、切除、倒角、阵列、装配
- （加分）有 3D 打印经验或 FreeCAD / SolidWorks 使用经验
- （加分）有 ROS / Gazebo 基础

## 硬件与软件清单

| 物品 | 数量 | 用途 | 必需 |
|---|---|---|---|
| 笔记本电脑 | 1 | 开发环境 | 是 |
| Node.js 18+ | 1 | Skills CLI | 是 |
| Python 3.10+ | 1 | 脚本与验证 | 是 |
| LLM API Key | 1 | 模型调用 | 是 |
| FreeCAD 1.0+ | 1 | STEP 验证 | 强烈建议 |
| Bambu Studio | 1 | 切片理解 | 建议 |
| Bambu Lab 打印机 | 1 | 实物打印 | Day 2 上午可选 |
| M5Stack / ESP32 | 1 | 设备接入 | 本课程不涉及 |
| Gazebo | 1 | 机器人仿真 | Day 2 下午可选 |

## 推荐阅读

1. [Text2CAD: Generating Sequential CAD Models from Beginner-to-Expert Level Text Prompts](https://arxiv.org/abs/2411.15279) — NeurIPS 2024
2. [Text-to-CadQuery: A New Paradigm for CAD Generation](https://arxiv.org/abs/2505.06507) — 代码生成路线
3. [earthtojake/text-to-cad README](https://github.com/earthtojake/text-to-cad) — Skill 列表与 CLI 用法
4. [CadQuery Documentation](https://cadquery.readthedocs.io/) — 底层建模 API
5. [Bambu Studio Wiki](https://wiki.bambulab.com/en/software/bambu-studio) — 切片参数详解
