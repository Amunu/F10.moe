title: vim 配置 NERDTree
date: 2014-10-29 11:00:13
tags: [vim, NERDTree]
---

###  pathogen 管理套件

接下来的操作都是建立在已经有了 **pathogen** 的前提下，所以没有搭过的可以看我[这篇博文](http://f10.moe/2014/10/20/Vim%E9%85%8D%E7%BD%AENode-js%E5%BC%80%E5%8F%91%E5%B7%A5%E5%85%B7/)。接下就进入正题。

### NERDTree + NERDTreeTabs 

```

cd ~/.vim/bundle
git clone http://github.com/scrooloose/nerdtree.git
git clone http://github.com/jistr/vim-nerdtree-tabs.git

```

接下来在 **.vimrc** 中加入下面的配置：`vim ~/.vimrc`

```
let g:NERDTreeWinSize = 20 "设定 NERDTree 视窗大小
map <Leader>n <plug>NERDTreeTabsToggle<CR> "设置 \ + n 打开 NERDTree
set mouse=nv "设置鼠标只能在 Visual, Normal mode时有作用
set cursorline "设置目前行下划线提示

```

### 基本操作

设置完上面的配置后，就可以用 `/ + n` 来打开 NERDTree ， 如果你觉得这种打开姿势不对0. 0你可以在 `~/.vimrc` 里面修改成你喜欢的姿势。

和编辑文件一样，通过h j k l移动光标定位
- o 打开关闭文件或者目录，如果是文件的话，光标出现在打开的文件中
- go 效果同上，不过光标保持在文件目录里，类似预览文件内容的功能
- i和s可以水平分割或纵向分割窗口打开文件，前面加g类似go的功能
- t 在标签页中打开
- T 在后台标签页中打开
- p 到上层目录
- P 到根目录
- K 到同目录第一个节点
- J 到同目录最后一个节点
- m 显示文件系统菜单（添加、删除、移动操作）
- ? 帮助
- q 关闭
- [Enter] 打开文件/文件夹
- C 設定該目錄為根目錄
- u 返回上个目录

