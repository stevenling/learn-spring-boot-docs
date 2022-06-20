## 一、概述
平常我们的网站内容是不对非登录用户开放的，因此要对除了登录和注册的页面的其他任何页面进行访问拦截。
## 二、步骤
### 2.1 自定义 UserLoginInterceptor 拦截器
在 `config`目录下新建 `UserLoginInterceptor` 类，然后去实现 `org.springframework.web.servlet.HandlerInterceptor`这个接口。

重写 `preHandle`方法，这个方法会在控制器接受请求前调用，如果 `return false`则不往下进行了，则可以实现拦截。
```java
import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @author: yunhu
 * @date: 2022/6/8
 *
 * 拦截器：验证用户是否登录
 */
public class UserLoginInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // 从 request 的 header 中获得 token 值
        String token = request.getHeader("authorization");
        if (token == null || token.equals("")) {
            return false;
        }
        // 验证 token, JwtUtil 是自己定义的类，里面有个方法验证 token 
        String sub = JwtUtil.validateToken(token);
        if (sub == null || sub.equals("")) {
            return false;
        }
        // 更新 token 有效时间 
        if (JwtUtil.isNeedUpdate(token)) {
            // 过期就创建新的 token 给前端
            String newToken = JwtUtil.createToken(sub);
            response.setHeader(JwtUtil.USER_LOGIN_TOKEN, newToken);
        }
        return true;
    }
}
```
### 2.2 配置拦截器
定义好拦截器后，还不能直接使用，必须配置。

在 `config`目录下新建 `WebMvcConfig`类，实现 `WebMvcConfigurer`接口。

然后把刚刚自定义的用户登录的拦截器注册上。
```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.*;

/**
 * @author 云胡
 * @date 2022/6/8
 * 
 * 注册自定义拦截器
 */
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 注册 registration 拦截器
        InterceptorRegistration registration = registry.addInterceptor(new UserLoginInterceptor());
        
        // 拦截所有的路径
        registration.addPathPatterns("/**");
        
        // 添加不拦截路径 /api/user/login 是登录的请求, /api/user/register 注册的请求
        registration.excludePathPatterns(
                "/api/user/login",
                "/api/user/register",
                // html 静态资源
                "/**/*.html",
                // js 静态资源
                "/**/*.js",
                // css 静态资源
                "/**/*.css"
        );
    }
}
```

