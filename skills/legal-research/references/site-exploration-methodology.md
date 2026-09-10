# Site Exploration Methodology for Chinese Legal Databases

## Overview

How to systematically discover the structure of undocumented Chinese legal sites (GB2312-encoded, no API, no sitemap) and build lookup tables for skills.

## General Approach

### 1. Identify encoding first
Chinese legal sites from mainland China often use GB2312/GBK, not UTF-8. Always try multiple encodings:
```python
for enc in ['gb2312', 'gbk', 'gb18030', 'utf-8']:
    try: return raw.decode(enc)
    except: continue
```

### 2. Fetch via terminal curl, not browser
- Browser tools may be blocked by proxy (ERR_BLOCKED_BY_CLIENT)
- `web_extract` may fail due to encoding mismatch
- `terminal curl` + Python decode is the most reliable path
- `web_search site:domain` works for discovery but gives truncated snippets

### 3. Link extraction for structure mapping
```python
import re
pattern = re.compile(r'<a[^>]*href=["\']([^"\']+)["\'][^>]*>(.*?)</a>', re.DOTALL)
```

## Site-Specific Patterns

### drxsfd.com — Sequential bh Number Scanning

**Pattern**: Every article/section has a numeric `bh` parameter.
- URL: `http://www.drxsfd.com/xf/xx.asp?bh={N}` (article text)
- URL: `http://www.drxsfd.com/xf/xx_sj.asp?bh={N}` (article + sentencing)

**Discovery method**: Loop through `bh` from 1 upward. The numbering roughly follows the Criminal Law article order:
- bh 1–12: Chapter 1 (危害国家安全罪)
- bh 13–58: Chapter 2 (危害公共安全罪)
- bh 59–165: Chapter 3 (破坏社会主义市场经济秩序罪)
- bh 166–203: Chapter 4 (侵犯公民人身权利罪)
- bh 204–215: Chapter 5 (侵犯财产罪)
- bh 216–344: Chapter 6 (妨害社会管理秩序罪)
- bh 345–366: Chapter 7 (危害国防利益罪)
- bh 367–377: Chapter 8 (贪污贿赂罪)
- bh 379–413: Chapter 9 (渎职罪)
- bh 414–444: Chapter 10 (军人违反职责罪)
- bh 485–506: General Provisions (总则)

Density ~95% (nearly every bh maps to content). Time: ~200 requests for full scan.

### xsba.vip — Directory Tree Traversal

**Pattern**: Hierarchical directory structure with descriptive filenames.
- No numeric IDs — use link extraction to map the tree
- Start from index pages: `/sfjs/sfjs.htm`, `/xfzm/xfzm.htm`, `/xsal/xsal.htm`, etc.
- Each section has its own subdirectory conventions:
  - `/sfjs/` — judicial interpretations by year/category/crime
  - `/xfzm/zm01/` through `/xfzm/zm10/` — crime commentary by chapter
  - `/xfzm/xfzz/` — general provisions interpretation
  - `/xsal/xfal/` — cases by category
  - `/xfsw/zdnd/` — judicial practice viewpoints
  - `/dfgd/labz/` — local standards (Jiangsu)

**Key discovery**: The site maintains parallel structure across sections:
- `/sfjs/sfjs(fz04).htm` = judicial interpretations for Chapter 4 crimes
- `/xsal/xfal/xsal(fz04).htm` = cases for Chapter 4 crimes
- `/xfsw/xfsw(fz04).htm` = judicial viewpoints for Chapter 4 crimes
- `/dfgd/dfgd(fz04).htm` = local regulations for Chapter 4 crimes

### yuanli.ailaw.cn — API Skill Downloads

**Pattern**: Skills served as ZIP files (sometimes with `.tar.gz` extension).
```bash
curl -sL -o skill.zip "https://yuanli.ailaw.cn/api/skills/{name}/download"
# If file says "Zip archive" but has .tar.gz extension:
python3 -c "import zipfile; zipfile.ZipFile('skill.zip').extractall('out')"
```

Skills may be bundles with nested sub-skills in `skills/` directory. Install both the parent directory and copy sub-skills to top-level.

## Performance Notes

- Scan 200 bh numbers from drxsfd.com: ~50 seconds
- Scan 8 index pages from xsba.vip: ~10 seconds
- Use `timeout=8` for individual requests, batch with `% 20` progress logging
