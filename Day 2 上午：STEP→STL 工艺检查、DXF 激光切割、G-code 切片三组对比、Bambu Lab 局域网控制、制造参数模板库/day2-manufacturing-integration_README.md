# Day 2 上午：制造链路 — 从 CAD 到实物

> 目标：掌握从 STEP 到 STL 到 G-code 到 Bambu Lab 打印的完整制造链路，理解 2D 工程图（DXF）与激光切割对接。

## 前置要求

- 已完成 Day 1 所有 Lab，拥有至少 3 个合格的 STEP 零件
- 安装 Bambu Studio（用于理解切片参数，无论是否有实体打印机）
- （可选）实验室有 Bambu Lab A1/P1P/X1C 打印机，且在同一局域网

## 核心概念

### 数据流

```
概念描述 → CAD Skill → STEP（可编辑）→ STL（网格）→ G-code（机器指令）→ Bambu Lab（实物）
                ↓
              DXF（2D切割图）→ SendCutSend / 激光切割机
```

### 格式对比

| 格式 | 用途 | 是否可编辑 | 制造环节 |
|---|---|---|---|
| STEP | 设计、归档、跨软件交换 | ✅ 参数化 | 机加工、注塑模具 |
| STL | 3D 打印 | ❌ 三角网格 | FDM / SLA / SLS |
| 3MF | 多材料/多颜色打印 | ⚠️ 有限 | FDM（AMS） |
| G-code | 打印机直接执行 | ❌ 机器指令 | FDM 打印 |
| DXF | 激光切割、线切割、钣金 | ✅ 2D 矢量 | 激光/水刀/冲床 |

### 切片关键参数

| 参数 | 典型值 | 影响 |
|---|---|---|
| 层高 | 0.2 mm（标准）/ 0.12 mm（精细） | 表面质量 vs 打印时间 |
| 壁厚 | 3 层（≈1.2 mm @ 0.4mm 喷嘴） | 强度、密封性 |
| 填充率 | 20%（标准）/ 100%（实心） | 重量、强度、耗材 |
| 支撑 | 树状支撑 / 常规支撑 | 悬垂 > 45° 必须加 |
| 材料 | PLA / PETG / ABS / ASA | 强度、耐温、收缩率 |
| 底板温度 | 60°C（PLA）/ 80°C（PETG） | 首层附着、翘曲 |

## 任务清单

### Lab 11：STEP → STL 与工艺检查（30 分钟）

将 Day 1 的 3 个零件全部导出 STL：

```bash
# 批量转换
for part in block flange lbracket; do
  npx skills run cad --input ./day1/${part}.step --export stl --output ./day2/${part}.stl
done
```

**工艺检查清单**：
- [ ] 壁厚 ≥ 1.2 mm（3 层壁 @ 0.4mm 喷嘴）
- [ ] 悬垂角度 ≤ 45°（否则需加支撑）
- [ ] 孔径 ≥ 2 mm（太小无法打印或需后处理）
- [ ] 底板平整，无明显倒扣（overhang）

**交付物**：`stl_files/`（3 个 STL）+ `design_checklist.md`

### Lab 12：DXF 生成与激光切割（30 分钟）

为 L 型支架生成 2D 切割图：

```bash
# 从 3D 模型投影生成 DXF
npx skills run dxf --from-cad ./lbracket.step --projection top --output ./lbracket_top.dxf

# 直接文本生成垫片
npx skills run dxf --prompt "Create a gasket profile for a 50mm pipe flange with 4 bolt holes on 65mm PCD and 3mm material thickness" --output ./gasket.dxf

# 钣金可行性检查
npx skills run sendcutsend --validate ./lbracket_top.dxf
```

**要求**：
- 理解 DXF 的图层：切割线（红色）、标注（蓝色）、中心线（黄色）
- 检查孔边距 ≥ 材料厚度（避免撕裂）
- 记录 SendCutSend 反馈的警告或建议

**交付物**：`dxf_files/`（≥2 个 DXF）+ `dxf_notes.md`

### Lab 13：G-code 切片与参数调优（45 分钟）

```bash
# 基础切片
npx skills run gcode --input ./flange.stl \
  --profile "Bambu Lab A1 0.4mm nozzle" \
  --material PLA \
  --infill 20% \
  --supports auto \
  --output ./flange_pla_20.gcode

# 精细切片（对比实验）
npx skills run gcode --input ./flange.stl \
  --profile "Bambu Lab A1 0.4mm nozzle" \
  --material PLA \
  --layer-height 0.12 \
  --infill 50% \
  --supports tree \
  --output ./flange_pla_fine.gcode

# 用 CAD Viewer 预览 G-code 路径
npx skills run cad-viewer --file ./flange_pla_20.gcode
```

