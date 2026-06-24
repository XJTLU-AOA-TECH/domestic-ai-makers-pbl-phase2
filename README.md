# Text-to-CAD 实战：从 Prompt 到 Loop Engineering

> 用自然语言生成工程级 CAD 模型，用 Loop Engineering 让 AI 自主迭代到合格，最后用 Bambu Studio 切片直送 3D 打印机。
> 

---

## 这是什么？

本课程教学生用 AI 完成 "设计→制造" 完整链路：

```
文本描述 → Text-to-CAD → STEP → STL → Bambu Studio 切片 → G-code → 3D 打印
```

核心不是 "让 AI 画图"，而是 **Loop Engineering**——你设定验收标准（如"尺寸误差 ≤ 2%"），AI 自己循环生成→测量→修正→直到达标。

---

## 上午：AI 生成 CAD

### 课前科普
- 3D 建模：STEP（参数化/可编辑） vs STL（三角网格/只打印）
- Text-to-CAD：自然语言 → 工程文件
- Skills：给 Claude Code 装的 CAD 工具箱
- Loop Engineering：从 "发球机"（手动 Prompt）进化为 "裁判"（设定标准让 AI 自己跑）

### 5 个 Benchmark 零件

| # | 零件 | 方式 | 考验 |
|---|------|------|------|
| 01 | 矩形校准块（四孔） | 手动 Prompt | 尺寸精度 |
| 02 | 圆形法兰盘（螺栓孔） | 手动 Prompt | 圆周均布推理 |
| 03 | L 型支架（加强筋） | 手动 Prompt | 多特征空间关系 |
| 04 | 阶梯轴（键槽） | **Loop Engineering** | 尺寸自动迭代 |
| 05 | 电子外壳（支撑柱） | **Loop Engineering** | 结构完整性 |

前 3 个体验手动修正的痛点，后 2 个感受 Loop 的自动化。

---

## 下午：制造落地

### 切片科普
- 切片 = 把 3D 模型翻译成 G-code（打印机只懂 "走到哪、挤多少料"）
- Bambu Studio：导入 → 摆放 → 修复 → 参数 → 切片 → 预览 → 导出

### 制造约束（设计时就要考虑）
- 壁厚 ≥ 1.2mm（0.4mm nozzle × 3 层）
- 悬空角度 ≤ 45°，否则需支撑
- 孔径 ≥ 2mm（太小打不圆）
- 特征高度是层厚的整数倍
- 支撑不能封死内部空腔
- 最大平面朝下

---

## 快速开始

### 方式 A：Claude Code 一键部署（推荐）

打开 Claude Code，直接说：

```
帮我本地部署 earthtojake/text-to-cad，并用 Loop Engineering 方式运行。
要求：
1. 自动检查并安装 Node.js、npm、@anthropic-ai/skills
2. 安装 text-to-cad 插件
3. 写一个 loop_cad.py，实现：生成→测量→修正→直到达标（误差 ≤ 2%，最多 5 轮）
4. 最后跑一个测试验证整个 loop 是否通
```

### 方式 B：手动安装

```bash
# 1. 确认 Node.js 18+
node -v

# 2. 安装 Skills CLI 和 Text-to-CAD
npm install -g @anthropic-ai/skills
npx skills install earthtojake/text-to-cad

# 3. 验证
npx skills list | grep cad

# 4. 生成第一个零件
npx skills run cad --prompt "Create a 20 mm cube" --output ./test.step

# 5. 浏览器预览
npx skills run cad-viewer --file ./test.step
```

---

## 技术栈

| 层级 | 工具 |
|------|------|
| AI Agent | Claude Code / OpenAI Codex |
| Skills | `@anthropic-ai/skills` + `earthtojake/text-to-cad` |
| 测量验证 | FreeCAD |
| 切片软件 | Bambu Studio（基于 PrusaSlicer） |
| 制造终端 | Bambu Lab / FDM 打印机 |

---

## 资源

- [text-to-cad 源码](https://github.com/earthtojake/text-to-cad)
- [Text2CAD 论文](https://sadilkhan.github.io/text2cad-project/)（NeurIPS 2024）
- [Bambu Studio 下载](https://bambulab.com/en/download/studio)
- [FreeCAD](https://www.freecad.org/)

---

MIT © XJTLU AOA TECH
