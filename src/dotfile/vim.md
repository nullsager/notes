_vimrc文件：
```
set guioptions-=T " 隐藏gvim的工具栏
set guioptions-=m " 隐藏gvim的菜单栏
set guioptions-=r " 隐藏gvim的右侧滚动条
set guioptions-=l " 隐藏gvim的左侧滚动条
set guioptions-=R " 隐藏gvim的右侧滚动条，当分割窗口时
set guioptions-=L " 隐藏gvim的左侧滚动条，当分割窗口时
let mapleader = " "

set encoding=utf-8
set fileencoding=utf-8
set number
set relativenumber
set tabstop=2        " Tab 键占用 4 个空格
set shiftwidth=2     " >> 和 << 命令每次缩进 4 个空格
set expandtab        " 使用空格替代 Tab
set autoindent       " 自动缩进
set smartindent      " 智能缩进
set guifont=FiraCode\ Nerd\ Font\ Mono:h12
syntax on
set nobackup
set nowritebackup
set noswapfile
set noshowmode
set hlsearch
set ignorecase        " 搜索时忽略大小写
set smartcase         " 当搜索关键字包含大写字母时，启用大小写敏感搜索
set nowrap
set wildmenu

inoremap jk <Esc>
nnoremap <leader>v :vsplit<CR>
nnoremap <leader>h :split<CR>
nnoremap <C-h> <C-w>h
nnoremap <C-j> <C-w>j
nnoremap <C-k> <C-w>k
nnoremap <C-l> <C-w>l
nnoremap <M-h> :wincmd H<CR>
nnoremap <M-j> :wincmd J<CR>
nnoremap <M-k> :wincmd K<CR>
nnoremap <M-l> :wincmd L<CR>
nnoremap [b :bprevious<CR>
nnoremap ]b :bnext<CR>

call plug#begin()
Plug 'morhetz/gruvbox'
Plug 'vim-airline/vim-airline'
Plug 'preservim/nerdtree'
Plug 'jiangmiao/auto-pairs'
Plug 'voldikss/vim-floaterm'
call plug#end()

" 配色
colorscheme gruvbox
set background=light

" nerdtree
nnoremap <leader>e :NERDTreeToggle<CR>
nnoremap <leader>o :NERDTreeFind<CR>

" let g:floaterm_width = 0.8
" let g:floaterm_height = 0.8
let g:floaterm_position = 'center'
nnoremap   <silent>   <F7>    :FloatermNew<CR>
tnoremap   <silent>   <F7>    <C-\><C-n>:FloatermNew<CR>
nnoremap   <silent>   <F8>    :FloatermPrev<CR>
tnoremap   <silent>   <F8>    <C-\><C-n>:FloatermPrev<CR>
nnoremap   <silent>   <F9>    :FloatermNext<CR>
tnoremap   <silent>   <F9>    <C-\><C-n>:FloatermNext<CR>
nnoremap   <silent>   <F12>   :FloatermToggle<CR>
tnoremap   <silent>   <F12>   <C-\><C-n>:FloatermToggle<CR>

let g:airline#extensions#tabline#enabled = 1
let g:airline#extensions#tabline#left_sep = ' '
let g:airline#extensions#tabline#left_alt_sep = '|'
```