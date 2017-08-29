# 源代码管理方案

【发布者】

| 版本   | 日期         | 编辑                    | 说明   | 审定      |
| ---- | ---------- | --------------------- | ---- | ------- |
| 0.1  | 2017-08-29 | 李斌(libin@elastos.org) | 初步作成 | 【审定人列表】 |

# 1. 目的

用于规范Elastos开源软件开发中的源代码管理。

# 2. 角色及权限

## 2.1 外围开发人员

-   定义：互联网上招募的软件开发人员，具备一定软件开发经验
-   权限
    -   可以Fork代码仓库到自己账户
    -   可以提交代码变更到自己账户
    -   可以向基金会提交自己的代码变更



## 2.2 核心开发人员

-   定义：对本模块整体有明确认识
-   权限
    -   具备外围开发人员的所有权限
    -   可以审核(Review)外围开发人员提交的工作
    -   可以直接向代码仓库提交代码


## 2.3 仓库负责人

-   定义：通常是本仓库的开发负责人，负责合并其它开发人员的工作成果
-   权限
    -   具备核心开发人员的权限
    -   可以审核(Review)外围开发人员提交的工作
    -   可以直接在代码仓库建立／删除分支
    -   可以直接在仓库进行分支合并操作


## 2.4 仓库管理员

定义：SCM维护人员

权限
-   具备仓库负责人的权限
-   可以新建代码仓库

## 2.5 白盒测试人员


# 3. 工具

| **工具用途**        | **工具名称**            | **工具获取** | **工具用户** |
| --------------- | ------------------- | -------- | -------- |
| 代码仓库服务          | GitHub Organization | Web      | 工程人员     |
| Git客户端(Linux)   | Git命令行              | Ubuntu自带 | 工程人员     |
| Git客户端(Mac)     | SourceTree          | 免费给开源项目  | 工程人员     |
| Git客户端(Windows) | TortoiseGit         | 免费       | 工程人员     |

# 4. 操作

## 4.1 注册账户(Register)

相关角色：所有开发人员。

核心开发者用elastos.org邮件注册Github账户。

### 建立个人秘钥

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
$ ssh-keygen
$ cat ~/.ssh/id_rsa.pub
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

### 添加个人公钥

访问<https://github.com/settings/keys>，点击New SSH key。

### 配置Git客户端

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
$ git config --global user.name "name"
$ git config --global user.email "email"
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

## 4.2 拷贝自有仓库(Fork)

相关角色：所有开发人员。

以Elastos.RT为例。访问https://github.com/elastos/Elastos.RT，点击页面上Fork，则拷贝一份仓库到自己账户下。

后续就可以工作在自有仓库上了 git@github.com:elastos/Elastos.RT。

参考：<https://help.github.com/articles/syncing-a-fork/>

## 4.3 代码下载(Clone)

相关角色：所有开发人员。

开发人员用任一选定的Git客户端，下载自有仓库到本地操作系统。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
$ git clone git@github.com:elastos/Elastos.RT 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

## 4.4 代码编译、代码调试、本地自测（Build, Debug, BVT)

相关角色：所有开发人员。

各子项目有不同要求。

## 4.5 代码提交(Commit, Push, Pull Request)

相关角色：所有开发人员。

开发人员要用三个步骤将工作成果提交到Elastos项目。

1.  首先将代码变更提交到本地(Commit)
2.  然后提交到个人仓库(Push)
3.  最后提交到Elastos仓库接受代码审核(Review)

参考：<https://help.github.com/articles/creating-a-pull-request-from-a-fork/> 

## 4.6 代码审核(Review) 

相关角色：核心开发人员、仓库负责人。

参考：<https://help.github.com/articles/reviewing-changes-in-pull-requests/> 

## 4.7 分支管理(Branching)

相关角色：仓库负责人。

各子项目有不同要求。

## 4.8 仓库数据备份(Backup) 

相关角色：仓库管理员

备份方式：脚本自动增量异地备份，备份频率1小时1次。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
mirror_github()
{
    if [ "$1" == "" ]; then
        echo Usage: $0 PROJ_NAME
        echo Mirror github.com/PROJ_NAME repositories
        return
    fi

    PROJ_NAME=$1

    IS_AVAILABLE=$(curl -s -I "https://api.github.com/users/$PROJ_NAME" | grep 'HTTP/1.1 200 OK')

    if [ "$IS_AVAILABLE" == "" ]; then
        echo ERROR: unable to reach github.com/$PROJ_NAME
        return
    fi

    GIT_REPOS=$(curl -s "https://api.github.com/users/$PROJ_NAME/repos?page=1&per_page=200" | grep clone_url | sed -e "s/.*https:\/\/github.com\/$PROJ_NAME\///" -e "s/\",$//")

    HOME=/data/git/git_config/proxy

    mkdir -p /data/git/github.com/$PROJ_NAME
    cd /data/git/github.com/$PROJ_NAME

    for i in $GIT_REPOS; do
        if [ -d $i ]; then
            echo Updating $i...
            1>/dev/null pushd $i
            git remote update
            test_return
            git update-server-info
            test_return
            echo >description
            1>/dev/null popd
        else
            PARENT_DIR=`dirname $i`
            mkdir -p $PARENT_DIR
            1>/dev/null pushd $PARENT_DIR
            git clone --mirror https://github.com/$PROJ_NAME/$i
            test_return
            1>/dev/null popd
        fi
    done
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

## 4.9 仓库数据恢复(Restore)

相关角色：仓库管理员

恢复方式：脚本自动恢复

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
#!/bin/bash

BASE=/data/git/github.com/elastos

GIT_REPOS="LIST GIT REPOSITORIES HERE"

for i in $GIT_REPOS; do
    pushd $BASE/$i 1>/dev/null
    echo Pushing $BASE/$i to git@github.com:elastos/$i...
    git push --all  git@github.com:elastos/$i
    git push --tags git@github.com:elastos/$i
    popd 1>/dev/null
done
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

# 5. 成本

| **项目** | **说明**                                   | **成本**      |
| ------ | ---------------------------------------- | ----------- |
| 互联网管制  | 中国国内存在互联网管制，平时GitHub也存在被GFW封锁的可能性。每个开发人员需要配备VPN工具，推荐vpnso，流量配额100GB/月。 | 108 RMB/人年。 |
