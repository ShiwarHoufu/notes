# Git 多人协作指南

## 1. 第一次准备

每个人配置身份，并克隆仓库：

```bash
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
git clone https://github.com/用户名/仓库名.git
```

## 2. 分支命名规范
- 主分支：`main`
- `feature/` 功能分支
- `bugfix/` 修复分支
- `docs/` 文档分支


## 3. 日常开发流程
> 仓库已对 main 分支开启分支保护，任何人无法直接推送代码到 main 分支。   
另外，为了减少在个人开发中其他的麻烦，最好不要在 main 分支上进行文件的编辑。   

**每次开发前先更新main分支**：

```bash
git switch main
git pull origin main
```

然后创建分支（以feature/login为例）：

```bash
git switch -c feature/login  
# 创建并切换到feature/login分支
# 此时仅在本地存在分支
```

写代码后提交并推送。

```bash
git add .
git commit -m "实现登录功能"
git push -u origin feature/login  
# 推送当前本地分支到远程仓库,这样远程仓库也有对应分支了。
# 之后在该分支上开发就只需要 git push 即可。
```

要把feature/login分支合并到main分支，就需要发起Pull Request ;   
而在发起Pull Request之前，可能main已经合并了别人新提交的内容，所以你的功能分支需要同步。  
（这一步非必须，但建议做）

### 方式一：merge 合并

```bash
git switch main
git pull origin main
git switch feature/login
git merge main
#解决冲突后：
git add .
git commit
git push
```

### 方式二：rebase 合并

（让提交历史更整洁）

```bash
git switch main
git pull origin main
git switch feature/login
git rebase main
# 解决冲突后
git add .
git rebase --continue
git push --force-with-lease
```

> **注意**：如果这个分支只有你一个人用，可以用 `rebase` ；如果多人共用，改用 `merge`。

接着在 Gitee 上发起Pull Request，等待代码审查通过后合并到 `main`。

成功合并到main分支后进行本地清理：   
（合并后该分支的生命周期就结束了，之后开发会创建新的分支）

```bash
git switch main    # 删除当前分支前必须要先切换到其他分支
git branch -d 分支名  # 删除本地分支
git push origin --delete 分支名   # 删除远程分支
git fetch --prune  # 清掉对远程已删除的分支的引用，使本地分支列表与远程分支列表一致
```

## 4. 关于解决冲突

冲突时 Git 会提示。查看状态：

```bash
git status
```

根据提示找到并打开冲突文件，会看到：

```text
<<<<<<< HEAD
你的代码
=======
别人的代码
>>>>>>> main
```

手动改成正确代码，删除标记，然后：

```bash
git add 冲突文件
git commit        # merge 冲突时
# 或
git rebase --continue   # rebase 冲突时
```


