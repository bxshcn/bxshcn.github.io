---
layout: post
title:  "git介绍"
comments: true
categories: 工具
tags: git
---
## working directory, repository
working directory存放代码，你可以从任何地方获取代码，将其放到working directory中，或者直接在working directory中编辑修改代码。

working directory中的`.git`是repository版本库，存放代码的所有信息，除代码存储区外，还包括所有提交的历史信息，HEAD、branch等meta信息，以及一些hook脚本等。

repository的代码存储区又分为stage（index）和分支区，我们总是先将working directory中的修改`add`到stage区，积累到一定程度后，再一次性的`commit`到分支区。

![architecture](/assets/2021-06-11-git-introduction/architecture.png)

如果我们从生命周期的角度来看待文件新增、修改、`add`和`commit`的状态，就有：
![lifecycle](/assets/2021-06-11-git-introduction/lifecycle.png)


`--`表示当前work directory
`git diff HEAD -- readme.txt`，针对readme.txt文件，查看工作区和版本库里面最新版本的区别。

`git checkout -- readme.txt`，抛弃当前工作区中对readme.txt的任何修改：
1. 如果之前已经将readme.txt `add`到了stage区，那么readme.txt就会和stage区一致
2. 否则（还没有将readme.txt`add`到了stage区），那么readme.txt就会和分支区一致

不过新版（比如git 2.28.0）已经将上述命令调整为更清晰的`git restore --staged readme.txt`和`git restore readme.txt`，分别让工作区的readme.txt和stage区一致，让工作区的readme.txt和分支区一致。

## 什么是three-way merge？
merge有一个前提，即我们总是在某个当前分支上，去merge另外一个分支的修改内容。但如果当前分支也有修改呢？如下图所示

![3-way-merge](/assets/2021-06-11-git-introduction/3-way-merge.png)

创建分支experiment后，分支experiment有了新的修改C4，与此同时分支master也有新的修改C3，如果我们现在在master上，希望将experiment的C4 merge进来，需要知道什么信息呢？

显然，我们需要知道分支experiment相对于C2的更新，以及分支master头相对于C2的更新，结合着两者进行merge来判断是否存在冲突，这就是three-way merge，哪three-way？the two latest branch snapshots(C3 and C4) and the most recent common ancestor of the two (C2)，其合并结果为：
![3-way-merge-result](/assets/2021-06-11-git-introduction/3-way-merge-result.png)
这会在master上生成一个新的commit，且master分支的历史**保留了experiment分支分叉出去的特征形状**，而这对master分支来说并没有意义：我们只需要协作的分支（比如master或者dev）包含所有人的修改，但其log保持线性。

## 什么是rebase？
你在某个experiment分支上做了一些新的commits并测试通过，然后你希望将这些commits合并到master分支上，如前一节所述，你在master上merge experiment分支时，会让master形成复杂的log。

rebase的意思就是replay，`git rebase base-branch modified-branch`就是将modify-branch上修改的内容，在base-branch上replay。基于上面的例子，`git rebase master experiment`的结果如下：
![rebase](/assets/2021-06-11-git-introduction/rebase.png)

四个关键点：
1. replay的目标。是在base-branch的最新提交(C3)之后replay
2. replay的内容。是modified-branch从base-branch上off出来之后的所有commits（本例仅包括为C4）。
3. replay操作的环境。由rebase命令参数modified-branch决定。如果不指定则默认为当前分支。
4. rebase的结果，会导致被replay内容所属的分支，自动包含base-branch的最新内容。

最后，你可以在master分支上merge rebase之后的experiment：
```
$ git checkout master
$ git merge experiment
```
从而得到线性的协作分支log：
![rebase-result](/assets/2021-06-11-git-introduction/rebase-result.png)

### 更复杂的例子
如果你想将C8、C9(即分支client相对于分支server off出来的修改commits)添加到master上：
![rebase-complex](/assets/2021-06-11-git-introduction/rebase-complex.png)
可以使用命令
![rebase-complex-cmd](/assets/2021-06-11-git-introduction/rebase-complex-cmd.png)
从而得到：
![rebase-complex-result](/assets/2021-06-11-git-introduction/rebase-complex-result.png)

### 注意
如果使用rebase不当会造成很麻烦的问题，但只要你遵循一条原则，就万事大吉：
> Do not rebase commits that exist outside your repository and that people may have based work on.

这句话的意思，简单说就是只rebase（replay）你自己的commits。不论何种情况，你需要弄清楚哪些是你的commits，哪些是其他人的commits，确保只将你自己的commits在目标分支上rebase（replay）。

