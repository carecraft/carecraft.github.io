---
layout: post
layout:     post
title:      "打造 vim 编辑 C/C++ 环境"
category :  language-instrument
date:       2018-06-20
author:     "Max"
header-img: "img/post-2018.jpg"
catalog:    true
tags:
    - vim
    - linux
---

## 升级vim8

使用了 fedora 一个非官方源便捷升级。
```
# curl -L https://copr.fedorainfracloud.org/coprs/lantw44/vim-latest/repo/epel-7/lantw44-vim-latest-epel-7.repo -o /etc/yum.repos.d/lantw44-vim-latest-epel-7.repo
# yum update "vim*"
```
验证
```
# vim --version
VIM - Vi IMproved 8.1 (2018 May 17, compiled Jun 19 2018 22:44:37)
Included patches: 1-89
```

## 配置vim

### 1. 插件管理  [vim-plug](https://github.com/junegunn/vim-plug)

```
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

这样就可以在 `.vimrc` 中便捷添加插件了。示例：
```
" Specify a directory for plugins
" - Avoid using standard Vim directory names like 'plugin'
call plug#begin('~/.vim/plugged')

Plug 'tpope/vim-sensible'
Plug 'vim-airline/vim-airline'
Plug 'vim-airline/vim-airline-themes'
let g:airline_theme='light'

" Initialize plugin system
call plug#end()
```

定义好后使用 `:PlugInstall` 命令开始安装指定的插件，每次需要更新 vim-plug 时只需要 `:PlugUpgrade`。

vim-plug也支持延迟按需加载，等到使用到命令的时候再加载或者打开对应文件类型才加载。详情参见文档。

### 2. 符号索引 [Universal CTags](https://github.com/universal-ctags/ctags)

使用符号索引插件 ctags 以及 tags 异步生成工具 [vim-gutentags](https://github.com/ludovicchabant/vim-gutentags):
```
Plug 'universal-ctags/ctags'
Plug 'ludovicchabant/vim-gutentags'                                                                                                                                                          
" gutentags 搜索工程目录的标志，碰到这些文件/目录名就停止向上一级目录递归
let g:gutentags_project_root = ['.root', '.svn', '.git', '.hg', '.project']
" 所生成的数据文件的名称
let g:gutentags_ctags_tagfile = '.tags'
" 将自动生成的 tags 文件全部放入 ~/.cache/tags 目录中，避免污染工程目录
let s:vim_tags = expand('~/.cache/tags')
let g:gutentags_cache_dir = s:vim_tags
" 配置 ctags 的参数
let g:gutentags_ctags_extra_args = ['--fields=+niazS', '--extra=+q']
let g:gutentags_ctags_extra_args += ['--c++-kinds=+px']
let g:gutentags_ctags_extra_args += ['--c-kinds=+px']
" 检测 ~/.cache/tags 不存在就新建
if !isdirectory(s:vim_tags)
   silent! call mkdir(s:vim_tags, 'p')
endif
```

如果一个文件没有托管在 .git/.svn 中，gutentags 找不到工程目录的话，就不会为该野文件生成 tags，这也很合理。想要避免的话，你可以在你的野文件目录中放一个名字为 .root 的空白文件，主动告诉 gutentags 这里就是工程目录。

### 3. 编译运行 [AsyncRun](https://github.com/skywind3000/asyncrun.vim)

```
Plug 'skywind3000/asyncrun.vim'
" asyncrun 搜索工程目录的标志，碰到这些文件/目录名就停止向上一级目录递归。
" 如果递归到根目录还没找到，那么文件所在目录就被当作项目目录。
let g:asyncrun_rootmarks = ['.svn', '.git', '.root', '_darcs', 'build.xml'] 
" 自动打开 quickfix window ，高度为 6
let g:asyncrun_open = 6
" 任务结束时候响铃提醒
let g:asyncrun_bell = 1
" 设置 F10 打开/关闭 Quickfix 窗口
nnoremap <F10> :call asyncrun#quickfix_toggle(6)<cr>
" 设置 F7 从工程根目录编译整个工程
nnoremap <silent> <F7> :AsyncRun -cwd=<root> make <cr>
```


### 4. 代码检查 [ALE](https://github.com/w0rp/ale)

ALE（Asynchronous Lint Engine）支持多种语言的各种代码分析器，如 gcc、cppcheck 等，但需要另行安装并放入 PATH。ALE 能在你修改了文本后自动调用这些 linter 来分析最新代码，然后将各种 linter 的结果进行汇总并显示再界面上。

```
Plug 'w0rp/ale'
let g:ale_linters_explicit = 1
let g:ale_linters = {
  \   'csh': ['shell'],
  \   'zsh': ['shell'],
  \   'go': ['gofmt', 'golint'],
  \   'python': ['flake8', 'mypy', 'pylint'],
  \   'c': ['gcc', 'cppcheck'],
  \   'cpp': ['gcc', 'cppcheck'],
  \   'text': [],
  \}
