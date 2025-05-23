# vscode 配置
## 键位映射
`gq`: 在v模式下将代码块进行压缩

`gb`: 在找到的下一个单词上添加另一个光标，该单词与光标下的单词相同

`af`: 在v模式下扩大选择范围

`gh`: 相当于将鼠标悬停在光标所在的位置。无需使用鼠标即可轻松查看类型和错误消息
## 具体配置
```json
{
  "window.titleBarStyle": "custom",
  "window.commandCenter": true,
  "editor.fontSize": 16,
  // 字体设置
  "editor.fontFamily": "'FiraCode Nerd Font' ,Consolas, 'Courier New', monospace",
  "editor.fontLigatures": true,
  // 图标设置
  "workbench.iconTheme": "material-icon-theme",
  // 缩进设置
  "[python]": {
    "editor.tabSize": 4
  },
  "editor.tabSize": 2,
  // 关闭左侧的缩略图
  "editor.minimap.enabled": false,
  // 粘滞滚动关闭 end
  "editor.stickyScroll.enabled": false,
  //////////////////////////////////////////////////////////////////////////////////////////
  // vim 设置
  // 相对行号
  "vim.leader": "<space>",
  "editor.lineNumbers": "relative",
  // 系统粘贴板
  "vim.useSystemClipboard": true,
  // 突出显示搜索文本
  "vim.hlsearch": true,
  // 高亮要粘贴的内容
  "vim.highlightedyank.enable": true,
  // insert模式映射
  "vim.insertModeKeyBindings": [
    {
      "before": [
        "j",
        "k"
      ],
      "after": [
        "<Esc>"
      ]
    },
  ],
  // normal模式映射
  "vim.normalModeKeyBindingsNonRecursive": [
    {
      "before": [
        "<Esc>"
      ],
      "commands": [
        ":nohl"
      ]
    },
    {
      "before": [
        "<C-h>"
      ],
      "commands": [
        "workbench.action.navigateLeft"
      ]
    },
    {
      "before": [
        "<C-j>"
      ],
      "commands": [
        "workbench.action.navigateDown"
      ]
    },
    {
      "before": [
        "<C-k>"
      ],
      "commands": [
        "workbench.action.navigateUp"
      ]
    },
    {
      "before": [
        "<C-l>"
      ],
      "commands": [
        "workbench.action.navigateRight"
      ]
    },
    {
      "before": [
        "<leader>",
        "e"
      ],
      "commands": [
        "workbench.action.toggleSidebarVisibility"
      ]
    },
    {
      "before": [
        "<leader>",
        "h"
      ],
      "commands": [
        "workbench.action.previousEditor"
      ]
    },
    {
      "before": [
        "<leader>",
        "l"
      ],
      "commands": [
        "workbench.action.nextEditor"
      ]
    },
    {
      "before": [
        "<leader>",
        "z"
      ],
      "commands": [
        "workbench.action.toggleZenMode"
      ]
    }
  ],
  // visual模式映射
  "vim.visualModeKeyBindings": [
    {
      "before": [
        ">"
      ],
      "commands": [
        "editor.action.indentLines"
      ]
    },
    {
      "before": [
        "<"
      ],
      "commands": [
        "editor.action.outdentLines"
      ]
    },
  ],
  "vim.handleKeys": {
    "<C-a>": false,
    "<C-f>": false,
    "<C-b>": false,
    "<C-p>": false,
  },
  // vim底部状态栏
  "vim.statusBarColorControl": true,
  "vim.statusBarColors.normal": [
    "#8FBCBB",
    "#434C5E"
  ],
  "vim.statusBarColors.insert": "#BF616A",
  "vim.statusBarColors.visual": "#B48EAD",
  "vim.statusBarColors.visualline": "#B48EAD",
  "vim.statusBarColors.visualblock": "#A3BE8C",
  "vim.statusBarColors.replace": "#D08770",
  "vim.statusBarColors.commandlineinprogress": "#007ACC",
  "vim.statusBarColors.searchinprogressmode": "#007ACC",
  "vim.statusBarColors.easymotionmode": "#007ACC",
  "vim.statusBarColors.easymotioninputmode": "#007ACC",
  "vim.statusBarColors.surroundinputmode": "#007ACC",
  // To improve performance
  "extensions.experimental.affinity": {
    "vscodevim.vim": 1
  },
  // 插件
  "vim.easymotion": true,
  //////////////////////////////////////////////////////////////////////////////////////////
  // latex
  "latex-workshop.latex.autoBuild.run": "never",
  "latex-workshop.showContextMenu": true,
  "latex-workshop.intellisense.package.enabled": true,
  "latex-workshop.message.error.show": false,
  "latex-workshop.message.warning.show": false,
  "latex-workshop.latex.tools": [
    {
      "name": "xelatex",
      "command": "xelatex",
      "args": [
        "-synctex=1",
        "-interaction=nonstopmode",
        "-file-line-error",
        "%DOC%"
      ]
    },
    {
      "name": "pdflatex",
      "command": "pdflatex",
      "args": [
        "-synctex=1",
        "-interaction=nonstopmode",
        "-file-line-error",
        "%DOC%"
      ]
    },
    {
      "name": "latexmk",
      "command": "latexmk",
      "args": [
        "-synctex=1",
        "-interaction=nonstopmode",
        "-file-line-error",
        "-pdf",
        "-outdir=%OUTDIR%",
        "%DOC%"
      ]
    },
    {
      "name": "bibtex",
      "command": "bibtex",
      "args": [
        "%DOC%"
      ]
    }
  ],
  "latex-workshop.latex.recipes": [
    {
      "name": "XeLaTeX",
      "tools": [
        "xelatex"
      ]
    },
    {
      "name": "PDFLaTeX",
      "tools": [
        "pdflatex"
      ]
    },
    {
      "name": "BibTeX",
      "tools": [
        "bibtex"
      ]
    },
    {
      "name": "LaTeXmk",
      "tools": [
        "latexmk"
      ]
    },
    {
      "name": "xelatex -> bibtex -> xelatex*2",
      "tools": [
        "xelatex",
        "bibtex",
        "xelatex",
        "xelatex"
      ]
    },
    {
      "name": "pdflatex -> bibtex -> pdflatex*2",
      "tools": [
        "pdflatex",
        "bibtex",
        "pdflatex",
        "pdflatex"
      ]
    },
  ],
  "latex-workshop.latex.clean.fileTypes": [
    "*.aux",
    "*.bbl",
    "*.blg",
    "*.idx",
    "*.ind",
    "*.lof",
    "*.lot",
    "*.out",
    "*.toc",
    "*.acn",
    "*.acr",
    "*.alg",
    "*.glg",
    "*.glo",
    "*.gls",
    "*.ist",
    "*.fls",
    "*.log",
    "*.fdb_latexmk"
  ],
  "latex-workshop.latex.autoClean.run": "onFailed",
  "latex-workshop.latex.recipe.default": "lastUsed",
  "latex-workshop.view.pdf.internal.synctex.keybinding": "double-click",
  "latex-workshop.view.pdf.viewer": "tab",
  "latex-workshop.view.pdf.ref.viewer": "auto",
  "explorer.confirmDelete": false,
  //////////////////////////////////////////////////////////////////////////////////////////
  "clangd.arguments": [
    "--header-insertion=never",
  ],
  "cmake.pinnedCommands": [
    "workbench.action.tasks.configureTaskRunner",
    "workbench.action.tasks.runTask"
  ],
  "cmake.showOptionsMovedNotification": false,
  "cmake.configureOnOpen": false,
  "cmake.autoBuildAfterConfigure": false,
  "cmake.buildOnSave": false,
  "workbench.colorTheme": "One Dark Pro",
  "workbench.colorCustomizations": {
    "statusBar.background": "#8FBCBB",
    "statusBar.noFolderBackground": "#8FBCBB",
    "statusBar.debuggingBackground": "#8FBCBB",
    "statusBar.foreground": "#434C5E",
    "statusBar.debuggingForeground": "#434C5E"
  },
  "[markdown]": {
    "editor.formatOnSave": false,
    "editor.renderWhitespace": "all",
    "editor.quickSuggestions": {
      "other": false,
      "comments": true,
      "strings": true
    },
    // "editor.snippetSuggestions": "top",
    // "editor.tabCompletion": "on",
    // "editor.acceptSuggestionOnEnter": "on",
    "editor.defaultFormatter": "mervin.markdown-formatter"
  },
  // 是否启用格式化
  "markdownFormatter.enable": true,
  // 将配置里的全角符号转化为半角符号, 例如 `，：；！“”‘’（）？。` 
  // 当设置为 '_' 或者 '' 的时候, 不转换符号
  // 当设置为 'auto' 的时候, 自动根据上下文转换符号
  "markdownFormatter.fullWidthTurnHalfWidth": "auto",
  // 转换代码块为代码区，默认为''（空字符）,不转换
  // 可以设置为任意合法英文
  // 只有设置成js或者javascript，才会按照js语言语法格式化
  "markdownFormatter.codeAreaToBlock": "",
  // 是否格式化代码
  // false: 不格式化代码
  "markdownFormatter.formatCodes": true,
  // 配置js-beautify
  // {}: beautifyjs的配置设置
  // 配置 `js-beautify` , 可以参考[这里](https://github.com/beautify-web/js-beautify)
  "markdownFormatter.formatOpt": {
    "indent_size": 2
  },
  // 格式化无序列表的符号 
  "markdownFormatter.formatULSymbol": true,
  // 各种符号后是否需要空格
  "markdownFormatter.spaceAfterFullWidthOrHalfWidth": "half",
  // 在文件首部显示时间
  "markdownFormatter.displayTime": false,
  // 是否格式化表格
  "markdownFormatter.formatTable": true,
  // 代表中文是英文的二倍宽度,用于表格对齐
  "markdownFormatter.formatTableOpt": {
    "chineseCharterWidth": 2
  },
  "leetcode.endpoint": "leetcode-cn",
}
```