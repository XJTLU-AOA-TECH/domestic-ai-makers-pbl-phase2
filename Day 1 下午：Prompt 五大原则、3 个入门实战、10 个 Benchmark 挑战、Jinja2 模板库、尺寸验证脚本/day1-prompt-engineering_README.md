# Day 1 下午：Prompt 工程与 10 个 Benchmark

> 目标：掌握工业级 Prompt 编写方法，复现 10 个官方 Benchmark 中的至少 3 个，建立可复用的 Prompt 模板库。

## 前置要求

- 已完成 Day 1 上午所有 Lab
- 理解 Text-to-CAD 的基本调用方式
- FreeCAD 已安装，用于尺寸验证

## 核心原则

### Prompt 五大原则

1. **精确性**：使用数值 + 单位，避免相对描述（大/小/厚/薄）
2. **方向性**：明确 vertical / horizontal / along X / top / bottom
3. **层级性**：先主体后细节 — base → holes → fillets/chamfers
4. **约束性**：使用 concentric / centered / symmetric / through / blind
5. **可验证性**：生成后必须能用 FreeCAD 测量，误差 ≤ 2%

### 反模式 vs 正模式

| 反模式 ❌ | 正模式 ✅ |
|---|---|
| 一个大一点的圆盘 | 直径 80 mm、厚度 10 mm 的圆盘 |
| 打几个孔 | 6 个直径 6 mm 的通孔，均布在直径 60 mm 的圆周上 |
| 边缘圆滑一点 | 所有外边缘倒圆角 R2 mm |
| 做一个支架 | 创建一个 L 型支架，底板 100×60×10 mm，立板 100×80×10 mm，配 2 个三角加强筋 |

## 任务清单

### Lab 05：Prompt 基础（30 分钟）

完成以下 3 个入门零件，记录 Prompt 和生成时间：

```bash
# Lab 1：矩形校准块
npx skills run cad --prompt "Create a 100 x 60 x 20 mm block with four 8 mm vertical through-holes at the corners. Add only a 2 mm chamfer on the top outer perimeter." --output ./lab05_block.step

# Lab 2：圆形法兰盘
npx skills run cad --prompt "Create an 80 mm diameter, 10 mm thick circular flange with a 30 mm central through-bore. Add six 6 mm through-holes on a 60 mm bolt circle and fillet the outside circular edges." --output ./lab05_flange.step

# Lab 3：L 型支架
npx skills run cad --prompt "Create an L-bracket from a base plate and rear vertical plate. Add vertical base holes, horizontal back-plate holes, two triangular gussets, and a filleted base/back transition." --output ./lab05_lbracket.step
```

**验收标准**：每个零件在 FreeCAD 中可打开，关键尺寸误差 ≤ 2%。

### Lab 06-08：10 个 Benchmark 挑战（90 分钟）

从以下 10 个官方 Benchmark 中**自选 3 个**完成（建议 1 简单 + 1 中等 + 1 困难）：

| 编号 | 名称 | 难度 | 考察点 |
|---|---|---|---|
| 01 | 矩形校准块 | ⭐ | 尺寸精度、孔阵列、倒角 |
| 02 | 圆形法兰盘 | ⭐ | 圆周阵列、配合尺寸 |
| 03 | L 型支架 | ⭐⭐ | 多方向特征、加强筋 |
| 04 | 阶梯轴 | ⭐⭐ | 旋转体、键槽、端部倒角 |
| 05 | 电子外壳 | ⭐⭐ | 薄壁、空心、内部支柱 |
| 06 | 航空叉形支架 | ⭐⭐⭐ | 复杂轮廓、减重孔 |
| 07 | 径向发动机气缸 | ⭐⭐⭐ | 散热片阵列、曲面 |
| 08 | 离心叶轮 | ⭐⭐⭐ | 旋转阵列、叶片扭曲 |
| 09 | 螺旋楼梯 | ⭐⭐⭐⭐ | 扫掠/放样、helical 路径 |
| 10 | 行星齿轮组 | ⭐⭐⭐⭐⭐ | 装配体、干涉检查 |

**要求**：
- 记录每个 Benchmark 的完整 Prompt（保存为 `benchmark_0N_prompt.txt`）
- 记录生成时间（保存为 `benchmark_0N_log.md`）
- 在 FreeCAD 中测量关键尺寸，填写 `benchmark_0N_measurements.csv`
- 计算误差率，分析误差来源

**交付物**：`benchmark_0{1-10}/` 目录（每个含 .step + prompt.txt + log.md + measurements.csv）

### Lab 09：Prompt 模板库（30 分钟）

将完成的零件按类型分类，编写 Jinja2 模板：

```
prompt_templates/
├── plate_class/          # 板类零件（底座、盖板、法兰）
│   └── template.jinja2
├── shaft_class/          # 轴类零件（阶梯轴、传动轴）
│   └── template.jinja2
├── housing_class/        # 壳体类（电子外壳、防护罩）
│   └── template.jinja2
└── bracket_class/        # 支架类（L 型、叉形、悬臂）
    └── template.jinja2
```

每个模板应包含：
- 变量定义（尺寸、孔径、数量、材料厚度）
- System Prompt（角色设定：你是一个参数化 CAD 工程师）
- Few-shot 示例（1 个完整案例）
- 输出格式要求（STEP + 可选 STL）

**交付物**：`prompt_templates/` 目录 + `template_usage_guide.md`

### Lab 10：尺寸验证与误差分析（30 分钟）

编写 Python 脚本 `measure_step.py`，使用 FreeCAD API 批量测量：

```python
# 示例：测量边界框尺寸
import FreeCAD
import Part

doc = FreeCAD.newDocument()
Part.insert("./lab05_block.step", doc.Name)
shape = doc.Objects[0].Shape
bbox = shape.BoundBox
print(f"L={bbox.XLength:.2f}, W={bbox.YLength:.2f}, H={bbox.ZLength:.2f}")
```

**要求**：
- 能自动测量：边界框长/宽/高、孔径（通过识别圆柱面）、板厚
- 输出 CSV 对比表：设计值 | 实测值 | 误差% | 是否合格（≤2%）
- 对不合格项给出 Prompt 优化建议

**交付物**：`measure_step.py` + `error_analysis.md`

## 今日交付物汇总

| 文件/目录 | 位置 | 验收标准 |
|---|---|---|
| `lab05_block.step` 等 | `lab05-prompt-basics/` | 3 个零件，FreeCAD 可测 |
| `benchmark_0{1-10}/` | `lab06-08-benchmarks/` | 自选 3 个，含 prompt + log + 测量 |
| `prompt_templates/` | `lab09-templates/` | 4 大类 Jinja2 模板 |
| `measure_step.py` | `lab10-evaluation/` | 能运行，输出 CSV 对比表 |
| `error_analysis.md` | `lab10-evaluation/` | 分析误差来源，给出优化建议 |

## 进阶挑战

- 完成 5 个及以上 Benchmark（加分）
- 尝试用图像输入生成 CAD（`--image ./sketch.png`）（加分）
- 编写自动化评测脚本：批量运行 10 个 Benchmark，自动评分（大幅加分）
