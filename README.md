# AI × CAD：Text-to-CAD 全链路实战工作坊

> 基于 [earthtojake/text-to-cad](https://github.com/earthtojake/text-to-cad) 开源项目的两天密集课程

## 课程信息

- **时长**：2天（每天 6 小时）
- **对象**：本科生 / 研究生 / 对AI+制造感兴趣的工程师
- **前置知识**：基础Python、了解3D建模概念、有GitHub账号
- **硬件要求**：笔记本电脑（建议带独显）、Node.js 18+ 和 Python 3.10+

## 仓库结构

```
ai-cad-workshop/
├── README.md
├── syllabus.md
├── setup.md
├── day01/
│   ├── slides/
│   └── labs/
├── day02/
│   ├── slides/
│   └── labs/
├── examples/
└── resources.md
```

## 快速开始

```bash
# 安装 text-to-cad 技能库
npx skills install earthtojake/text-to-cad

# 验证安装
npx skills run cad --prompt "Create a 20mm cube" --output ./test.step
npx skills run cad-viewer --file ./test.step
```

## Day 1：AI建模革命

| 时间 | 主题 | 内容 |
|------|------|------|
| 上午 | Text-to-CAD 原理 | 技术路线对比、项目架构、环境搭建 |
| 下午 | 基础实战 | 矩形块、法兰盘、L支架生成；Prompt工程；10个Benchmark |

## Day 2：制造闭环

| 时间 | 主题 | 内容 |
|------|------|------|
| 上午 | 制造链路 | STEP→STL→G-code→打印全流程；Bambu Lab集成 |
| 下午 | 机器人自动化 | URDF/SRDF/SDF生成与仿真；综合大作业 |

## 核心技能清单

| Skill | 功能 | 输出格式 |
|-------|------|----------|
| CAD | 文本/图像生成3D模型 | STEP, STL, 3MF, GLB |
| CAD Viewer | 本地浏览器预览 | Web |
| step.parts | 标准件检索 | STEP |
| DXF | 2D工程图/切割图 | DXF |
| URDF | 机器人结构描述 | URDF |
| SRDF | MoveIt规划配置 | SRDF |
| SDF | 仿真世界 | SDF |
| G-code | 切片 | G-code |
| Bambu Labs | 打印控制 | - |
| Implicit CAD | 隐式建模（实验性） | GLSL → STL |

## 综合大作业

**任务**：设计一个桌面级机器人夹具系统

1. 用自然语言生成 ≥3 个零件
2. 生成装配体 STEP 并检查干涉
3. 输出 STL 并切片生成 G-code
4. （选做）生成 URDF 并在 Gazebo 中仿真抓取
5. （选做）用 Implicit CAD 设计有机形态外壳

## 资源链接

- [earthtojake/text-to-cad](https://github.com/earthtojake/text-to-cad)
- [Text2CAD Paper](https://sadilkhan.github.io/text2cad-project/)
- [awesome-cad](https://github.com/mlightcad/awesome-cad)

## License

CC BY-NC-SA 4.0
