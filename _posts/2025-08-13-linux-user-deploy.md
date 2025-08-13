---
title: Deploy用户配置与权限管理指南
author: ZQ7
date: 2025-08-13 14:00:00 +0800
categories: [Linux,User]
tags: [Linux]
---

## 目录

1. [创建 Deploy 用户](#1-创建-deploy-用户)
2. [创建 Deploy 用户组](#2-创建-deploy-用户组)
3. [配置目录权限](#3-配置目录权限)
4. [SSH 配置](#4-ssh-配置)
5. [权限验证](#5-权限验证)
6. [故障排查](#6-故障排查)
7. [安全建议](#7-安全建议)

---

## 1. 创建 Deploy 用户

### 1.1 创建新用户

```bash
# 创建 deploy 用户（带home目录）
sudo useradd -m -s /bin/bash deploy

# 设置密码（会提示输入密码）
sudo passwd deploy

# 或者创建用户时直接指定密码（不推荐在生产环境使用）
# echo "deploy:your_password" | sudo chpasswd
```

### 1.2 检查用户是否创建成功

```bash
# 查看用户信息
id deploy

# 查看用户家目录
ls -la /home/deploy/

# 查看所有用户列表
cat /etc/passwd | grep deploy
```

---

## 2. 创建 Deploy 用户组

### 2.1 创建部署专用组

```bash
# 创建 deploy 组（如果不存在）
sudo groupadd deploy 2>/dev/null || echo "组 deploy 已存在"

# 查看组信息
getent group deploy
```

### 2.2 将用户加入组

```bash
# 将 deploy 用户加入 deploy 组
sudo usermod -a -G deploy deploy

# 如果有其他用户需要部署权限，也可以加入此组
# sudo usermod -a -G deploy other_username

# 验证用户组设置
groups deploy
```

---

## 3. 配置目录权限

### 3.1 创建部署目录结构

```bash
# 创建必要的目录
sudo mkdir -p /srv/mobile/dev/dist
sudo mkdir -p /srv/mobile/dev/.bak
sudo mkdir -p /srv/mobile/prod/dist
sudo mkdir -p /srv/mobile/prod/.bak
```

### 3.2 设置目录权限

```bash
# 方案A：deploy 用户完全控制（简单但安全性较低）
sudo chown -R deploy:deploy /srv/mobile/
sudo chmod -R 755 /srv/mobile/

# 方案B：root 拥有，deploy 组可读写（推荐）
sudo chown -R root:deploy /srv/mobile/
sudo chmod -R 775 /srv/mobile/

# 方案C：更细粒度的权限控制
sudo chown -R root:deploy /srv/mobile/
sudo chmod 2775 /srv/mobile/              # SGID 确保新文件继承组
sudo chmod -R 775 /srv/mobile/dev/
sudo chmod -R 755 /srv/mobile/prod/       # 生产环境更严格
```

### 3.3 设置 SGID（推荐）

```bash
# 设置 SGID，确保新创建的文件自动归属 deploy 组
sudo chmod g+s /srv/mobile/
sudo chmod g+s /srv/mobile/dev/
sudo chmod g+s /srv/mobile/prod/
```

---

## 4. SSH 配置

### 4.1 配置 SSH 密钥认证（推荐）

```bash
# 切换到 deploy 用户
sudo su - deploy

# 创建 .ssh 目录
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# 生成 SSH 密钥对（在本地机器执行）
ssh-keygen -t rsa -b 4096 -C "deploy@yourserver"

# 将公钥添加到服务器（在服务器执行）
echo "your_public_key_content" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# 退出 deploy 用户
exit
```

### 4.2 配置 SSH 服务

```bash
# 编辑 SSH 配置（可选）
sudo vim /etc/ssh/sshd_config

# 建议的安全配置：
# PermitRootLogin no
# PasswordAuthentication yes  # 如果使用密钥，可以设为 no
# PubkeyAuthentication yes
# AllowUsers deploy  # 只允许特定用户

# 重启 SSH 服务
sudo systemctl restart sshd
# 或
sudo service ssh restart
```

### 4.3 配置 sudo 权限（可选）

```bash
# 如果 deploy 用户需要某些 sudo 权限
sudo visudo

# 添加以下行（根据需要选择）：
# 允许 deploy 用户无密码重启服务
deploy ALL=(ALL) NOPASSWD: /bin/systemctl restart nginx
deploy ALL=(ALL) NOPASSWD: /bin/systemctl reload nginx

# 或者允许所有 sudo 操作（不推荐）
# deploy ALL=(ALL) NOPASSWD: ALL
```

---

## 5. 权限验证

### 5.1 完整验证脚本

```bash
#!/bin/bash
# 保存为 check_deploy_permissions.sh

echo "========== Deploy 用户权限检查 =========="
echo ""

# 1. 检查用户
echo "1. 用户信息："
if id deploy &>/dev/null; then
    id deploy
else
    echo "✗ 用户 deploy 不存在"
    exit 1
fi
echo ""

# 2. 检查组
echo "2. 用户组信息："
groups deploy
echo ""

# 3. 检查目录权限
echo "3. 目录权限："
ls -la /srv/mobile/ 2>/dev/null || echo "✗ /srv/mobile/ 不存在"
ls -la /srv/mobile/dev/ 2>/dev/null || echo "✗ /srv/mobile/dev/ 不存在"
ls -la /srv/mobile/dev/dist/ 2>/dev/null || echo "✗ /srv/mobile/dev/dist/ 不存在"
echo ""

# 4. 测试写权限
echo "4. 测试写权限："
TEST_FILE="/srv/mobile/test_$(date +%s).txt"
if sudo -u deploy touch ${TEST_FILE} 2>/dev/null; then
    echo "✓ deploy 用户可以在 /srv/mobile/ 创建文件"
    sudo -u deploy rm ${TEST_FILE}
else
    echo "✗ deploy 用户无法在 /srv/mobile/ 创建文件"
fi
echo ""

# 5. 测试 SSH 登录
echo "5. SSH 配置："
if [ -d /home/deploy/.ssh ]; then
    echo "✓ SSH 目录存在"
    ls -la /home/deploy/.ssh/
else
    echo "✗ SSH 目录不存在"
fi
echo ""

# 6. 检查 sudo 权限
echo "6. Sudo 权限："
sudo -l -U deploy 2>/dev/null || echo "无特殊 sudo 权限"
echo ""

echo "========== 检查完成 =========="
```

### 5.2 运行验证

```bash
# 赋予执行权限
chmod +x check_deploy_permissions.sh

# 执行检查
sudo ./check_deploy_permissions.sh
```

### 5.3 手动测试部署

```bash
# 测试 SSH 登录
ssh deploy@localhost

# 测试文件操作
touch /srv/mobile/dev/test.txt
echo "test" > /srv/mobile/dev/test.txt
rm /srv/mobile/dev/test.txt

# 测试目录创建
mkdir -p /srv/mobile/dev/test_dir
rmdir /srv/mobile/dev/test_dir

# 退出
exit
```

---

## 6. 故障排查

### 6.1 常见问题及解决方案

#### 问题1：Permission denied

```bash
# 检查目录所有者和权限
ls -la /srv/mobile/

# 重新设置权限
sudo chown -R root:deploy /srv/mobile/
sudo chmod -R 775 /srv/mobile/

# 确保用户在正确的组
sudo usermod -a -G deploy deploy

# 用户需要重新登录生效
```

#### 问题2：用户无法 SSH 登录

```bash
# 检查 SSH 服务状态
sudo systemctl status sshd

# 检查 SSH 配置
sudo grep -E "AllowUsers|DenyUsers|PermitRootLogin" /etc/ssh/sshd_config

# 查看 SSH 日志
sudo tail -f /var/log/auth.log  # Ubuntu/Debian
sudo tail -f /var/log/secure    # CentOS/RHEL
```

#### 问题3：组权限不生效

```bash
# 用户需要重新登录
pkill -u deploy  # 强制用户重新登录

# 或重启 SSH 服务
sudo systemctl restart sshd
```

### 6.2 权限修复脚本

```bash
#!/bin/bash
# 保存为 fix_deploy_permissions.sh

echo "修复 deploy 用户权限..."

# 确保用户和组存在
sudo useradd -m -s /bin/bash deploy 2>/dev/null || echo "用户已存在"
sudo groupadd deploy 2>/dev/null || echo "组已存在"

# 将用户加入组
sudo usermod -a -G deploy deploy

# 创建目录
sudo mkdir -p /srv/mobile/dev/dist
sudo mkdir -p /srv/mobile/dev/.bak
sudo mkdir -p /srv/mobile/prod/dist
sudo mkdir -p /srv/mobile/prod/.bak

# 设置权限
sudo chown -R root:deploy /srv/mobile/
sudo chmod -R 775 /srv/mobile/
sudo chmod g+s /srv/mobile/

# 重启服务
sudo systemctl restart sshd

echo "修复完成！"
echo "请让 deploy 用户重新登录以使权限生效。"
```

---

## 7. 安全建议

### 7.1 基本安全措施

1. **使用 SSH 密钥认证**：禁用密码登录
2. **限制 sudo 权限**：只给必要的权限
3. **定期审计**：检查用户活动和文件变更
4. **备份策略**：保留部署历史和回滚能力

### 7.2 高级安全配置

```bash
# 1. 限制 SSH 访问
# 编辑 /etc/ssh/sshd_config
AllowUsers deploy
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes

# 2. 设置文件审计
sudo apt-get install auditd  # Ubuntu/Debian
sudo yum install audit       # CentOS/RHEL

# 添加审计规则
sudo auditctl -w /srv/mobile/ -p wa -k deploy_changes

# 3. 配置防火墙
sudo ufw allow from 特定IP to any port 22  # 只允许特定 IP SSH
```

### 7.3 监控和日志

```bash
# 查看部署用户的操作历史
last deploy

# 查看文件修改记录
find /srv/mobile -type f -mtime -1 -ls  # 最近一天修改的文件

# 设置日志轮转
cat > /etc/logrotate.d/deploy << EOF
/srv/mobile/dev/.bak/*.tar.gz {
    weekly
    rotate 4
    compress
    missingok
    notifempty
}
EOF
```

---
