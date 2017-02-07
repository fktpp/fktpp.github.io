---
layout: post
title:  "ssh-dss key not in PubkeyAcceptedKeyTypes"
date:   Sat Feb  4 20:47:12 DST 2017
categories: ssh dss
---
# DSS 不能用啦！ ssh-dss key not in PubkeyAcceptedKeyTypes

忙里偷闲鼓捣自建基于[Jekyll][Jekyll]的blog，发现无论如何都不能在心爱的linux里面通过ssh clone新创建的 git repository。

提示如下：

```
Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

遂坠入无限ssh检查中无法自拔：

1. ```ssh-add -l``` 显示 ssh-keyagent 内容无碍
2. ```ssh-keygen -l -f``` 显示各个key的finger print 看不出错误
3. 进入windows通过 git bash clone 失败，遂发现 windows 中 plink 太久未用，已经生锈。不认识 github 的 host key。用 putty github.com 弹出的新host key 对话框确认了事。
4. windows下终于clone成功。
5. 再次尝试将 windows 下的ppk导出为openssh格式，遂发现几年前曾经有一份导出的 key 躺在那里……
6. 各种折腾之后拷贝的key诺到了linux中，以为万事大吉，结果，结果，结果虽然 ```diff -u``` 显示新来的 key 是新的，但是```ssh-keygen -l -f``` 说他们两个是一样的……
7. 无奈出下下策 ```ssh -vvvv git@github.com```，一阵垃圾滚屏过后终于发现

```
debug1: Skipping ssh-dss key /home/fktpp/.ssh/fktpp.openssh - not in PubkeyAcceptedKeyTypes
```

## Google 说 (keyword: Skipping ssh-dss key not in PubkeyAcceptedKeyTypes)

[stackexchange: ssh keeps skipping my pubkey and asking for a password][stackexchangeSshSkipPubKey]

> The new openssh version (7.0+) deprecated DSA keys and is not using
> DSA keys by default (not on server or client). The keys are not
> preferred to be used anymore, so if you can, I would recommend to use
> RSA keys where possible.
>
> If you really need to use DSA keys, you need to explicitly allow them
> in your client config using
>
> PubkeyAcceptedKeyTypes +ssh-dss
>
> Should be enough to put that line in ~/.ssh/config, as the verbose
> message is trying to tell you.


[perezdecastro.org: Locked out of SSH? Renew all the keys!][perezdecastroSshLockOut]

> Looking at OpenSSH 7.0 release notes, there is the culpript:
>
>  * Support for ssh-dss, ssh-dss-cert-* host and user keys is disabled
>    by default at run-time. These may be re-enabled using the
>    instructions at http://www.openssh.com/legacy.html
>
> So it turns out that support for ssh-dss keys like the one I was
> trying to use is still available, but disabled by default.

## Wrapping Up

大势所趋，换key啦～～

# 新问题 agent refused operation

命苦的娃总会碰到各种挫折！！

为了换key我也是拼了，Gnome 环境下执行了 ```ssh-add -D``` 然后就被恶狠
狠的一句话挡在了门外

```
[fktpp@fedora workspace]$ git clone git@github.com:fktpp/fktpp.github.io.git
正克隆到 'fktpp.github.io'...
sign_and_send_pubkey: signing failed: agent refused operation
sign_and_send_pubkey: signing failed: agent refused operation
Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

## Google 说 (keyword: sign_and_send_pubkey: signing failed: agent refused operation)

我看出Google说的都不靠谱！！

重启 Gnome by logout and login 试试... 不灵

## Google 继续说 (keyword)

[Chris Jean: Ubuntu ssh fix fo agent admitted fail to sign using the key][ChrisJeanUbuntuFix]

> After some digging, I found out that issues with the gnome-keyring were at fault. gnome-keyring doesn’t always handle specific formats of SSH keys correctly. Unfortunately, gnome-keyring was trying to handle all SSH key usage, preventing the keys from working.

## Wrapping Up

为了给私钥加个comments我听从 ```ssh-keygen``` 的建议使用了 ```-o``` 参数

Linux 用户是需要多么顽qiang的生命力啊

[Jekyll]: http://jekyllrb.com
[stackexchangeSshSkipPubKey]: http://unix.stackexchange.com/questions/247612/ssh-keeps-skipping-my-pubkey-and-asking-for-a-passworde
[perezdecastroSshLockOut]: https://perezdecastro.org/2015/old-ssh-keys.html
[ChrisJeanUbuntuFix]: https://chrisjean.com/ubuntu-ssh-fix-for-agent-admitted-failure-to-sign-using-the-key/
