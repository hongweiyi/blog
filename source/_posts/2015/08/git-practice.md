title: GIT 最佳实践
date: 2015-08-28 22:32:00
categories: 最佳实践
tags: GIT
---

以下是我整理的自己使用及见过比较好的 GIT 实践，草草总结如下，有空会详细描述各项内容：
<!--more-->

## Labels

* BUG
* P0
* P1
* P2
* TIPS
* TODO
* 功能Label
	* 交互
	* 视觉
	* ...
* 子功能Label
    * 业务1
    * 业务2
	* ...


## Branch

* 方式1: [Git版本控制与工作流](http://www.jianshu.com/p/67afe711c731)
	* master
	* release
	* develop
	* feature/*
	* hotfix
* 方式2
	* master
	* name-dev

## Issue

* 命名
	* XX1-XX2-Description
	* XX1: 功能
	* XX2: 子功能
* Push
	* comment 中添加 #{Issue Id}

## Push

* 最小功能提交原则


## Milestone

### 常规

* 命名
	* v1.0.0
	* v1.0.1
	* ...
* 内容
	* 基本描述
	* 版本负责人

### 其它

* Application Talk
	* 针对 Application 的讨论与展望
	* 同时也可以发设计文档

## 其它实践

* 可以建立 project-management 分支
	* 更多的关注项目管理的内容
		* 文档
		* 项目管理 issue
	* 如果有些 issue 同时涉及多个不同子工程，可以将这些写在这里面
