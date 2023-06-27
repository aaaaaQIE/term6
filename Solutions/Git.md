# Git基本教程

## 基础配置

```
git config --system --list	#查看关于git的系统配置
git config -l	#查看关于git的所有配置，l指list
git config --global --list	#查看关于git的全局用户配置，必须配置
git config --global user.name "your_github_username"	#设置用户名
git config --global user.email "your_github_email"	#设置邮箱
```

## 仓库搭建

### 一、克隆远程仓库法

进入 github ，登录后点击右上角头像中的 Settings

点击左边栏 Access 中的 SSH and GPG keys

在 SSH keys 旁点击 New SSH key

回到电脑本地，在终端中输入

```
cd /home/user/.ssh	#进入电脑的.ssh目录，通常在用户的根目录，为隐藏文件夹，也可以直接进行下一步
ssh-keygen -t rsa	#生成公钥，建议选择默认路径，之后一路回车下去
vim id_rsa.pub	#把公钥内容全部复制下来，另一个id_rsa文件为私钥不用管
```

将复制的内容粘贴到 GitHub 的 Key 中， Title 可以随意填写， Key type 选择 Authentication Key ，完成后点击 Add SSH key

点击右上角加号中的 New repository ，建议勾选 Add a README file，填写完成后 Create repository

回到首页进入新的 repo（repository），点击绿色 Code 按钮，复制 Clone-HTTPS 里的链接 ``` https://github.com/user/repo.git``` 

回到电脑本地，在终端中输入

```
git clone https://github.com/user/repo.git
```

即完成本地+远程仓库的搭建，repo文件夹则成为工作文件夹

### 二、本地仓库搭建法

在终端中输入

```
git init	#在当前目录初始化git环境，会生成一个隐藏的.git文件夹
git remote add origin https://github.com/user/repo.git	#将本地文件与GitHub远程仓库关联
```

```origin``` ——项目主机名，亦可随意选定，可通过 ```git remote``` 查询

## 文件操作

```
git add .	#将git工作目录中所有文件添加至暂存区，参与版本控制，状态变为Staged暂存状态
git commit -m <memo>	#将暂存区文件添加到本地仓库，m代表message，可为每次commit添加说明
git push	#将本地仓库文件推送至远程服务器
```

```git status``` ：查看git状态

```git remote``` ：查看当前项目主机名

```git remote -v``` ：查看当前仓库的远程 URL

```git remote set-url origin git@github.com:anotherrepo.git``` ：修改远程 URL

```git rm <file>``` ：从暂存区和工作区删除文件

可建立 ```.gitignore``` 配置不纳入版本控制的文件，具体语法如下

```
file.txt	#忽略file.txt文件
*.txt		#忽略所有.txt结尾的文件
!lib.txt	#但lib.txt除外
temp/		#忽略temp目录下的所有文件和目录
/folder		#忽略folder目录中的文件，而其子目录中的文件不忽略
```

## 错误总结

1. ```git push``` 时提示： ```SSL certificate problem: unable to get local issuer certificate``` 

   这是因为 Git 默认开启了 SSL 验证，关闭即可，执行以下命令：

   ```
   git config --global http.sslVerify false
   ```

2. ```git push``` 时提示： ```Updates were rejected because the remote contains work that you do```

   这是因为原先的项目中还有文件，先 ```git pull``` 下来，合并处理一下冲突

3. ```git push``` 时提示： ```Updates were rejected because a pushed branch tip is behind its remote``` 

   错误原因其实是本地没有 ```README.md``` 这个文件，而远程仓库中有，执行下列命令：

   ```
   git pull --rebase origin master
   ```

4. ```git push``` 每次都需要输入帐号密码，浪费时间，执行以下命令

   ```
   git config --global credential.helper store
   ```

   执行完上面的 git 命令后，在命令行正常执行 pull、push ，如果是在以上操作完之后第一次执行 pull、push ，需要输入一次用户名密码，以后不再需要输入。

5. ```sign_and_send_pubkey: signing failed: agent refused operation``` 

   这是由于ssh的环境发生了变化，需要执行如下命令即可解决：

   ```
   eval "$(ssh-agent -s)"
   ssh-add
   ```

   需要注意的是.ssh里面文件的权限非常重要，权限错了否则不能正常登陆！

   因为sshd为了安全，对属主的目录和文件权限有所要求，需要保障其他用户不能有写权限。如果权限不对，则ssh的免密码登陆不生效。

   ```.ssh``` 目录权限一般为 ```755``` 或者 ```700``` 。

   ```id_rsa.pub``` 及 ```authorized_keys``` 权限一般为 ```644``` 

   ```id_rsa``` 权限必须为 ```600``` 

   处理完成后再次 ```ssh-keygen -t rsa``` ，成功会生成一张由ascii组成的随机图片。 

   操作完进行测试 ```ssh -Tv git@github.com``` 

6. ```
   remote: error: Trace: ee978cd0172947d342c66ef1196903c98e74787df2e1ee55efcd7a308514eabe
   remote: error: See https://gh.io/lfs for more information.
   remote: error: File 就业指导/resources/南林TTSC7-8.pptx is 151.40 MB; this exceeds GitHub's file size limit of 100.00 MB
   remote: error: GH001: Large files detected. You may want to try Git Large File Storage - https://git-lfs.github.com.
   To https://github.com/aaaaaQIE/term6.git
    ! [remote rejected] main -> main (pre-receive hook declined)
   error: failed to push some refs to 'https://github.com/aaaaaQIE/term6.git'
   ```

   Github只允许上传最大100MB的文件，如果超过，则会被拒绝上传。

   解决这个问题的方式：运行如下命令删除有关某个文件的push

   `git filter-branch --force --index-filter "git rm --cached --ignore-unmatch <file>"  --prune-empty --tag-name-filter cat -- --all`

   修改 log 信息后保存返回

   `git commit --amend` （也许不需要）

   重新提交

   `git push` 或 `git push --force`

7. ```
   Changes not staged for commit:
   ```

   提交时加上参数： `-a` ，表示新增。

   ```
   git commit -am "提交说明"
   ```

## 实际案例

#### 一键push脚本

```
vim push.sh
	------------------------------------------
	#! /bin/bash
	$(git add .;git commit -m push;git push)
	------------------------------------------
chmod +x push.sh
```

push 脚本一定要放在 git 工作目录下

#### 克隆下载Github某仓库中的部分文件

```
git init <repo> && cd <repo>

git remote add -f origin git@IP:XXX.git	#从Code-Clone-SSH里面复制

git config core.sparsecheckout true	#设置允许克隆子目录

echo /<repo> >> .git/info/sparse-checkout	#将需要下载的文件路径加入到配置文件，需要添加多个则多写一条记录，比如：拉取<传热学>文件夹中的所有文件 $ echo /传热学 >> .git/info/sparse-checkout

git checkout master
git pull origin master	#获取文件，之后就可以正常操作其他命令了
```

## 参考文献

[【狂神说Java】Git最新教程通俗易懂](https://www.bilibili.com/video/BV1FE411P7B3) 

[Github快速入门——详细教程 ](https://zhuanlan.zhihu.com/p/437280775) 

[Typora+GitHub打造专属云笔记](https://blog.csdn.net/weixin_44924882/article/details/108936635) 

[Message "Support for password authentication was removed. Please use a personal access token instead."](https://stackoverflow.com/questions/68775869/message-support-for-password-authentication-was-removed-please-use-a-personal) 