# Offline Contact-Prior Table Implementation Plan

> **面向 AI 代理的工作者：** 必需子技能：使用 superpowers:subagent-driven-development（推荐）或 superpowers:executing-plans 逐任务实现此计划。步骤使用复选框（`- [ ]`）语法来跟踪进度。

**目标：** Add a compact Phase/Progress/VPCE offline contact-prediction comparison to the one-page CoRL rebuttal and publish the verified source and PDF.

**架构：** Keep real-robot task success in the existing table and add a separate three-row offline-prior table under Q1. Shorten the Q1 prose, explicitly distinguish oracle offline progress from fixed-clock rollout conditioning, and leave unavailable metrics as the manuscript's intentional `TBD` entries.

**技术栈：** IEEEtran LaTeX, booktabs, Tectonic, Poppler (`pdfinfo`, `pdftotext`), Git.

---

### 任务 1：Add the offline contact-prior comparison

**文件：**
- 修改：`rebuttal_template.tex:54-63`

- [ ] **步骤 1：验证目标内容尚未存在**

运行：

```bash
! rg -q 'Offline contact-prior prediction' rebuttal_template.tex
```

预期：命令退出码为 0，证明新表尚未加入。

- [ ] **步骤 2：插入紧凑表格并收紧 Q1 文本**

在 Q1 标题后加入：

```latex
\begin{table}[h]
  \centering
  \footnotesize
  \caption{Offline future-contact prediction on held-out SWIPE trajectories.
  AUC/F1 measure the conditioning signal, not task success.}
  \label{tab:offline_prior}
  \begin{tabular}{lccc}
    \toprule
    Prior & Test-time source & AUC & F1 \\
    \midrule
    Phase    & Oracle trajectory phase$^\dagger$ & TBD & TBD \\
    Progress & Oracle trajectory progress$^\dagger$ & TBD & TBD \\
    VPCE     & RGB + robot motion & TBD & TBD \\
    \bottomrule
  \end{tabular}
\end{table}
```

Replace the Q1 discussion with text that says the offline oracle diagnostic and
fixed-clock policy rollout are distinct, and that VPCE is available online from
RGB and motion.

- [ ] **步骤 3：检查 LaTeX 源文件内容**

运行：

```bash
rg -n 'Offline future-contact|Oracle trajectory|fixed-clock|RGB and motion' rebuttal_template.tex
git diff --check
```

预期：四类文本均被找到，`git diff --check` 无输出并退出 0。

### 任务 2：Compile and enforce the one-page limit

**文件：**
- 更新：`rebuttal_template.pdf`
- 生成但不提交：`rebuttal_template.aux`、`rebuttal_template.log`

- [ ] **步骤 1：编译 LaTeX**

运行：

```bash
/home/hkust/miniconda3/envs/latex_cv/bin/tectonic rebuttal_template.tex
```

预期：退出码为 0，并生成 `rebuttal_template.pdf`。

- [ ] **步骤 2：验证一页限制和正文内容**

运行：

```bash
pdfinfo rebuttal_template.pdf | rg '^Pages:\s+1$'
pdftotext rebuttal_template.pdf - | rg 'Offline future-contact prediction'
pdftotext rebuttal_template.pdf - | rg 'Oracle trajectory phase'
pdftotext rebuttal_template.pdf - | rg 'RGB \+ robot motion'
```

预期：四条命令均成功，新 PDF 恰好一页。

- [ ] **步骤 3：检查编译日志**

运行：

```bash
! rg -n '^!|LaTeX Error|Undefined control sequence|Overfull \\vbox' rebuttal_template.log
```

预期：退出码为 0，不存在致命 LaTeX 错误或纵向溢页。

### 任务 3：Commit and publish

**文件：**
- 提交：`rebuttal_template.tex`
- 提交：`rebuttal_template.pdf`
- 提交：`docs/superpowers/plans/2026-08-09-offline-contact-table.md`

- [ ] **步骤 1：审计最终 diff**

运行：

```bash
git status --short
git diff --check
git diff -- rebuttal_template.tex
```

预期：只有计划、LaTeX 源文件和编译 PDF 是待提交的实质变更。

- [ ] **步骤 2：提交**

运行：

```bash
git add rebuttal_template.tex rebuttal_template.pdf docs/superpowers/plans/2026-08-09-offline-contact-table.md
git commit -m "docs: add offline contact-prior comparison"
```

预期：创建一个包含源文件、PDF 和计划的提交。

- [ ] **步骤 3：推送并验证远端**

运行：

```bash
git push origin main
git status --short --branch
git ls-remote --heads origin main
```

预期：推送成功，本地 `main` 与 `origin/main` 对齐，远端 main 指向本地 HEAD。
