# Day 2 下午：机器人、Implicit CAD 与综合大作业

> 目标：将 Text-to-CAD 能力扩展到机器人领域（URDF/SDF），了解前沿 Implicit CAD 技术，完成桌面夹具系统综合大作业。

## 前置要求

- 已完成 Day 2 上午所有 Lab
- 拥有至少 3 个合格的 STEP 零件和对应的 G-code
- （可选）安装 Gazebo 用于机器人仿真
- （可选）安装 ROS2 Humble 用于 MoveIt 规划

## 核心概念

### URDF 结构

```xml
<robot name="arm">
  <link name="base"> ... </link>
  <link name="shoulder"> ... </link>
  <joint name="base_to_shoulder" type="revolute">
    <parent link="base"/>
    <child link="shoulder"/>
    <axis xyz="0 0 1"/>
    <limit lower="-3.14" upper="3.14" effort="100" velocity="1.0"/>
  </joint>
</robot>
```

### 从 CAD 到 URDF 的工作流

```
自然语言描述 → URDF Skill → URDF 文件
                                    ↓
                              Gazebo SDF Skill
                                    ↓
                              Gazebo 仿真世界
                                    ↓
                              MoveIt SRDF Skill
                                    ↓
                              运动规划与碰撞检测
```

## 任务清单

### Lab 16：生成机器人 URDF（45 分钟）

```bash
# 生成六轴机械臂
npx skills run urdf --prompt "Create a 6-DOF articulated arm with 600mm reach,
  base rotation ±180°, shoulder ±90°, elbow ±135°, wrist roll/pitch/yaw.
  End-effector: parallel gripper with 50mm stroke. All links aluminum,
  joints revolute with position limits." --output ./robot_arm.urdf

# 生成 SRDF（MoveIt 配置）
npx skills run srdf --urdf ./robot_arm.urdf \
  --group "manipulator" \
  --end-effector "gripper" \
  --default-pose "home" \
  --output ./robot_arm.srdf

# 生成夹爪 URDF（单独）
npx skills run urdf --prompt "Create a parallel jaw gripper with 50mm stroke,
  20mm jaw width, M3 mounting holes on 30mm spacing." --output ./gripper.urdf
```

**验证要求**：
- 在 FreeCAD 或 ROS2 中加载 URDF，检查 link 和 joint 的层级关系
- 确认每个 joint 的 axis 和 limit 合理（limit 不为 0）
- 检查碰撞几何（collision）是否简化，避免过于复杂影响仿真速度

**交付物**：`urdf_robot/`（robot_arm.urdf + robot_arm.srdf + gripper.urdf）

### Lab 17：Gazebo 仿真世界搭建（30 分钟）

```bash
# 生成空仓库仿真世界，加入机械臂
npx skills run sdf --world "empty_warehouse" \
  --add-robot ./robot_arm.urdf \
  --pose "0 0 0.5 0 0 0" \
  --add-camera "overhead" --pose "2 2 3 -1.0 0.8 0" \
  --add-ground-plane \
  --output ./sim_world.sdf

# 启动 Gazebo（如已安装）
gazebo ./sim_world.sdf
```

**要求**：
- 仿真世界能正常加载，无模型丢失（红色错误）
- 机械臂能手动拖动关节运动（Gazebo 的 Joint Control 插件）
- 相机视角能俯视工作区

**交付物**：`sim_world.sdf` + Gazebo 加载截图 `gazebo_screenshot.png`

### Lab 18：Implicit CAD 实验（30 分钟）

```bash
# 生成 Voronoi 风格外壳
npx skills run implicit-cad --prompt "Create a Voronoi-style cell structure
  enclosure with 2mm wall thickness and 50mm overall diameter." \
  --output ./voronoi.glsl

# 浏览器实时预览（光线 marching 渲染）
npx skills run cad-viewer --file ./voronoi.glsl --mode implicit

# 导出为 STL（体素化转换）
npx skills run implicit-cad --input ./voronoi.glsl \
  --resolution 0.1mm \
  --export stl \
  --output ./voronoi.stl
```

**思考题**：
1. Implicit CAD 与传统 B-rep CAD 的本质区别是什么？
2. 为什么体素化分辨率会影响 STL 的精度与文件大小？
3. 在什么场景下 Implicit CAD 优于传统 CAD？（拓扑优化、晶格结构、有机形态）

**交付物**：`implicit_cad/`（voronoi.glsl + voronoi.stl + `implicit_cad_notes.md`）

### Lab 19：Agent 多技能编排（15 分钟）

设计一个复杂场景，观察 Agent 自动调用多个 Skill：

