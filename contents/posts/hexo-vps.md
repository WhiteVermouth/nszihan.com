---
title: "在VPS上使用Git部署Hexo博客"
date: "2018-09-14 17:14:47"
---

## 搭建步骤

1. 在本地完成 hexo 博客的初始化
2. 在服务端完成 git 仓库的初始化
3. 使用 git 一键部署

## 在本地搭建 hexo

1. 安装 hexo-cli

```
yarn gload add hexo-cli
```

2. 初始化 hexo 博客

```
hexo init blog
```

3. 安装 git 部署插件

```
yarn add hexo-deployer-git
```

至此 hexo 的本地初始化已完成，使用`hexo server`可在本地启动一个服务，用于测试预览博客。

## 在服务端建立 git 仓库

1. 安装 git

```
sudo apt-get install git
```

2. 建立 git 裸库

```
sudo mkdir /var/repo
cd /var/repo
sudo git init --bare blog.git
```

3. 添加 git hook 用于同步网站根目录

```
cd /var/repo/blog.git/hooks
sudo vim post-receive
```

输入以下内容

```shell
#!/bin/sh
git --work-tree=/var/www/hexo --git-dir=/var/repo/blog.git checkout -f
```

注意修改 work-tree 的路径

添加权限

```
sudo chmod +x post-receive
```

4. 创建 git 账户

此处我使用了`Ansible`来快速自动化完成创建

关键代码如下

```yaml
- name: add git user
  become: yes
  user:
    name: git
    shell: /usr/bin/git-shell
    group: git
- name: add ssh key
  become: yes
  authorized_key:
    user: git
    key: "{{ item }}"
  with_file:
    - ~/.ssh/id_rsa.pub
```

出于安全考虑，建议将 git 账户的`shell`设置为`git-shell`

5. 修改 git 库的权限

```
sudo chown -R git:git blog.git
```

## 修改本地\_config.yml

在\_config.yml 找到 deploy，加入以下代码

```yaml
deploy:
  type: git
  repo: git@serverip:/var/repo/blog.git
  branch: master
```

## 部署

在本地执行

```
hexo g
hexo d
```

至此，已完成博客的部署
