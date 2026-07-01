# Text-to-CAD 从 Prompt 到 Loop：一条完整的学习路径

> 先试试一句话生成零件，再发现问题，最后用 Loop Engineering 让 AI 自己改到合格  
> 基于 [earthtojake/text-to-cad](https://github.com/earthtojake/text-to-cad)

---

## 先试试：一句话生成零件

Text-to-CAD = **AI 翻译官**。你说"我要一个带螺丝孔的 L 型支架"，它输出工程师能直接生产的 **STEP 文件**。

| 传统方式 | Text-to-CAD |
|---------|-------------|
| 打开 SolidWorks → 画草图 → 拉伸 → 打孔 → 导出（30 分钟） | 说一句"50mm L 支架，底座 2 个 M6 孔" → 拿到 STEP（30 秒） |

> **核心区别**：STEP 是真正的参数化 CAD，包含尺寸、公差、装配关系，可以修改。STL 只是三角网格，只能打印不能编辑。

**试试这个 Prompt**：

```bash
npx skills run cad   --prompt "Create a 50mm cube with a 20mm cylindrical hole through the center"   --output ./cube.step
```

生成了？打开看看。大概率你会发现：
- 孔径不是精确的 20mm，可能是 19.8 或 20.5
- 立方体边长可能偏了 1-2mm
- 孔的位置可能不在正中心

**问题出现了**：AI 听懂了你的意思，但执行有误差。

---

## 然后：手动修正的循环

传统做法是：你测量 → 写修正指令 → 再生成 → 再测量……

```
你: "生成一个 50mm 立方体，中心 20mm 通孔"
AI: 生成零件（孔径 19.8mm）
你: 用 FreeCAD 测量，发现孔小了
你: "孔应该是 20mm 直径，不是 19.8，请修正"
AI: 生成零件（孔径 20.2mm）
你: 还是不准，再改……
```

你变成了 **"发球机"**——不断给 AI 喂指令，AI 被动执行。citeweb_search:8#0

这很烦，对吧？尤其是要批量生成 10 个零件的时候。

---

## 最后：Loop Engineering——让 AI 自己循环改到合格

**Loop Engineering（循环工程）** 是 2026 年 AI 工程化的新范式。citeweb_search:8#5web_search:8#4

它的核心思想：**你设计一个循环，让 AI 自己判断、自己修正、自己停。**

| 层级 | 你做什么 | AI 做什么 |
|-----|---------|----------|
| **Prompt Engineering** | 写一句好指令 | 给一次答案 |
| **Harness Engineering** | 给 Agent 工具和环境 | 在约束内自主调用工具 |
| **Loop Engineering** | 设计"会自己下指令的系统" | 自主循环：行动→观察→推理→重来，直到达标 |

> **本质**：你不再是 AI 的"发球机"，而是设计一个 `while` 循环——AI 自己判断零件是否合格，不合格就自动改，改到对为止。citeweb_search:8#0

**Text-to-CAD 的 Loop 长什么样？**

```python
# 你写的验收标准
target = {"边长": 50, "孔径": 20, "误差": "≤2%"}

# AI 自己循环
for i in range(5):  # 最多 5 次
    step = generate_cad(prompt)      # 生成
    actual = measure(step)           # 测量
    error = compare(target, actual)  # 对比
    if error <= 2%:
        break  # 合格，停
    else:
        prompt = f"修正：{actual} 偏差 {error}%，请调整"
        # 继续下一轮
```

**你只需做一件事**：设定验收标准（如"尺寸误差 ≤ 2%"）。剩下的，AI 自己搞定。

---

## 一键部署（Claude Code）

打开 Claude Code，直接粘贴：

```
帮我本地部署 earthtojake/text-to-cad，并用 Loop Engineering 方式运行。

要求：
1. 自动检查并安装 Node.js、npm、@anthropic-ai/skills
2. 安装 text-to-cad 插件
3. 写一个 loop_cad.py，实现以下循环：
   - 接收零件描述（如"50mm 立方体，中心 20mm 通孔"）和验收标准（如"尺寸误差 ≤ 2%"）
   - 调用 text-to-cad 生成 STEP
   - 用 FreeCAD 或脚本测量关键尺寸
   - 对比目标尺寸，如果误差 > 2%，自动写修正指令重新生成
   - 循环最多 5 次，成功或达上限后停止
   - 输出最终零件 + 生成报告（几轮迭代、最终误差、修改记录）
4. 如果缺环境，告诉我怎么装，不要擅自装系统级的东西
5. 最后跑一个测试验证整个 loop 是否通
```

Claude Code 会自动完成全部部署和脚本编写。

**你可能需要手动做的**：如果系统没有 Node.js，Claude Code 会给你下载链接，装完重启它继续。

---

## Loop 的核心：六原语

设计一个 CAD Loop，需要这六个组件：citeweb_search:8#6

| 原语 | 在 Text-to-CAD 中的体现 |
|-----|----------------------|
| **指令（Instruction）** | "生成 50mm 立方体，中心 20mm 通孔，误差 ≤ 2%" |
| **工具（Tools）** | text-to-cad 生成、FreeCAD 测量、尺寸对比脚本 |
| **上下文（Context）** | 上一轮生成的 STEP 文件、测量结果、修正历史 |
| **记忆（Memory）** | 记录每轮迭代的目标尺寸、实际尺寸、修正指令 |
| **终止（Termination）** | 误差 ≤ 2% 或迭代达 5 次上限 |
| **评估（Evaluation）** | 尺寸对比脚本输出误差百分比 |

---

## 手动安装（备用）

如果 Claude Code 不可用：

```bash
# 1. 确认 Node.js 18+
node -v

# 2. 安装 Skills CLI
npm install -g @anthropic-ai/skills

# 3. 安装 Text-to-CAD
npx skills install earthtojake/text-to-cad

# 4. 验证
npx skills run cad --prompt "Create a 20mm cube" --output ./test.step
```

---

## 常用命令

```bash
# 生成零件
npx skills run cad --prompt "描述" --output ./零件.step

# 导出 STL（3D 打印用）
npx skills run cad --input ./零件.step --export stl --output ./零件.stl

# 浏览器预览
npx skills run cad-viewer --file ./零件.step

# 查标准件
npx skills run step.parts --query "M6 hex socket head screw" --standard ISO
```

---

## Prompt 技巧

越像工程图纸语言，结果越准：

```
"A rectangular mounting plate, 100mm x 60mm x 5mm, with 4 corner holes M4"
"L-bracket, 50mm x 50mm x 5mm, fillet radius 3mm, 2 holes on each leg"
"Cylindrical spacer, OD 20mm, ID 10mm, height 15mm"
```

> 带上单位（mm）、数量、位置关系。

---

## 制造闭环（可选）

```
文本描述 → STEP → STL → Bambu Studio 切片 → G-code → 3D 打印
         ↓
      DXF → 激光切割
         ↓
      URDF → 机器人仿真（Gazebo）
```

**第一步（文本→STEP）已经能解决 80% 的需求**。

---

## 资源

- [text-to-cad 源码](https://github.com/earthtojake/text-to-cad)
- [Text2CAD 论文](https://sadilkhan.github.io/text2cad-project/)（NeurIPS 2024）
- [Loop Engineering 详解](https://www.huxiu.com/article/4867297.html)citeweb_search:8#5
- [FreeCAD](https://www.freecad.org/)（免费 STEP 编辑器）
- [Bambu Studio](https://bambulab.com/en/download/studio)（切片软件）

---

MIT © XJTLU AOA TECH
