```
cd .ssh
```

```
ssh-keygen -t rsa -b 4096
```

cd到文件夹下，初始化git

```
git init
```

为本地仓库添加一个远程仓库引用，DatasetMake是自己起的远程仓库别名，后面的是GitHub仓库地址

```
git remote add DatasetMake git@github.com:dio3379/DatasetMake.git
```

修改当前分支名称为main

```
git branch -M main
```

查看远程仓库别名

```
git remote -v
```

例如，远程仓库的别名为DatasetMake，本地和远程分支名都为main

```
git push -u DatasetMake main
```

强制合并，以本地进度为主

```
git push -u DatasetMake main --force
```

