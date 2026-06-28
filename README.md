# domestic-ai-makers-pbl-phase2

本仓库为西交利物浦大学 AOA Lab 的 PBL 课程资料，围绕 AI 辅助工程设计与制造展开，涵盖从文本生成 CAD 模型到机器人仿真与操控的多个环节。

---

## 课程目标

通过自然语言与 AI 工具链的配合，完成以下任务：
- 将文本描述转换为可编辑的 CAD 模型（STEP）及可打印的网格模型（STL）
- 对 AI 生成的模型进行自动测量与迭代修正（Loop Engineering）
- 使用切片软件生成 G-code 并输出至 3D 打印机
- 在 NVIDIA Isaac Lab 中对机器人动作进行物理仿真
- 使用 LeRobot 框架训练机械臂的端到端控制策略
- 操作桌面五轴 CNC 进行零件加工
- 使用 Blender 结合 AI 工具生成与修改 3D 资产

---

## 内容模块

| 模块 | 内容 | 主要产出 |
|:---|:---|:---|
| Text-to-CAD | 自然语言生成参数化 CAD 模型 | STEP / STL 文件 |
| Loop Engineering | 设定验收标准，由 AI 自动迭代生成-测量-修正 | 达标后的 CAD 模型 |
| 3D 打印制造 | Bambu Studio 切片、工艺参数设置、G-code 输出 | 可打印的 G-code 文件 |
| 机器人仿真 | Isaac Lab 环境搭建、SO101 机械臂远程操控 | 仿真与实机操控代码 |
| 机器人学习 | LeRobot 框架、Diffusion Policy 训练与部署 | 训练策略与实机运行 |
| 桌面 CNC | Xhorse 五轴机床操作、刀具路径与加工参数 | 加工完成的零件 |
| Blender + AI | Blender MCP 配置、AI 生成 3D 资产与手动精修 | 3D 模型文件 |

---

## 使用的工具

- **Claude Code** / OpenAI Codex：AI Agent 交互环境
- **@anthropic-ai/skills** + **earthtojake/text-to-cad**：自然语言生成 CAD
- **FreeCAD**：模型测量与验证
- **Bambu Studio**（基于 PrusaSlicer）：切片与 G-code 生成
- **NVIDIA Isaac Lab**：机器人物理仿真
- **Hugging Face LeRobot**：开源机器人学习框架
- **Xhorse / Xmachine**：桌面五轴 CNC 机床
- **Blender** + **Blender MCP**：3D 内容创作与 AI 联动


---

## 相关资源

- [text-to-cad](https://github.com/earthtojake/text-to-cad)
- [Isaac Lab](https://github.com/isaac-sim/IsaacLab)
- [LeRobot](https://github.com/huggingface/lerobot)
- [Bambu Studio](https://bambulab.com/en-us/download/studio)
- [FreeCAD](https://www.freecad.org)

---

MIT © XJTLU AOA Lab
