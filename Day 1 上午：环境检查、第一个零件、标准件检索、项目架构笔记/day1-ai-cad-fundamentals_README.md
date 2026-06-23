# Day 1 上午：环境搭建、原理与第一个零件

> 目标：跑通 text-to-cad 完整环境，生成并验证第一个参数化零件，理解 Text-to-CAD 的技术路线与项目架构。

## 前置要求

- 已完成 `setup.md` 中的所有安装步骤
- Node.js 18+、Python 3.10+、LLM API Key 已配置
- （建议）FreeCAD 已安装，用于尺寸验证

## 任务清单

### Lab 01：环境检查（15 分钟）

```bash
# 检查版本
node -v
python -V
npx skills list | grep cad

# 检查 API 连通性
python -c "import openai; print('API OK')"
```

**验收标准**：`npx skills list` 能显示 `cad`、`cad-viewer` 等 Skill。

### Lab 02：生成第一个零件（30 分钟）

```bash
# 创建今日工作目录
mkdir -p day1-ai-cad-fundamentals/lab02-first-cad
cd day1-ai-cad-fundamentals/lab02-first-cad

# 生成 20mm 立方体
npx skills run cad --prompt "Create a 20 mm cube" --output ./cube.step

# 导出 STL
npx skills run cad --input ./cube.step --export stl --output ./cube.stl

# 启动本地查看器
npx skills run cad-viewer --file ./cube.step
```

**验收标准**：浏览器打开 `http://localhost:8080` 能看到立方体；`cube.step` 和 `cube.stl` 文件存在且大小 > 0。

### Lab 03：标准件检索（15 分钟）

```bash
# 检索 M6 内六角圆柱头螺钉
npx skills run step.parts --query "M6 hex socket head screw" --standard ISO --output ./m6_screw.step

# 检索 6204 深沟球轴承
npx skills run step.parts --query "6204 deep groove ball bearing" --standard ISO --output ./bearing_6204.step
```

**验收标准**：成功输出 `.step` 文件，在 FreeCAD 中可打开并测量。

### Lab 04：项目架构理解（30 分钟）

阅读 [earthtojake/text-to-cad README](https://github.com/earthtojake/text-to-cad)，完成以下笔记：

1. 列出所有 11 个 Skill 的名称与功能
2. 画出数据流图：文本 → [?] → STEP → [?] → STL → [?] → G-code → [?] → 打印
3. 思考：为什么项目选择 STEP 作为主输出格式，而不是 STL 或 OBJ？

**交付物**：`lab04-project-overview.md`（包含上述 3 个问题的答案）

## 今日交付物

| 文件 | 位置 | 说明 |
|---|---|---|
| `cube.step` | `lab02-first-cad/` | 第一个生成的零件 |
| `cube.stl` | `lab02-first-cad/` | 导出用于打印的网格 |
| `m6_screw.step` | `lab03-std-parts/` | 标准件检索示例 |
| `lab04-project-overview.md` | `lab04-project/` | 项目架构笔记 |
| 环境截图 | `lab01-environment/` | 证明 Viewer 正常运行 |

## 常见问题

**Q：API 调用失败？**  
A：检查 `.env` 文件是否在项目根目录，或环境变量是否已 export。可用 `echo $OPENAI_API_KEY` 验证。

**Q：生成的零件尺寸不对？**  
A：text-to-cad 默认单位是 mm，但 LLM 可能误解。在 Prompt 中明确写 "20 mm cube" 而不是 "20 cube"。

**Q：FreeCAD 打不开 STEP？**  
A：确保 FreeCAD 版本 ≥ 1.0。旧版可能不支持某些 AP242 特性。
