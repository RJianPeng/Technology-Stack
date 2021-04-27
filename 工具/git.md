* [分支合并](#分支合并)

## 分支合并
在git的指令中，有两种常见指令用于执行分支的合并操作，分别是git merge和git rebase两个指令。

不同之处：
* 1.merge会保留所有commit的历史时间。每个人对代码的提交是各式各样的。尽管这些时间对于程序本身并没有任何意义。这样也就形成了merge以时间为基准的网状历史结构，即commit的顺序和分支无关，而是按照提交的时间进行排列的。
* 2.而rebase会始终把你最新的修改放到最前头。比如你的A分支对branch进行rebase之后，你在分支A上的所有修改都会在branch当前所有修改之前。
 
 merge和rebase的区别如下图所示（图片来自https://www.jianshu.com/p/493c68a48047）
 <div align="center"> <img src="https://github.com/RJianPeng/Technology-Stack/blob/master/%E5%B7%A5%E5%85%B7/photo/merge%E5%92%8Crebase%E7%9A%84%E5%8C%BA%E5%88%AB.webp"/></div><br>

## 删除远程commit
提交了非常弱智（bushi）的commit想要删除应该如何操作
* 1.本地仓库使用git reset —hard head~1 //回退一个版本
* 2.git push -f //强行将本地的版本覆盖到远程仓库
* 3.done

## 删除远程分支
如何删除远程仓库没有用的分支
* 1.git checkout other //切换到其他分支
* 2.git push origin --delete useless_branch //删除掉目标分支
* 3.done

## git提交时候的名字设置
git提交时需要设置一个帅气的名字如何操作
* 1.本地执行 git config --global user.name [username]//配置本地提交时候的用户名
* 2.done

## 在分支间转移commit
cherry-pick commit_hash 将某个提交转移到当前分支 作为一个新的提交

## 存储变更
git stash save 将当前未commit的变更保存到一个堆中，可以在合适的地方通过git stash pop释放到当前代码中，
释放顺序默认为堆，先进后出，可以通过命令指定
```
//常见的相关指令
git stash save "message"//存储当前变更并备注
git stash list//查看当前存储了哪些变更
git stash show//查看某次存储做了哪些变更，默认为最新放进去的存储，可以指定
git stash apply//应用某次存储，不删除，可以指定
git stash pop//应用某次存储，删除，可以指定
git stash drop//删除某次存储，不应用，可以指定
git stash clear//清空当前堆中的存储
```