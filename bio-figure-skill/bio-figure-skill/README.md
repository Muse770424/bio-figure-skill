# Bio-Figure Skill

让 Claude 直接从实验数据（CSV/Excel）生成符合 Nature / Cell / Science 投稿标准的科研图表。

## 支持的图表类型

- **柱状图** — 自动叠加 jitter 散点、误差棒、显著性标注
- **散点图** — 线性拟合 + 95% CI、火山图（volcano plot）
- **热图** — z-score / 原始值色板、聚类 dendrogram、分组色带
- **折线图** — 时间序列 / 剂量效应、误差带（fill_between）

## 核心特性

- NPG 风格配色（源自 R 包 [ggsci](https://github.com/nanxstats/ggsci)，低饱和、高区分度；非 Nature 官方色板）
- 永远显示原始数据点（禁止纯均值柱状图）
- 自动选择统计检验并标注显著性（\* / \*\* / \*\*\* / \*\*\*\*）
- 300 DPI TIFF/PNG 输出，单栏 89mm / 双栏 183mm
- 无衬线字体（Arial/Helvetica），去掉 top/right 边框

## 安装方法

### 方法一：Claude Desktop App / Cowork

1. 打开 **Settings → Skills**
2. 点击 **Add Skill** 或 **Import Skill**
3. 选择这个文件夹（包含 `SKILL.md` 的那个）
4. 技能会出现在列表中，名称为 **bio-figure**

### 方法二：Claude Code（命令行）

```bash
# 用户级（所有项目可用）
cp -r bio-figure-skill/ ~/.claude/skills/bio-figure/

# 项目级（仅当前仓库可用）
cp -r bio-figure-skill/ .claude/skills/bio-figure/
```

下次启动 session 即自动加载。

## 使用示例

安装后，给 Claude 发数据并说要画什么图即可：

- "这是我的 CSV 数据，画个柱状图比较三组差异"
- "用这个 DESeq2 结果画火山图"
- "画一个基因表达热图，按行聚类"
- "画四组细胞 72 小时生长曲线"

Claude 会按流程执行：审数据 → 确认方案 → 写代码 → 出图 → 自查。

## 依赖

生成的代码使用标准 Python 数据科学包：

- Python 3.8+
- matplotlib / seaborn / numpy / pandas
- scipy（统计检验）

## 文件结构

```
bio-figure-skill/
├── SKILL.md    ← 技能定义（YAML frontmatter + 完整指令）
├── README.md   ← 本文件
└── LICENSE     ← MIT 许可证
```

## 许可证

本项目以 [MIT 许可证](LICENSE) 开源。你可以自由使用、修改和再分发，只需保留原作者的版权声明和许可证文本。

## 致谢

默认配色取自 R 包 [ggsci](https://github.com/nanxstats/ggsci) 的 `npg` 色板。
