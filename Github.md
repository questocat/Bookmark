## 怎样删除 GitHub 项目上打的标签

删除标签  
git tag -d [tag]  
git push origin :[tag]  

如果你的标签与其中某一个分支具有相同的名称，那么使用下面的命令  
git tag -d [tag]  
git push origin :refs/tags/[tag]  
