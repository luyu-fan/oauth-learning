# 利用GitHub API开发OAuth应用

GitHub可以提供REST API以满足用户定制化的需求，而OAuth则是一种以用户为中心的授权开发方式。

在Go中可以借助OAuth-Client这个库完成OAuth应用的开发，本质上这个客户端就是一个Http请求的代理和封装（按照RFC实现）。

在GitHub的OAuth应用开发过程中，GitHub主要提供了两种授权方式，一种是标准的授权码形式，还有一种是设备流的方式。实际上关于授权部分涉及到的Http流部分相对较少，主要还是要掌握GitHub内部大量的REST API。

## 授权处理部分

### 授权 [使用GET]

接入GitHub进行授权的URL为`https://github.com/login/oauth/authorize`, 需要包括以下主要参数：

|      名称      |   类型   | GitHub要求必填 |                                  描述                                  |
|:------------:|:------:|:----------:|:--------------------------------------------------------------------:|
|  client_id   | string |     必      |                       在GitHub中创建OAuth应用时给分配的ID                       |
| redirect_uri | string |     可      | 在被GitHub账户授权使用之后的OAuth应用回调URI，一般在刚开始的时候就已经注册了，也可以在刚开始的时候仅仅注册一个回调的模式。 |
|    login     | string |     可      |                             用这个参数指定某一个用户                             |
|    scope     | string |     可      |                           表明需要授权的[范围](#1)                            |
|    state     | string |     可      |                         一段不可猜测的随机字符串，防止CSRF。                         |
| allow_signup | string |     可      |                    表示是否允许在没有注册的情况下去注册一个新的GitHub账号                    |

这个重定向请求是GitHub提供的由user-agent完成的交互动作。

用户在完成这个授权动作之后，会由GitHub重定向返回至回调URI，其中包含了授权码code。

### Access Token [使用POST]

按照OAuth的授权码类型授权流，使用重定向URL中的授权码code获取访问Token。

|      名称       |   类型   | 必须  |                    描述                    |
|:-------------:|:------:|:---:|:----------------------------------------:|
|   client_id   | string |  必  |                    同上                    |
| client_secret | string |  必  |             在创建OAuth应用时提供的密钥             |
|     code      | string |  必  |                   授权码                    |
| redirect_uri  | string |  可  | 在获得访问token之后被发送到的应用程序中的URL，即访问token的回调地址 |

在重定向的请求中指定`Accept`时可以将返回数据指明为`aplication/x-www-form-urlencoded`、`application/xml`或者是`apploication/json`。

### 使用Token访问API

在获取到访问token之后，就可以根据约定的token类型以及token内容从GitHub的API中访问数据。

### 设备验证

GitHub也提供了设备验证流，需要用户根据设备码进行授权，一般用于不基于Web的形式，此处省略。

### User Guidance

GitHub也提供了用于指引用户授权信息的授权说明页，满足RFC要求。

## REST API部分

REST API是目前GitHub上提供的比较标准的第三方应用开发API，详情参[参考](https://docs.github.com/cn/rest),
此处做一些简单的归类和整理。

### 基础

#### 版本

目前GitHub中使用的是v3版本，推荐在Accept字段中显式指明选择使用该版本的API和数据格式，即
设置为：`application/vnd.github.v3+json`。

GitHub中所有的API都是通过HTTPS完成的，他的根API路径是`https://api.githubc.com`, 所有的数据无论
是发送还是接收（授权过程除外）都采用`json`格式。

#### 验证

GitHub支持两种请求API时的授权方式，一种是简单的`Basic`授权，这个主要使用的是PAT，另外一种则是
OAuth Token。后者也是比较推荐的一种方式。

#### 通用

HTTP状态、动词符合REST标准。

所有的请求必须要在头部指明`user-agent` 字段，可以是APP名称，也可以是GitHub的用户名。这个字段为
空将会被拒绝。

#### 限速

GitHub将所有的类型（PAT，GitHub APP，OAuth）统一视作user-server类型的请求，
遵循如下限流策略：

1. 每个授权通过的用户每小时最多可以有**5000次**请求。
2. 当这个应用属于被GitHub企业云授权的应用时，最高可以达到**15000**每小时。
3. 没有授权的请求限制在每小时**60**。

可以通过限流API`https://api.github.com/rate_limit` 查询自己的限流情况。

为了尽可能地降低流量，应该尽可能地使用条件请求，即对某些资源，根据其Etag值来判断
是否可以得到304响应。主要通过指定`If-None-Match` 以及`If-Modified-Since` 来
决定是否需要重新计算资源，对于返回304的请求并不会消耗流量。

#### 跨源共享策略

API支持跨源资源共享问题CORS来满足简单的安全性问题。

支持JSON-P回调。

#### 时区

#### 媒体类型

API能够支持的最基本媒体类型：

```js
application/json
application/vnd.github+json
```

可以在响应标头文件中检查此次返回的媒体类型：`X-GitHub-Media-Type`。获得的二进制块数据
实际上是用json的base64编码字符串表示的。

## Appendix

### <span id="1">scope</span>

作用域表明了可以授权OAuth应用访问那些内容，主要包括可读写权限等。<通过作用域指定所需的访问类型，作用域限制OAuth令牌的访问权限，不会授予超出用户权限范围
的任何额外权限>

GitHub中划分的可用作用域主要有以下几个部分（[详细](https://docs.github.com/cn/developers/apps/building-oauth-apps/scopes-for-oauth-apps)）：

|  名称  |                     描述                     |
|:----:|:------------------------------------------:|
|      | 在请求时不提供scope字段表明只授权那些公共信息的只读权限，公共仓库信息和gist |
| repo |             表示对用户仓库等所有资产的读写权限。             |
| user |                对用户身份的读写全写。                 |

scope字段指明了在请求中应该限制访问的内容，在请求中可以通过%20空格的形式表示多个作用域，应该
时刻维持最小作用域策略。请求的作用域和令牌的作用域应该相协调，用户可以修改令牌的作用域。
注意作用域之间的包含关系。

GitHub提供的API囊括了管理、代码生产、版本控制以及协作的方方面面，但这个API都分散在数十个相对独立的
领域中，需要根据实际需求来选择合适的API构建应用功能。

各个独立功能API[参考](https://docs.github.com/cn/rest)。