let g:ale_completion_delay = 500
let g:ale_echo_delay = 20
let g:ale_lint_delay = 500
let g:ale_echo_msg_format = '[%linter%] %code: %%s'
let g:ale_lint_on_text_changed = 'normal'
let g:ale_lint_on_insert_leave = 1
let g:airline#extensions#ale#enabled = 1
let g:ale_c_gcc_options = '-Wall -O2 -std=c99'
let g:ale_cpp_gcc_options = '-Wall -O2 -std=c++11'
let g:ale_c_cppcheck_options = ''
let g:ale_cpp_cppcheck_options = ''
```

### 5. 修改比较[vim-signify](https://github.com/mhinz/vim-signify)

```
Plug 'mhinz/vim-signify'
let g:signify_vcs_list = [ 'git' ]
let g:signify_sign_show_text = 1
```

### 6. 语法高亮 [vim-cpp-enhanced-highlight](https://github.com/octol/vim-cpp-enhanced-highlight)

```
Plug 'octol/vim-cpp-enhanced-highlight'
let c_no_curly_error = 1
```

### 7. 编辑辅助 [vim-unimpaired](https://github.com/tpope/vim-unimpaired)

```
Plug 'tpope/vim-unimpaired'
```

unimpaired 插件定义了一系列括号开头的快捷键，被称为官方 Vim 中丢失的快捷键。详见插件文档。

### 8. 代码补全 [YouCompleteMe](https://github.com/Valloric/YouCompleteMe)
```
Plug 'Valloric/YouCompleteMe', { 'do': './install.py --clang-completer --go-completer' }
let g:ycm_global_ycm_extra_conf='~/.vim/plugged/YouCompleteMe/third_party/ycmd/examples/.ycm_extra_conf.py'
let g:ycm_add_preview_to_completeopt = 0
let g:ycm_show_diagnostics_ui = 0
let g:ycm_server_log_level = 'info'
let g:ycm_min_num_identifier_candidate_chars = 2
let g:ycm_collect_identifiers_from_comments_and_strings = 1
let g:ycm_complete_in_strings=1
let g:ycm_key_invoke_completion = '<c-z>'                    " 使用 Ctrl+Z 主动触发语义补全
noremap <c-z> <NOP>
set completeopt=menu,menuone

" 修改补全列表配色
highlight PMenu ctermfg=0 ctermbg=242 guifg=black guibg=darkgrey
highlight PMenuSel ctermfg=242 ctermbg=8 guifg=darkgrey guibg=black

" 对指定源文件，输入两个字母后即触发语义补全
let g:ycm_semantic_triggers =  {
           \ 'c,cpp,python,java,go,erlang,perl': ['re!\w{2}'],
           \ 'cs,lua,javascript': ['re!\w{2}'],
           \ }

let g:ycm_filetype_whitelist = { 
            \ "c":1,
            \ "cpp":1, 
            \ "go":1,
            \ "python":1,
            \ "sh":1,
            \ "zsh":1,
            \ }

let g:ycm_filetype_blacklist = {
        \ 'markdown' : 1,
        \ 'text' : 1,
        \ 'pandoc' : 1,
        \ 'infolog' : 1,
        \}
