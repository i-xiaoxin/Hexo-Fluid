---
title: Maven依赖爆红Dependency not found 问题
date: 2022-10-15 23:55:02
tags: 
- Maven
index_img: 
banner_img: /img/article8.webp
categories:
- Maven
---

## 问题

Maven依赖倒入后pom文件IDEA提示错误：dependency not found

## 原因&解决

### 1. IDEA中Maven配置

原因：项目中Maven的路径&xml文件&仓库路径配置变更；由于copy其他项目，默认路径未修正；由于文件配置存在多个导致配置异常等

解决： 重新配置

### 2. 本地仓库依赖不错在持续爆红

原因：

- pom坐标错误
- 网络环境异常

解决

- 搜索当前依赖pom坐标 [查询链接](https://mvnrepository.com/)
- 项目依赖提示多个坐标错误，通过以上配置刷新后某个依赖仍旧爆红；IDEA提示`dependency not found`刷新多次无效
  - 尝试直接从Maven远程仓库下载手工部署到本地仓库，注意路径，不推荐
  - 新建空白Maven项目，导入坐标刷新下载，爆红问题解决





😷遇到问题沉着冷静，认真阅读，方可豁然！！