---
title: 使用hexo+github pages建立静态blog
date: 2020-01-14 19:32:51
tags: [git,hexo]
categories: hexo
---
> 好记性不如烂笔头

记录一下使用hexo + github pages搭建博客的过程,以及过程中遇到的坑;同时也给阅读这篇文章,想要搭建博客的人一点帮助
<!--more-->
# 为什么搞独立blog
`市面上blog service多如牛毛为啥非要搞自己的,因为独立的才是自己的`

# 为什么使用gihub pages 
`因为不用买服务器,免费而且稳定,也无需域名,只需要你有github账号就能用`

> #  Talk is cheap, show me the code

# Git

1. [git](https://git-scm.com/downloads)根据自己电脑下载相应版本,本人是win10电脑使用git bash.
2. 安装完成打开```git bash```, 输入```git --version```测试是否安装成功
3. 注册[github](https://github.com/)账号(全球最大的同性交友网站,你值得拥有).
4. 新建```<username>.github.io```仓库.

# node

1. [node](http://nodejs.cn/download/])根据自己电脑下载相应版本
2. 安装完成之后,在之前的```git bash```输入```node --version```测试是否安装成功

# Hexo

1. 安装好 Node.js 后，通过 npm 安装 Hexo
    > npm install hexo-cli -g

    ps: 可能安装失败,如果失败,安装cnpm,然后在安装hexo;或者挂代理,走梯子

    > npm install -g cnpm --registry=https://registry.npm.taobao.org
2. 安装 Hexo 完成后,执行
    > hexo init \<folder>  
    > cd \<folder>  
    > npm install  

    执行完成之后目录会像
    
    > ├── \_config.yml  
    ├── node_modules  
    │   ├── hexo  
    │   ├── hexo-generator-archive   
    │   ├── hexo-generator-category  
    │   ├── hexo-generator-index   
    │   ├── hexo-generator-tag  
    │   ├── hexo-renderer-ejs  
    │   ├── hexo-renderer-marked  
    │   ├── hexo-renderer-stylus  
    │   └── hexo-server  
    ├── package.json  
    ├── scaffolds  
    │   ├── draft.md  
    │   ├── page.md  
    │   └── post.md  
    ├── source  
    │   └── \_posts  
    └── themes  
        └── landscape  

    ps: 简单说明一下目录作用   

    >  \_config.yml  配置文件,网站的标题,作者,主题配置等
    >  node_modules hexo的模块,较少关心  
    >  package.json 项目描述文件,不用关心  
    >  scaffolds  模版配置,较少关心
    >  source-->\_post  主要存放我们写的文章  
    >  themes  [hexo主题](https://hexo.io/themes/),文件夹名称对应为主题名称  
3. 配置\_config.yml
    ```
    # Site
    title: Hexo #标题
    subtitle: #副标题
    description: #描述
    author: #你的名字
    language: zh #网站使用的语言,注意使用主题下面的语言(themes--><要使用的主题>-->languages)不然会出现未知结果
    timezone: Asia/Shanghai #网站时区 

    # URL
    url: https://dengbojing.com #地址(如果未申请域名则不需要填写)
    root: / #根目录
    permalink: :year/:month/:day/:title/ #文章的永久链接格式
    permalink_defaults: #
        trailing_index: true # Set to false to remove trailing 'index.html' from permalinks
        trailing_html: true # Set to false to remove trailing '.html' from permalinks

    # Directory
    source_dir: source #资源文件夹
    public_dir: public #由资源文件夹生成而来
    tag_dir: tags #标签文件夹,默认是没有的需要使用hexo new page tags 自行创建
    archive_dir: archives #归档文件夹
    category_dir: categories #分类文件夹,默认是没有的需要使用hexo new page tags 自行创建
    code_dir: downloads/code #Include code 文件夹
    i18n_dir: :lang #国际化（i18n）文件夹
    skip_render: #跳过指定文件的解析

    # Writing
    new_post_name: title.md # 新文章的文件名称
    default_layout: post #预设布局
    titlecase: false # 把标题转换为单词首字母大写
    external_link: 
        enable: true # 在新标签中打开链接
        field: site
        exclude: '' #排除文件
    filename_case: 0 #把文件名称转换为 (1) 小写或 (2) 大写
    render_drafts: false #显示草稿
    post_asset_folder: false #启动 Asset 文件夹，为 true 时，每次建立文件时，Hexo 会自动建立一个与文章同名的文件夹
    relative_link: false #把链接改为与根目录的相对位址
    future: true #显示未来的文章
    highlight: #代码块高亮,很多主题要求此项为false
        enable: true
        line_number: true
        auto_detect: true
        tab_replace:

    # Category & Tag
    default_category: uncategorized #默认分类
    category_map: #分类别名
    tag_map: #标签别名

    # Date / Time format
    date_format: YYYY-MM-DD #日期格式
    time_format: HH:mm:ss #时间格式

    # Pagination
    per_page: 10 #每页显示的文章量 (0关闭分页功能)
    pagination_dir: page #分页目录

    # Extensions
    theme: next #当前主题名称(本人使用的非默认主题)

    # Deployment
    deploy: #部署
        type: git
        repo: https://github.com/username/username.github.io.git #仓库地址
        branch: master #分支名称
    ```
4. 打开看看, 在```git bash```中使用 ```hexo g``` 命令生成文章,```hexo s```命令启动服务,下面提示访问[localhost:4000](http://localhost:4000),访问一下看到使用默认主题的网站  

# 写文章   
  使用```hexo new post <filename>``` 创建自己的第一篇文章  
  找到source-->\_post,打开```<filename>.md```  
  **_更多[写作](https://hexo.io/zh-cn/docs/writing)用法_**
```
  ---
  title: 使用hexo+github pages建立静态blog
  date: 2020-01-14 19:32:51
  tags: [git,hexo]
  categories: hexo
  ---
```
  在date下面添加分类和标签,可选  
  在---下面写正文内容,可以使用```<!--more-->```分割  
  > 比如:   
        
        简介  
        <!--more-->
        正文  

# 本地预览  

hexo g生成  
hexo s启动  
打开浏览器,输入[localhost:4000](localhost:4000)看看吧  

# 部署到服务器  
  安装一键部署
  > npm install hexo-deployer-git --save  

  执行 ```hexo clean```(可选,正常情况不需要)   
  ```hexo d``` 部署到```<username>.github.io```

# 后记
  使用过程中遇到很多问题,目前都没有解决,  
  比如有些主题莫名看不到tags和categories仓库,最后选来选去只能使用next主题  
  还有写modules(比如七牛云)安装之后即使你不启用你也得写配置  
  