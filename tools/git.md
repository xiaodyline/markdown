# Linux 中 Git 的安装、配置与常用命令方案

## 1. 目标

本文整理 Linux 环境下 Git 的安装、基础配置、常用命令与日常使用流程，适合初学者快速搭建本地 Git 开发环境，并掌握最常见的版本管理操作。

------

## 2. Git 是什么

Git 是一种分布式版本控制工具，主要用于：

- 管理代码版本
- 记录文件修改历史
- 支持多人协作开发
- 支持分支开发与代码合并
- 支持与 GitHub、GitLab、Gitee 等远程仓库协同

------

## 3. Linux 中安装 Git

不同 Linux 发行版安装方式不同。

## 3.1 Ubuntu / Debian

```bash
sudo apt update
sudo apt install git -y
```

## 3.2 CentOS / Rocky / RHEL

```bash
sudo yum install git -y
```

较新的系统也可能使用：

```bash
sudo dnf install git -y
```

## 3.3 Arch Linux

```bash
sudo pacman -S git
```

## 3.4 检查是否安装成功

```bash
git --version
```

如果输出类似下面内容，说明安装成功：

```bash
git version 2.xx.x
```

------

## 4. Git 的基础配置

Git 安装完成后，建议先做全局配置。

## 4.1 配置用户名

```bash
git config --global user.name "你的名字"
```

例如：

```bash
git config --global user.name "zhangsan"
```

## 4.2 配置邮箱

```bash
git config --global user.email "你的邮箱"
```

例如：

```bash
git config --global user.email "2606275152@qq.com"
```

这个邮箱通常建议与 GitHub 或 GitLab 绑定邮箱保持一致。

## 4.3 配置默认分支名

```bash
git config --global init.defaultBranch main
```

表示以后新建仓库时，默认主分支名称为 `main`。

## 4.4 配置命令行颜色

```bash
git config --global color.ui auto
```

这样 Git 输出会更清晰。

## 4.5 查看当前配置

```bash
git config --list
```

如果只想查看某一项：

```bash
git config user.name
git config user.email
```

------

## 5. Git 配置文件说明

Git 配置一般分为三类。

## 5.1 系统级配置

对整台机器所有用户生效：

```bash
/etc/gitconfig
```

## 5.2 用户级全局配置

对当前用户生效：

```bash
~/.gitconfig
```

或：

```bash
~/.config/git/config
```

## 5.3 当前仓库配置

只对当前项目生效：

```bash
.git/config
```

------

## 6. 创建与初始化仓库

## 6.1 新建一个项目目录

```bash
mkdir my_project
cd my_project
```

## 6.2 初始化 Git 仓库

```bash
git init
```

执行后，当前目录会生成一个 `.git` 隐藏目录，表示该目录已经被 Git 管理。

## 6.3 查看仓库状态

```bash
git status
```

该命令非常常用，用于查看：

- 当前分支
- 哪些文件被修改
- 哪些文件已暂存
- 哪些文件未跟踪

------

## 7. Git 基本工作流程

Git 的常见工作流程可以概括为：

```text
工作区 -> 暂存区 -> 本地仓库
```

对应的操作通常是：

1. 修改文件
2. `git add` 加入暂存区
3. `git commit` 提交到本地仓库

------

## 8. Git 常用命令整理

## 8.1 查看状态

```bash
git status
```

用途：

- 查看文件是否修改
- 查看文件是否已经暂存
- 查看当前所在分支

------

## 8.2 查看提交历史

```bash
git log
```

常用简写形式：

```bash
git log --oneline
```

更简洁地查看提交记录。

如果想图形化查看分支历史：

```bash
git log --oneline --graph --all
```

------

## 8.3 添加文件到暂存区

添加单个文件：

```bash
git add 文件名
```

例如：

```bash
git add main.py
```

添加所有改动：

```bash
git add .
```

------

## 8.4 提交到本地仓库

```bash
git commit -m "提交说明"
```

