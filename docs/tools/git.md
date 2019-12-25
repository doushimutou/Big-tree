## git简介：

![](https://note.youdao.com/yws/public/resource/7f5bc7c68877511de7441b2db224ad2f/xmlnote/BD75E7B277EB4BD9B0D7C868FEB62A01/8318)

1. 工作区：本地电脑存放项目文件的地方，暂时没有和git产生任何关系

2. 暂存区：git管理文件的时候，在项目的文件夹下面生成的.git文件夹，里面是git的版本库。其中.git包含两个部分，一个是暂存区，房子暂存区的文件，通过add命令将工作区的文件添加到暂存区里

3. 本地仓库：.git文件夹还包括git自动创建的master分支，并且将HEAD指针指向master分支。使用commit将暂存区的文件添加到本地仓库中

4. 远程仓库：项目代码在git服务器上

   

## git 使用场景，纯命令方式：

- 第一次使用git,列出的git的配置

  ```
  git  config --list
  ```



- 第一次使用git，配置用户信息

  1. 配置用户名：

     ```
     git config --global user.name "your name"
     ```

     

  2. 配置用户邮箱：

     ```
     git config --global user.email "youremail@github.com"
     ```

     

- 第一次从服务器拉取代码：

  ```git
  git   clone  git@192.168.1.66:test/weekReport.git
  ```

- 创建新的分支

  ```
  git push --set-upstream origin gitdemo
  ```



## 在工作区使用git命令

#### 新建仓库

1. 将工作区的文件使用git进行管理，即创建一个新的本地仓库：`git init`
2. 从远程git仓库复制项目：`git clone `，如：git clone git://github.com/wasd/example.git;克隆项目时如果想定义新的项目名，可以在clone命令后指定新的项目名：`git clone git://github.com/wasd/example.git mygit`；

#### 提交

1. 提交工作区所有文件到暂存区：`git add .`
2. 提交工作区中指定文件到暂存区：`git add   ...`;
3. 提交工作区中某个文件夹中所有文件到暂存区：`git add [dir]`;

#### 撤销

1. 删除工作区文件，并且也从暂存区删除对应文件的记录：`git rm  `;
2. 从暂存区中删除文件，但是工作区依然还有该文件:`git rm --cached `;
3. 取消暂存区已经暂存的文件：`git reset HEAD ...`;
4. 撤销上一次对文件的操作：`git checkout --`。要确定上一次对文件的修改不再需要，如果想保留上一次的修改以备以后继续工作，可以使用stashing和分支来处理；
5. 隐藏当前变更，以便能够切换分支：`git stash`；
6. 查看当前所有的储藏：`git stash list`；
7. 应用最新的储藏：`git stash apply`，如果想应用更早的储藏：`git stash apply stash@{2}`；重新应用被暂存的变更，需要加上`--index`参数：`git stash apply --index`;
8. 使用apply命令只是应用储藏，而内容仍然还在栈上，需要移除指定的储藏：`git stash drop stash{0}`；如果使用pop命令不仅可以重新应用储藏，还可以立刻从堆栈中清除：`git stash pop`;
9. 在某些情况下，你可能想应用储藏的修改，在进行了一些其他的修改后，又要取消之前所应用储藏的修改。Git没有提供类似于 stash unapply 的命令，但是可以通过取消该储藏的补丁达到同样的效果：`git stash show -p stash@{0} | git apply -R`；同样的，如果你沒有指定具体的某个储藏，Git 会选择最近的储藏：`git stash show -p | git apply -R`；

#### 查看信息

1. 查询当前工作区所有文件的状态：`git status`;

2. 比较工作区中当前文件和暂存区之间的差异，也就是修改之后还没有暂存的内容：git diff；指定文件在工作区和暂存区上差异比较：`git diff `;

   



##  暂存区上的操作命令

#### 提交文件到版本库

1. 将暂存区中的文件提交到本地仓库中，即打上新版本：`git commit -m "commit_info"`;
2. 将所有已经使用git管理过的文件暂存后一并提交，跳过add到暂存区的过程：`git commit -a -m "commit_info"`;
3. 提交文件时，发现漏掉几个文件，或者注释写错了，可以撤销上一次提交：`git commit --amend`;

#### 查看信息

1. 比较暂存区与上一版本的差异：`git diff --cached`;
2. 指定文件在暂存区和本地仓库的不同：`git diff  --cached`;
3. 查看提交历史：git log；参数`-p`展开每次提交的内容差异，用`-2`显示最近的两次更新，如`git log -p -2`;

#### 分支管理

1. 创建新的分支：`git branch <branch name>`
2. 从当前所处分支切到其他分支：git checkout <branch-name>
3. 从当前所处分支切到其他分支：
4. 新建并切到新建分支上：`git checkout -b `;
5. 删除分支：`git branch -d `；
6. 将当前分支与指定分支合并：`git merge `;
7. 显示本地仓库所有分支:`git branch`
8. 查看各个分支最后一个对象的信息： git branch -v
9. 查看那些分支已经合并到当前分支：`git branch --merged`;
10. 查看当前那些分支还没有合并到当前分支：`git branch --no-merged`;
11. 推送：`git push`  拉取数据：`git  pull`

 

#### 本地仓库上的操作



1. 查看本地仓库关联的远程仓库：`git remote`；在克隆完每个远程仓库后，远程仓库默认为`origin`;加上`-v`的参数后，会显示远程仓库的`url`地址；

2. 添加远程仓库，一般会取一个简短的别名：`git remote add [remote-name] [url]`，比如：`git remote add example git://github.com/example/example.git`;

3. 从远程仓库中抓取本地仓库中没有的更新：`git fetch [remote-name]`，如`git fetch origin`;使用fetch只是将远端数据拉到本地仓库，并不自动合并到当前工作分支，只能人工合并。如果设置了某个分支关联到远程仓库的某个分支的话，可以使用`git pull`来拉去远程分支的数据，然后将远端分支自动合并到本地仓库中的当前分支；

4. 将本地仓库某分支推送到远程仓库上：`git push [remote-name] [branch-name]`，如`git push origin master`；如果想将本地分支推送到远程仓库的不同名分支：`git push  :`，如`git push origin serverfix:awesomebranch`;如果想删除远程分支：`git push [romote-name] :`，如`git push origin :serverfix`。这里省略了本地分支，也就相当于将空白内容推送给远程分支，就等于删掉了远程分支

5. 移除远程仓库：`git remote rm [remote-name]`；

   
## git 几种角色的区别：

<img src="http://img1.tuicool.com/7n63uyI.png!web" alt="img" style="zoom:67%;" />





