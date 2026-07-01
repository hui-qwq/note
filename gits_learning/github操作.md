## github操作
 [github网址](https://github.com)

### 1.1 创建远程仓库
远程库的创建在网站上， bilibili搜索下应该很容易就能学会
#### HTTPS 类
```bash
git remote add 别名 远程地址 # 给库网址起一个别名，方便提交
git remote -v # 查看所有别名
git push 仓库名 分支
git pull 仓库名 分支
git clone 远程地址.git
```
#### SSH
```bash
ssh-keygen -t rsa -C 邮箱 #ssh的免密登录， rsa就是非对称加密
```
将公匙赋入github里
之后以公钥为远程地址名， 就可以免密