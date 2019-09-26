---
title: 搭建Hexo的一些小事
date: 2019-09-26 17:07:53
tags:
---
# 一、将Hexo本体和生成的静态网页都放到Github中

在Hexo中，hexo deploy命令是将编译生成的.deploy_git文件夹的内容上传到Github上。

在Hexo的配置文件_config.yml中有hexo deploy的配置

```yml
deploy:
  type: git
  repo: git@github.com:ZZFlying/zzflying.github.io.git
  branch: master
```

Github Page使用的是master分支，而branch可以指定deploy发布的分支，所以可以新建一个hexo分支用于专门存放Hexo本体。

新建一个hexo分支并设置为默认分支后，每次新增主题或修改hexo配置文件后执行

```
git add .
git commit -m "message"
git push origin hexo
```

就会将新增的主题或修改的文件提交到Github，这样就算原始文件丢失了也不用怕了！

在npm install hexo时生成的.gitignore的文件默认排除了.DS_Store、Thumbs.db、db.json、.log、node_modules/、public/、.deploy*/这些自动生成的文件或文件夹。

如果重新安装了系统，只需输入如下命令

```
git clone git@github.com:ZZFlying/zzflying.github.io.git
npm install hexo
npm install
npm install hexo-deployer-git
```

因为将hexo分支设置成了默认分支就不需要指定分支名了，在取回的文件夹中重新安装hexo，就跟原来的一模一样。