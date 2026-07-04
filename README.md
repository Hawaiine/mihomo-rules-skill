<p align="center">
  <img src="https://raw.githubusercontent.com/MetaCubeX/mihomo/Meta/docs/logo.png" width="80" alt="mihomo logo"/>
</p>

<h1 align="center">🔄 mihomo-rules-workflow</h1>

<p align="center">
  <b>通用 mihomo / clash-meta RULE-SET 规则集管理项目模板</b>
</p>

<p align="center">
  <img alt="License" src="https://img.shields.io/badge/license-MIT-blue?style=flat-square"/>
  <img alt="PRs Welcome" src="https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat-square"/>
  <img alt="bash 4.0+" src="https://img.shields.io/badge/bash-4.0%2B-orange?style=flat-square"/>
  <img alt="Powered by v2fly" src="https://img.shields.io/badge/powered%20by-v2fly-9cf?style=flat-square"/>
</p>

---

## ✨ 特性一览

| 特性 | 说明 |
|------|------|
| 🔄 **上游同步** | 从 v2fly/Loyalsoldier 自动拉取域名数据 |
| 🔍 **include 递归** | 解析 v2fly 的 include 链，最多 5 层深度 |
| 🤖 **自动发现** | 空 ruleset 文件自动识别、重命名、填充 |
| ✅ **格式校验** | 自动修复裸域名、标签、大小写、计数 |
| ⚙️ **配置生成** | 一键生成 Nikki / Clash for Android 配置 |
| 📅 **CI/CD** | GitHub Actions 每日同步 + Discord 通知推送 |
| 📝 **变更日志** | 自动记录每次更新的规则变化 |

---

## 🚀 快速上手

```bash
# 1️⃣ 创建品牌规则集（随便写格式）
echo "  - example.com
  - +.cdn.com" > ruleset/MyBrand

# 2️⃣ 自动校验修复 → MyBrand.yaml
bash scripts/validate-ruleset.sh

# 3️⃣ 同步上游补充数据
bash scripts/sync-upstream.sh MyBrand

# 4️⃣ 提交推送（pre-commit 自动校验）
git add -A
git commit -m "✨ add MyBrand ruleset"
git push
```

> 💡 也支持 `bagumi` 裸文件（无 `.yaml` 后缀），脚本会自动补后缀、转 PascalCase、修格式。

---

## 📂 目录结构

```
mihomo-rules/
├── 📦 ruleset/                  # → 规则集文件（.yaml）
│   ├── 🎯 Direct.yaml           #   国内直连域名
│   ├── 🔧 Proxy.yaml            #   默认代理域名
│   ├── 🛑 Reject.yaml           #   广告/恶意域名
│   ├── 🏠 Private.yaml          #   私有地址
│   ├── 🏠 LanCIDR.yaml          #   局域网 IP 段
│   ├── 🌏 CNCIDR.yaml           #   中国 IP 段
│   ├── 📲 Telegram.yaml         #   Telegram 直连
│   ├── 📱 Applications.yaml     #   应用直连
│   └── 🏷️ *.yaml               #   品牌规则集
├── ⚙️ configs/                  # → 平台配置文件
│   ├── Nikki-mihomo.yml         #   OpenWrt Nikki
│   └── Android-mihomo.yml       #   Clash for Android
├── 📜 scripts/                  # → 自动化脚本
│   ├── 🔄 sync-upstream.sh      #   上游同步
│   ├── ✅ validate-ruleset.sh   #   格式校验+修复
│   └── ⚙️ generate-config.sh    #   配置生成
├── 🔌 providers/                # → 节点模板
│   ├── airport1.yaml.example    #   机场订阅
│   └── nodes.yaml.example       #   单节点协议
├── 🤖 .github/workflows/
│   └── daily-sync.yml           # → 每日同步工作流
├── 📝 CHANGELOG.md
└── 📖 README.md
```

---

## 📏 命名规范

```
✅ PascalCase          → Netflix.yaml, AIService.yaml
✅ 去括号/去符号       → U-NEXT → UNext, Karaoke@DAM → KaraokeDam
✅ 缩写保留大写        → AI, TV, CDN, IP 等
✅ Apple i-前缀保留    → iCloud.yaml（不转 Icloud）
✅ 裸文件自动修复      → bagumi → Bagumi.yaml + 内容修正
```

---

## 🧪 脚本全家桶

### 🔄 sync-upstream.sh