**实验要求**：
- 同一零件做 3 组切片参数对比（标准/精细/高强度）
- 记录：预计打印时间、耗材重量、支撑体积
- 分析：哪种参数最适合功能零件？哪种适合展示件？

**交付物**：`gcode_files/`（≥3 个 G-code）+ `slicing_comparison.md`

### Lab 14：Bambu Lab 局域网控制（45 分钟）

```bash
# 1. 干运行验证
npx skills run bambu-labs --dry-run ./flange_pla_20.gcode

# 2. 上传文件到打印机（不启动）
npx skills run bambu-labs --upload ./flange_pla_20.gcode --printer "Bambu A1"

# 3. 启动打印（确认打印机就绪后）
npx skills run bambu-labs --upload ./flange_pla_20.gcode \
  --printer "Bambu A1" --start-print

# 4. 监控状态（可选，需要 MQTT 配置）
npx skills run bambu-labs --monitor --printer "Bambu A1" --duration 30m
```

**安全协议**：
1. 必须 dry-run 通过后才能上传
2. 上传后检查预览图，确认方向、支撑位置合理
3. 首层打印时必须在场观察
4. 发现 spaghetti / 翘曲立即暂停

**交付物**：`bambu_log.md`（记录 dry-run 结果、上传时间、打印状态）

### Lab 15：制造参数模板库（30 分钟）

编写 `print_profiles.json`，覆盖三种常用材料：

```json
{
  "profiles": {
    "PLA_standard": {
      "material": "PLA",
      "nozzle": "0.4mm",
      "layer_height": "0.2mm",
      "infill": 20,
      "wall_count": 3,
      "support": "tree",
      "bed_temp": 60,
      "nozzle_temp": 210,
      "use_case": "一般结构件、原型"
    },
    "PETG_functional": {
      "material": "PETG",
      "nozzle": "0.4mm",
      "layer_height": "0.2mm",
      "infill": 50,
      "wall_count": 4,
      "support": "normal",
      "bed_temp": 80,
      "nozzle_temp": 240,
      "use_case": "高韧性、耐水、户外"
    },
    "ABS_engineering": {
      "material": "ABS",
      "nozzle": "0.4mm",
      "layer_height": "0.2mm",
      "infill": 100,
      "wall_count": 5,
      "support": "tree",
      "bed_temp": 100,
      "nozzle_temp": 260,
      "enclosure": true,
      "use_case": "高强度、耐高温、需封闭仓"
    }
  }
}
```

**要求**：每个 profile 必须说明：适用场景、强度预期、注意事项（如 ABS 需封闭仓防翘曲）。

**交付物**：`print_profiles.json` + `profile_selection_guide.md`

## 今日交付物汇总

| 文件/目录 | 位置 | 验收标准 |
|---|---|---|
| `stl_files/` | `lab11-step-to-stl/` | 3 个 STL，通过工艺检查 |
| `dxf_files/` | `lab12-dxf-laser/` | ≥2 个 DXF，SendCutSend 无严重警告 |
| `gcode_files/` | `lab13-gcode-slicing/` | ≥3 组对比切片，含预览截图 |
| `bambu_log.md` | `lab14-bambu-control/` | dry-run 通过，有状态记录 |
| `print_profiles.json` | `lab15-manufacturing-log/` | 3 套完整参数，含使用场景说明 |

## 故障排除

**Q：G-code 切片失败？**  
A：检查 STL 是否水密（watertight），可用 FreeCAD 的 Mesh → Analyze → Check 功能修复。

**Q：Bambu Lab 连接超时？**  
A：确认打印机开启 LAN Mode；电脑与打印机同网段；关闭 VPN 或防火墙测试。

**Q：打印后尺寸收缩？**  
A：PLA 收缩率约 0.2-0.5%，PETG 约 0.4-0.8%，ABS 可达 1-2%。设计时在 CAD 中预留补偿量，或在切片软件中调整 XY 补偿。

**Q：支撑难以拆除？**  
A：改用树状支撑（tree support），减少接触面；提高支撑顶部 Z 距离；使用水溶性支撑材料（Bambu Support W / PVA）。
