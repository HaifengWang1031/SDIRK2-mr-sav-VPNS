# 编译与 Skim 联动（macOS）

## 项目结构与编译方式

- `ex_article.tex`：主文。
- `ex_supplement.tex`：补充材料，与主文通过 `\externaldocument` 双向引用。
- `ex_shared.tex`：两者共用的宏包和元信息，已指定主文为 LaTeX Workshop 的根文件。
- `references.bib` + `siamplain.bst`：BibTeX 参考文献。
- 已移除未参与论文编译的 SIAM 模板手册、示例图片及 `data.dat`。
- `.gitignore`：排除可重新生成的 PDF、SyncTeX 和 LaTeX 辅助文件。

使用 pdfLaTeX，由 latexmk 自动调用 BibTeX 并重复编译。PDF、辅助文件和
`.synctex.gz` 保留在项目根目录，便于两份文档互相读取 `.aux`。
EPS 图片由 TeX Live 的 epstopdf 在编译时转换。

在项目目录执行（任一步失败即停止）：

```sh
for target in ex_article ex_supplement ex_article ex_supplement; do
  /Library/TeX/texbin/latexmk -pdf -synctex=1 \
    -interaction=nonstopmode -file-line-error -halt-on-error "$target.tex" || break
done
```

主文、补充材料依次各编译两轮，以更新相互引用；每轮内部的重复编译由 latexmk 管理。

## VS Code

用 VS Code 打开整个项目目录，扩展推荐写在 `.vscode/extensions.json`。
本机已有 LaTeX Workshop 10.18.0、TeX Live 2025 和 Skim。

`.vscode/settings.json` 配置了：

- 已关闭保存时自动编译；手动触发编译时，默认联合编译主文与补充材料。
- 整个编译流程成功后，自动清理主文和补充材料的辅助文件（含 `.aux`、`.bbl`、`.blg`、`.log`、`.thm` 等）；失败时保留日志。PDF 与 `.synctex.gz` 保留，Skim 同步继续可用。清理后请使用默认联合编译流程重建双向引用；单文件 recipe 不会自动补建另一份文档的 `.aux`。此自动清理仅适用于 LaTeX Workshop，终端命令不触发。
- 另一个 recipe `latexmk: current root (pdfLaTeX / BibTeX)` 用于独立编译当前根文件。
- 使用 Skim 显示 PDF，编译后在后台同步到光标所在位置。
- 显式指定 TeX 工具路径，避免从 Dock 启动 VS Code 时找不到命令。

默认快捷键：

| 操作 | 快捷键 |
| --- | --- |
| 编译 | `⌘⌥B` |
| 在 Skim 打开 PDF | `⌘⌥V` |
| 源码位置 → PDF | `⌘⌥J` |
| Skim PDF → 源码 | 在正文上 `⇧⌘` + 单击 |

也可以从命令面板使用 LaTeX Workshop 的 Build、View、SyncTeX 命令。

## Skim 的反向同步

反向同步属于 **Skim 全局偏好**，无法只靠项目的 settings.json 设置。
本机已写入以下设置，并开启自动检测和重新加载 PDF：

- Preset：Visual Studio Code
- Command：`/usr/local/bin/code`
- Arguments：`--reuse-window --goto "%file:%line"`

如果 Skim 原先已运行且未采用新设置，请正常退出后重新打开；也可以在
Skim → Settings/Preferences → Sync 中检查以上配置。
其他电脑需在该面板选择 Visual Studio Code，或按实际路径设置自定义命令。
修改前的偏好备份位于 `/tmp/vpns-skim-preferences-before.plist`（临时文件）。

## SIAM 2022 类与新版 LaTeX 的兼容

实际编译曾在 theorem 处出现：

```text
Use of \cref@override@label@type doesn't match its definition.
```

`siamart220329.cls` 重定义了内核的 `\label` 和 `\refstepcounter`，破坏了新版
LaTeX/hyperref/cleveref 的引用处理。`siam-compat.tex` 在加载类前保存内核命令，
并在 hyperref 加载前恢复；两份论文入口均在 `\documentclass` 前载入该补丁。
原始 `.cls` 未修改。将来升级 SIAM 模板后，应重新检查是否仍需要此补丁。

## 验证范围

主文和补充材料均已实际编译，最终日志无未定义引用；已检查 SyncTeX 正向位置映射。
图形界面的快捷键及鼠标反向跳转尚未实际操作验证。

配置参考：[LaTeX Workshop 官方 Skim 文档](https://github.com/James-Yu/LaTeX-Workshop/wiki/View#macos)。
