### Token 防盗链

Token 防盗链适合下载类网站或应用使用，可设置签名过期时间来控制文件的访问时限。

> Token 防盗链原理图

![](https://upyun-assets.b0.upaiyun.com/docs/cdn/upyun-cdn-token.png)

> 签名方式说明

签名：

```
_upt = MD5(token 密匙&etime&URI){中间 8 位} + etime
```

参数：

* token 密匙：用户所填的密钥
* etime：过期时间，必须是 UNIX TIME 格式，如： `1378092990`
* URI：请求地址（不包含?及后面的 Query String），如： `/dir/pic.jpg`

备注：

* 对于使用 Cookie 类无法给每个链接进行单独签名的情况，可使用 / 作为 URI 进行统一的泛签名，如：

```
MD5(token 密匙&过期时间&/){中间 8 位} + 过期时间
```

* 注意：泛签名在过期时间内，对服务域名内的所有链接资源都有效

#### 目录级别 Token 防盗链：_upp 参数

```
+----------------------+------------------------------------------------------------+
|                      | Ex. uri = '/a/b/c.jpg', ex_time = now() + 600              |
|    token secret      |                                                            |
|                      | _upt = md5(token_secret&ex_time&uri)[12:20] + ex_time      |
+----------------------+------------------------------------------------------------+
```

`_upp` 参数是用来配合 Token 防盗链的 `_upt` 参数使用的，以便支持用户可自定义目录级别进行签名，默认情况下 `_upt` 均以完整的 URI 进行签名，当 `_upp = 2` 那么就以相应的 `uri = '/a/b/'` 来进行签名。这样就很容易实现对某个目录的防盗链权限控制了。

> 示例

```
http://upyun-assets.b0.upaiyun.com/docs/cdn/upyun-cdn-architecture.png?_upp=2&_upt=abcdefgh1370000600
```

例如对于以上这条结合 `_upp` 参数的 Token 防盗链请求来说，对应 `_upt` 参数正确的签名应按如下公式进行计算（其中被签名 URI 此时为 `/docs/cdn/`）：

```
MD5(token 密匙&过期时间&/docs/cdn/){中间 8 位} + 过期时间
```

> 详细说明

1. 其值应为正整数，设置范围 `0 <= x <= 20`，超过该范围或其它不合法的字符均视为无效值，此时仍然按照 URI 全路径进行匹配.
2. 当 `_upp = 0` 时，总是匹配 `/`；其它 `_upp = 1,2,3 ...` 分别对应一级目录，二级目录，三级目录 ...

例如，对于 `uri = '/a/b/c/foo.jpg'` 来说：

```
[v] /a/b/c/foo.jpg => '/a/b/c/foo.jpg'

[v] /a/b/c/foo.jpg?_upp=0 => '/'
[v] /a/b/c/foo.jpg?_upp=1 => '/a/'
[v] /a/b/c/foo.jpg?_upp=2 => '/a/b/'
[v] /a/b/c/foo.jpg?_upp=3 => '/a/b/c/'

[x] /a/b/c/foo.jpg?_upp=4 => '/a/b/c/foo.jpg'
[x] /a/b/c/foo.jpg?_upp=-1 => '/a/b/c/foo.jpg'
[x] /a/b/c/foo.jpg?_upp=abc => '/a/b/c/foo.jpg'
```
