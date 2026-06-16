# Doubao Agent 完整部署方案

## 📅 文档版本
- 版本: v1.0
- 日期: 2026年6月17日
- 作者: Doubao Agent System

---

## 🎯 项目概述

本指南提供在个人服务器上完整还原 Doubao AI Agent 系统的详细步骤。包含系统架构、硬件要求、软件安装、配置优化等全部内容。

---

## 📋 目录

1. [系统架构概述](#1-系统架构概述)
2. [硬件要求](#2-硬件要求)
3. [系统准备](#3-系统准备)
4. [基础环境安装](#4-基础环境安装)
5. [AI 模型部署](#5-ai-模型部署)
6. [Agent 框架部署](#6-agent-框架部署)
7. [Skill 系统集成](#7-skill-系统集成)
8. [浏览器自动化](#8-浏览器自动化)
9. [开发环境服务](#9-开发环境服务)
10. [安全与隔离](#10-安全与隔离)
11. [快速启动脚本](#11-快速启动脚本)
12. [故障排查](#12-故障排查)

---

## 1. 系统架构概述

### 1.1 完整架构图

```
┌─────────────────────────────────────────────────────────────┐
│                    用户接入层                                │
│  Web UI (8080) / VNC (5900) / SSH / API                     │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│                   MainAgent 主控层                           │
│  对话理解 │ 任务调度 │ Skill 路由 │ 工具编排                  │
│  Open Interpreter / LangChain / Custom Agent                 │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│                    模型推理层                                │
│  Ollama / vLLM │ Qwen2.5 / Llama 3 │ Function Calling        │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│                    工具执行层                                │
│  Python (8888) │ Node.js │ Chrome (9222) │ Code Server (8200)│
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│                    隔离沙箱层                                │
│  Docker │ nsjail │ cgroup │ namespaces                       │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 核心组件清单

| 组件 | 版本 | 用途 |
|------|------|------|
| Ubuntu Server | 22.04 LTS | 操作系统 |
| Docker | 26.0+ | 容器运行时 |
| Ollama | 0.3+ | LLM 推理引擎 |
| Open Interpreter | 0.2+ | Agent 执行框架 |
| Python | 3.10+ | 代码执行环境 |
| Node.js | 22+ | JS 执行环境 |
| Chrome/Chromium | 140+ | 浏览器自动化 |
| Playwright | 1.40+ | 浏览器控制 |
| Jupyter Lab | 4.0+ | 交互式开发 |
| Code Server | 4.9+ | Web IDE |
| TinyProxy | 1.10+ | 网络代理 |

---

## 2. 硬件要求

### 2.1 配置分级

| 等级 | CPU | 内存 | 显存 | 存储 | 推荐模型 |
|------|-----|------|------|------|---------|
| **入门级** | 4核 | 8GB | 8GB | 50GB SSD | Qwen2.5:7b |
| **进阶级** | 8核 | 16GB | 16GB | 100GB SSD | Qwen2.5:14b |
| **发烧级** | 16核 | 32GB | 24GB+ | 200GB SSD | Qwen2.5:32b |
| **旗舰级** | 32核 | 64GB | 48GB+ | 500GB SSD | Qwen2.5:72b-AWQ |

### 2.2 显卡兼容性

| 显卡 | 显存 | 支持模型 | 推荐 |
|------|------|---------|------|
| RTX 3060 | 12GB | 7B/14B | ✅ 性价比 |
| RTX 3090 | 24GB | 7B/14B/32B | ✅ 强烈推荐 |
| RTX 4060 | 8GB | 7B | ✅ |
| RTX 4090 | 24GB | 7B/14B/32B | ✅ 最佳 |
| A10 | 24GB | 7B-72B | ⭐ 专业 |
| A100 | 80GB | 全系列 | ⭐ 顶级 |

---

## 3. 系统准备

### 3.1 操作系统安装

```bash
# 推荐: Ubuntu Server 22.04 LTS
# 下载地址: https://releases.ubuntu.com/22.04/
```

### 3.2 系统初始化

```bash
#!/bin/bash
# system-init.sh

# 更新系统
apt update && apt upgrade -y

# 安装基础工具
apt install -y \
  build-essential git curl wget vim htop \
  python3 python3-pip python3-venv \
  nodejs npm \
  docker.io docker-compose \
  chromium-browser chromium-chromedriver \
  tightvncserver novnc websockify \
  tinyproxy

# 配置 Docker
usermod -aG docker $USER
systemctl enable docker
systemctl start docker

# 配置防火墙
ufw allow 22/tcp    # SSH
ufw allow 8080/tcp  # Web UI
ufw allow 8888/tcp  # Jupyter
ufw allow 8200/tcp  # Code Server
ufw allow 5900/tcp  # VNC
ufw enable

echo "✅ 系统初始化完成"
```

---

## 4. 基础环境安装

### 4.1 Python 环境

```bash
#!/bin/bash
# install-python.sh

# 创建虚拟环境
python3 -m venv ~/agent-env
source ~/agent-env/bin/activate

# 安装核心包
pip install --upgrade pip
pip install \
  open-interpreter \
  langchain langchain-community \
  playwright pandas numpy requests \
  jupyterlab notebook \
  python-dotenv pyyaml rich \
  openpyxl python-docx \
  matplotlib seaborn

# 安装浏览器
playwright install chromium
playwright install-deps

echo "✅ Python 环境安装完成"
```

### 4.2 Node.js 环境

```bash
#!/bin/bash
# install-node.sh

# 安装 nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc

# 安装 Node.js 22
nvm install 22
nvm use 22
nvm alias default 22

# 全局工具
npm install -g yarn ts-node typescript

echo "✅ Node.js 环境安装完成"
```

---

## 5. AI 模型部署

### 5.1 Ollama 安装 (推荐)

```bash
#!/bin/bash
# install-ollama.sh

# 安装 Ollama
curl -fsSL https://ollama.ai/install.sh | sh

# 启动服务
systemctl enable ollama
systemctl start ollama

# 下载模型 (按配置选择)
echo "正在下载 Qwen2.5:7b ..."
ollama pull qwen2.5:7b

# 可选更大模型:
# ollama pull qwen2.5:14b
# ollama pull qwen2.5:32b
# ollama pull llama3.1:8b
# ollama pull deepseek-coder-v2:16b

# 验证安装
ollama list
ollama run qwen2.5:7b --verbose

echo "✅ Ollama 模型部署完成"
```

### 5.2 vLLM 高性能部署 (可选)

```bash
#!/bin/bash
# install-vllm.sh

pip install vllm

# 启动 API 服务
vllm serve Qwen/Qwen2.5-7B-Instruct \
  --host 0.0.0.0 \
  --port 8000 \
  --tensor-parallel-size 1 \
  --gpu-memory-utilization 0.9

echo "✅ vLLM 服务启动: http://localhost:8000"
```

---

## 6. Agent 框架部署

### 6.1 Open Interpreter 安装配置

```bash
#!/bin/bash
# install-interpreter.sh

# 安装
pip install open-interpreter

# 创建配置目录
mkdir -p ~/.config/interpreter

# 配置文件
cat > ~/.config/interpreter/config.yaml << EOF
llm:
  model: ollama/qwen2.5:7b
  api_base: http://localhost:11434
  context_window: 128000
  temperature: 0.7

system_message: |
  你是 Doubao AI Agent，一个强大的本地 AI 助手。
  你可以执行代码、操作文件、控制浏览器、分析数据。
  请专业、高效地完成用户的任务。

computer:
  enabled: true
  timeout: 120

safety:
  confirm_commands: false
  offline: true
EOF

# 测试运行
interpreter --config ~/.config/interpreter/config.yaml

echo "✅ Open Interpreter 配置完成"
```

### 6.2 自定义 MoA 多 Agent 架构

```python
#!/usr/bin/env python3
# moa-agent.py

from langchain.agents import AgentExecutor, create_openai_tools_agent
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool
from langchain_core.prompts import ChatPromptTemplate

class MoAOrchestrator:
    def __init__(self):
        self.llm = ChatOpenAI(
            base_url="http://localhost:11434/v1",
            model="qwen2.5:7b",
            api_key="ollama"
        )
        
    async def plan_task(self, query: str):
        """任务规划 Agent"""
        pass
    
    async def execute_task(self, subtask: dict):
        """执行 Agent"""
        pass
    
    async def synthesize_result(self, results: list):
        """结果合成 Agent"""
        pass
```

---

## 7. Skill 系统集成

### 7.1 Skill 系统部署

```bash
#!/bin/bash
# install-skills.sh

# 解压 Skill 文件
mkdir -p ~/agent/skills
tar -xzf doubao-skills.tar.gz -C ~/agent/

# 目录结构
# ~/agent/skills/
#   ├── lark-doc/
#   ├── lark-sheets/
#   ├── office-excel/
#   ├── doubao-creative-design/
#   └── ... (共28个 Skill)

echo "✅ Skill 系统部署完成"
```

### 7.2 Skill 加载器

```python
#!/usr/bin/env python3
# skill-loader.py

import os
import yaml
import re
from pathlib import Path

class SkillLoader:
    def __init__(self, skills_path: str):
        self.skills_path = Path(skills_path)
        self.skills = {}
        self.load_all_skills()
    
    def load_all_skills(self):
        """加载所有 Skill"""
        for skill_dir in self.skills_path.iterdir():
            if skill_dir.is_dir():
                skill_md = skill_dir / "SKILL.md"
                if skill_md.exists():
                    self.skills[skill_dir.name] = self.parse_skill(skill_md)
    
    def parse_skill(self, skill_md: Path) -> dict:
        """解析 Skill 元数据"""
        content = skill_md.read_text()
        
        # 提取 frontmatter
        frontmatter = {}
        if content.startswith("---"):
            match = re.search(r"---\n(.*?)\n---", content, re.DOTALL)
            if match:
                frontmatter = yaml.safe_load(match.group(1))
        
        return {
            "name": frontmatter.get("name", skill_md.parent.name),
            "description": frontmatter.get("description", ""),
            "content": content,
            "references": list((skill_md.parent / "references").glob("*.md"))
        }
    
    def match_skill(self, query: str) -> list:
        """根据用户查询匹配 Skill"""
        matched = []
        for name, skill in self.skills.items():
            desc = skill["description"].lower()
            query_lower = query.lower()
            if any(keyword in query_lower for keyword in name.split("-")):
                matched.append(name)
        return matched
```

---

## 8. 浏览器自动化

### 8.1 Chrome 自动化配置

```bash
#!/bin/bash
# install-browser.sh

# 安装 Chrome
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
dpkg -i google-chrome-stable_current_amd64.deb
apt -f install -y

# 启动 Chrome 远程调试模式
cat > /etc/systemd/system/chrome-debug.service << EOF
[Unit]
Description=Chrome Remote Debugging
After=network.target

[Service]
User=$USER
ExecStart=/usr/bin/google-chrome-stable \
  --remote-debugging-port=9222 \
  --remote-debugging-address=0.0.0.0 \
  --no-sandbox \
  --headless=new \
  --disable-gpu \
  --disable-dev-shm-usage
Restart=always

[Install]
WantedBy=multi-user.target
EOF

systemctl enable chrome-debug
systemctl start chrome-debug

echo "✅ Chrome 调试服务启动: http://localhost:9222"
```

### 8.2 Playwright 工具封装

```python
#!/usr/bin/env python3
# browser-tool.py

from playwright.sync_api import sync_playwright

class BrowserTool:
    def __init__(self, cdp_url: str = "http://localhost:9222"):
        self.cdp_url = cdp_url
    
    def screenshot(self, url: str, output: str = "screenshot.png"):
        """网页截图"""
        with sync_playwright() as p:
            browser = p.chromium.connect_over_cdp(self.cdp_url)
            page = browser.new_page()
            page.goto(url)
            page.screenshot(path=output, full_page=True)
            browser.close()
        return f"截图已保存: {output}"
    
    def extract_content(self, url: str) -> str:
        """提取网页内容"""
        with sync_playwright() as p:
            browser = p.chromium.connect_over_cdp(self.cdp_url)
            page = browser.new_page()
            page.goto(url)
            content = page.content()
            browser.close()
        return content
```

---

## 9. 开发环境服务

### 9.1 Jupyter Lab 服务

```bash
#!/bin/bash
# install-jupyter.sh

pip install jupyterlab

# 配置
jupyter lab --generate-config
cat >> ~/.jupyter/jupyter_lab_config.py << EOF
c.ServerApp.ip = '0.0.0.0'
c.ServerApp.port = 8888
c.ServerApp.allow_origin = '*'
c.ServerApp.token = ''
c.ServerApp.password = ''
c.ServerApp.open_browser = False
EOF

# 系统服务
cat > /etc/systemd/system/jupyter.service << EOF
[Unit]
Description=Jupyter Lab
After=network.target

[Service]
User=$USER
ExecStart=$HOME/agent-env/bin/jupyter lab
WorkingDirectory=$HOME/agent
Restart=always

[Install]
WantedBy=multi-user.target
EOF

systemctl enable jupyter
systemctl start jupyter

echo "✅ Jupyter Lab: http://localhost:8888"
```

### 9.2 Code Server (VSCode Web)

```bash
#!/bin/bash
# install-code-server.sh

curl -fsSL https://code-server.dev/install.sh | sh

# 配置
mkdir -p ~/.config/code-server
cat > ~/.config/code-server/config.yaml << EOF
bind-addr: 0.0.0.0:8200
auth: none
password: ""
cert: false
EOF

# 系统服务
cat > /etc/systemd/system/code-server.service << EOF
[Unit]
Description=Code Server
After=network.target

[Service]
User=$USER
ExecStart=/usr/bin/code-server
Restart=always

[Install]
WantedBy=multi-user.target
EOF

systemctl enable code-server
systemctl start code-server

echo "✅ Code Server: http://localhost:8200"
```

### 9.3 VNC 远程桌面

```bash
#!/bin/bash
# install-vnc.sh

# 设置密码
vncpasswd

# 启动脚本
cat > ~/.vnc/xstartup << EOF
#!/bin/bash
xrdb $HOME/.Xresources
startxfce4 &
EOF
chmod +x ~/.vnc/xstartup

# 启动 VNC
vncserver :1 -geometry 1920x1080 -depth 24

# noVNC Web 访问
websockify --web /usr/share/novnc 6080 localhost:5901 &

echo "✅ VNC: 5901, Web VNC: http://localhost:6080/vnc.html"
```

---

## 10. 安全与隔离

### 10.1 Docker 沙箱

```bash
#!/bin/bash
# sandbox-setup.sh

# Agent 沙箱镜像
cat > Dockerfile.agent << EOF
FROM ubuntu:22.04
RUN apt update && apt install -y python3 python3-pip nodejs chromium-browser
RUN useradd -m agent
USER agent
WORKDIR /home/agent
EOF

docker build -f Dockerfile.agent -t agent-sandbox .

# 运行沙箱
docker run -d \
  --name agent-sandbox \
  --memory 4g \
  --cpus 2 \
  --pids-limit 512 \
  --read-only \
  --tmpfs /tmp \
  --net none \
  agent-sandbox sleep infinity

echo "✅ 沙箱容器已启动"
```

### 10.2 nsjail 轻量级隔离

```bash
#!/bin/bash
# install-nsjail.sh

apt install -y nsjail

# 配置文件
cat > nsjail.cfg << EOF
name: "agent-sandbox"
mode: ONCE
cwd: "/tmp"
time_limit: 60
rlimit_as: 4096
rlimit_cpu: 60
rlimit_fsize: 1024
rlimit_nproc: 64
mount {
  src: "/usr"
  dst: "/usr"
  is_bind: true
  ro: true
}
mount {
  src: "/lib"
  dst: "/lib"
  is_bind: true
  ro: true
}
EOF

echo "✅ nsjail 隔离配置完成"
```

---

## 11. 快速启动脚本

### 11.1 一键部署脚本

```bash
#!/bin/bash
# quick-deploy.sh
# 一键部署 Doubao Agent

set -e

echo "🚀 开始部署 Doubao Agent ..."

# 1. 系统初始化
echo "[1/6] 系统初始化..."
apt update && apt upgrade -y
apt install -y build-essential git curl wget python3 python3-pip docker.io

# 2. 安装 Ollama
echo "[2/6] 安装 Ollama..."
curl -fsSL https://ollama.ai/install.sh | sh
sleep 3
ollama pull qwen2.5:7b

# 3. 安装 Python 环境
echo "[3/6] 安装 Python 环境..."
python3 -m venv ~/agent-env
source ~/agent-env/bin/activate
pip install open-interpreter playwright jupyterlab
playwright install chromium

# 4. 部署 Skill 系统
echo "[4/6] 部署 Skill 系统..."
mkdir -p ~/agent/skills
tar -xzf doubao-skills.tar.gz -C ~/agent/

# 5. 配置 Open Interpreter
echo "[5/6] 配置 Agent..."
cat > ~/.config/interpreter/config.yaml << EOF
llm:
  model: ollama/qwen2.5:7b
  context_window: 128000
computer:
  enabled: true
EOF

# 6. 启动服务
echo "[6/6] 启动服务..."
systemctl start ollama

echo ""
echo "✅ 部署完成！"
echo ""
echo "📋 服务地址:"
echo "  Agent: interpreter"
echo "  模型: http://localhost:11434"
echo ""
echo "🚀 启动命令:"
echo "  source ~/agent-env/bin/activate"
echo "  interpreter"
```

### 11.2 Docker Compose 一键启动

```yaml
# docker-compose.yml
version: '3.8'

services:
  ollama:
    image: ollama/ollama:latest
    ports:
      - "11434:11434"
    volumes:
      - ollama-data:/root/.ollama
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]

  jupyter:
    image: jupyter/scipy-notebook:latest
    ports:
      - "8888:8888"
    volumes:
      - ./workspace:/home/jovyan/work

  chrome:
    image: browserless/chrome:latest
    ports:
      - "9222:3000"
    environment:
      MAX_CONCURRENT_SESSIONS: 10

volumes:
  ollama-data:
```

启动命令:
```bash
docker-compose up -d
```

---

## 12. 故障排查

### 12.1 常见问题

| 问题 | 解决方案 |
|------|---------|
| **Ollama 模型下载慢** | 配置镜像源或手动下载 GGUF |
| **GPU 不识别** | 安装 NVIDIA Container Toolkit |
| **Chrome 启动失败** | 添加 --no-sandbox 参数 |
| **内存不足** | 启用 SWAP，使用量化模型 |
| **端口冲突** | 修改配置文件端口 |

### 12.2 健康检查脚本

```bash
#!/bin/bash
# health-check.sh

echo "=== Doubao Agent 健康检查 ==="

# 检查 Ollama
curl -s http://localhost:11434/api/tags > /dev/null && echo "✅ Ollama: 正常" || echo "❌ Ollama: 未启动"

# 检查 Chrome
curl -s http://localhost:9222/json/version > /dev/null && echo "✅ Chrome: 正常" || echo "⚠️ Chrome: 未启动"

# 检查 Docker
docker ps > /dev/null && echo "✅ Docker: 正常" || echo "❌ Docker: 异常"

# 显存检查
nvidia-smi --query-gpu=memory.used,memory.total --format=csv,noheader 2>/dev/null && echo "✅ GPU: 可用" || echo "⚠️ GPU: 无或未配置"

echo "=== 检查完成 ==="
```

---

## 📚 参考资源

- Ollama 文档: https://ollama.ai/docs
- Open Interpreter: https://docs.openinterpreter.com
- Playwright: https://playwright.dev
- LangChain: https://python.langchain.com

---

*本部署指南基于 Doubao Agent 实际运行环境编写，可直接用于个人服务器部署。*
