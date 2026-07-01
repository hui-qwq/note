## Git 常用命令

### 用户签名， 用于确定用户身份 
```bash
git config --global user.name 用户名    # 设置用户签名
git config --global user.email 邮箱     # 设置用户签名
```

### 初始化本地库
```bash
git init 
```
### 查看状态
```bash
git status 
```
### 提交
```bash
git add 文件等
```
### 删除
```bash
git rm --cached 文件等
```
### 提交本地库
```bash
git commit -m "日志信息"
```
### 查看版本信息
```bash
git reflog  # 简约版
git log     # 详细版
```
### 版本穿梭
```bash
git reset --hard 版本号
```
