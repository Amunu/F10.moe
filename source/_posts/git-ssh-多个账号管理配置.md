title: git ssh 多个账号管理配置
date: 2014-10-16 11:08:05
tags: [git,ssh]
---

同时使用了github和gitlab托管项目时, 因为某些原因不想用同一个ssh key, 就需要ssh多个账号配置了.

## ssh-keygen

用ssh-keygen命令生成一组新的id_rsa_new和id_rsa_new.pub。
    
    ssh-keygen -t rsa -C "new email"

可以用 ``ls -al ~/.ssh`` 查看当前的 rsa。 

所以， 当前我有两组rsa ：id_rsa， id_rsa_new，  id_rsa_new.pub，  id_rsa.pub。

## ~/.ssh/config
  
  首先，找到本地 .ssh 的 config 文件，若没有，则新建一个 （vi config）。

  修改配置：
   ssh://gitlab@ **Host** :XXX/ **User** /XXX.git
  (这里配置的Host 和 User 关系到之后的clone等操作)
  
```
Host gitlab.widget-inc.com      
	HostName gitlab.widget-inc.com
	User Minary
	IdentityFile ~/.ssh/id_rsa_new

Host github.com
	HostName github.com
	User Amunu
	IdentityFile ~/.ssh/id_rsa
	
```
  
  若为多个 *github* 账号 上面配置可以改成：

```
Host Minary.github.com
	HostName github.com
	User Minary
	IdentityFile ~/.ssh/id_rsa_new

Host Amunu.github.com
	HostName github.com
	User Amunu
	IdentityFile ~/.ssh/id_rsa
```
  
  以 "git@github.com:Amunu/amunu.github.io.git" 为例，改成上面那样后，以后 ssh 操作，比方说 *clone* 时，若需要 *Amunu* 这个账号操作："git@Amunu.github.com:Amunu/amunu.github.io.git" ；
  若需要  *Minary* 操作："git@Minary.github.com:Amunu/amunu.github.io.git" 。 
  
## git config
  同时，你可以通过在特定的项目下执行下面的命令，生成区别于全局设置的user.name和user.email。
  
```
git config user.name "newname"
git config user.email "newemail"
 
#git config --global --unset user.name 取消全局设置
#git config --global --unset user.email 取消全局设置
```

想看自己全局的设置可以执行： ``cat ~/.gitconfig ``
