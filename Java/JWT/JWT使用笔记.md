# JWT 使用笔记

* 一般认证方式有三种, Basic Authentication, Token 认证, OAuth2.0 , 其中最后一种认证方式安全等级最高, 但是也是最复杂的。

## JWT 介绍

* JWT（Java Web Token），为了在网络应用环境中传递声明而执行的一种基于JSON的开发标准（RFC 7519），该Token设计紧凑，比较适合于分布式站点的单点登录（SSO）场景。主要是用于认证。

* JWT 一般由三段信息构成，这三段信息用 “`.`” 进行链接就组成了JWT字符串，示例：

  ```java
  eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
  ```

* JWT构成，JWT由三部分构成：头部 -- header、payload -- 载荷、signature -- 签证

  * header -> JWT的头部承载两部分信息

    * 声明类型， 一般是 jwt
    * 声明加密算法， 通常使用的是 HMAC SHA256 单向散列算法

    ```json
    // 完整的头部类似这样
    {
        'typ': 'JWT',
        'alg': 'HS256'
    }
    ```

    然后将头部信息进行 base64 加密 （该加密是可以对称加密的），这样就构成了第一部分：

    ```java
    eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
    ```

  * payload  -> 载荷，即存放有效信息的地方，该信息一般包含三个部分：

    - 标准中注册的声明
    - 公共的声明
    - 私有的声明

    标准中注册的声明 (建议但不强制使用) ：

    - iss: jwt签发者
    - sub: jwt所面向的用户
    - aud: 接收jwt的一方
    - exp: jwt的过期时间，这个过期时间必须要大于签发时间
    - nbf: 定义在什么时间之前，该jwt都是不可用的.
    - iat: jwt的签发时间
    - jti: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击。

    定义一个payload：

    ```json
    {
        "sub": "1234567890";
        "name": "zhangsan";
        "admin": true
    }
    ```

    然后将其进行 base64 加密，于是得到 JWT 的第二部分：

    ```java
    eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9
    ```

  * signature -> 即签证信息， 签证信息由三部分组成：

    ```java
    header(base64后的)
    payload(base64后的)
    secret(即密码)
    ```

    这部分是由加密后的 header 和 payload 用 “`.`” 连接组成的字符串， 然后通过 header 中声明的加密方式进行加盐 secret 组合加密，然后就构成了 JWT 的第三部分。

    ```java
    String encodedString = base64UrlEncode(header) + "." + base64UrlEncode(payload);
    String signature = HMACSHA256(encodedString, secret); // TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
    ```

  这三部分用 “`.`”连起来就是一个完整的字符串，即 JWT

  **注意**：secret是保存在服务器端的，jwt的签发生成也是在服务器端的，secret就是用来进行jwt的签发和jwt的验证， 所以，它就是你服务端的私钥，在任何场景都不应该流露出去。一旦客户端得知这个secret, 那就意味着客户端是可以自我签发jwt了。

  应用：一般在请求头里加入Authorization， 并加上 Bearer 标注：

  ```js
  fetch('api/user/1', {
      headers: {
          'Authorization': 'bearer' + token
      }
  })
  ```

  ## 攻击面

  ### 敏感信息泄露

  JWT中的header 和 payload 虽然看起来不可读，但实际上都只经过简单编码， 开发者可能误将敏感信息存储在里面。使用上述工具可以方便地解码JWT中前两部分的信息。

  ### 指定算法为none

  上面提到算法 none 是JWT规范中强制要求实现的，但有些实现JWT的库直接将使用none 算法的token视为已经过校验。 这样攻击者就可以设置alg 为none ，使signature 部分为空，然后构造包含任意payload 的JWT来欺骗服务端。

  ### 将签名算法从非对称类型改为对称类型

  使用非对称加密算法（主要基于RSA、ECDSA，如S256）分发JWT的过程是使用私钥（private）加密生成JWT，使用公钥（public）解密验证。

  使用对称加密算法（主要基于HMAC，如HS256）分发JWT的过程是使用同一个密钥（secret）生成和验证JWT。

  如果服务端期待收到的算法类型为RS256，然后以RS256和public去验证JWT，而实际上收到的算法类型是HS256， 那么服务端就可能尝试把public当作secret，然后用HS256算法解密验证JWT。

  由于RS256的public人人都可获得，攻击者可以预先以public为密钥，用HS256算法伪造包含任意payload 的JWT，从而成功通过服务端的验证。

  ### 爆破密钥

  WT的安全性依赖于密钥的保密性，任何拥有密钥的人都可以构造任何内容的合法token。

  当一个JSON Web Token 被分发出去，如果密钥不够强壮就存在被爆破的风险，而且整个爆破过程可以离线进行。

## SpringBoot 集成 JWT

为什么要使用 JWT ? 因为前后端分离情况下不可能使用 session， cookie的方式进行鉴权，所以通过加密密钥 - JWT实现鉴权。

* 程序逻辑:
  1. 我们POST用户名与密码到/login进行登入，如果成功返回一个加密token，失败的话直接返回401错误。
  2. 之后用户访问每一个需要权限的网址请求必须在header中添加Authorization字段，例如Authorization: token，token为密钥。
  3. 后台会进行token的校验，如果不通过直接返回401。

添加 maven 依赖：

```xml
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>3.4.0</version>
</dependency>
```

写一个简单的 JWT 加密、校验工具，默认使用用户的密码充当密钥

```java
public class JWTUtil {
    // 默认 5 分钟过期
    private static final long EXPIRE_TIME = 5 * 10 * 1000;
    
    /**
    * 生成 token 签名，5 分钟后过期
    *
    * @param username 用户名
    * @param password 密码
    * @return 加密后的 token
    */
    pulic static String sign(String username, String password) {
        Date date = new Date(Instance.now() + EXPIRE_TIME);
        // 加密失败，即password为null 会抛出异常
        Algorithm algorithm = Algorithm.HHAC256(password);
        return JWT.create()
            	  .withClaim("username", username)
            	  .withExpireAt(date)
            	  .sign(algorithm);
    }
    
    /**
    * 校验 token 是否正确
    * @param token    密钥
    * @param password 用户密码
    * @return         是否正确
    */
    pulic static boolean verify(String token, String username, Stirng password) {
        // 加密失败，即password为null 会抛出异常
        Algorithm algorithm = Algorithm.HHAC256(password);
        JWTVerifier verifier = JWT.require(algorithm)
            					.withClaim("username",usernmae)
            					.build();
        DecodedJWT jwt = verifier.verify(token);
        return true;
    }
    
    /**
     * 获得token中的信息无需secret解密也能获得
     *
     * @return token中包含的用户名
     */
    public static String getUsername(String token) {
        DecodedJWT jwt = JWT.decode(token);
        return jwt.getClaim("username").asString();
    }
}
```

