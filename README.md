# nvim.bak

> **已归档（Archived）· 仅作留档，不再维护。**
> 这是把 LazyVim 默认行为“去包装”后的一份完整 Neovim 配置。作者计划从零重写一份新配置作为后续项目，
> 因此本仓库冻结在这里，供日后参考。

一份自建的 Neovim 配置：不依赖 LazyVim/LazyVim 仓库，去掉 `LazyVim.*` 抽象与 extras 机制，
只保留原生 `vim.opt` / `vim.keymap.set` / `nvim_create_autocmd` 与普通 lazy.nvim spec。

```
init.lua ──> lua/config/lazy.lua ──> lua/plugins/{core,ui,editor,…}
                                     lua/plugins/lang/{lua,python,…}
```

## 分支

三个分支均已推送并冻结：

| 分支 | 说明 |
|---|---|
| `main` | 去 LazyVim 后的稳定线（默认分支） |
| `feat/system-toolchain` | 在 `main` 之上把 mason 换成系统工具链（Arch + 本机判定） |
| `feat/native-statusline` | 用 Neovim 原生 statusline 替换 lualine 的实验 |

## 环境

- Neovim **0.12.x**（按 0.12.5 写死；不保留跨版本分支）
- **仅 Linux**（Windows/macOS 分支已删除）
- Wayland（niri）+ kitty / Ghostty；背景透明由终端 opacity 与高亮处理协作
- 需要 Nerd Font 才能显示图标
- 插件由 lazy.nvim 管理，首次启动自动 clone（需要 git / ripgrep / C 编译器 / tree-sitter-cli 等系统依赖）

## 目录

```
~/.config/nvim/
├── init.lua                  # 只做 bootstrap，转 require("config.lazy")
├── lazy-lock.json            # 插件版本锁
└── lua/
    ├── config/               # 启动骨架
    │   ├── lazy.lua          # lazy.nvim setup（spec 来自 modules.lua）
    │   ├── modules.lua       # 唯一的功能 / 语言模块开关
    │   ├── options.lua       # vim.opt
    │   ├── keymaps.lua       # <leader> 键位
    │   ├── autocmds.lua      # 原生 autocmd
    │   ├── theme.lua         # 主题 / 变体 / 透明选项一张表
    │   ├── icons.lua         # 图标 + kind_filter
    │   └── health.lua        # 自定义 :checkhealth config
    ├── plugins/              # 功能模块，一个文件一块
    │   ├── core.lua ui.lua editor.lua completion.lua picker.lua explorer.lua
    │   ├── coding.lua treesitter.lua lsp.lua formatting.lua linting.lua shell.lua util.lua
    │   └── lang/             # 语言模块，一个文件一种语言
    └── util/                 # 替代 LazyVim.* 的本地工具库
        ├── transparency.lua  # 背景透明总表 + 运行时开关
        ├── styles.lua        # 字体样式层（斜体 / 粗体…）
        ├── keytrace.lua      # 键位覆盖追踪（health 用）
        └── root.lua format.lua lsp.lua lspkeys.lua cmp.lua mini.lua treesitter.lua …
```

## 设计取向

- **单一开关**：`lua/config/modules.lua` 用两组布尔值控制所有 feature / lang 模块是否加载；关掉只是不加载，文件仍在。
- **语言自报需求**：每个 `lua/plugins/lang/*.lua` 顶部 `register_lang()` 声明其 LSP / parser / formatter / linter，
  `:checkhealth config` 据此给出「语言就绪度」。
- **系统工具链优先**：`util/init.lua` 的 `system_toolchain()`（`/etc/arch-release` ∧ 本机 hostname 白名单）为真时，
  `mason.nvim` 直接 `enabled = false`（不加载也不下载），工具改走 pacman / AUR / 自维护 PKGBUILD；其他机器行为不变。见 `SYSTEM-TOOLS.md`。
- **主题与透明**：`config/theme.lua` 一张表管理 catppuccin / tokyonight / everforest 的变体与各自的透明选项，
  `<leader>uC` 运行时切换并记忆；背景透明另由 `util/transparency.lua` 的总表 + 快照机制负责，`<leader>uT` 开关，切主题后精确恢复。见 `TRANSPARENCY.md`。
- **键位自检**：`util/keytrace.lua` 在任何映射注册前先跑，记录每一次 `keymap.set`；`:checkhealth config` 报告覆盖、
  缺 `desc`、「既是动作又是前缀」。防误触：`q` 禁用，宏录制改到 `Q`。
- **自带健康检查**：`:checkhealth config` 覆盖模块登记、键位自洽、语言就绪度、工具链、终端 / 复用器等。

## 主要插件

- 基础 / UI：lazy.nvim、snacks.nvim、nui.nvim、noice.nvim、bufferline.nvim、mini.icons、trouble.nvim、
  todo-comments.nvim、nvim-treesitter-context、persistence.nvim、which-key.nvim
- 编辑：mini.ai、mini.pairs、mini.surround、mini.animate、flash.nvim、yanky.nvim、dial.nvim、grug-far.nvim、
  harpoon、ts-comments.nvim、vim-illuminate、inc-rename.nvim
- 补全 / LSP：blink.cmp、friendly-snippets、nvim-lspconfig、clangd_extensions.nvim、cmake-tools.nvim、
  venv-selector.nvim、nvim-navic、lazydev.nvim、SchemaStore.nvim（rust / scala 模块默认关闭：rustaceanvim / nvim-metals / crates.nvim）
- treesitter：nvim-treesitter、nvim-treesitter-textobjects、nvim-ts-autotag
- 格式化 / lint：conform.nvim、nvim-lint
- 主题：catppuccin、tokyonight.nvim、everforest-nvim（默认 catppuccin / frappe）
- Git / 其他：gitsigns.nvim、direnv.nvim、plenary.nvim、vim-dadbod(-ui, -completion)

> `main` 仍用 lualine；`feat/system-toolchain` 在本机停用 mason；`feat/native-statusline` 已移除 lualine。

## 文档

| 文件 | 内容 |
|---|---|
| `PLAN.md` | 「去 LazyVim」计划原文（历史） |
| `AUDIT.md` | 全量审计报告快照，含键位 / 自动命令冲突扫描 |
| `CHEATS.md` | 快捷键手册（运行时 dump 后人工归类） |
| `TRANSPARENCY.md` | 背景透明的机制、硬规则、诊断流程 |
| `SYSTEM-TOOLS.md` | mason → 系统工具链的决策与「工具→包名」对照（`feat/system-toolchain`） |
| `TODO.md` | 待办与决策记录 |
| `.dsh/skills/nvim-transparency/SKILL.md` | 透明化问题的诊断 SOP（DSH skill） |
| `STATUSLINE.md` | 原生状态栏实验记录（`feat/native-statusline`） |

## 为什么归档

不再在本仓库上继续迭代，而是从零重写。主要原因：

1. 想借重写重新整理结构，而不是在旧历史上继续叠加；
2. 现有提交历史无法在**不引入 ssh-agent** 的前提下，用当前签名方式（FIDO2 resident SK）批量重签；
   保留一段签名不一致的历史不如新开仓库——新仓库从第一次提交起就是签名过的。

## 许可

个人配置，未附许可证。插件 / 主题版权归各自上游；结构参考了 [LazyVim](https://github.com/LazyVim/LazyVim)（Apache-2.0）。
