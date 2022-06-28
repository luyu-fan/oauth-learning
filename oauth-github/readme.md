# 利用GitHub API开发OAuth应用

GitHub可以提供REST API以满足用户定制化的需求，而OAuth则是一种以用户为中心的授权开发方式。

在Go中可以借助OAuth-Client这个库完成OAuth应用的开发，本质上这个客户端就是一个Http请求的代理和封装（按照RFC实现）。

在GitHub的OAuth应用开发过程中，GitHub主要提供了两种授权方式，一种是标准的授权码形式，还有一种是设备流的方式。实际上关于授权部分涉及到的Http流部分相对较少，主要还是要掌握GitHub内部大量的REST API。
