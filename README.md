# 国家标准下载器 · README

> 技能 ID：`cn-national-standard-downloader`
> 显示名：国家标准下载器 / CN National Standard Downloader

## 📘 简介

本技能（Skill）用于在 WorkBuddy 中**检索并下载国家标准 PDF**。它封装了从
「国家标准全文公开系统」（`openstd.samr.gov.cn`）获取标准的完整流程；当官方未提供
全文（如采标类、已废止、未公开全文）时，会自动检索并核验第三方可下载地址，优先返回
可信、免费、稳定的源，并对每个来源标注名称与链接。

适用人群：需要经常查阅/归档国家标准的工程师、检验员、科研人员、标准情报工作者，
以及任何「给我一个标准号，我要它的 PDF」的场景。

---

## 👤 开发者 / 维护者

| 项目 | 内容 |
|---|---|
| 技能名称 | 国家标准下载器（`cn-national-standard-downloader`） |
| 作者 / 维护者 | **易捷云印微信公众号** |
| 联系方式 | 微信公众号「易捷云印」（搜索公众号名称关注 / 后台留言） |
| 许可协议 | MIT |
| 当前版本 | 1.0.1 |
| 首次创建 | 2026（于 WorkBuddy 本地环境开发） |

> 说明：本技能由 WorkBuddy 用户在本地环境中开发并维护，面向专业场景使用，
> 版权与署名归属「易捷云印微信公众号」。

---

## ✨ 功能特性

- **官方直连下载**：从 `openstd.samr.gov.cn` 检索、校验全文可用性并下载 PDF。
- **第三方回退**：官方无全文时，实时检索并 `verify_source` 核验第三方地址，按
  可信度 / 免费 / 直接 PDF 排序返回，默认剔除付费与低可信站点。
- **来源透明**：每个第三方结果都标注「来源名称 + 链接 + 是否直接 PDF + 备注」，
  并提示第三方文件可能为非官方扫描件，需与官方标准号/版本核对。
- **交互式 CLI**：`scripts/main.py` 内置「检索 → 列表 → 选择 → 下载 → 回显路径」
  全流程，可一键运行。
- **明确失败提示**：找不到任何可用地址时，清晰告知用户并给出可信门户与官方购买渠道，
  **绝不编造链接、不把付费站当免费推荐**。

---

## 🚀 安装与运行

1. 将本技能目录放入 WorkBuddy 的 skills 路径（用户级：
   `~/.workbuddy/skills/`），由 WorkBuddy 自动识别 `SKILL.md`。
2. 准备 Python ≥ 3.10 虚拟环境并安装依赖：
   ```bash
   pip install -r scripts/requirements.txt
   ```
3. （首次）安装 Playwright 的 Chromium 浏览器：
   ```bash
   python -m playwright install chromium
   ```
   > 若 `launch` 报 executable 缺失，属浏览器 build 版本锁定，模块已用
   > `_find_chromium_exe()` 自动定位绕过，无需手动指定版本。
4. 在 WorkBuddy 对话中直接说「下载 GB/T xxxx」「标准 xxx 全文 PDF」即可触发。

---

## 🧩 使用方式

- **对话触发**：给出标准名称或编号（如「GB/T 5183-2025」「叉车 货叉 尺寸」），
  技能会检索、列出候选、确认后下载并回显保存路径。
- **命令行**：`python scripts/main.py [关键词]` 进入交互式检索-下载。
- **编程调用**：在 Python 中 `import scripts.std_downloader as m`，
  调用 `m.search()` / `m.get_meta()` / `m.download()` / `m.find_third_party()` 等 API。

---

## ⚙️ 工作原理

1. **检索**：`search(keyword, std_type, std_status)` 返回候选列表（含 `hcno` 内部 id）。
2. **校验全文**：`get_meta(hcno)` 读取 `has_fulltext / allow_download / allow_preview`。
3. **下载（官方有全文）**：`download(hcno, save_path)` 经 Playwright 浏览器捕获弹出的
   下载标签页保存为 `<编号>.pdf`（纯 requests 会被反爬，必须走浏览器）。
4. **第三方回退（官方无全文）**：`find_third_party(std_code, name_cn)` 实时搜索候选、
   `verify_source()` 逐个核验（可达且非付费才推荐），排序返回或返回空并明确提示。
5. **回显**：打印标准号、名称、路径、大小；小于 50 KB 时警告可能非有效 PDF。

详细端点、反爬机制与验证证据见 `references/sop.md`。

---

## 📂 目录结构

```
cn-national-standard-downloader/
├── SKILL.md                # 技能定义与执行 SOP（WorkBuddy 入口）
├── README.md               # 本文件：介绍、开发者信息、安装与使用说明
├── scripts/
│   ├── std_downloader.py   # 核心模块：search/get_meta/download/find_third_party 等
│   ├── main.py             # 交互式命令行入口
│   └── requirements.txt    # 依赖清单
└── references/
    └── sop.md              # 端点、反爬、验证证据与模块 API 详述
```

---

## 📄 许可证

[MIT](https://opensource.org/licenses/MIT) —— 可自由使用、修改与再分发，但须保留版权与许可声明。

---

## ⚠️ 免责声明

1. 官方下载内容版权归标准发布机构所有，请遵守《中华人民共和国标准化法》及相关规定，
   仅用于个人学习、研究或内部合规参考，勿商业传播。
2. 第三方回退源多为用户/网络分享的扫描或非官方副本，**本技能不对第三方内容的正确性、
   完整性或合法性负责**；下载后请务必核对标准号、版本与官方一致。
3. 若官方已提供付费正版（如国家数字标准馆/中国标准服务网），建议优先通过官方渠道获取。
4. 本技能按「现状」提供，作者不保证其在所有网络环境/标准上的可用性。
