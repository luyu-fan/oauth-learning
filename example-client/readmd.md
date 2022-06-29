# OAuth应用

GitHub的OAuth应用（OAuth客户端）示例：WebulusTest。

## 描述

搭建一个简单的客户端（同时也是WebulusTest应用的前端所对应的数据后端），完成以下基本功能即可:

1. OAuth授权, 获取到访问token。

2. 根据token获取到仓库列表。

3. 对仓库内的某一个分支进行提交（完成简要的Git功能）。

4. 选择使用比较合适的OAuth客户端或者是GitHub-REST客户端。

## Dev

1. 语言：Go(go1.18)
2. 框架或库：Gin/GoAuthClient