## Git分支管理
### HEAD与branch name分支名的关系
分支名用于跟踪分支的时间线，它实际上是一个指针，指向对应分支的最后一个commit。每当我们在对应分支中添加一个新的commit，则分支名就被更新为这个最新的commit。

HEAD始终指向下一次commit的父节点，一般来说，The HEAD in Git is the pointer to the current branch reference, which is in turn a pointer to the last commit you made or the last commit that was checked out into your working directory. That also means it will be the parent of the next commit you do. 

在正常状态下，HEAD指向当前分支的分支名指针，它就是下一次新增commit的父节点。在detached HEAD状态下，HEAD指向下一次commit的父节点这一点仍然成立。

那么HEAD有什么作用呢？
1. 如果我们只用branch name来跟踪不同的分支的最后一次commit，那么我们就需要针对每条指令明确其操作的目标branch，采用HEAD指向的当前分支可以显著简化指令。
2. 如果我们只用branch name来跟踪冈分支的最后一次commit，我们如何以某个分支的某个commit为基础来创建一个新的commit呢？HEAD可以解决这个问题。

### 分支合并的冲突和解决
如果两个分支（比如master和feature1）相对分叉点（commit C3）有了新的commits，那么当在master上合并feature1就不能采用fast-forward策略，而只能采用3-way merge，此时就可能存在冲突。所谓冲突，就是存在两个不同的commits，修改了共同基commit的同一行。

git在merge指令时，如果发现存在冲突，就会在执行结果中输出提示，并将存在冲突的文件中的行用`<<<<<<<，=======，>>>>>>>`标记出来，比如：
```
Git is a distributed version control system.
Git is free software distributed under the GPL.
Git has a mutable index called stage.
Git tracks changes of files.
<<<<<<< HEAD
Creating a new branch is quick & simple.
=======
Creating a new branch is quick AND simple.
>>>>>>> feature1
```

解决冲突的办法，就是打开冲突文件，修改并取出上述特殊标记，最后保存。我们可以使用git status查看存在冲突的所有文件。

### 分支策略和日常工作场景
master分支应该是非常稳定的，仅用来发布新版本

dev分支用于协作干活，它是不稳定的。到某个时候，比如1.0版本发布时，再把dev分支合并到master上，在master分支发布1.0版。

每个人都有自己的分支并在自己的分支上干活，时不时地往dev分支上合并集成。

对于bug fix，我们往往会直接从master上创建分支，比如创建分支`issue-101`，然后修改测试通过后，将其合并到master分支。

对于开发新的feature，我们往往会从dev上创建一个分支比如feature1，然后修改测试通过后，将其合并到dev分支进行进一步的集成测试。

需要协作的分支才需要提交到远程repository中，否则留在本地自己玩就好。

#### 临时修改bug
如果在当前工作区中有未完成工作的情况下，需要临时切换到其他分支下工作，比如在feature1（基于dev分支创建）未开发完成的情况下，需要修复一个线上bug（基于master分支创建），我们可以使用`git stash`指令来临时保存当前工作区。

我们可以使用`git stash list`查看所有保存的工作区，比如：
```Bash
$ git stash list
stash@{0}: WIP on dev: f52c633 add merge
```
然后使用`git stash apply`来恢复指定的备份，使用`git stash drop`来删除指定的备份。

更常用的指令是`git stash pop`，它会恢复上次备份的工作区并删除相关备份。

stash的本意是put something valuable in a secret place to keep it safe.

#### cherry-pick
cherry-pick的意思是to pick or accept the best people or things in a group。

当我们在bug分支上修改了某个bug，然后合并到了master分支上，对应一个提交C1，而dev分支则是从更早的master分支上创建的，因此这个bug必然也存在于dev分支上，我们我们如何修改dev分支上的对应bug呢？

依照bug修复的过程，手动修改并commit一遍当然可以，但更常用的是使用cherry-pick，将相关的commits应用到dev分支上:`git cherry-pick C1`。


## Git internal
### git objects
Git is a content-addressable filesystem. 也就是说the core of Git is a simple key-value data store. 你可以insert any kind of content into a Git repository, for which Git will hand you back a unique key you can use later to retrieve that content。

这个key-value数据库就是.git/objects目录。

git将任意一次针对特定文件的修改内容视为value，并为其计算得到一个SHA-1的40位key值。这就是git objects。我们可以使用`git hash-object`来计算key，使用`git cat-file`来恢复value，比如：
```Bash
# -w，除计算hash key值外，还将value内容写到数据库中。--stdin表示从stdin获取内容，否则它需要从文件获取内容。
$ echo 'test content' | git hash-object -w --stdin
d670460b4b4aece5915caf5c68d12f560a9fe3e4
# -p 推测对象内容的类型
$ git cat-file -p d670460b4b4aece5915caf5c68d12f560a9fe3e4
test content
```

