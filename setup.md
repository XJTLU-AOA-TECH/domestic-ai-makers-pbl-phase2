# 环境配置指南

## 系统要求

- **OS**: Windows 11 (WSL2 推荐) / macOS 14+ / Ubuntu 22.04+
- **CPU**: 4 核+
- **RAM**: 16GB+
- **GPU**: NVIDIA GTX 1060+（本地 LLM 推理可选，云端 API 不需要）
- **磁盘**: 10GB 可用空间

## 第一步：安装 Node.js 18+

```bash
# macOS
brew install node

# Ubuntu
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Windows: 下载 https://nodejs.org/dist/v20.x.x/node-v20.x.x-x64.msi

# 验证
node -v   # v20.x.x
npm -v    # 10.x.x
```

## 第二步：安装 Python 3.10+

```bash
# 推荐用 conda/miniconda
conda create -n text2cad python=3.11
conda activate text2cad

# 安装基础包
pip install -r requirements.txt
```

`requirements.txt`:
```
requests>=2.31.0
pydantic>=2.0.0
jinja2>=3.1.0
openai>=1.0.0
python-dotenv>=1.0.0
pytest>=7.4.0
numpy>=1.24.0
```

## 第三步：安装 text-to-cad Skills

```bash
# 全局安装 Skills CLI
npm install -g @anthropic-ai/skills

# 安装 CAD Skills
npx skills install earthtojake/text-to-cad

# 验证安装
npx skills list | grep cad
```

## 第四步：配置 LLM API

### 方案 A：云端 API（推荐，快速开始）

```bash
# 创建 .env 文件
cp resources/api-configs/.env.example .env

# 编辑 .env，填入至少一个 Key
OPENAI_API_KEY=sk-...
# 或
ANTHROPIC_API_KEY=sk-ant-...
```

### 方案 B：本地 Ollama（隐私优先，无需联网）

```bash
# 安装 Ollama
curl -fsSL https://ollama.com/install.sh | sh

# 拉取代码生成模型（推荐 codellama 或 qwen-coder）
ollama pull codellama:13b
ollama pull qwen2.5-coder:14b

# 配置本地端点
export OLLAMA_HOST="http://localhost:11434"
```

## 第五步：安装 CAD 查看器与工具（强烈推荐）

```bash
# FreeCAD（用于 STEP 验证、尺寸测量、手动编辑）
# macOS
brew install --cask freecad

# Ubuntu
sudo apt-get install freecad

# Windows: https://www.freecad.org/downloads.php

# Bambu Studio（用于理解切片参数、预览 G-code）
# 官网下载: https://bambulab.com/en/download/studio
```

## 第六步：验证完整链路

```bash
# 1. 创建测试目录
mkdir -p ~/text2cad-lab && cd ~/text2cad-lab

# 2. 生成零件
npx skills run cad \
  --prompt "Create a 50 mm cube with a 20 mm cylindrical hole through the center" \
  --output ./test_part.step

# 3. 导出 STL
npx skills run cad --input ./test_part.step --export stl --output ./test_part.stl

# 4. 启动本地查看器
npx skills run cad-viewer --file ./test_part.step

# 5. 标准件检索
npx skills run step.parts --query "M6 hex socket head screw" --standard ISO

echo "✅ 环境验证完成！"
```

## 故障排除

### 问题：Skills 安装失败或卡住

```bash
# 清除缓存重试
rm -rf ~/.skills/cache
npx skills install earthtojake/text-to-cad --force
```

### 问题：CAD Viewer 无法启动或端口冲突

```bash
# 检查端口占用
lsof -i :8080

# 指定其他端口
npx skills run cad-viewer --file ./part.step --port 8888
```

### 问题：生成模型尺寸偏差大

- 检查 Prompt 中是否明确指定单位（mm）
- 在 FreeCAD 中打开 STEP，用 Part → Measure 工具验证
- 向 Agent 反馈修正："The hole should be 20 mm diameter, not radius"
- 使用迭代策略：先粗生成 → 测量 → 追加编辑指令

### 问题：Bambu Lab 连接失败（Day 2 上午）

- 确保电脑与打印机在同一局域网（建议 5GHz Wi-Fi 或网线）
- 检查打印机设置：开启局域网连接（LAN Mode）
- 防火墙需开放端口 322（Bambu 局域网协议）
- 使用 dry-run 先行验证 G-code 格式

### 问题：URDF 在 Gazebo 中无法加载（Day 2 下午）

- 检查 mesh 文件路径是否为相对路径或绝对路径
- 确保碰撞几何（collision）简化，避免过于复杂
- 验证 joint 的 axis 和 limit 是否合理（避免 limit 为 0）

## Docker 环境（可选，高级用户）

```dockerfile
FROM node:20-slim
RUN apt-get update && apt-get install -y python3 python3-pip freecad
RUN pip3 install pydantic jinja2 openai python-dotenv
RUN npm install -g @anthropic-ai/skills
RUN npx skills install earthtojake/text-to-cad
WORKDIR /workspace
```

```bash
docker build -t text2cad-workshop .
docker run -it -v $(pwd):/workspace -p 8080:8080 text2cad-workshop
```

## 下一步

环境就绪后，进入 `day1-ai-cad-fundamentals/README.md` 开始第一天课程。
