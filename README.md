# xsba-drxsfd-agent-by-hermes

Hermes Agent skill for Chinese criminal law research using [xsba.vip](http://xsba.vip/) and [drxsfd.com](http://www.drxsfd.com/).

## 内容

```
skills/
└── legal-research/
    ├── SKILL.md                             # 技能主体：数据库资源、检索策略、案例分析工作流
    └── references/
        ├── companion-skills.md              # 法律技能生态 — 配套技能目录
        ├── drxsfd-bh-full-mapping.md        # drxsfd.com 完整 bh 编号映射表（200+ 条）
        ├── github-auth.md                   # 更新本仓库时的 GitHub 认证说明
        └── site-exploration-methodology.md  # 中文法律站点探索方法论
```

### SKILL.md 涵盖

- **xsba.vip**（刑事司法办案业务资料汇编）完整路径速查表 — 8 大板块
- **drxsfd.com**（东人刑事法典）完整 bh 编号速查表（200+ 条）
- 免费 / 学术 / 实务三类法律数据库资源对照
- 类案检索策略与检索式模板
- 刑法案例分析工作流（法条 → 类案 → 量刑 → 理论 → 综合分析）
- 法律信息验证原则（时效性、效力层级、引证格式）
- GB2312 站点抓取技术说明

## 使用

克隆后把整个技能目录复制到 Hermes 的 skills 目录：

```bash
git clone https://github.com/MuQ411/xsba-drxsfd-agent-by-hermes.git /tmp/xsba
cp -r /tmp/xsba/skills/legal-research ~/.hermes/skills/
```

只取 SKILL.md（会缺少 references 中的补充材料）：

```bash
cp skills/legal-research/SKILL.md ~/.hermes/skills/legal-research/SKILL.md
```

## 两个站点

| 站点 | URL | 定位 |
|------|-----|------|
| xsba.vip | http://xsba.vip/ | 刑事办案实务资料汇编（案例、量刑、司法观点） |
| drxsfd.com | http://www.drxsfd.com/ | 东人刑事法典（法条 + 司法解释关联查询） |

两个站点均为 **GB2312** 编码，`web_extract` 可能解码失败。推荐用 `web_search` 的 `site:` 操作符，或 `curl` 抓取后转码：

```python
urllib.request.urlopen(url).read().decode('gb2312')
```

## 更新本仓库

推送需要 **Classic PAT**（`ghp_` 前缀）或 `gh` CLI。
Fine-grained PAT（`github_` 前缀）**不支持 Git HTTPS 推送**。

```bash
git clone https://MuQ411:<TOKEN>@github.com/MuQ411/xsba-drxsfd-agent-by-hermes.git
# 修改后
git add -A && git commit -m "..." && git push
```

详见 `skills/legal-research/references/github-auth.md`。
