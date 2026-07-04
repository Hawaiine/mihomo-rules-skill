---
name: mihomo-rules-management
description: >-
  Use when managing mihomo/clash-meta compatible RULE-SET proxy rulesets —
  syncing upstream domain lists, adding/reseeding brand rulesets, validating
  YAML format, generating platform configs (Nikki/Clash for Android), and
  operating the associated GitHub Actions CI/CD pipeline. Covers the
  Hawaiine/mihomo-rules project conventions.
version: 1.2.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [mihomo, clash, proxy, ruleset, sync]
    related_skills: [github-actions-workflows, bash-projects, systematic-debugging]
---

# mihomo-rules 管理 Skill

## Overview

Manage a mihomo-compatible RULE-SET-level proxy ruleset repository. The project syncs upstream domain lists from v2fly/domain-list-community and Loyalsoldier/clash-rules, auto-discovers empty rulesets, validates YAML format, generates platform-specific configs, and notifies via Discord webhook.

## When to Use

- 用户要求添加/修改/删除 ruleset 规则集
- 需要同步上游域名数据（v2fly / Loyalsoldier）
- 校验 ruleset 文件格式是否符合规范
- 生成 Nikki（OpenWrt）或 Clash for Android 配置文件
- 调试 GitHub Actions CI/CD 工作流
- 排查 Discord 通知失败

## 项目结构

```
mihomo-rules/
├── ruleset/                   # 规则集文件（.yaml）
│   ├── Direct.yaml            # 国内直连域名
│   ├── Proxy.yaml             # 默认代理域名
│   ├── Reject.yaml            # 广告/恶意域名
│   ├── Private.yaml           # 私有地址
│   ├── LanCIDR.yaml           # 局域网 IP 段（IP-CIDR）
│   ├── CNCIDR.yaml            # 中国 IP 段（IP-CIDR）
│   ├── Telegram.yaml          # Telegram 直连域名
│   ├── Applications.yaml      # 应用程序直连
│   └── *.yaml                 # 品牌规则集（如 Netflix.yaml）
├── configs/                   # 平台配置文件
│   ├── Nikki-mihomo.yml       # Nikki (OpenWrt / sing-box)
│   └── Android-mihomo.yml     # Clash for Android
├── providers/                 # 节点占位模板
│   ├── airport1.yaml.example  # 机场订阅模板
│   └── nodes.yaml.example     # 单节点协议模板
├── scripts/                   # 自动化脚本
│   ├── sync-upstream.sh       # 上游同步（v2fly + Loyalsoldier）
│   ├── generate-config.sh     # 配置生成
│   ├── validate-ruleset.sh    # 格式校验 + 自动修复
│   ├── import-openwrt.sh      # 从 OpenWrt-Domain-list 导入
│   └── test_upstream.sh       # 上游存在性测试
├── .github/workflows/
│   └── daily-sync.yml         # 每日同步工作流
├── CHANGELOG.md               # 变更日志
└── README.md                  # 项目文档
```

## 核心规范

### 1. 命名规范

- **文件名**: PascalCase（如 `Netflix.yaml`, `AIService.yaml`）
- **去符号**: 去掉括号、`@`、`-`、`+` 等（`U-NEXT` → `UNext`, `Karaoke@DAM` → `KaraokeDam`）
- **缩写保留**: `AI`, `TV`, `CDN`, `IP`, `ID` 等常见缩写保持大写
- **Apple i-前缀**: `iCloud`, `iTunes` 等保持首字母小写
- **裸文件**: 无后缀的文件自动加 `.yaml`（`bagumi` → `Bagumi.yaml`）

### 2. Ruleset 文件格式

```yaml
# ===========================================
# Rule Name: Netflix
# Author: Hawaiine
# Updated: 2026-07-04 07:05:58
# DOMAIN: 27
# DOMAIN-SUFFIX: 1
# IP-CIDR: 0
# ===========================================
payload:
  # --- DOMAIN 条目（按字母序） ---
  - DOMAIN,netflix.com
  # --- DOMAIN-SUFFIX 条目（按字母序） ---
  - DOMAIN-SUFFIX,nflxvideo.net
  # --- IP-CIDR 条目（按字母序） ---
  - IP-CIDR,10.0.0.0/8
```

