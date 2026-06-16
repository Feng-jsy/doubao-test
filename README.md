# Doubao Agent Skills & System Info

## 📦 项目说明

本仓库包含 Doubao AI Agent 系统的完整 Skill 文件和系统信息文档。

## 📁 文件列表

### Skill 压缩包
- **doubao-skills.tar.gz** - 完整 Skill 系统 (TAR.GZ 格式, 757KB)
- **doubao-skills.zip** - 完整 Skill 系统 (ZIP 格式, 1.1MB)

### 系统文档
- **system-info.md** - 完整系统信息文档
  - 系统基本信息 (内核/CPU/内存/存储)
  - 网络信息 (IP/端口/代理配置)
  - 系统限制与安全机制
  - 技术架构原理 (MoA/Skill/浏览器/存储)
  - 完整目录结构
  - Skill 系统统计

## 📊 Skill 系统概览

| 项目 | 数值 |
|------|------|
| Skill 总数 | 28 个 |
| 总目录数 | 65 个 |
| 总文件数 | 298 个 |
| Markdown 文档 | 275 个 |
| Python 脚本 | 16 个 |
| 原始大小 | 3.2 MB |

## 🏷️ Skill 分类

### 飞书生态 (21个)
- lark-doc - 飞书文档
- lark-sheets - 电子表格
- lark-ppt - PPT
- lark-drive - 云盘
- lark-im - 即时通讯
- lark-calendar - 日历
- lark-mail - 邮件
- lark-base - 多维表格
- lark-whiteboard - 画板
- lark-wiki - 知识库
- lark-task - 任务管理
- lark-okr - OKR
- lark-project - 项目管理
- lark-vc - 视频会议
- lark-minutes - 会议纪要
- lark-markdown - Markdown
- lark-contact - 通讯录
- lark-approval - 审批
- lark-attendance - 考勤
- lark-workflow-meeting-summary
- lark-workflow-standup-report

### 创意设计 (1个)
- doubao-creative-design - 创意设计

### Office 处理 (2个)
- office-excel - Excel 数据处理
- office-word - Word 文档处理

### 系统工具 (4个)
- skill-creator-for-task - Skill 创建工具
- lark-openapi-explorer - API 调试

## 🚀 技术架构

```
用户 ↔ MainAgent ↔ OrganizeAgent ↔ SubAgent
```

### Skill 分层加载
```
Level 1: Metadata (name + description)  ~100词
Level 2: SKILL.md 正文                  <5000词
Level 3: references/*.md                按需读取
Level 4: scripts/*.py                   直接执行
```

---

*自动生成于 2026年6月17日*
