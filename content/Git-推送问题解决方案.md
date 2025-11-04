---
tags:
  - Git
  - GitHub
created: 2025-10-29T14:48
updated: 2025-11-04T13:54
share: true
---

# Git 推送到 GitHub 失败 - 解决方案

> 相关导航：[[Git版本控制-MOC|Git版本控制-MOC]]

## 问题描述

错误信息：`fatal: unable to access 'https://github.com/luckybearbear/kb.git/': recv failure: connection was reset`

这是访问 GitHub 时的网络连接问题，在中国大陆比较常见。

---

## 🚀 推荐解决方案（按优先级）

### 方案1：配置代理（推荐 - 如果您有代理）

如果您使用科学上网工具（如 Clash、V2Ray 等），需要配置 Git 使用代理。

#### 1.1 查看代理端口
- Clash：通常是 `7890`
- V2Ray：通常是 `10808` 或 `1080`
- 其他工具：查看软件设置中的代理端口

#### 1.2 配置 HTTP 代理
```bash
# HTTP/HTTPS 代理（最常用）
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890

# 如果是其他端口，修改端口号，例如：
git config --global http.proxy http://127.0.0.1:10808
git config --global https.proxy http://127.0.0.1:10808
```

#### 1.3 配置 SOCKS5 代理（可选）
```bash
# SOCKS5 代理
git config --global http.proxy socks5://127.0.0.1:7890
git config --global https.proxy socks5://127.0.0.1:7890
```

#### 1.4 只为 GitHub 配置代理（推荐）
```bash
# 只对 GitHub 使用代理，国内仓库不受影响
git config --global http.https://github.com.proxy http://127.0.0.1:7890
git config --global https.https://github.com.proxy http://127.0.0.1:7890
```

#### 1.5 取消代理（如果配置错误）
```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
git config --global --unset http.https://github.com.proxy
git config --global --unset https.https://github.com.proxy
```

---

### 方案2：使用 SSH 方式（更稳定）

SSH 方式通常比 HTTPS 更稳定，推荐长期使用。

#### 2.1 生成 SSH 密钥
```bash
# 生成 SSH 密钥（如果还没有）
ssh-keygen -t ed25519 -C "your_email@example.com"

# 或者使用 RSA（兼容性更好）
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# 一路回车即可
```

#### 2.2 查看并复制公钥
```bash
# 查看公钥
cat ~/.ssh/id_ed25519.pub
# 或
cat ~/.ssh/id_rsa.pub

# Windows 用户可以用：
type %USERPROFILE%\.ssh\id_ed25519.pub
```

#### 2.3 添加到 GitHub
1. 复制输出的公钥内容
2. 打开 https://github.com/settings/keys
3. 点击 "New SSH key"
4. 粘贴公钥，保存

#### 2.4 测试 SSH 连接
```bash
ssh -T git@github.com

# 成功会显示：
# Hi username! You've successfully authenticated...
```

#### 2.5 修改远程仓库为 SSH
```bash
# 查看当前远程地址
git remote -v

# 修改为 SSH 地址
git remote set-url origin git@github.com:luckybearbear/kb.git

# 验证修改
git remote -v

# 推送测试
git push origin main
```

#### 2.6 配置 SSH 代理（如果 SSH 也需要代理）
Windows 用户在 `~/.ssh/config` 中添加：
```
Host github.com
    User git
    ProxyCommand connect -H 127.0.0.1:7890 %h %p
```

---

### 方案3：修改 hosts 文件（临时方案）

通过修改 hosts 文件，使用 GitHub 的 IP 地址直接访问。

#### 3.1 查询 GitHub IP
访问 https://www.ipaddress.com/ 查询以下域名的 IP：
- `github.com`
- `github.global.ssl.fastly.net`

#### 3.2 编辑 hosts 文件
**Windows**：`C:\Windows\System32\drivers\etc\hosts`
**Mac/Linux**：`/etc/hosts`

添加（IP 地址可能需要更新）：
```
140.82.113.4 github.com
199.232.69.194 github.global.ssl.fastly.net
```

#### 3.3 刷新 DNS
```bash
# Windows
ipconfig /flushdns

# Mac
sudo killall -HUP mDNSResponder

# Linux
sudo systemd-resolve --flush-caches
```

---

### 方案4：使用 GitHub 镜像站（临时）

如果以上方法都不行，可以临时使用镜像。

```bash
# 修改为镜像地址（仅做临时推送）
git remote set-url origin https://hub.fastgit.xyz/luckybearbear/kb.git

# 推送
git push origin main

# 推送完成后改回原地址
git remote set-url origin https://github.com/luckybearbear/kb.git
```

⚠️ **注意**：镜像站可能不稳定，仅用于临时应急。

---

### 方案5：增加重试和超时设置

```bash
# 增加缓冲区
git config --global http.postBuffer 524288000

# 禁用低速限制
git config --global http.lowSpeedLimit 0
git config --global http.lowSpeedTime 999999

# 增加超时时间
git config --global http.timeout 300
```

---

## 🔍 问题诊断命令

### 测试网络连接
```bash
# 测试 GitHub 连接
ping github.com

# 测试 HTTPS 连接
curl -I https://github.com

# 查看 Git 配置
git config --list | grep -E "(http|https|proxy)"

# 测试 SSH 连接
ssh -T git@github.com
```

### 查看详细错误
```bash
# 使用详细模式推送
GIT_CURL_VERBOSE=1 git push origin main

# 或
GIT_TRACE=1 git push origin main
```

---

## ✅ 最终推荐配置（组合使用）

对于中国大陆用户，推荐的最佳配置组合：

```bash
# 1. 生成并配置 SSH（一次性）
ssh-keygen -t ed25519 -C "your_email@example.com"
# 添加公钥到 GitHub

# 2. 切换到 SSH 方式
git remote set-url origin git@github.com:luckybearbear/kb.git

# 3. 如果有代理，配置代理
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890

# 4. 优化性能配置
git config --global http.postBuffer 524288000
git config --global http.lowSpeedLimit 0
git config --global http.lowSpeedTime 999999
```

---

## 🆘 常见问题

### Q1: 我不知道代理端口怎么办？
**A**:
1. 打开你的代理工具（Clash/V2Ray 等）
2. 查看设置 → 端口设置
3. 找到 HTTP 代理端口或 SOCKS5 端口
4. 常见端口：7890、10808、1080、8080

### Q2: 配置了代理还是不行？
**A**:
1. 确认代理工具正在运行
2. 在浏览器中测试能否访问 GitHub
3. 尝试切换 HTTP 和 SOCKS5 代理
4. 尝试使用 SSH 方式

### Q3: SSH 方式也连不上？
**A**:
1. 测试 SSH 连接：`ssh -T git@github.com`
2. 如果超时，可能需要配置 SSH 代理
3. 编辑 `~/.ssh/config` 添加代理配置

### Q4: 临时推送一次怎么办？
**A**:
使用镜像站临时推送，或者使用 Obsidian-git 插件的自动推送功能（它会自动重试）。

---

## 📝 当前状态

- 当前仓库：https://github.com/luckybearbear/kb.git
- 使用方式：HTTPS
- 建议：切换到 SSH + 配置代理

---

## 🔗 相关链接

- [GitHub SSH 文档](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh)
- [Git 代理配置](https://gist.github.com/laispace/666dd7b27e9116faece6)
- [常见 Git 网络问题](https://github.com/hawtim/blog/issues/10)

---

**💡 提示**：推荐先尝试方案1（配置代理）或方案2（SSH），这两种是最稳定的长期解决方案。
