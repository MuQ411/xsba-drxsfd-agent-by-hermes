# 法律技能生态 — 配套技能目录

本目录记录与 `legal-research` 协同使用的法律技能。当检索到法条/案例后，可路由到以下技能完成后续工作。

## 已安装技能一览

### 刑法案例分析
| 技能 | 来源 | 用途 |
|------|------|------|
| `legal-research` | 本地创建 | 刑法法条/案例/司法解释检索（xsba.vip + drxsfd.com） |

### 民法案例分析
| 技能 | 来源 | 用途 |
|------|------|------|
| `gutachten-civil-case` | github.com/Youchu-lawhub | 鉴定式民法案例研习（请求权基础+ Gutachten 格式）v3.0.0 |

### 法律文书生成
| 技能 | 来源 | 用途 |
|------|------|------|
| `legal-document-generator` | yuanli.ailaw.cn | 泛法律文书生成（合同/诉状/辩护词/律师函/法律意见书） |
| `contract-gen` | github.com/zh-xx | 合同生成（支持立场倾斜） |
| `contract-review` | github.com/zh-xx | 合同审核（四层审查：主体/基础/商业/法律） |

### 合规审核
| 技能 | 来源 | 用途 |
|------|------|------|
| `ad-compliance-review` | github.com/zh-xx | 广告合规审核（广告法/反不正当竞争法） |
| `food-label-review` | github.com/zh-xx | 食品标签合规（GB 7718 / GB 28050，2011/2025双版） |

### 法律可视化
| 技能 | 来源 | 用途 |
|------|------|------|
| `legal-visualization` | yuanli.ailaw.cn | 法律图表化总控（判决书/案件关系/时间线/证据链）v1.1.0 |
| ├─ `drawio` | 捆绑子技能 | 正式结构图（draw.io 格式） |
| └─ `excalidraw-diagram-generator` | 捆绑子技能 | 手绘白板图（Excalidraw 格式） |
| `legal-architecture` | github.com/zh-xx | 法律结构可视化（SVG/HTML 输出） |
| `legal-risk-visualization` | github.com/zh-xx | 法律风险可视化（风险矩阵/雷达图/因果图） |

### 法学学习
| 技能 | 来源 | 用途 |
|------|------|------|
| `law-student-session` | yuanli.ailaw.cn | 法学学习统一入口（路由到以下12个技能） |
| `law-student-bar-prep-questions` | yuanli.ailaw.cn | 法考题目练习（客观题/主观题） |
| `law-student-case-brief` | yuanli.ailaw.cn | 案例摘要批改 |
| `law-student-cold-call-prep` | yuanli.ailaw.cn | 课堂提问预演 |
| `law-student-cold-start-interview` | yuanli.ailaw.cn | 首次使用个性化访谈 |
| `law-student-customize` | yuanli.ailaw.cn | 学习设定调整 |
| `law-student-exam-forecast` | yuanli.ailaw.cn | 历年试题规律分析 |
| `law-student-flashcards` | yuanli.ailaw.cn | 记忆卡片（Leitner间隔重复） |
| `law-student-irac-practice` | yuanli.ailaw.cn | IRAC/鉴定式案例分析评析 |
| `law-student-legal-writing` | yuanli.ailaw.cn | 法律写作批改 |
| `law-student-outline-builder` | yuanli.ailaw.cn | 课程知识大纲构建 |
| `law-student-socratic-drill` | yuanli.ailaw.cn | 苏格拉底式追问练习 |
| `law-student-study-plan` | yuanli.ailaw.cn | 法考长期备考计划 |

### 工具类
| 技能 | 来源 | 用途 |
|------|------|------|
| `law-to-markdown` | github.com/zh-xx | 法条/规范文件转 Markdown（.txt/.docx/.pdf） |
| `legal-job-search` | github.com/zh-xx | 法律AI求职助手（律所/公司调研、简历生成） |

## 技能下载源

| 来源 | URL | 格式 |
|------|-----|------|
| yuanli.ailaw.cn API | `https://yuanli.ailaw.cn/api/skills/{name}/download` | ZIP（可能伪装为 .tar.gz 扩展名） |
| yuanli.ailaw.cn 工具包 | `https://yuanli.ailaw.cn/api/toolkits/{name}/download` | ZIP + install.py |
| zh-xx GitHub | `https://github.com/zh-xx/legal-assistant-skills` | Git 仓库 |
| Youchu-lawhub GitHub | `https://github.com/Youchu-lawhub/gutachten-civil-case` | Git 仓库 |

## 安装方法

### yuanli.ailaw.cn 技能
```bash
# 下载并解压（注意：扩展名可能是 .tar.gz 但实际是 ZIP）
curl -sL "https://yuanli.ailaw.cn/api/skills/{name}/download" -o skill.zip
python3 -c "import zipfile; zipfile.ZipFile('skill.zip').extractall('/tmp/skill')"
# 安装
cp -r /tmp/skill/* ~/.hermes/skills/{name}/
```

### yuanli.ailaw.cn 工具包
```bash
curl -sL "https://yuanli.ailaw.cn/api/toolkits/{name}/download" -o toolkit.zip
python3 -c "import zipfile; zipfile.ZipFile('toolkit.zip').extractall('/tmp/toolkit')"
cd /tmp/toolkit && python3 install.py --target ~/.hermes/skills
```

### GitHub 仓库
```bash
git clone https://github.com/{user}/{repo}.git /tmp/repo
# 安装全部技能
for dir in /tmp/repo/*/; do
  [ -f "$dir/SKILL.md" ] && cp -r "$dir" ~/.hermes/skills/$(basename "$dir")
done
```

## 工作流建议

1. **刑法案例分析**：`legal-research`（检索法条/案例） → 输出事实和法律依据 → 手动分析或结合 `gutachten-civil-case` 方法
2. **法考备考**：`law-student-study-plan`（制定计划） → `law-student-session`（每日练习）
3. **法律文书**：`legal-research`（查法条依据） → `legal-document-generator`（生成文书）
4. **合同工作**：`contract-review`（审核） → `contract-gen`（生成修改版）
