# Linux 常用操作笔记

## 1. 文件操作

### 查看文件
```bash
ls                 # 查看当前目录文件
ls -l              # 详细列表
ls -a              # 显示隐藏文件
cat 文件名         # 查看文件内容
less 文件名        # 分页查看文件内容
```

### 创建与复制
```bash
touch demo.txt                 # 创建空文件
cp a.txt b.txt                 # 复制文件
cp -r dir1 dir2                # 复制目录
```

### 移动与重命名
```bash
mv old.txt new.txt             # 重命名文件
mv a.txt /tmp/                 # 移动文件到目录
```

### 删除
```bash
rm a.txt                       # 删除文件
rm -r test_dir                 # 递归删除目录
rm -rf test_dir                # 强制递归删除（危险，谨慎使用）
```

## 2. 目录操作

### 当前路径与切换目录
```bash
pwd                            # 显示当前路径
cd /home/user                  # 进入指定目录
cd ..                          # 返回上一级目录
cd ~                           # 回到用户主目录
```

### 创建与删除目录
```bash
mkdir demo                     # 创建目录
mkdir -p a/b/c                 # 递归创建多级目录
rmdir empty_dir                # 删除空目录
rm -r demo                     # 删除非空目录
```

### 查找目录或文件
```bash
find . -name "*.md"            # 在当前目录查找 md 文件
find /var -type d -name "log*" # 查找目录名以 log 开头的目录
```

## 3. 使用 nano 编辑文件

### 打开或新建文件
```bash
nano notes.txt
```

### 常用快捷键
- `Ctrl + O`：保存（Write Out）
- `Ctrl + X`：退出
- `Ctrl + K`：剪切当前行
- `Ctrl + U`：粘贴
- `Ctrl + W`：搜索
- `Ctrl + \`：替换
- `Ctrl + G`：打开帮助

### 示例流程
1. 执行 `nano notes.txt` 打开文件。
2. 输入内容后按 `Ctrl + O` 保存，回车确认文件名。
3. 按 `Ctrl + X` 退出编辑器。

## 4. 实用补充命令

```bash
history                        # 查看历史命令
clear                          # 清屏
man ls                         # 查看命令手册
```

