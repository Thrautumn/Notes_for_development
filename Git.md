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

# Git 常用命令 —— 基于远程仓库

## 配置 VPN

首先为 Git 配置 VPN ，不然连不上 github 。

对于 MinGW 版本 Git，底层依赖 .NET 的 ServicePointManager，它不支持 socks5 代理，需要把代理换成 HTTP/HTTPS 格式（Windows 原生支持）。

```bash
git config --global http.proxy http://127.0.0.1:7892
git config --global https.proxy http://127.0.0.1:7892
```

**验证代理配置**：

```bash
git config --global --get http.proxy
```

## push

push 即将本地仓库中的内容推送至远程仓库。

### 连接远程仓库

```bash
git remote add <repoName> <remoteRepoAddr>
```

### 推送

```bash
git push <remoteRepoName> <localBranchName> [:remoteBranchName]
```
> 如果本地分支名与远程仓库的待提交分支名一致，则无需写后面 `[]` 中的内容。


我们可以也将远端和本地的分支进行绑定，绑定后就不需要指定分支名称了。这种情况适用于**一个本地仓库对应一个远程仓库**，即这个 Repository 是完全自用的，不存在协作。

```bash
git push --set-upstream <remoteRepoName> <localBranchName> :<remoteBranchName>
git push origin
```

## clone

如果我们已经存在一个远程仓库的情况下，我们希望在远程仓库的代码上继续编写代码。此时可以使用克隆操作来将远端仓库的内容全部复制到本地：

```bash
git clone <remoteRepoAddr>
```

# 协作流程

下面是我看到的一个很好的比喻例子，共飨。

**一个比喻**：
你的仓库 (Repository) = 这本小说的原稿，正放在一个公共的桌子上。
main 分支 (main branch) = 这本小说的最终正式出版稿，非常干净、重要，不能随便乱动。

现在，有人想帮你修改小说（写代码），正确的流程应该是这样的：

1. **Fork 远程仓库到自己的仓库**（这样就变成纯单人开发，无协作压力）：他应该先把你的整本小说原稿完整地复印一份，拿回自己的桌子上。这样他随便怎么改，都不会弄脏你的原稿。

2. 新建 feature branch：你的原稿旁边有一叠叫 develop 的草稿纸，大家平时的新想法都先写在这里。他应该从这叠草稿纸里拿一张新的。他拿到的这张新纸，叫做 feature/confession (功能分支)，专门用来写 “表白” 这个新章节。这样，他的修改就不会和别人的修改混在一起。

3. 写代码，写测试：他得把 “表白” 的章节写好（写代码）。写完后，还要自己读几遍，确保没有错别字，情节也合理（写测试）。还要检查句子通不通顺，符不符合你们的写作风格（跑 Linter）。

4. 按时 commit ：他写好一页后，需要在旁边贴个小纸条，写上 “feat: 新增了浪漫的表白情节”（遵循提交规范），而不是只写 “改了点东西”。

5. 将修改 Push 到自己的仓库中：他把这张写满了的草稿纸，放回他自己那本复印的小说稿里。

6. 发起 PR ( Pull Request )：然后他对你说：“嘿，我写了个新章节，你看看怎么样？如果可以，就把它放进你的草稿里吧！” 这就是 提一个 Pull Request (PR)。他还会在请求里详细解释他写了什么，为什么这么写，并且 @你和另外两个朋友一起来帮忙检查。

7. 审稿 (Review)：你和你的朋友会仔细看他的草稿，可能会说：“这里剧情不太合理”、“这个词用错了”。他需要根据你们的意见，把所有问题都修改好。

8. 最终审核通过 (CI/CD & LGTM)：你们会用一套自动检查工具 (CI/CD) 来扫描他的草稿，确保没有低级错误。至少需要两个人看完后说 “Looks Good To Me (LGTM)” —— “我看没问题！”。

9. 最终合并 (Squash and merge)：最后，你把他那张写得乱七八糟、但内容很好的草稿，重新誊写得干干净净，然后放进你自己的 develop 那叠草稿里，准备以后一起出版。这个过程就叫 Squash and Merge。