```

### 9. 参数提示 [echodoc](https://github.com/Shougo/echodoc.vim)

写 C/C++ 时函数忘了可以用上面的 YCM 补全，若是参数忘记了则可以使用 echodoc 插件。当用 YCM 的 tab 补全了一个函数名后，只要输入左括号，下面命令行就会显示出该函数的参数信息，并且随着光标移动高亮出当前参数位置。

```
Plug 'Shougo/echodoc.vim'
set noshowmode
```

### 10. 函数列表 [LeaderF](https://github.com/Yggdroot/LeaderF)

这里定义了 CTRL+P 在当前项目目录打开文件搜索，CTRL+N 打开 MRU 搜索，ALT+P 打开函数搜索，ALT+N 打开 Buffer 搜索，ALT+M 打开 Tag 搜索：
```
Plug 'Yggdroot/LeaderF', { 'do': './install.sh' }
let g:Lf_ShortcutF = '<c-p>'
let g:Lf_ShortcutB = '<m-n>'
noremap <c-n> :LeaderfMru<cr>
noremap <m-p> :LeaderfFunction!<cr>
noremap <m-n> :LeaderfBuffer<cr>
noremap <m-m> :LeaderfTag<cr>
let g:Lf_StlSeparator = { 'left': '', 'right': '', 'font': '' }
let g:Lf_RootMarkers = ['.project', '.root', '.svn', '.git']
let g:Lf_WorkingDirectoryMode = 'Ac'
let g:Lf_WindowHeight = 0.30
let g:Lf_CacheDirectory = expand('~/.vim/cache')
let g:Lf_ShowRelativePath = 0
let g:Lf_HideHelp = 1
let g:Lf_StlColorscheme = 'powerline'
let g:Lf_PreviewResult = {'Function':0, 'BufTag':0}
```

### 11. 文件浏览 [vim-dirvish](https://github.com/justinmk/vim-dirvish)

头文件和源文件之间可以使用 `:A` 命令快速切换，其它可以折腾下 dirvish。
```
Plug 'vim-scripts/a.vim'
Plug 'justinmk/vim-dirvish'
```

### 12. 其它

```
set autoindent             " Indent according to previous line.
set expandtab              " Use spaces instead of tabs.
set softtabstop =4         " Tab key indents by 4 spaces.
set shiftwidth  =4         " >> indents by 4 spaces.
set shiftround             " >> indents to next multiple of 'shiftwidth'.

set hidden                 " Switch between buffers without having to save first.
set display     =lastline  " Show as much as possible of the last line.

set ttyfast                " Faster redrawing.
set lazyredraw             " Only redraw when necessary.

set splitbelow             " Open new windows below the current window.
set splitright             " Open new windows right of the current window.

set cursorline             " Find the current line quickly.
set wrapscan               " Searches wrap around end-of-file.
set report      =0         " Always report changed lines.
set synmaxcol   =120       " Only highlight the first 200 columns.

set list                   " Show non-printable characters.
if has('multi_byte') && &encoding ==# 'utf-8'
  let &listchars = 'tab:▸ ,extends:❯,precedes:❮,nbsp:±'
else
  let &listchars = 'tab:> ,extends:>,precedes:<,nbsp:.'
endif

" Put all temporary files under the same directory.
let s:vim_backup = expand("$HOME/.vim/files/backup/")
if !isdirectory(s:vim_backup)
   silent! call mkdir(s:vim_backup, 'p')
endif
let s:vim_swap = expand("$HOME/.vim/files/swap/")
if !isdirectory(s:vim_swap)
   silent! call mkdir(s:vim_swap, 'p')
endif
let s:vim_undo = expand("$HOME/.vim/files/undo/")
if !isdirectory(s:vim_undo)
   silent! call mkdir(s:vim_undo, 'p')
endif
let s:vim_info = expand("$HOME/.vim/files/info/")
if !isdirectory(s:vim_info)
   silent! call mkdir(s:vim_info, 'p')
endif
set backup
set backupdir   =$HOME/.vim/files/backup/
set backupext   =-vimbackup
set backupskip  =
set directory   =$HOME/.vim/files/swap/
set updatecount =100
set undofile
set undodir     =$HOME/.vim/files/undo/
set viminfo     ='100,n$HOME/.vim/files/info/viminfo
```



## 参考

1. [知乎《如何在 Linux 下利用 Vim 搭建 C/C++ 开发环境?》](https://www.zhihu.com/question/47691414/answer/373700711)

