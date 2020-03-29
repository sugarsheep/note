# git简单使用

1. 创建一个作为本地库的目录，然后使用git bash进入该目录执行git init初始化本地库

2. 设置签名（备注：与github账号和密码不是同一个概念）

   - 系统用户级别

   ```shell
   git config --global user.name tom
   git config --global user.email tom@qq.com
   信息保存位置：~/.gitconfig 文件
   ```

   - 项目级别

   ```shell
   git config user.name tom
   git config user.email tom@qq.com
   信息保存位置：./.git/config 文件
   ```

3. 设置远程库别名

   ```shell
   git remote -v 查看当前所有远程地址别名
   git remote add [别名] [远程地址]
   信息保存位置：./.git/config 文件
   ```

4. 下载远程库

   ```shell
   git clone https://github.com/sugarsheep/note.git
   ```

5. 推送远程库

   ```shell
   git add .        （注：别忘记后面的.，此操作是把Test文件夹下面的文件都添加进来）
   
   git commit  -m  "提交信息"  （注：“提交信息”里面换成你需要，如“first commit”）
   
   git push -u origin master   （注：此操作目的是把本地仓库push到github上面，此步骤需要你输入帐号和密码）
   ```

6. 拉取远程库更新

   ```shell
   git pull [别名] master
   ```

7. 解决每次push都要输入用户名和密码的问题

   > - 先执行**git config --global credential.helper store**，执行后下一次输入用户名和密码后将会保存起来，以后就不用再次输入了

8. 