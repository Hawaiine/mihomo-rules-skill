# mihomo-rules-workflow

通用 mihomo/clash-meta 兼容的 RULE-SET 规则集管理项目模板。

## 特性

- 从 v2fly/domain-list-community 和 Loyalsoldier/clash-rules 自动同步上游数据
- 递归解析 v2fly include 指令（最多 5 层深度）
- 自动发现并填充空 ruleset 文件
- 规则集格式自动校验与修复
- 多平台配置文件生成（Nikki/Clash for Android）
- GitHub Actions 每日同步 + Discord 通知

## 快速开始

```bash
# 1. 添加品牌规则集
echo "  - example.com" > ruleset/MyBrand

# 2. 自动校验修复
bash scripts/validate-ruleset.sh

# 3. 同步上游数据
bash scripts/sync-upstream.sh MyBrand

# 4. 生成配置文件
bash scripts/generate-config.sh

# 5. 提交
git add -A
git commit -m "✨ add MyBrand ruleset"
git push
```

## 目录结构

```
├── ruleset/              # 规则集 (.yaml)
├── configs/              # 平台配置
├── scripts/              # 自动化脚本
├── providers/            # 节点模板
└── .github/workflows/    # CI/CD 工作流
```

## 依赖

- bash 4.0+
- curl
- awk
- git
