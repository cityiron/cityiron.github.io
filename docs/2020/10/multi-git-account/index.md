# Mac 上配置多个 GitHub 的 ssh 账号

本文介绍如何在一台电脑上，配置多个 Git 的连接信息，同时连接 GitHub 和公司的 GitLab。

<!--more-->

## 一、前言
日常工作时我本机需要连接公司的 Gitlab，还需要连接两个 GitHub 账号，本地使用的时候免不了需要不同的 SSH 认证登录。

## 二、如何配置

- 添加 ssh-agent

```bash
$ eval "$(ssh-agent -s)"
Agent pid 65799
```

- 查看初始化的认证信息

```bash
$ ssh-add -L
The agent has no identities.
```

- 把 rsa 私钥添加到 ssh

```
$ ssh-add ~/.ssh/id_rsa2
Identity added: /Users/tc/.ssh/id_rsa2 (/Users/tc/.ssh/id_rsa2)
```

- 查看添加后的认证信息

```
$ ssh-add -L
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCmz+FyF/uuIYAedTbzQoAE+wnHG3/uzqQ2Mv9QQbjcWOOjOlTmL48rYTH6hA1rJQl9hFsTrYzTbKYlV1h7oNqT/7nUsmNOhItg51DcvSbA/PSw+2jADkbMV6bnKUtJmx7gxu7RKuWi48smEewYny/zMIJSPRu5Tr1G6WX/zEtonJntgEJELfgrJntgBikTZ/QJOGtejwWJjEPyKyNji0O4o0tmK5Xql+MG0O109Xa5toRBADFWqz+Popk7MNFuooRp/Fz0b4tLTvvlfwrEGDu0z4qmlShTmiZNQhYQsmiwpt27/IUo2Cw8J62EEUqviykuNeZBzYCGMXd3Mcg/CpTPbtBb /Users/tc/.ssh/id_rsa2
```

- 查看 github.com 交互的账号信息

```bash
$ ssh -T git@github.com

Hi cityiron! You've successfully authenticated, but GitHub does not provide shell access.
```

当出现 `Hi [yourname]`， name 和你要操作的仓库所在的 github 账号一样，那么就可以正常使用 Git 命令。

> 每一个 terminal 可以是独立的 session，当电脑启动的时候，默认所有的 terminal 共享一个账号信息

## 三、参考资源

https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent

