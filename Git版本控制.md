# Git版本控制

1. ### 选择远程仓库（GitHub、Gitee、GitCode、GitLab）

2. ### 新建仓库

   默认不勾选README.md选项

3. ### 获取SSH keys

   输入 cd ~/.ssh，返回"no such file or directory"表明电脑没有ssh key，需要创建ssh key，然后输入 ssh-keygen -t rsa -C “git账号”，输入之后一路（三次）Enter（确认）就可以了，按路径进入.ssh，里面存储的是两个ssh key的秘钥，id\_rsa.pub文件里面存储的是公钥，id\_rsa文件里存储的是私钥，不能告诉别人（也可以输入cat ~/.ssh/id_rsa.pub获取）。打开id\_rsa.pub文件，复制里面的内容。如果要生成多对密钥对，输入ssh-keygen -t rsa -b 4096 -C "git账号"（4096代表密钥更复杂，可选），提示输入保存路径Enter file in which to save the key (/c/Users/YourName/.ssh/id_rsa): /c/Users/YourName/.ssh/id_rsa_new，编辑SSH配置文件（输入nano ~/.ssh/config），添加以下内容

   ```
   # 公司 GitLab 账户（使用默认密钥）
   Host company-gitlab #公司gitlab地址
       HostName company-gitlab #公司gitlab地址
       User git
       IdentityFile ~/.ssh/id_rsa
   
   # 个人 GitHub 账户（使用新密钥）
   Host personal-github 
       HostName personal-github  
       User git
       IdentityFile ~/.ssh/id_rsa_new
   ```

   可能出现的问题：无法连接到远程仓库

   ```
   Host github.com  #别名/匹配模式，根据远程仓库地址，匹配对应的Host配置
       HostName ssh.github.com  #实际连接地址
       User git
       Port 443
       IdentityFile ~/.ssh/id_rsa_new
       TCPKeepAlive yes
       IdentitiesOnly yes
   ```

   验证连接（ssh -T git@github.com）

4. ### 个人配置

   - 全局配置：

     ```
     git config --global user.name "用户名"
     git config --global user.email "邮箱"
     ```

   - 局部配置：进入项目文件夹

     ```
     git config user.name "用户名"
     git config user.email "邮箱"
     ```

5. ### 进入文件夹

   - 初始化仓库：git init
   - 暂存文件：git add .
   - 提交文件：git commit -m "说明"
   - 新建分支：git checkout -b (分支名称)
   - 连接远程仓库：git remote add origin (远程仓库SSH链接)
   - 推送：git push -u origin main(第一次需要设置上游分支，后续直接 git push)