例如：

```bash
git commit -m "完成登录功能"
```

说明应尽量写清楚本次修改做了什么。

------

## 8.5 查看文件差异

查看工作区与暂存区差异：

```bash
git diff
```

查看暂存区与本地仓库差异：

```bash
git diff --cached
```

------

## 8.6 删除文件

删除工作区和 Git 跟踪记录中的文件：

```bash
git rm 文件名
```

例如：

```bash
git rm test.txt
```

删除目录：

```bash
git rm -r 目录名
```

------

## 8.7 重命名文件

```bash
git mv 旧文件名 新文件名
```

例如：

```bash
git mv a.txt b.txt
```

------

## 8.8 查看某个文件的修改历史

```bash
git log -- 文件名
```

例如：

```bash
git log -- README.md
```

------

## 9. 分支相关命令

分支是 Git 中非常重要的功能，用于隔离不同开发任务。

## 9.1 查看分支

```bash
git branch
```

查看本地和远程所有分支：

```bash
git branch -a
```

------

## 9.2 创建分支

```bash
git branch 分支名
```

例如：

```bash
git branch dev
```

------

## 9.3 切换分支

```bash
git switch 分支名
```

例如：

```bash
git switch dev
```

旧写法也可以：

```bash
git checkout dev
```

------

## 9.4 创建并切换到新分支

```bash
git switch -c 分支名
```

例如：

```bash
git switch -c feature-login
```

旧写法：

```bash
git checkout -b feature-login
```

------

## 9.5 合并分支

先切换到目标分支，例如主分支：

```bash
git switch main
```

再执行合并：

```bash
git merge dev
```

表示把 `dev` 分支的内容合并到当前分支。

------

## 9.6 删除分支

删除已合并分支：

```bash
git branch -d 分支名
```

强制删除分支：

```bash
git branch -D 分支名
```

------

## 10. 远程仓库相关命令

远程仓库通常托管在 GitHub、GitLab、Gitee 等平台上。

## 10.1 查看远程仓库

```bash
git remote -v
```

------

## 10.2 添加远程仓库

```bash
git remote add origin 仓库地址
```

例如：

```bash
git remote add origin git@github.com:user/demo.git
```

或者：

```bash
git remote add origin https://github.com/user/demo.git
```

------

## 10.3 推送到远程仓库

第一次推送：

```bash
git push -u origin main
```

后续推送：

```bash
git push
```

------

## 10.4 拉取远程仓库内容

```bash
git pull
```

等价于先抓取再合并。

也可以先抓取：

```bash
git fetch
```

再自己决定是否合并：

```bash
git merge
```

------

## 10.5 克隆远程仓库

```bash
git clone 仓库地址
```

例如：

```bash
git clone git@github.com:user/demo.git
```

------

## 11. 撤销与恢复常用命令

这部分很重要，也是最容易混淆的部分。

## 11.1 取消暂存

```bash
git restore --staged 文件名
```

例如：

```bash
git restore --staged main.py
```

作用是把文件从暂存区撤回到工作区，但不会删除工作区修改。

------

## 11.2 撤销工作区修改

```bash
git restore 文件名
```

例如：

```bash
git restore main.py
```

作用是丢弃工作区中尚未提交的修改。

------

## 11.3 撤销最后一次提交但保留修改

```bash
git reset --soft HEAD~1
```

适合提交信息写错，或提交内容需要重新整理。

------

## 11.4 撤销最后一次提交并取消暂存

```bash
git reset --mixed HEAD~1
```

这是默认形式，保留工作区修改，但取消暂存和提交。

------

## 11.5 强制回退到某次提交

```bash
git reset --hard 提交ID
```

例如：

```bash
git reset --hard a1b2c3d
```

注意，这会丢失之后的工作区和暂存区修改，使用时要非常谨慎。

------

## 12. Git 忽略文件配置

项目中通常会有一些不需要提交的文件，例如：

