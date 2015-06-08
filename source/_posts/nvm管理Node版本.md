title: nvm 和 n 管理Node版本
date: 2014-10-27 11:38:41
tags: [nvm, node]
---

node 版本变化太快，做很多项目的时候就已经出现了很多因为不同版本带来的问题。为了更快的切换 node 版本，现在比较常用的node版本管理有 [**n**](https://github.com/tj/n) 和 [**nvm**](https://github.com/creationix/nvm)。

---

## n installation

    $ npm install -g n

or

    $ make install
    
to `$HOME`. Prefix later calls to `n` with `N_PREFIX=$HOME`

    $ PREFIX=$HOME make install

### Installing Binaries

Install a few nodes:

    $ n 0.8.14
    $ n 0.8.17
    $ n 0.9.6

Type `n` to prompt selection of an installed node. Use the up /
down arrow to navigate, and press enter or the right arrow to
select, or ^C to cancel:

    $ n

      0.8.14
    ο 0.8.17
      0.9.6

Use or install the latest official release:

    $ n latest

Use or install the stable official release:

    $ n stable

Switch to the previous version you were using:

    $ n prev

### Removing Binaries

Remove some versions:

    $ n rm 0.9.4 v0.10.0

Instead of using `rm` we can simply use `-`:

    $ n - 0.9.4

---

我个人比较喜欢用 **nvm** , 需要注意的是 nvm 不支持 windows，  下面是nvm的基本用法：

## Installation

    curl https://raw.githubusercontent.com/creationix/nvm/v0.17.3/install.sh | bash

    wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.17.3/install.sh | bash

    git clone https://github.com/creationix/nvm.git ~/.nvm && cd ~/.nvm && git checkout `git describe --abbrev=0 --tags`

To activate nvm, you need to source it from your shell:

    source ~/.nvm/nvm.sh

## Usage

To download, compile, and install the latest v0.10.x release of node, do this:

    nvm install 0.10

And then in any new shell just use the installed version:

    nvm use 0.10

Or you can just run it:

    nvm run 0.10 --version

Or, you can run any arbitrary command in a subshell with the desired version of node:

    nvm exec 0.10 node --version

In place of a version pointer like "0.10", you can use the special default aliases "stable" and "unstable":

    nvm install stable
    nvm install unstable
    nvm use stable
    nvm run unstable --version

If you want to use the system-installed version of node, you can use the special default alias "system":

    nvm use system
    nvm run system --version

If you want to see what versions are installed:

    nvm ls

If you want to see what versions are available to install:

    nvm ls-remote

To restore your PATH, you can deactivate it.

    nvm deactivate

To set a default Node version to be used in any new shell, use the alias 'default':

    nvm alias default stable

---

大部分摘自各自的github 的 readme ，之所以把他们拿过来是为了方便查阅。