**格式要求：**
- Header 必须包含 `# Rule Name:`, `# Author:`, `# Updated:`, `# DOMAIN:`, `# DOMAIN-SUFFIX:`, `# IP-CIDR:`
- 无 `# Source:` 行
- 无 `@ads`、`@cn` 等标签
- DOMAIN 在前，DOMAIN-SUFFIX 在后，IP-CIDR 最后，各自按字母序
- 计数必须与实际条目数一致

### 3. 路由架构

项目采用 **RULE-SET 级分流**，每个品牌规则集即同名策略组：

```
RULE-SET,Netflix,Netflix      # Netflix 流量 → Netflix 策略组
RULE-SET,Disney,Disney        # Disney 流量 → Disney 策略组
RULE-SET,Direct,🎯 全球直连    # 国内流量 → 直连
```

### 4. 规则集类别

- **基础规则集**: Direct, Proxy, Reject, Private, LanCIDR, CNCIDR, Telegram, Applications
- **品牌规则集**: 流媒体、AI、社交、音乐、游戏、云服务等
- **Porn/PornChina**: 特殊处理，不在 README 公开

## 脚本操作

### sync-upstream.sh

上游同步脚本，从 v2fly 和 Loyalsoldier 拉取数据。

```
# 同步所有品牌
bash scripts/sync-upstream.sh

# 同步单个品牌
bash scripts/sync-upstream.sh Netflix

# 同步基础规则集
bash scripts/sync-upstream.sh  # 自动同步全部
```

**关键特性：**
- 递归解析 v2fly `include:` 指令（最多 5 层深度）
- 品牌名映射：`Porn` → `category-porn`, `NHK` → `nhk`
- 自动发现空 ruleset 文件并填充上游数据
- 并行品牌同步（`sync_brand &` + `wait`）
- Loyalsoldier 大文件用 awk 解析（11 万行 ~0.03 秒）
- CI 超时控制：connect-timeout 20s, max-time 60s
- 自动重命名不规范文件名（`tik-tok` → `TikTok`）

### validate-ruleset.sh

格式校验和自动修复脚本。

```
# 校验所有 ruleset
bash scripts/validate-ruleset.sh

# 校验指定文件
bash scripts/validate-ruleset.sh ruleset/Netflix.yaml
```

**自动修复能力：**
- 裸域名 → `DOMAIN,xxx`（`example.com` → `DOMAIN,example.com`）
- `+.xxx` / `.xxx` → `DOMAIN-SUFFIX,xxx`
- `full:xxx` → `DOMAIN-SUFFIX,xxx`
- 小写 `domain,` → `DOMAIN,`
- 去掉 `@ads` 标签
- 裸文件自动加 `.yaml` 后缀
- 小写文件名自动转 PascalCase（`bagumi` → `Bagumi`）
- 更新 header 计数
- 排序（DOMAIN → SUFFIX → IP-CIDR，各自按字母序）

### generate-config.sh

生成平台配置文件。

```
bash scripts/generate-config.sh
```

自动扫描 `ruleset/` 下的 `.yaml` 文件，生成：
- `rule-providers` 块（含 URL 和路径）
- `rules` 块（RULE-SET 引用）
- 同名策略组映射

### import-openwrt.sh

从 OpenWrt-Domain-list 仓库批量导入品牌规则集。

```
bash scripts/import-openwrt.sh
```

处理文件名特殊字符：`GameJP(GAME JAPAN)` → `GameJapan.yaml`

## 上游数据源

| 源 | 用途 | URL |
|----|------|-----|
| v2fly/domain-list-community | 品牌域名数据 | `raw.githubusercontent.com/v2fly/domain-list-community/master/data/` |
| Loyalsoldier/clash-rules | 基础规则集 | `raw.githubusercontent.com/Loyalsoldier/clash-rules/release/` |
| MetaCubeX/meta-rules-dat | geox 数据 | `fastly.jsdelivr.net/gh/MetaCubeX/meta-rules-dat@release/` |

