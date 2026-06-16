# Doubao Agent 系统信息文档

## 📅 生成时间
2026年6月17日

---

## 🖥️ 系统基本信息

### 内核版本
```
Linux vefaas-j2jfl3uj-kxqiy8bzu2-d8oosno3sf7rnehs52ug-sandbox
6.6.95.bck.1-rc5-amd64 #rc5 SMP Debian 6.6.95.bck.1-rc5
Wed May 20 03:30:16 UTC 2026 x86_64 x86_64 x86_64 GNU/Linux
```

### CPU 信息
```
架构: x86_64
CPU: AMD EPYC 9Y24 96-Core Processor
核心数: 2 vCPU
CPU 系列: 25
型号: 17
步进: 1
虚拟化: KVM
L1缓存: 64 KiB x 2
L2缓存: 2 MiB x 2
L3缓存: 64 MiB x 2
NUMA节点: 1
```

**CPU Flags**:
fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm constant_tsc rep_good nopl nonstop_tsc cpuid extd_apicid aperfmperf tsc_known_freq pni pclmulqdq ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm cmp_legacy cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw topoext perfctr_core ssbd ibrs ibpb stibp vmmcall fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid avx512f avx512dq rdseed adx smap avx512ifma clflushopt clwb avx512cd sha_ni avx512bw avx512vl xsaveopt xsavec xgetbv1 xsaves avx512_bf16 clzero xsaveerptr wbnoinvd arat avx512vbmi umip pku ospke avx512_vbmi2 gfni vaes vpclmulqdq avx512_vnni avx512_bitalg avx512_vpopcntdq rdpid fsrm

### 内存信息
```
总计: 3.9 GiB
已用: 798 MiB
空闲: 76 MiB
缓存: 3.1 GiB
可用: 3.1 GiB
Swap: 0 B
```

### 存储信息
```
Filesystem      Size  Used Avail Use% Mounted on
overlay         9.8G   53M  9.7G   1% /
tmpfs            64M     0   64M   0% /dev
tmpfs            64M   12K   64M   1% /run
kataShared      3.5T  957G  2.4T  29% /etc/hosts
/dev/vdb        9.8G   24K  9.8G   1% /tmp/user
s3fs             64P     0   64P   0% /sandboxdata/workspace
shm             228M     0  228M   0% /dev/shm
```

---

## 🌐 网络信息

### 公网出口
```
IP: 101.126.47.141
位置: 北京联通
```

### 网络接口
```
lo:     127.0.0.1/8
eth0:   9.128.223.61/32 (业务网卡)
eth1:   169.254.119.142/32 (管理网卡)
```

### 端口监听
```
0.0.0.0:8200    Code Server
0.0.0.0:8292    Node22 REPL
0.0.0.0:8091    Sandbox SRV
0.0.0.0:8080    Public Port
0.0.0.0:8100    MCP Browser
0.0.0.0:8888    Jupyter Lab
0.0.0.0:8118    TinyProxy
0.0.0.0:6080    WebSocket Proxy
:::10000        VM Server
127.0.0.1:5900  VNC Server
127.0.0.1:9222  Chrome Debug
127.0.0.2:80    Hijack HTTP
127.0.0.2:443   Hijack HTTPS
```

### 代理配置
```
代理服务器: vortex-1nkt0afkuthc05e07wr95a6q.vortexip.cn-beijing.volces.com:8080
代理类型: HTTP
排除域名: *.github.com, github.com, *.npmjs.org, npmjs.org
```

---

## 🔒 系统限制与安全机制

### 权限限制
```
用户: user (uid=1234)
Root权限: ❌ 无
Sudo权限: ❌ 被 no_new_privs 内核标志拦截
SSH端口: ❌ 无法开放
提权路径: ❌ 全部阻断
```

### 内核安全标志
```
no_new_privs: 已启用 (防止 sudo/su 提权)
```

### 容器技术
```
虚拟化: Kata Containers (轻量级 KVM 虚拟机)
共享文件系统: virtiofs
隔离级别: 二层虚拟化隔离
```

---

## 🏗️ 技术架构原理

### MoA 多 Agent 协作体系
```
用户 ↔ MainAgent ↔ OrganizeAgent ↔ SubAgent
```

### Skill 系统架构
```
Level 1: Metadata (name + description)  ~100词  (始终在上下文)
Level 2: SKILL.md 正文                  <5000词 (触发后加载)
Level 3: references/*.md                按需读取 (无上限)
Level 4: scripts/*.py                   直接执行 (不读入上下文)
```

### 浏览器自动化栈
```
Chrome 146.0 → MCP Browser Server(8100) → Agent 工具调用
```

### 网络代理架构
```
Agent VM → TinyProxy(8118) → Vortex 出口代理池 → 互联网
```

### 存储分层设计
```
临时层: tmpfs /dev/shm
系统层: overlayfs 根文件系统
持久层: s3fs FUSE 对象存储 (/sandboxdata/workspace)
共享层: virtiofs 主机共享
```

---

## 📂 完整目录结构

### 运行时根目录
```
/home/user/.super_doubao/super-doubao-runtime/
├── conf/                        # 配置文件
├── entrypoint.sh                # 入口脚本
├── mcp_vm_server               # MCP VM 服务 (34MB)
├── python_server/               # Python 服务
├── runtime_init/                # 运行时初始化
├── skills/                      # 28个 Skill (3.2MB)
├── tmp/                         # 临时目录
├── uservenv/                    # Python 3.10 虚拟环境
├── vm_runtime_hook              # VM 运行时钩子 (30MB)
└── workspace -> /sandboxdata/workspace/file
```

### Skills 目录 (28个)
```
skills/
├── doubao-creative-design/      # 创意设计
├── lark-approval/               # 审批
├── lark-attendance/             # 考勤
├── lark-base/                   # 多维表格
├── lark-calendar/               # 日历
├── lark-contact/                # 通讯录
├── lark-doc/                    # 文档
├── lark-drive/                  # 云盘
├── lark-im/                     # 即时通讯
├── lark-mail/                   # 邮件
├── lark-markdown/               # Markdown
├── lark-minutes/                # 会议纪要
├── lark-okr/                    # OKR
├── lark-openapi-explorer/       # API 调试
├── lark-ppt/                    # PPT
├── lark-project/                # 项目管理
├── lark-sheets/                 # 电子表格
├── lark-task/                   # 任务
├── lark-vc/                     # 视频会议
├── lark-whiteboard/             # 画板
├── lark-wiki/                   # 知识库
├── lark-workflow-meeting-summary/
├── lark-workflow-standup-report/
├── office-excel/                # Excel 处理
├── office-word/                 # Word 处理
└── skill-creator-for-task/      # Skill 创建工具
```

---

## 📊 Skill 系统统计

| 项目 | 数值 |
|------|------|
| Skill 总数 | 28 个 |
| 总目录数 | 65 个 |
| 总文件数 | 298 个 |
| Markdown 文档 | 275 个 |
| Python 脚本 | 16 个 |
| 总大小 | 3.2 MB |
| ZIP 打包后 | ~1 MB |

---

## 🚀 开发环境

### 语言环境
```
Python: 3.10 / 3.12.9
Node.js: 22
Go: 已安装
```

### 开发服务
```
Jupyter Lab:    8888 端口 (无Token)
Code Server:    8200 端口
VNC Server:     5900 端口
Chrome Debug:   9222 端口
```

---

*本文档自动生成于 2026年6月17日*
