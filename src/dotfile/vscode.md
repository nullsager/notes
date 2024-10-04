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
  "workbench.colorCustomizations": {
    "statusBar.background": "#BF616A",
    "statusBar.noFolderBackground": "#BF616A",
    "statusBar.debuggingBackground": "#BF616A",
    "statusBar.foreground": "#434C5E",
    "statusBar.debuggingForeground": "#434C5E"
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
}
```