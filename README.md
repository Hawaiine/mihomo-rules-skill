<p align="center">
  <img src="https://raw.githubusercontent.com/MetaCubeX/mihomo/Meta/docs/logo.png" width="64" alt="mihomo logo"/>
</p>

<h1 align="center">📖 mihomo-rules-skill</h1>

<p align="center">
  <b>Hermes Agent Skill — mihomo/clash-meta RULE-SET 规则集管理知识库</b>
</p>

<p align="center">
  <img alt="License" src="https://img.shields.io/badge/license-MIT-blue?style=flat-square"/>
  <img alt="Hermes Skill" src="https://img.shields.io/badge/Hermes-Skill-purple?style=flat-square"/>
</p>

---

## 🎯 这是什么

**mihomo-rules-skill** 是一个 [Hermes Agent](https://github.com/NousResearch/hermes) Skill，专门为管理 mihomo/clash-meta RULE-SET 规则集项目而编写。

如果你需要向 Hermes 提需求：
- ✋ "帮我加个 Netflix 规则集"
- 🔄 "同步一下上游数据"
- ✅ "校验 ruleset 格式"
- ⚙️ "生成配置"

加载这个 skill 后，Hermes 就掌握了完整的规范、脚本、陷阱和检查清单，可以直接上手干活。

---

## 📂 仓库内容

```
├── 📖 README.md    # 本文件 —— 使用说明
└── 📜 SKILL.md     # → Hermes Agent Skill（核心）
```

> 🔗 参考实现：[Hawaiine/mihomo-rules](https://github.com/Hawaiine/mihomo-rules)
> 真实的 70+ 规则集项目，展示了本 skill 描述的完整工作流。

---

## 🚀 使用方式

### 💬 对话中临时加载

告诉 Hermes：

```
skill_view(name='mihomo-rules-management')
```

### 📦 永久安装

```bash
git clone https://github.com/Hawaiine/mihomo-rules-skill.git
cp mihomo-rules-skill/SKILL.md ~/.hermes/skills/mihomo-rules-management/
```

下次启动 Hermes 自动生效。

---

## 📋 Skill 涵盖的内容

| 模块 | 说明 |
|------|------|
| 📏 **命名规范** | PascalCase 命名、去符号规则、缩写处理、i-前缀例外 |
| 📄 **文件格式** | YAML header 规范、DOMAIN/SUFFIX/IP-CIDR 排序与计数 |
| 🗺️ **路由架构** | RULE-SET 级分流、策略组一一映射 |
| 📜 **脚本参考** | `sync-upstream.sh`、`validate-ruleset.sh`、`generate-config.sh` 完整用法与代码 |
| 🌐 **上游数据源** | v2fly/Loyalsoldier/MetaCubeX 拉取机制与性能优化（awk vs bash） |
| 🤖 **CI/CD 参考** | GitHub Actions 每日同步工作流配置 |
| ⚠️ **常见陷阱** | include 递归丢失、JSON 破坏、CI 超时等 6 个已踩过的坑 |
| ✅ **检查清单** | 12 项提交前验证标准 |

---

## 🔗 相关资源

| 资源 | 链接 |
|------|------|
|📦 参考实现 | [Hawaiine/mihomo-rules](https://github.com/Hawaiine/mihomo-rules) |
|⚡ mihomo 内核 | [MetaCubeX/mihomo](https://github.com/MetaCubeX/mihomo) |
|📡 v2fly 域名数据 | [v2fly/domain-list-community](https://github.com/v2fly/domain-list-community) |
|🌐 Loyalsoldier 规则 | [Loyalsoldier/clash-rules](https://github.com/Loyalsoldier/clash-rules) |

---

<p align="center">
  <sub>Made with ❤️ for Hermes Agent · 同名 skill 已在本地安装</sub>
</p>