## CI/CD 工作流

`.github/workflows/daily-sync.yml`:

| 步骤 | 描述 |
|------|------|
| checkout | actions/checkout@v7 |
| sync-upstream.sh | 上游同步（20min 超时） |
| validate-ruleset.sh | 格式校验（5min 超时） |
| generate-config.sh | 配置生成（10min 超时） |
| git commit + push | 中文 emoji 提交，北京时间 |
| Discord webhook | 成功通知（提交者+分支+时间） |

**环境变量：**
- `TZ: Asia/Shanghai` — 北京时间
- `DISCORD_WEBHOOK` — 通过 GitHub Secrets 传入

## 常见工作流

### 添加新品牌

1. 创建 bare 文件或 `.yaml` 文件
2. 写入不规范域名（支持裸域名、`+.` 前缀等）
3. 运行 `bash scripts/validate-ruleset.sh` 自动修复
4. 运行 `bash scripts/sync-upstream.sh <Brand>` 补充上游数据
5. `git add` + `git commit`（pre-commit 钩子自动校验）

### 手动补充域名

1. 编辑 `ruleset/<Brand>.yaml`
2. 添加裸域名（如 `example.com`）
3. 运行 `bash scripts/validate-ruleset.sh` 自动修正格式
4. 更新 `# Updated:` 时间

### 排查 CI 失败

1. `gh run view <run-id> --log` 查看日志
2. 常见问题：
   - 上游超时 → 检查 curl 超时设置
   - 数据量过大 → 检查 awk 解析
   - include 递归未生效 → 检查 `v2fly_all.txt` 是否保留了原始内容
   - 403 推送失败 → 检查 `permissions: contents: write`

## 关键配置参考

### v2fly 品牌名映射

当品牌名 ≠ v2fly 数据文件名时，需在 `sync-upstream.sh` 中添加映射：

```bash
case "$brand" in
  Porn) v2fly_lower="category-porn" ;;
  NHK) v2fly_lower="nhk" ;;
esac
```

### Discord Webhook Payload

```json
{
  "username": "mihomo-rules",
  "embeds": [{
    "title": "🔄 mihomo-rules 同步完成",
    "url": "${COMMIT_URL}",
    "description": "`${COMMIT_MSG}`",
    "color": 3447003,
    "fields": [
      {"name": "提交者", "value": "${COMMIT_AUTHOR}", "inline": true},
      {"name": "分支", "value": "main", "inline": true}
    ]
  }]
}
```

注意：commit message 必须用 `git log --format='%s'` 单行格式，多行会破坏 JSON。

## 常见陷阱

1. **include 递归丢失原始内容**：`cat >> v2fly_all.txt` 前必须 `cp v2fly.txt v2fly_all.txt`，否则原始内容被覆盖
2. **文件名规范化破坏已有品牌名**：只应在首字母小写时重命名，且排除 `iCloud` 等 i-前缀
3. **Commit message 破坏 JSON**：必须用 `git log --format='%s'`（单行），不能用 `--pretty=%B`
4. **CI 超时**：Loyalsoldier 大文件 11 万行，bash 逐行解析 5.4 秒，aww 只需 0.03 秒
5. **Discord 通知静默失败**：删除 `|| echo` 让失败可见，不要 mask 错误
6. **Porn/PornChina 不应在 README 公开**

## 验证检查清单

- [ ] 新 ruleset 文件命名 PascalCase
- [ ] Header 格式正确（Rule Name, Author, Updated, 计数）
- [ ] 无 `@ads`、`@cn` 等标签
- [ ] 无 `# Source:` 行
- [ ] DOMAIN/SUFFIX/IP-CIDR 按字母序排列
- [ ] Header 计数与实际条目一致
- [ ] 裸文件已加 `.yaml` 后缀
- [ ] 上游 include 递归解析正确（多品牌时检查 `v2fly_all.txt`）
- [ ] CI 工作流语法正确 (`bash -n`)
- [ ] 敏感信息（token, webhook URL）未提交到仓库
- [ ] README 已更新（如新增品牌）
- [ ] CHANGELOG 已更新（中文 + emoji）