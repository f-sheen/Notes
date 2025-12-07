# Git版本控制

1. ### 安装Git

2. ### 注册远程仓库（GitHub、Gitee、GitCode、GitLab）

3. ### 新建仓库

   ​	默认不勾选README.md选项

4. ### 远程连接 GitHub 有两种传输协议

   - *HTTPS：*需要个人访问令牌。即使没有配置个人访问令牌，也是可以 git clone 的，但是 git push 的时候需要输入用户名和个人访问令牌，由于访问 GitHub 的网络原因，走 HTTPS 协议可能会出现 git push 失败。如果是自己的个人项目，建议走 SSH 协议
   - *SSH：*需要密钥对。如果没有配置密钥对，既不能 git clone，也不能 git push

5. ### 基于 SSH 协议配置 Git 连接 GitHub

   - 为本机生成 SSH 密钥对

     ​	输入 cd ~/.ssh，返回"no such file or directory"表明电脑没有ssh key，需要创建ssh key，然后输入 ssh-keygen -t rsa -C “本机标识”，输入之后一路（三次）Enter（确认）就可以了，按路径进入.ssh，里面存储的是两个ssh key的秘钥，id\_rsa.pub文件里面存储的是公钥，id\_rsa文件里存储的是私钥，不能告诉别人（也可以输入cat ~/.ssh/id_rsa.pub获取）。打开id\_rsa.pub文件，复制里面的内容。如果要生成多对密钥对，输入ssh-keygen -t rsa -b 4096 -C "本机标识"（4096代表密钥更复杂，可选）
     ​	如果需要多个仓库可以编辑SSH配置文件（输入nano ~/.ssh/config），添加以下内容

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

   - 将公钥拷贝到 GitHub 上

   - SSH测试

     ```
     ssh -T git@github.com
     ```

6. ### 关联本地和远程仓库

   - 第一种方法：为本地仓库添加远程仓库

     1. 初始化仓库：git init

     2. 将想要上传的文件放到这个本地仓库文件夹下

     3. 暂存文件：git add .

        如果出现这个警告“LF will be replaced by CRLF the next time Git touches it”，可以直接忽略

     4. 提交文件：git commit -m "说明"

     5. 新建分支：git checkout -b (分支名称)

     6. 查看远程仓库信息：git remote -v

     7. 连接远程仓库：git remote add origin (远程仓库SSH链接)

     8. 推送：git push -u origin main(第一次需要设置上游分支，后续直接 git push)

   - 第二种方法：克隆远程仓库到本地

     1. 克隆远程仓库：git clone 远程仓库的SSH地址

        这样就跟 GitHub 上的远程仓库相关联了，所以就省去了 git init、git remote add 等操作

     2. 将想要上传的文件放到这个本地仓库文件夹下

     3. 暂存文件：git add .

     4. 提交文件：git commit -m "说明"

     5. 新建分支：git checkout -b (分支名称)

     6. 推送：git push -u origin main(第一次需要设置上游分支，后续直接 git push)

## Conventional Commits 约定式提交规范

​	Conventional Commits 是一种用于给提交信息增加人机可读含义的规范。约定式提交规范是一种基于消息的轻量级约定。它提供了一组用于创建清晰的提交历史的简单规则；这使得编写基于规范的自动化工具变得更容易。这个约定与 SemVer 相吻合，在提交信息中描述新特性、bug 修复和破坏性变更。
提交说明的结构如下所示：
​	

```ini
<类型>([可选的作用域]): <描述>

[可选的正文]

[可选的脚注]
```

类型（type）

- feat：类型为 feat 的提交表示在代码库中新增了一个功能（这和语义化版本中的 MINOR 相对应）。
- fix：类型为 fix 的 提交表示在代码库中修复了一个 bug （这和语义化版本中的 PATCH 相对应）。
- docs：只是更改文档。
- style：不影响代码含义的变化（空白、格式化、缺少分号等）。
- refactor：代码重构，既不修复错误也不添加功能。
- perf：改进性能的代码更改。
- test：添加确实测试或更正现有的测试。
- build：影响构建系统或外部依赖关系的更改（示例范围：gulp、broccoli、NPM）。
- ci：更改持续集成文件和脚本（示例范围：Travis、Circle、BrowserStack、SauceLabs）。
- chore：其他不修改src或test文件。
- revert：commit 回退。

范围（scope）
	可以为提交类型添加一个围在圆括号内的作用域，以为其提供额外的上下文信息。例如 feat(parser): adds ability to parse arrays.。

BREAKING CHANGE
	在可选的正文或脚注的起始位置带有 BREAKING CHANGE: 的提交，表示引入了破坏性 API 变更（这和语义化版本中的 MAJOR 相对应）。 破坏性变更可以是任意 类型 提交的一部分。

示例

- 包含了描述以及正文内有破坏性变更的提交说明

  ```ini
  feat: allow provided config object to extend other configs
  
  BREAKING CHANGE: `extends` key in config file is now used for extending other config files
  ```

  

- 包含了可选的 ！字符以提醒注意破坏性变更的提交说明

  ```ini
  chore!: drop Node 6 from testing matrix
  
  BREAKING CHANGE: dropping Node 6 which hits end of life in April
  ```

  

- 不包含正文的提交说明

  ```ini
  docs: correct spelling of CHANGELOG
  ```

  

- 包含作用域的提交说明

  ```ini
  feat(lang): add polish language
  ```

  

- 为 fix 编写的提交说明，包含（可选的） issue 编号

  ```ini
  fix: correct minor typos in code
  
  see the issue for details on the typos fixed
  
  closes issue #12
  ```

  

约定式提交规范

- 每个提交都必须使用类型字段前缀，它由一个名词组成，诸如feat或fix，其后接一个可选的作用域字段，以及一个必要的冒号（英文半角）和空格。
- 当一个提交为应用或类库实现了新特性时，必须使用feat类型。
- 当一个提交为应用修复 bug 时，必须使用fix类型。
- 作用域字段可以跟随在类型字段后面。作用有必须是一个描述某部分代码的名词，并用圆括号包围，例如：fix(parser)：
- 描述字段必须紧接在类型/作用域前缀的空格之后。描述指的是对代码变更的简短总结，例如：fix:array parsing issue when multiplejspaces were contained in string。
- 在简短描述之后，可以编写更长的提交正文，为代码变更提供额外的上下文信息。正文必须起始于描述字段结束的一个空行后。
- 在正文结束的一个空行之后，可以编写一行或或多行脚注。脚注必须包含关于提交的元信息，例如：关联的合并请求、Reviewer、破坏性变更、每条元信息一行。
- 破坏性变更必须标示在正文区域最开始处，或脚注区域中某一行的开始。一个破坏性变更必须包含大写的文本BREAKING CHANGE，后面紧跟冒号和空格。
- 在BREAKING CHANGE:之后必须提供描述，以描述对 API 的变更。例如：BREAKING CHANGE: enviroment variables now take precedence over cofig files。
- 在提交说明中，可以使用feat和fix之外的类型。
- 工具的实现必须不区分大小写地解析构成约定式提交的信息单元，只有BREAKING CHANGE 必须是大写的。
- 可以在类型/作用域前缀之后，:之前，附加!字符，以进一步提醒注意破坏性变更。当有!前缀时，正文或脚注内必须包含BREAKING CHANGE: description