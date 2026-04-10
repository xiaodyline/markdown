下面按“能直接照做”的顺序给你。

# 一、整体思路

SSH 免密登录不是“完全不要验证”，而是把原来的“输入服务器账户密码”，改成：

- 你本地保存一把**私钥**
- 服务器保存对应的**公钥**
- 连接时服务器拿公钥校验你本地私钥
- 验证通过后登录

你后面在 VS Code 里远程连接，通常就会顺畅很多。

------

# 二、先确认你的环境

你本机是 Windows，通常直接用 **OpenSSH** 就够了，不用额外装 Git Bash 或别的工具。

先在 **PowerShell** 里执行：

```powershell
ssh -V
```

如果能看到类似：

```powershell
OpenSSH_for_Windows_...
```

说明本机 SSH 可用。

再检查你有没有 `.ssh` 目录：

```powershell
dir $env:USERPROFILE\.ssh
```

一般路径就是：

```powershell
C:\Users\你的用户名\.ssh
```

------

# 三、先测试当前服务器能不能正常密码登录

先别急着配密钥，先确认服务器本身是通的。

在 PowerShell 执行：

```powershell
ssh 用户名@服务器IP
```

例如：

```powershell
ssh root@192.168.1.100
```

看是否出现：

```powershell
root@192.168.1.100's password:
```

这时直接输入密码，**不会显示字符**，按回车即可。

### 这里你要先确认三件事

#### 1. 能连通

如果报：

- `Connection timed out`
- `No route to host`
- `Connection refused`

那不是密钥问题，是网络或 SSH 服务问题。

#### 2. 用户名和密码正确

如果报：

- `Permission denied`

说明用户名、密码，或者服务器 SSH 配置有问题。

#### 3. 能正常进系统

只有“密码登录本来就正常”的前提下，再配免密才有意义。

------

# 四、在 Windows 本机生成 SSH 密钥

在 PowerShell 执行：

```powershell
ssh-keygen -t ed25519
```

如果你的环境太旧不支持 `ed25519`，再退回 RSA：

```powershell
ssh-keygen -t rsa -b 4096
```

执行后一般会看到类似提示：

```powershell
Enter file in which to save the key (C:\Users\你的用户名\.ssh\id_ed25519):
```

直接按回车，使用默认路径即可。

然后会提示：

```powershell
Enter passphrase (empty for no passphrase):
```

这里有两种选择。

### 方案 A：不设置私钥口令

直接回车两次。

优点：

- 最省事
- VS Code 基本不会再问你东西

缺点：

- 如果你本地私钥文件泄露，别人拿到就能尝试登录你的服务器

### 方案 B：设置私钥口令

输入一个你自己记得住的口令。

优点：

- 更安全

缺点：

- 第一次使用私钥时，可能需要输入这个口令
- 但后面可以配合 `ssh-agent` 缓存

### 建议

对你这种刚开始配置、主要想先用起来的情况，可以先：

- **先不设 passphrase**
- 等你完全用顺了，再考虑加口令和 agent

生成完成后，你会得到两个文件：

```powershell
C:\Users\你的用户名\.ssh\id_ed25519
C:\Users\你的用户名\.ssh\id_ed25519.pub
```

其中：

- `id_ed25519` 是**私钥**，不能发给别人
- `id_ed25519.pub` 是**公钥**，要放到服务器上

------

# 五、把公钥上传到服务器

## 方法一：手动复制，最稳

先查看公钥内容：

```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub
```

会输出一整行，类似：

```powershell
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI...... yourname@PC
```

把这一整行完整复制下来。

然后继续用密码方式登录服务器：

```powershell
ssh 用户名@服务器IP
```

登录后，在服务器里执行：

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
```

如果没有 `nano`，也可以用：

```bash
vi ~/.ssh/authorized_keys
```

把刚才复制的那一整行公钥粘贴进去，一行一个公钥。

保存后执行：

```bash
chmod 600 ~/.ssh/authorized_keys
```

然后退出服务器：

```bash
exit
```

------

## 方法二：一条命令追加公钥

如果你愿意，也可以在 Windows 本机这样做：

```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh 用户名@服务器IP "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

