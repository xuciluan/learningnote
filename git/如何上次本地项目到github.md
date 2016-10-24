# 如何上次本地项目到github

1. 第一步要先到github上创建帐号和仓库，不详细说

    ![github](C:\Users\Public\Pictures\Sample Pictures\github.PNG)

   ​

2.配置好上一步骤后，到本地文件下使用git的初始化命令

```
git init
```

3.为了把本地的仓库传到github，需要配置ssh key,配置命令如下：

```
$ssh-keygen -t rsa -C "xuciluan@126.com"
```

然后一路回车即可，之后会看到这个界面：

![ssh](C:\Users\Public\Pictures\Sample Pictures\ssh.PNG)

然后进入提示的ssh存放位置，在当前电脑下是在：

```
C:\Users\xcl\.ssh
```

打开`id_rsa.pub`复制里面的key。如下：（**记住不是id_rsa，要看清楚**）

 ![rsa](C:\Users\Public\Pictures\Sample Pictures\rsa.PNG)

最后到github的setting中进行添加ssh即可：

![github-ssh](C:\Users\Public\Pictures\Sample Pictures\github-ssh.PNG)









