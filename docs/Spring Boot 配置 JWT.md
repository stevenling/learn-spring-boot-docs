## 一、概述
Web 网站离不开用户认证，这边我们不用 `session`，而直接使用 `JWT`。
JWT 的背景知识可以看阮一峰老师的这篇文章: 
 
JWT 由三个部分组成：

- Header（头部）
- Payload（负载）
- Signature（签名）
## 二、步骤
### 2.1 application.yml 中添加 jwt 信息
在 `src/main/resources/application.yml`中添加 `jwt` 配置信息。
```yaml
# jwt 配置
jwt:
  header: "Authorization"     # token 返回头部
  tokenPrefix: "Bearer "      # token 前缀
  secret: "qwertyuiop1214156" # 私钥
  expireTime: 43200            # token 的有效时间，单位是分钟
```
### 2.2 新建 JwtUtil 工具类
新建 `util`目录，在 `util` 目录下新建 `JwtUtil.class`。
`@Component`将当前 `JwtUtil`注入到 `spring` 容器中。
`@ConfigurationProperties(prefix = "jwt")`，匹配 `application.yml`配置文件的前缀，然后将配置文件里面的数据加载到当前类。
```java
**
 * Jwt Token 生成的工具类
 *
 * @author yunhu
 * @date 2022/6/8
 */

@Component
@ConfigurationProperties(prefix = "jwt")
public class JwtUtil {
    /**
     * 定义 token 返回头部
     */
    public static String header;

    /**
     * token 前缀
     */
    public static String tokenPrefix;

    /**
     * 签名密钥
     */
    public static String secret;

    /**
     * 有效期
     */
    public static long expireTime;

    /**
     * 存进客户端的 token 的 key 名
     */
    public static final String USER_LOGIN_TOKEN = "token";

    /**
     * 设置 token 头部
     * @param header token 头部
     */
    public void setHeader(String header) {
        JwtUtil.header = header;
    }

    /**
     * 设置 token 前缀
     * @param tokenPrefix token 前缀
     */
    public void setTokenPrefix(String tokenPrefix) {
        JwtUtil.tokenPrefix = tokenPrefix;
    }

    /**
     * 设置 token 密钥
     * @param secret token 密钥
     */
    public void setSecret(String secret) {
        JwtUtil.secret = secret;
    }

    /**
     * 设置 token 有效时间
     * @param expireTimeInt token 有效时间
     */
    public void setExpireTime(int expireTimeInt) {
        JwtUtil.expireTime = expireTimeInt;
    }

    /**
     * 创建 TOKEN
     * JWT 构成: header, payload, signature
     * @param sub jwt 所面向的用户，即用户名
     * @return token 值
     */
    public static String createToken(String sub) {
        return tokenPrefix + JWT.create()
                .withSubject(sub)
                .withExpiresAt(new Date(System.currentTimeMillis() + expireTime))
                .sign(Algorithm.HMAC512(secret));
    }

    /**
     * 验证 token
     * @param token 验证的 token 值
     * @return 用户名
     */
    public static String validateToken(String token) throws Exception {
        try {
            Verification verification = JWT.require(Algorithm.HMAC512(secret));
            JWTVerifier jwtVerifier = verification.build();
            // 去除 token 的前缀
            String noPrefixToken = token.replace(tokenPrefix, "");
            DecodedJWT decodedJwt = jwtVerifier.verify(noPrefixToken);
            if(decodedJwt != null) {
                return decodedJwt.getSubject();
            }
            return "";
        } catch (TokenExpiredException e){
            e.printStackTrace();
        } catch (Exception e){
            e.printStackTrace();
            return "";
        }
        return "";
    }

    /**
     * 检查 token 是否需要更新
     * @param token token 值
     * @return 过期时间
     */
    public static boolean isNeedUpdate(String token) throws Exception {
        // 获取 token 过期时间
        Date expiresAt = null;
        try {
            expiresAt = JWT.require(Algorithm.HMAC512(secret))
                    .build()
                    .verify(token.replace(tokenPrefix, ""))
                    .getExpiresAt();
        } catch (TokenExpiredException e){
            return true;
        } catch (Exception e){
            throw new Exception(("token 验证失败"));
        }
        // 需要更新
        return (expiresAt.getTime() - System.currentTimeMillis()) < (expireTime >> 1);
    }
}

```
### 2.3 用户登录创建 token 
```java
    /**
     * 用户登录
     * @param request 登录请求
     * @return 响应信息
     */
    @GetMapping(value = "/user/login")
    public Map<String, Object> userLogin(HttpServletRequest request, HttpServletResponse response)  throws IOException {
        String formData = request.getParameter("formData");
        // 转为 json
        Map<String, Object> map = new HashMap<>(3);

        JSONObject json = JSONObject.parseObject(formData);
        if(CollectionUtils.isNotEmpty(json)) {
            String userName = (String) json.get("userName");
            String password = (String) json.get("password");

            // 验证账号密码是否正确
            QueryWrapper<UserTableEntity> queryWrapper = new QueryWrapper<>();
            queryWrapper
                    .eq("username", userName)
                    .eq("password", password);

            List<UserTableEntity> list = userTableMapper.selectList(queryWrapper);
            if(CollectionUtils.isNotEmpty(list)) {
                // 登录成功,加上 token
                String token = JwtUtil.createToken(list.get(0).getUsername());
                map.put("loginResult", true);
                map.put("msg", "登录成功");
                map.put("user",list.get(0));
                map.put("token", token);
                response.setHeader(JwtUtil.USER_LOGIN_TOKEN, token);
                return map;
            }else {
                // 登录失败
                map.put("loginResult", false);
                map.put("msg", "登录失败，请认真检查账号密码哦");
            }
        }
        return map;
    }
```
### 2.3 拦截器验证 token
如何自定义拦截器可以看我的这篇  。
```java
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // http 的 header 中获得token
        String token = request.getHeader("authorization");
        // token 不存在
        if (token == null || token.equals("")) {
            System.out.println("UserLoginInterceptor class token is not exist");
            return false;
        }
        // 验证 token
        String sub = JwtUtil.validateToken(token);
        if (sub == null || sub.equals("")) {
            System.out.println("token exist, but not same");
            return false;
        }
        // 更新 token 有效时间 (如果需要更新其实就是产生一个新的 token)
        if (JwtUtil.isNeedUpdate(token)){
            String newToken = JwtUtil.createToken(sub);
            response.setHeader(JwtUtil.USER_LOGIN_TOKEN, newToken);
        }
        return true;
    }
```
## 三、参考资料