这条命令的意思是：

- 读取本地公钥
- 通过 SSH 登录服务器
- 自动追加到 `authorized_keys`

第一次仍然会要求你输入**服务器密码**

### 注意

如果你重复执行多次，`authorized_keys` 里可能会有重复公钥，一般问题不大，但不够整洁。

------

# 六、测试是否免密成功

回到本机 PowerShell，执行：

```powershell
ssh 用户名@服务器IP
```

如果现在能直接进服务器，说明成功了。

如果你设置了私钥 passphrase，可能会提示输入的是**私钥口令**，不是服务器账户密码。

这两者区别很大：

- **服务器密码**：服务器上那个 Linux 用户的密码
- **私钥口令**：你本地给私钥文件加的一层保护

------

# 七、给 VS Code 配置 SSH Host

现在开始配 VS Code。

打开本地 SSH 配置文件：

```powershell
notepad $env:USERPROFILE\.ssh\config
```

如果文件不存在，会提示创建，直接创建。

写入类似内容：

```ssh
Host myserver
    HostName 你的服务器IP
    User 你的用户名
    IdentityFile C:\Users\你的用户名\.ssh\id_ed25519
```

例如：

```ssh
Host gpu-server
    HostName 192.168.1.100
    User root
    IdentityFile C:\Users\ld\.ssh\id_ed25519
```

### 每一项的意思

- `Host`
  这是你自己起的别名，后面 VS Code 里会显示它
- `HostName`
  服务器真实 IP 或域名
- `User`
  登录用户名
- `IdentityFile`
  你本地私钥路径

### 如果端口不是 22

加一行：

```ssh
Port 2222
```

完整例子：

```ssh
Host gpu-server
    HostName 192.168.1.100
    User root
    Port 22
    IdentityFile C:\Users\ld\.ssh\id_ed25519
```

------

# 八、在 VS Code 里连接

先确保你装了扩展：

- `Remote - SSH`

然后：

1. 按 `F1`
2. 输入 `Remote-SSH: Connect to Host...`
3. 选择你刚才配置的 `gpu-server`
4. 选择打开的窗口类型
5. 等它连接

第一次连接通常会多做几件事：

- 确认主机指纹
- 在远程安装 VS Code Server
- 建立远程工作区

如果密钥配置好了，原则上就不需要你再输服务器密码了。

------

# 九、如果你给私钥设置了口令，建议再配 ssh-agent

如果你给私钥设了 passphrase，每次都输也很烦，这时可以让 Windows 记住。

## 1. 启动 ssh-agent 服务

PowerShell 以管理员身份运行：

```powershell
Get-Service ssh-agent
```

如果没启动，执行：

```powershell
Set-Service -Name ssh-agent -StartupType Automatic
Start-Service ssh-agent
```

## 2. 把私钥加入 agent

普通 PowerShell 执行：

```powershell
ssh-add $env:USERPROFILE\.ssh\id_ed25519
```

输入一次私钥口令后，后面很多场景就不用反复输了。

## 3. 查看是否加入成功

```powershell
ssh-add -l
```

如果能看到你的密钥指纹，说明已经生效。

------

# 十、建议的最终配置方式

你可以按下面两种模式选一种。

## 模式 1：先求稳，用起来最省事

- 用 `ed25519`
- 私钥**不设口令**
- VS Code 直接走 `IdentityFile`

适合：

- 个人电脑
- 先快速搭建远程开发环境
- 当前最重视“先能用”

## 模式 2：长期使用，更规范

- 用 `ed25519`
- 私钥**设置口令**
- 配 `ssh-agent`
- VS Code 继续走 `IdentityFile`

适合：

- 长期开发
- 机器比较重要
- 你担心私钥文件安全

------

# 十一、常见问题排查

## 1. 明明配了公钥，还是要输密码

先看本机是否真的用上了指定私钥：

```powershell
ssh -v 用户名@服务器IP
```

或：

