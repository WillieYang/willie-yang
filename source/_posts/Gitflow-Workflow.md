---
title: 如何理解GitFlow工作流，以及其在产品研发过程中的使用方式
date: 2019-06-27 21:11:27
tags: 
- Git
- Version Control
---

## 前言

随着软件工程和项目管理的逐渐复杂化，各种各样的工具也都不断的涌现，其目的，是为了减轻我们软件工程师的工作量，让我们把更多的精力放到代码的编写上面。其中，Git，作为一个代码的版本控制工具，随着开源的大流，现今已经成为了工程师们在编写代码的过程中，最为青睐，也是最不可分割的一部分。

今天这篇文章会忽略一些简单的Git常识，而会着重阐述一下GitFlow的具体思路，以及在多人协作开发产品过程中，应该如何使用GitFlow来完成整个项目的开发。

## 什么是GitFlow?

GitFlow是Git的一个严格的分支模型设计，它相当于一种工作流模式，被Vincent Driessen at nvie首次提出，并从此流行。主要适用于管理大型的软件工程项目。`git-flow`是一个Git基础上的扩展与封装，它可以用来简化GitFlow的使用过程，单单使用原生的Git命令也可以进行GitFlow的工作流管理，不过略嫌麻烦。举例子来说，`git flow init`和`git init`这两个命令都是新建一个git仓库，不过前者会在后者的基础上，创建更多必要的分支。

## 主分支以及开发分支

GitFlow会使用两个分支来保存工程的历史，分别是主分支(Master Branch)以及开发分支(Develop Branch)，主分支主要是用来保存官方的发布历史，以及使用版本号来给主分支里面的提交(commit)打标签(tag)。开发分支则主要是被用来作为特性(features)的集成分支。(*当使用`git flow init`这个命令的时候，会自动生成master和develop两个分支，如下图*)

```bash
$ git flow init
Initialized empty Git repository in ~/project/.git/
No branches exist yet. Base branches must be created now.
Branch name for production releases: [master]
Branch name for "next release" development: [develop]

How to name your supporting branch prefixes?
Feature branches? [feature/]
Release branches? [release/]
Hotfix branches? [hotfix/]
Support branches? [support/]
Version tag prefix? []

$ git branch
* develop
  master
```

## 特性分支

对于一个正在开发的产品来说，每一个新的特性，都存在它自己的分支。这些分支，将会以开发分支，而不是主分支作为父分支。每当一个产品的特性被完成，这个特性分支(Feature Branch)就会被合并到开发分支里面，而所有的特性分支，都不能与主分支进行直接的交互。

以下是不使用`git-flow`创建特性分支的相关命令，需要切换到develop branch然后再创建一个相应feature的branch，之后再切换到该特性分支。

```shell
git checkout develop
git checkout -b feature_branch

git checkout develop
git merge feature_branch
```

以下是使用`git-flow`的相关命令，比较简洁

```shell
$ git flow feature start feature_branch

$ git flow feature finish feature_branch
```

## 发布分支

当开发分支已经积攒了足够多的feature的时候，我们可以在开发分支上面分叉出去一个release分支，用于版本发布。这个分支上将不会再添加新的feature，只会被用来修改bug。当bug被修复的差不多了，到了这个release分支可以被用于版本发布的时候，该分支将会被合并到master分支上面去，并打上版本标签。同时这个release分支也会被合并到develop分支上去。（*当release分支合并成功以后，该分支将会被删除*）

以下是不使用`git-flow`时的相关release分支操作命令

```shell
git checkout develop
git checkout -b release/0.1.0

git checkout master
git merge release/0.1.0
```

以下是使用`git-flow`时的相关release分支操作命令

```shell
$ git flow release start 0.1.0
Switched to a new branch 'release/0.1.0'

$ git flow release finish '0.1.0'
```

## 热修复分支

热修复分支(Hotfix Branch)可以被用来给生产发布的版本进行快速的补丁修复，它是基于主分支的，除此之外，和release, feature分支没有太大的区别，但是这是唯一一个直接从master分支里面分叉出去的分支。一旦bug被修复成功，这个分支将会被合并到master和develop分支里面去，然后master分支将会被打上版本更新的标签(tag)。

以下是不使用`git-flow`时的相关hotfix分支操作命令

```shell
git checkout master
git checkout -b hotfix_branch

git checkout master
git merge hotfix_branch
git checkout develop
git merge hotfix_branch
git branch -D hotfix_branch
```

以下是使用`git-flow`时的相关hotfix分支操作命令

```shell
$ git flow hotfix start hotfix_branch

$ git flow hotfix finish hotfix_branch
```
## 总结

总的来说，GitFlow适用于基于发布版本的软件开发工作流，它也为生产环境提供了bug热修复的流程。

总的工作流程可以分为以下几步：

1. 开发分支将会基于主分支被创建。
2. 发布分支将会基于开发分支被创建。
3. 特性分支将会基于开发分支被创建。
4. 当一个特性被完成以后，该分支将会被合并到开发分支。
5. 当一个发布分支完成以后(`bug修复完毕`)，该分支将会被合并到开发分支和主分支。
6. 当在主分支里面发现问题以后，一个热修复分支将会基于主分支被创建。
7. 一旦热更新分支被完成，它将会被合并到主分支和开发分支。

## 引用及参考

1. [Gitflow Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)
2. [Introducing GitFlow](https://datasift.github.io/gitflow/IntroducingGitFlow.html)




