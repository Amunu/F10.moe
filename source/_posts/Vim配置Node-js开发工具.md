title: Vim配置Node.js开发工具
date: 2014-10-20 12:54:58
tags: [vim, node]
---

vim 的基本操作就不涉及了，vim 在运行的时候会加载 **~/.vimrc** 文件里的配置，若不存在该文件可以手动创建。

### pathogen 

**~/.vim** 目录下是 vim 插件加载的位置。为了方便管理，首先先安装一个 vim 插件的管理器 [pathogen](https://github.com/tpope/vim-pathogen) 插件。按照常规把pathogen.vim安装在autoload文件夹下，这样vim就会自动加载插件的时候找到pathogen，然后我们把其余的插件都安装在bundle文件夹下，由pathogen路由给vim。

```
mkdir -p ~/.vim/autoload ~/.vim/bundle		#创建安装vim插件的文件夹bundle和antoload
git clone https://github.com/tpope/vim-pathogen.git #从git上同步下来pathogen项目
mv vim-pathogen/autoload/pathogen.vim autoload/		#把pathogen.vim移动到autoload文件夹下

```

这样pathogen就安装完毕。接下来需要[配置](https://github.com/tpope/vim-pathogen),按照readme粘贴复制就行。

### vim-node

安装node插件：

```
cd  ~/.vim/bundle
git clone https://github.com/moll/vim-node.git ~/.vim/bundle/node

```

### vimrc

对于vimrc的配置也一同配了。下面是我的配置, 每个配置旁都有注释。最好是一条条看下来，毕竟每个人的配置不会一模一样。

```
execute pathogen#infect()

filetype plugin indent on
set bs=2  " 在insert模式下用退格键删除
syntax enable

colorscheme monokai " 设置主题,这里我用的是monokai。具体的可以见后面的解释
set number " 显示行号

set cursorline " 突出显示当前行
set ruler " 打开状态栏标尺
set shiftwidth=4 " 设定 << 和 >> 命令移动时的宽度为 4
set softtabstop=4 " 使得按退格键时可以一次删掉 4 个空格
set tabstop=4 " 设定 tab 长度为 4
set nobackup " 覆盖文件时不备份
set autochdir " 自动切换当前目录为当前文件所在的目录
set backupcopy=yes " 设置备份时的行为为覆盖
set ignorecase smartcase " 搜索时忽略大小写，但在有一个或以上大写字母时仍保持对大小写敏感
set nowrapscan " 禁止在搜索到文件两端时重新搜索
set incsearch " 输入搜索内容时就显示搜索结果
set hlsearch " 搜索时高亮显示被找到的文本
set showmatch " 插入括号时，短暂地跳转到匹配的对应括号
set matchtime=2 " 短暂跳转到匹配括号的时间
set magic " 设置魔术
set hidden " 允许在有未保存的修改时切换缓冲区，此时的修改由 vim 负责保存
set guioptions-=T " 隐藏工具栏
set guioptions-=m " 隐藏菜单栏
set smartindent " 开启新行时使用智能自动缩进
set backspace=indent,eol,start " 不设定在插入状态无法用退格键和 Delete 键删除回车符
" set cmdheight=1 " 设定命令行的行数为 1
set laststatus=2 " 显示状态栏 (默认值为 1, 无法显示状态栏)
set statusline=\ %<%F[%1*%M%*%n%R%H]%=\ %y\ %0(%{&fileformat}\ %{&encoding}\ %c:%l/%L%)\  " 设置在状态行显示的信息
set foldenable " 开始折叠
set foldmethod=marker " 设置语法折叠
set foldcolumn=0 " 设置折叠区域的宽度
setlocal foldlevel=1 " 设置折叠层数为
set foldclose=all " 设置为自动关闭折叠 
nnoremap <space> @=((foldclosed(line('.')) < 0) ? 'zc' : 'zo')<CR> 
" 用空格键来开关折叠

```

### smartindent

在上面的配置中，我们设置了 ``set smartindent 开启新行时使用智能自动缩进`` ，这时候如果粘贴了一段已经缩进过的代码，结果会惨不忍睹。
所以当我们要粘贴文本到 vim 前，执行 ``:set paste``，粘贴完后再恢复 ``:set paste!`` 。

### colorscheme

Vim的颜色主题在 **/usr/share/vim/vim73/colors** 文件夹里。
打开 vim 后在 normal 模式下输入 ``：colorscheme`` 查看当前的主题，修改主题使用命令 ``：colorscheme mycolor`` ，其中 mycolor 是你usr/share/vim/vim73/colors文件夹包含的文件名。
也可以把这个命令写入~/.vimrc配置文件中，(也就是上面的 ``colorscheme monokai ``)这样每次打开Vim都是你设定的主题。

再说说这里的[monokai](https://github.com/sickill/vim-monokai) 主题，由于我比较喜欢 **sublime** 的配色，所以这个主题也比较像sublime的主题。

### foldmethod

折叠效果，对应上面配置的：

```
set foldmethod=marker " 设置语法折叠
set foldcolumn=0 " 设置折叠区域的宽度
setlocal foldlevel=1 " 设置折叠层数为
set foldclose=all " 设置为自动关闭折叠 
nnoremap <space> @=((foldclosed(line('.')) < 0) ? 'zc' : 'zo')<CR>  " 用空格键来开关折叠

```

``set foldmethod`` 可以设置折叠模式(下面有6种模式)：

- **manual**		手动建立折叠。
- **indent**		相同缩进距离的行构成折叠。
- **expr**			用表达式来定义折叠，'foldexpr' 给出每行的折叠级别。
- **marker**		标志用于指定折叠。
- **syntax**		语法高亮项目指定折叠。
- **diff**			没有改变的文本构成折叠。

每个模式有不同的特点，我这里用的 **marker** ，其他模式可以去了解一下 。

具体的代码折叠使用方法可以[戳这里](http://scmbob.org/vim_fdm.html)。由于某些特别原因就不再次写了。