```powershell
ssh -v gpu-server
```

重点看日志里有没有：

- `Offering public key`
- `Authentications that can continue`
- `Permission denied`

如果没看到它在尝试你的私钥，通常是本地配置没写对。

常见原因：

- `IdentityFile` 路径错了
- `.ssh\config` 格式不对
- 连接的不是你以为的那个 Host
- 实际用户名写错了

------

## 2. 服务器公钥文件权限不对

服务器端必须注意权限：

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

有些服务器权限不对时，会直接忽略公钥认证。

------

## 3. 服务器不允许公钥登录

服务器里看一下 SSH 配置：

```bash
sudo grep -E "PubkeyAuthentication|PasswordAuthentication" /etc/ssh/sshd_config
```

常见期望值是：

```bash
PubkeyAuthentication yes
```

如果改了 SSH 配置，通常要重启服务：

```bash
sudo systemctl restart ssh
```

有些系统服务名是：

```bash
sudo systemctl restart sshd
```

### 注意

这个步骤要谨慎。
如果你是远程机器，改错 SSH 配置可能把自己锁在外面。
所以建议先确保“当前会话不要断”，并且先别急着关密码登录。

------

## 4. root 用户不能密钥登录

有些服务器对 root 登录有限制。检查：

```bash
sudo grep PermitRootLogin /etc/ssh/sshd_config
```

常见情况：

- `PermitRootLogin prohibit-password`
- `PermitRootLogin yes`
- `PermitRootLogin no`

如果是 `prohibit-password`，意思通常是：

- root 可以用密钥登录
- 但不能用密码登录

------

## 5. VS Code 里还是没有密码输入框

这时不要盯着“弹窗”，而是看日志。

打开：

- `查看` → `输出`
- 下拉选择 `Remote - SSH`

看它到底卡在哪一步。最常见不是“没法输密码”，而是：

- 正在等你在终端里输入
- 正在安装 VS Code Server
- 网络慢
- 远程 shell 启动异常

------

# 十二、推荐你这样做

这是最适合你的落地步骤：

## 第一步

先在 PowerShell 验证：

```powershell
ssh 用户名@服务器IP
```

确认服务器密码登录本来就没问题。

## 第二步

生成密钥：

```powershell
ssh-keygen -t ed25519
```

## 第三步

把公钥写到服务器：

```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub
```

复制后放进服务器的：

```bash
~/.ssh/authorized_keys
```

## 第四步

本机测试：

```powershell
ssh 用户名@服务器IP
```

确认已经能免密或只输私钥口令。

## 第五步

配置：

```ssh
C:\Users\你的用户名\.ssh\config
```

例如：

```ssh
Host gpu-server
    HostName 你的服务器IP
    User 你的用户名
    IdentityFile C:\Users\你的用户名\.ssh\id_ed25519
```

## 第六步

在 VS Code 里通过 `Remote-SSH: Connect to Host...` 连接。

------

# 十三、给你一个可直接替换的模板

假设：

- Windows 用户目录：`C:\Users\ld`
- 服务器 IP：`192.168.1.100`
- 服务器用户名：`root`

那么你可以直接这样配。

## 本地生成密钥

```powershell
ssh-keygen -t ed25519
```

## 查看公钥

```powershell
type C:\Users\ld\.ssh\id_ed25519.pub
```

## 服务器写入公钥后，本地配置文件

文件：

```powershell
C:\Users\ld\.ssh\config
```

内容：

```ssh
Host gpu-server
    HostName 192.168.1.100
    User root
    IdentityFile C:\Users\ld\.ssh\id_ed25519
```

## 测试

```powershell
ssh gpu-server
```

成功后，再去 VS Code 里连 `gpu-server`。

------

# 十四、补充一个安全建议

刚配好免密后，**先不要立刻关闭密码登录**。

原因是：

- 你要先确认 VS Code 能正常连
- 确认密钥没问题
- 确认服务器重启后仍然能连

等你验证稳定了，再考虑把服务器改成：

- 禁止密码登录
- 只允许密钥登录