这种object的类型被标记为blog，但这种类型的key值很难记忆，且没有文件名信息，因此git还引入了tree类型的object来解决这个问题。

A single tree object contains one or more entries, each of which is the SHA-1 hash of a blob or subtree with **its associated mode, type, and filename**. For example, the most recent tree in a project may look something like this:
```Bash
$ git cat-file -p master^{tree}
100644 blob a906cb2a4a904a152e80877d4088654daad0c859 README
100644 blob 8f94139338f9404f26296befa88755fc2598c289 Rakefile
040000 tree 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0 lib
$ git cat-file -p 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0
100644 blob 47c6340d6459e05787f644c2447d2595f5d3a54b simplegit.rb
```
其中lib是一个目录，里面包含一个blog对象，the data that Git is storing looks something like this:
![tree-objects](/assets/2021-06-11-git-introduction/tree-objects.png)

tree objects表示了项目的一个snapshot（包含了你最近在stage区域所做的所有修改），但You also don’t have any information about who saved 439the snapshots, when they were saved, or why they were saved..为此git提供了commit object来存储这些信息。

你可以使用`commit-tree`来创建一个commit object，并使用 `git cat-file`查看其内容：
```Bash
$ echo 'First commit' | git commit-tree d8329f
fdf4fc3344e67ab068f836878b6c4951e3b15f3d
$ git cat-file -p fdf4fc3
tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579
author Scott Chacon <schacon@gmail.com> 1243040974 -0700
committer Scott Chacon <schacon@gmail.com> 1243040974 -0700

First commit
```

这样，结合commit、tree、blob objects，git就完整的记了和表达了一个项目的各次提交所及的所有数据和元信息：
![objects](/assets/2021-06-11-git-introduction/objects.png)

除了上述三类object外，还有一个tag object：
1. 它类似于commit对象，annotated tag contains a tagger, a date, a message, and a pointer. 
2. 但不同于commit对象，lightweight tag points to a commit rather than a tree. 注意commit object points to a tree object
3. 它类似于a branch reference, but it never moves。换句话说， it always points to the same commit but
gives it a friendlier name.

实际上有两种tag object，一是lightweight tag，just a reference to a commit object; 二是annotated tag，是一个指向lightweight tag的reference，附带message信息。

### Git references
git references也称为refs。是**为了方便**识记SHA-1值而引入的标记。

`.git/refs/heads/`目录下的文件即本地分支名，每个分支名文件中存放对应分支的最后一次commit对象的SHA-1值。

`.git/refs/tags`目录下的文件即tag objects，它们都是references。

## 其他
### tag标签
tag 是版本库的一个snapshot，对应到某个具体的commit，所以tag不能像分支那样修改：分支的修改体现在有一个分支名所表征的指针，不断指向新的commit。

你可以checkout一个tag, 从而你的HEAD不再指向分支名指针，这就是一种detached HEAD状态。当前的任何修改只能由HEAD跟踪，而没有特定的分支名来同步跟踪，所以一旦你转到其他分支或者tag导致HEAD指向其他commit，当前的哪些修改就不再可能被跟踪到(即可籍由某个commit的reference-tag or branch name-reach达到。注意一个commit指向一个tree object，参考后面git internal)，相当于丢失了。
![detached-HEAD](/assets/2021-06-11-git-introduction/detached-HEAD.png)

当我们执行`git checkout tag`指令，git会提示：
> You are in 'detached HEAD' state. You can look around, make experimental
> changes and commit them, and you can discard any commits you make in this
> state without impacting any branches by switching back to a branch.
>
> If you want to create a new branch to retain commits you create, you may
> do so (now or later) by using -c with the switch command. Example:
>
>  git switch -c `new-branch-name`

### submodule
while working on one project, you need to use another project（比如a library that a third party developed） from within it.

A common issue arises in these scenarios: you want to be able to treat the two projects as separate yet still be able to use one from within the other. 因为一旦你将the third party library包含进你的project，你在当前project中对它们的任何修改就很难被merge回the third party library，因为它们已经是你的project的不可分割的一部分，而不是the third party library的一个clone。

Git addresses this issue using submodules. Submodules allow you to keep a Git repository as a subdirectory of another Git repository. This lets you clone another repository into your project and keep your commits separate.


参考：
- [1] Scott Chacon, Ben Straub. Pro Git 
- [2] 廖雪峰. [Git教程](https://www.liaoxuefeng.com/wiki/896043488029600)
