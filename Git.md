# Git 常用命令 —— 基于本地仓库

## 创建仓库 & 提交

### git init

在当前文件夹里激活 Git，将当前文件夹内的修改交由 Git 进行管理。

### git add 类

#### `git add <fileName>`

让 Git 跟踪指定文件的修改，把这个文件放进「暂存区」（相当于 “告诉 Git，这个文件的修改要算进下一次快照”）。

#### git add ./

一次性把当前文件夹里所有修改过 / 新增的文件都加入暂存区。

### git commit 类

#### git commit -m 'description'

给暂存区的文件 “拍快照”，记录版本。-m 后面的文字是版本备注（必须写，方便后续找版本）

#### git commit -a -m 'description'

跳过 git add 步骤，直接把「已被 Git 追踪的文件」的修改提交（新增的文件不会被提交，因为没被追踪过）

## 查看日志

### git status
查看 “当前仓库的实时状态” —— 哪些文件改了、哪些文件没加暂存区、当前在哪个分支。

### git log 类

#### git log 

查看所有版本记录（提交哈希值、提交人、时间、备注），按时间倒序排列（最新的在最上面）。

#### git log --oneline

简化 git log 的输出，每行只显示 “简短哈希值 + 提交备注” 。

#### git log --oneline --graph

在极简版日志基础上，加「分支分叉 / 合并的图形化标识」（用 */|/\ 显示分支关系）。

#### git log --oneline --graph --all

显示「所有分支」的图形化极简日志（默认只显示当前分支的日志）。

#### `git show <hashCode>`

查看「某个版本 / 提交」的具体修改内容（默认看最新提交）。

## 回滚 & 历史操作

### `git reset --hard <hashCode>`

强制把当前分支 “回退到指定版本” 。

### git reflog

查看「所有 Git 操作记录」（包括 commit、reset、checkout 等），是 Git 的 “操作日志”。

## 分支操作

### git branch 类

#### git branch

查看所有本地分支，当前分支前会标 *。

#### `git branch <branchName>`

创建新分支（只创建，不切换到新分支）。

### `git checkout <branchName>`

切换到指定分支。

### 合并分支

#### `git merge <branchName>`

把 branchName 分支的修改同步到「当前分支」（比如在 master 执行 git merge test，就是把 test 的修改合到 master）。

Git 的合并从来不是 “把两个分支揉成一个、删掉其中一个”，而是将目标分支的修改（新增 / 修改 / 删除）同步到当前分支，**被合并的分支（比如 test）会完整保留，它的文件、提交记录、分支本身都不会有任何变化**。

### 变基

#### `git rebase <branchName>`

变基：先选定目标分支（比如团队公共的 master）的某个版本（通常是最新版本），把我们自己开发分支（比如 test）的起点对齐到这个版本（同步目标版本的所有内容），然后把你之前在自己分支上写的所有修改，按原来的顺序 “重新写” 在这个新起点后面 —— 最终既保留了你所有的开发内容，又让整个提交历史变成没有分叉的直线。

举个例子：

* master: test1.txt 、 test2.txt( V1 ) 

* test: test2.txt( V2 ) 、 test3.txt

变基之后（test 对齐 master），相当于 在 master 的 test1.txt 、 test2.txt( V1 ) 之后，添加上 test 的 test2.txt( V2 ) 、 test3.txt 。但是此时二者对于 test2.txt 的修改不一，所以需要我们进行修改。步骤如下

1. 打开冲突文件 test2.txt ，找到 Git 标记的冲突区域（由 `<<<<<<</=======/>>>>>>>` 标识）并修改。

2. 标记冲突已解决。执行：git add test2.txt 。

3. git rebase --continue 

4. 如果不想进行变基，可以执行 git reabse --abort

## .gitignore

不被 Git 追踪的文件列表。有以下的匹配规则：

```txt
# 这样就会匹配所有以txt结尾的文件
*.txt

# 虽然上面排除了所有txt结尾的文件，但是这个不排除
!666.txt

# 也可以直接指定一个文件夹，文件夹下的所有文件将全部忽略
test/

# 目录中所有以txt结尾的文件，但不包括子目录
xxx/*.txt

# 目录中所有以txt结尾的文件，包括子目录
xxx/**/*.txt
```