- 日志文件
- 编译生成文件
- 临时文件
- 虚拟环境目录
- IDE 配置文件

可以在项目根目录创建 `.gitignore` 文件。

例如：

```gitignore
node_modules/
dist/
*.log
.env
__pycache__/
.vscode/
```

这样 Git 就不会跟踪这些文件。

------

## 13. SSH 配置建议

如果你需要免密连接 GitHub、GitLab，建议配置 SSH Key。

## 13.1 生成 SSH 密钥

```bash
ssh-keygen -t ed25519 -C "2606275152@qq.com"
```

如果系统不支持，也可以用 RSA：

```bash
ssh-keygen -t rsa -b 4096 -C "你的邮箱"
```

一路回车后，密钥通常保存在：

```bash
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

## 13.2 查看公钥

```bash
cat ~/.ssh/id_ed25519.pub
```

复制输出内容，添加到 GitHub 或 GitLab 的 SSH Keys 页面。

## 13.3 测试 SSH 连接

GitHub：

```bash
ssh -T git@github.com
```

GitLab：

```bash
ssh -T git@gitlab.com
```

------

## 14. 日常开发推荐流程

下面给出一个典型的 Git 使用流程。

## 14.1 克隆项目

```bash
git clone 仓库地址
cd 项目目录
```

## 14.2 创建开发分支

```bash
git switch -c feature-login
```

## 14.3 编写代码并查看状态

```bash
git status
```

## 14.4 添加修改

```bash
git add .
```

## 14.5 提交代码

```bash
git commit -m "实现登录页面与表单校验"
```

## 14.6 推送到远程分支

```bash
git push -u origin feature-login
```

## 14.7 合并前更新主分支

```bash
git switch main
git pull
```

## 14.8 切回开发分支并合并最新主分支

```bash
git switch feature-login
git merge main
```

处理冲突后再提交并推送。

------

## 15. 常用命令速查表

| 功能           | 命令                          |
| -------------- | ----------------------------- |
| 查看版本       | `git --version`               |
| 初始化仓库     | `git init`                    |
| 查看状态       | `git status`                  |
| 添加文件       | `git add 文件名`              |
| 添加全部改动   | `git add .`                   |
| 提交           | `git commit -m "说明"`        |
| 查看日志       | `git log --oneline`           |
| 查看差异       | `git diff`                    |
| 查看分支       | `git branch`                  |
| 创建分支       | `git branch dev`              |
| 切换分支       | `git switch dev`              |
| 创建并切换分支 | `git switch -c dev`           |
| 合并分支       | `git merge dev`               |
| 删除分支       | `git branch -d dev`           |
| 添加远程仓库   | `git remote add origin 地址`  |
| 推送           | `git push -u origin main`     |
| 拉取           | `git pull`                    |
| 克隆仓库       | `git clone 地址`              |
| 取消暂存       | `git restore --staged 文件名` |
| 撤销修改       | `git restore 文件名`          |
| 回退提交       | `git reset --soft HEAD~1`     |

------

## 16. 注意事项

## 16.1 提交前先看状态

```bash
git status
```

这是最重要的习惯之一。

## 16.2 不要随便使用 `git reset --hard`

这个命令会直接丢弃修改，恢复难度较大。

## 16.3 不要把敏感信息提交到仓库

例如：

- 密码
- Token
- 私钥
- 数据库配置

这些应写入 `.gitignore`。

## 16.4 提交说明尽量清晰

不建议总写：

```text
update
fix
modify
```

更推荐：

```text
修复登录接口参数校验问题
新增用户列表分页功能
优化首页加载性能
```

------

## 17. 总结

在 Linux 中使用 Git，核心可以归纳为三步：

1. 安装 Git
2. 完成基础配置
3. 掌握常用工作流与命令

实际使用中，最常见的命令是：

- `git status`
- `git add`
- `git commit`
- `git log`
- `git branch`
- `git switch`
- `git merge`
- `git pull`
- `git push`