```bash
# 同步全部品牌
bash scripts/sync-upstream.sh

# 同步单个品牌（含 include 递归解析）
bash scripts/sync-upstream.sh Netflix
```

**特性：**
- 递归跟随 v2fly `include:` 指令（最多 5 层）
- 并行拉取（`&` + `wait`），大幅提升速度
- Loyalsoldier 大文件用 awk 解析（110k 行 → 0.03s）
- 自动发现空文件并填充

### ✅ validate-ruleset.sh

```bash
# 校验全部
bash scripts/validate-ruleset.sh

# 校验指定文件
bash scripts/validate-ruleset.sh ruleset/Netflix.yaml
```

| 你写的 | 自动修成 |
|--------|----------|
| `example.com` | ✅ `DOMAIN,example.com` |
| `+.example.com` | ✅ `DOMAIN-SUFFIX,example.com` |
| `.example.com` | ✅ `DOMAIN-SUFFIX,example.com` |
| `full:example.com` | ✅ `DOMAIN-SUFFIX,example.com` |
| `domain,xxx` | ✅ `DOMAIN,xxx` |
| `xxx @ads` | ✅ 去掉 `@ads` |
| `bagumi`（裸文件） | ✅ `Bagumi.yaml` + 内容修正 |

### ⚙️ generate-config.sh

```bash
bash scripts/generate-config.sh
```

自动扫描 `ruleset/` 下的所有 `.yaml`，生成 `rule-providers`、`rules` 和同名策略组。

---

## 📡 上游数据源

```
🌐 v2fly/domain-list-community    → 品牌域名数据
🌐 Loyalsoldier/clash-rules       → 基础/拦截/代理规则
🌐 MetaCubeX/meta-rules-dat       → geox 数据
```

---

## 🤖 GitHub Actions 工作流

每天 UTC 22:00（北京时间 06:00）自动执行：

```mermaid
graph LR
    A[checkout@v7] --> B[sync-upstream.sh 20m]
    B --> C[validate-ruleset.sh 5m]
    C --> D[generate-config.sh 10m]
    D --> E[git commit + push]
    E --> F[Discord 通知]
```

| 步骤 | 超时 | 说明 |
|------|------|------|
| `actions/checkout@v7` | — | 升级到 v7 避免 Node.js 弃用警告 |
| `sync-upstream.sh` | 20 分钟 | 拉取上游 + include 递归 + 自动发现 |
| `validate-ruleset.sh` | 5 分钟 | 自动修复格式问题 |
| `generate-config.sh` | 10 分钟 | 生成平台配置文件 |
| `git commit + push` | — | 中文 emoji + 北京时间 |

---

## 📥 安装到 Hermes Agent

其他 Hermes Agent 想用这套规范：

```bash
# 克隆项目
git clone https://github.com/Hawaiine/mihomo-rules-workflow.git my-project

# 加载 skill
# 在对话中加载 skill_view(name='mihomo-rules-management')
```

> SKILL.md 在仓库根目录，记录着完整的命名规范、脚本用法、常见陷阱。

---

## 🧩 依赖

| 工具 | 版本要求 | 用途 |
|------|---------|------|
| `bash` | ≥ 4.0 | 脚本运行 |
| `curl` | — | 上游数据拉取 |
| `awk` | — | 大文件高性能解析 |
| `git` | — | 版本管理 |
| `gh` | — | CI 调试（可选） |

---

## ⚠️ 常见陷阱

<details>
<summary>🔽 点击展开</summary>

| # | 陷阱 | 正确做法 |
|---|------|----------|
| 1️⃣ | include 递归丢失原始内容 | 先 `cp v2fly.txt v2fly_all.txt`，再 `cat >>` 追加 |
| 2️⃣ | 文件名规范化破坏已有品牌 | 只改首字母小写的，排除 `iCloud` 等 |
| 3️⃣ | commit message 破坏 JSON | 用 `git log --format='%s'`（单行） |
| 4️⃣ | CI 超时 | 大文件用 awk 替代 bash 逐行 |
| 5️⃣ | Discord 通知静默失败 | 删掉 `|| echo`，让错误暴露 |
| 6️⃣ | 敏感信息泄露 | webhook URL / token 只存 GitHub Secrets |
</details>

---

<p align="center">
  <sub>Built with ❤️ by <a href="https://github.com/Hawaiine">Hawaiine</a> · Powered by <a href="https://github.com/MetaCubeX/mihomo">mihomo</a> & <a href="https://github.com/v2fly/domain-list-community">v2fly</a></sub>
</p>