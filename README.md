# Text-to-CAD 全链路实战

> **Text-to-CAD Hands-On Workshop**  
> AI-PBL 课程 · 两天密集版 · 从自然语言到制造到机器人  
> 基于 [earthtojake/text-to-cad](https://github.com/earthtojake/text-to-cad) 开源项目

## 这是什么？

两天，从零开始走通 **Text-to-CAD 工程化的完整链路**。不是看论文，不是调 Demo——你自己动手生成零件、切片打印、控制机器人。

| 天 | 主题 | 你要做什么 |
|---|---|---|
| **Day 1 上午** | 环境搭建与原理 | 装环境、跑通第一个零件、理解 Text-to-CAD 技术路线 |
| **Day 1 下午** | Prompt 工程与 Benchmark | 写精确 Prompt、复现 10 个官方 Benchmark、建立模板库 |
| **Day 2 上午** | 制造链路 | STEP→STL→G-code→Bambu Lab 打印控制；DXF 激光切割 |
| **Day 2 下午** | 机器人与综合项目 | 生成 URDF/SDF 机械臂、Implicit CAD 实验、完成夹具系统设计 |

## 前置要求

- Node.js 18+ & Python 3.10+
- 一个 LLM API Key（OpenAI / Anthropic / 本地 Ollama）
- （Day 2 上午）Bambu Lab 3D 打印机 或 Bambu Studio 用于仿真切片
- （Day 2 下午）FreeCAD 用于验证尺寸；Gazebo（可选）用于机器人仿真

## 仓库结构

```
text2cad-workshop/
├── day1-ai-cad-fundamentals/           ← Day 1 上午：环境+原理+第一个零件
│   ├── lab01-environment/                # 环境检查与安装
│   ├── lab02-first-cad/                  # 生成第一个零件（20mm立方体）
│   ├── lab03-cad-viewer/                 # 本地浏览器预览
│   └── lab04-project-overview/           # 项目架构与技术路线
│
├── day1-prompt-engineering/              ← Day 1 下午：Prompt工程+Benchmark
│   ├── lab05-prompt-basics/              # Prompt 最佳实践
│   ├── lab06-benchmark-01-03/            # 矩形块/法兰盘/L支架
│   ├── lab07-benchmark-04-07/            # 阶梯轴/电子外壳/航空支架/发动机
│   ├── lab08-benchmark-08-10/            # 叶轮/螺旋楼梯/行星齿轮
│   ├── lab09-prompt-templates/           # 四大类模板库（板/轴/壳/支架）
│   └── lab10-evaluation/                 # 尺寸验证与误差分析
│
├── day2-manufacturing-integration/         ← Day 2 上午：制造链路
│   ├── lab11-step-to-stl/                # 格式转换与工艺检查
│   ├── lab12-dxf-laser/                  # 2D工程图与激光切割
│   ├── lab13-gcode-slicing/              # G-code 切片与参数调优
│   ├── lab14-bambu-control/              # Bambu Lab 局域网打印控制
│   └── lab15-manufacturing-log/          # 制造参数模板库
│
├── day2-robotics-advanced/               ← Day 2 下午：机器人+高级主题
│   ├── lab16-urdf-generation/            # 机械臂 URDF 生成
│   ├── lab17-sdf-simulation/             # Gazebo 仿真世界搭建
│   ├── lab18-implicit-cad/               # 隐式建模（实验性）
│   ├── lab19-agent-orchestration/        # 多技能 Agent 编排
│   └── lab20-final-project/              # 综合大作业：桌面夹具系统
│
├── resources/                            # 公共资源
│   ├── api-configs/                      # LLM API 配置模板
│   ├── print-profiles/                   # Bambu Lab 打印参数模板
│   ├── cad-standards/                    # 尺寸公差与制造规范
│   └── troubleshooting.md                # 故障排除手册
│
├── slides/                               # 课程课件
│   └── Text2CAD_Workshop_Detailed.pptx
│
├── README.md
├── syllabus.md
├── setup.md
├── requirements.txt
└── .gitignore
```

## 快速开始

```bash
# 1. 克隆仓库
git clone https://github.com/yourname/text2cad-workshop.git
cd text2cad-workshop

# 2. 按 setup.md 完成环境安装
# 3. 每天进入对应文件夹，先读 README.md
# 4. 按里面的步骤操作——可以不跑参考代码，但交付文件名和格式必须一致
# 5. 每一天的产物交给下一天，形成完整链路
```

## 课程交付物

| 阶段 | 交付物 | 验收标准 |
|---|---|---|
| Day 1 上午 | `cube.step` + `cube.stl` + 环境截图 | 能生成零件并在浏览器预览 |
| Day 1 下午 | `benchmark_report.md` + `prompt_templates/` | 完成 ≥3 个 Benchmark，尺寸误差 ≤ 2% |
| Day 2 上午 | `gcode_files/` + `dxf_files/` + `manufacturing_log.md` | 切片成功，参数合理，dry-run 通过 |
| Day 2 下午 | `urdf_robot/` + `final_project/` + 演示视频 | 机械臂可加载，夹具系统完整，视频 5 分钟 |

## 核心原则

1. **Mesh ≠ CAD** — 能打印不代表能制造，参数化模型才是工程语言
2. **Prompt 是设计意图** — 描述越精确，模型越准确；模糊描述 = 废品
3. **制造约束前置** — 设计时就要考虑壁厚、悬垂、支撑、材料收缩
4. **链路必须闭环** — 文本 → CAD → 切片 → 打印/仿真，缺一环就是 Demo

## 资源链接

- [earthtojake/text-to-cad](https://github.com/earthtojake/text-to-cad) — 本课程核心项目
- [Text2CAD Paper](https://sadilkhan.github.io/text2cad-project/) — NeurIPS 2024
- [Bambu Studio](https://bambulab.com/en/download/studio) — 切片引擎
- [FreeCAD](https://www.freecad.org/) — STEP 验证与编辑
- [Gazebo](https://gazebosim.org/) — 机器人仿真

## License

MIT © XJTLU AOA TECH