```bash
# 场景：设计一个机器人底盘，需要钣金切割件 + 3D 打印件 + 标准件 + URDF
# 用自然语言描述整个任务，Agent 应自动分解为：
# 1. CAD Skill 生成底盘主体
# 2. DXF Skill 生成切割图
# 3. step.parts 检索螺丝和轴承
# 4. URDF Skill 生成机器人描述
# 5. G-code Skill 切片打印件

npx skills run agent --prompt "Design a mobile robot chassis 200x150mm,
  with 4 wheel mounts, M3 mounting holes, and a central electronics bay.
  Output: chassis STEP, laser-cut DXF, wheel bearing parts list, and full URDF."
```

**交付物**：`agent_scenario.md`（记录 Agent 的 Skill 调用链、遇到的问题、优化建议）

### Lab 20：综合大作业 — 桌面级机器人夹具系统（90 分钟）

**需求描述**：
设计一个可 3D 打印的桌面夹具系统，用于固定直径 20-50 mm 的圆柱形工件（如电机、轴承、管件），适配标准工作台（M6 螺纹孔，孔距 50 mm）。

**必须完成（基础分）**：
1. 用自然语言生成 ≥3 个零件：
   - 底座（带 M6 安装孔，与工作台配合）
   - 活动夹爪（V 型槽，适配 20-50 mm 圆柱）
   - 连接板/导轨（连接底座与夹爪，允许调节位置）
2. 生成装配体 STEP，在 FreeCAD 中检查：
   - 零件间无干涉
   - 配合间隙 ≥ 0.2 mm（打印公差补偿）
   - 螺栓孔对齐
3. 输出所有零件的 STL，切片生成 G-code
4. 编写 `manufacturing_plan.md`：材料选择、打印方向、支撑策略、后处理步骤

**选做（加分项）**：
5. 生成 URDF，描述夹具作为机器人末端执行器（固定于机械臂法兰）
6. 用 Implicit CAD 设计一个有机形态的防护外壳（覆盖夹具顶部，防飞溅）
7. 编写自动化脚本：一键从文本描述生成全部零件 + 装配 + 切片

**提交要求**：
- GitHub 仓库（或压缩包）：`final_project/` 目录
- 5 分钟演示视频：展示从文本描述到 CAD 模型到切片预览的完整链路
- 设计报告 `design_report.md`（≥1500 字）：
  - 设计需求与约束分析
  - Prompt 迭代记录（至少 3 轮优化）
  - 尺寸验证结果（设计值 vs 实测值 vs 切片预览值）
  - 制造可行性分析（壁厚、悬垂、支撑、材料）
  - 遇到的问题与解决方案
  - 如果重做，会如何改进

**评分标准**：

| 维度 | 权重 | 说明 |
|---|---|---|
| 功能完整性 | 30% | 零件是否满足设计需求，装配是否合理 |
| 制造可行性 | 25% | 壁厚、悬垂、支撑、材料选择、公差补偿 |
| Prompt 质量 | 20% | 描述是否精确、可复现、迭代记录是否清晰 |
| 文档与视频 | 15% | README 是否完整，视频是否展示端到端链路 |
| 创新性 | 10% | URDF、Implicit CAD、自动化脚本等进阶功能 |

## 今日交付物汇总

| 文件/目录 | 位置 | 验收标准 |
|---|---|---|
| `urdf_robot/` | `lab16-urdf-generation/` | 机械臂 URDF + SRDF + 夹爪 URDF |
| `sim_world.sdf` | `lab17-sdf-simulation/` | Gazebo 可加载，无红色错误 |
| `implicit_cad/` | `lab18-implicit-cad/` | GLSL + STL + 思考题笔记 |
| `agent_scenario.md` | `lab19-agent-orchestration/` | 记录多 Skill 调用链 |
| `final_project/` | `lab20-final-project/` | 完整夹具系统 + 视频 + 报告 |

## 答辩指南

**5 分钟 Demo 结构建议**：

1. **30 秒**：介绍设计需求（夹什么、怎么夹、为什么这样设计）
2. **1 分钟**：展示 Prompt 迭代过程（从模糊到精确，至少 3 轮）
3. **1 分钟**：展示 CAD 模型（FreeCAD 中旋转、剖切、测量）
4. **1 分钟**：展示切片与制造计划（G-code 预览、支撑位置、材料选择）
5. **30 秒**：展示 URDF/Implicit CAD 等进阶成果（如有）
6. **30 秒**：总结收获与反思

**评委可能提问**：
- 如果打印后夹爪尺寸收缩了 0.5 mm，你的设计如何补偿？
- 如果要把夹具从 PLA 换成 ABS，需要修改哪些设计参数？
- 你的 Prompt 如果交给另一个同学，他能复现出一样的结果吗？为什么？
- 如果预算只有 50 元，你会如何简化这个夹具系统？
