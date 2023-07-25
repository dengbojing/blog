---
titile: neovim学习笔记
date: 2023-04-16
tags: [vim, neovim]
---

<!--more-->
# 0x00 

从18年开始一直使用 `idea-vim` 插件作为编辑工具, 日常使用虽然比较频繁, 但是很少做深入了解, 只是局限于一些简单的操作, 虽然速度提升了一些, 但是远远不够, `idea` 号称内存黑洞, 所以一直在考虑是不是有一种很多好的工具能够替代它,
最开始想到的是`eclipse`系列(`sts, myeclipse`等), 虽然现在`eclipse`也很强大,但是还是老问题, 如果`workspaces`打开项目多了,确实还是会卡, `idea` 没有 `workspaces` 这个概念,一个项目开一个窗口,造成内存吃紧;后面队任何 `IDE` 都失去兴趣了,
就考虑是否能直接用`vim`, 问题在于 `java` 的提示和编译, 目前找到一个 `lunarvim`, 使用感觉和 `vs code` 添加 `java` 插件差不多.至此,实验失败, 不过过程中收获很多, 让我对 `打造一个自己的专属编辑器` 这个概念产生了浓厚的兴趣, 而且确实 `vim` 或者 `neovim` 确实是简单,灵活,强大.   

# Install  

1. [官方安装包下载](https://github.com/neovim/neovim/releases) 
2. `windows`下使用[chocolatey](https://chocolatey.org/install)安装, 命令为`choco install neovim --pre`, 此处`--per` 为安装`beta`版本(目前为`0.10.0), 默认安装版本为 `0.8.0`, `neovim 0.9.0`版本之后使用`lua`语言管理插件.  

# Configuration

1. 使用`choco`安装`nvim`默认是在`c:\tools`下面, 具体的安装位置, 可以通过环境变量`ChocolateyToolsLocation`来指定.  
2. `windows`操作系统下,`neovim`的配置文件`$env:localappdata\nvim`(默认为`c:\user\<username>\appdata\local\nvim, 该目录也是大多数应用程序的配置文件所在目录, 详见[what's appdata](https://www.xda-developers.com/appdata/#:~:text=It's%20a%20hidden%20folder%20that,User%2Dspecific%20installations)`.   
3. 创建`init.lua文件`, `0.8.0`之前是`init.vim`, 之后版本也兼容`init.vim`但是大部分使用`init.lua`.  
4. 创建lua\user文件夹, `md lua\user`.  
5. 进入lua\user目录, 创建`options.lua`文件, 用于做`vim`的通用配置, 创建`globals.lua`用于全局配置, 创建`plugins.lua` 用于加载插件.  
6. `nvim init.lua` 然后写入一下信息   

```vim
    vim.g.mapleader = ' '
    require('user.plugins')
    require('user.global')
    require('user.options')
```

此处一定要先加载插件文件,如果其他文件里面有用到插件配置,会导致插件未加载,配置不生效.  
7. 添加插件管理, `git clone https://github.com/wbthomason/packer.nvim "$env:LOCALAPPDATA\nvim-data\site\pack\packer\start\packer.nvim"` 编辑插件配置文件 `nvim lua/user/plugins.lua` 添加nvim插件管理此处使用`packer`, 这个是之前没用用过的插件管理工具, 以前`vim`使用`vim-plug`感觉很简单,而且还可以指定插件位置, 这个工具初次接触有一点点难受.   

```
    vim.cmd [[packadd packer.nvim]]

    vim.cmd([[
      augroup packer_user_config
        autocmd!
        autocmd BufWritePost plugins.lua source <afile> | PackerSync
      augroup end
    ]])

    local status_ok, packer = pcall(require, "packer")
    if not status_ok then
      return
    end


    return require('packer').startup(function(use)
      -- Packer can manage itself
      use 'wbthomason/packer.nvim'
    end)
```

此处添加插件自身管理,具体使用方法可以看[`github`](https://github.com/wbthomason/packer.nvim)链接.  
9. `nvim options.lua`文件, 添加一些之前`vim`的常用配置.  

```
    local o = vim.opt       -- nvim 对vim的配置映射
    o.nu = true             -- 显示行号
    o.ignorecase = true     -- 忽略大小写
    o.cursorline = true     -- 鼠标行高亮
    o.langmenu = zh_CN      -- 中文菜单, 貌似没啥用
    o.encoding = 'utf-8'    -- 文件编码
    o.bk = false            -- backfile 不生成
    o.udf = false           -- undofile 不生成, 可以指定undofie的目录 
    o.tabstop = 2           -- tab键多少个字宽
    o.shiftwidth = 2        -- >> 推进的字宽
    o.rnu = true            -- 行对行号
    o.syntax = 'on'         -- 语法高亮
    o.bg = 'dark'           -- 背景色
```

# Plugins Config (未完待续)
nvim真的是有很多插件, `nvm-tree` 文件树可以像文件资源管理器一样,管理文件,`telescope`文件搜索, 简直太强大了, 但是非常依赖, 键位记忆以及键位配置, 慢慢学习, 慢慢配置
