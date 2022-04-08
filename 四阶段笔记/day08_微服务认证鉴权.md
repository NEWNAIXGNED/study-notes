### Gateway网关鉴权+JWT（高 60）

#### gateway微服务中添加全局过滤器

```java
package com.woniuxy.gateway.filter;

import com.commons.enums.TokenEnum;
import com.commons.result.ResponseResult;
import com.commons.utils.JWTUtil;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.core.io.buffer.DataBuffer;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.http.server.reactive.ServerHttpRequest;
import org.springframework.http.server.reactive.ServerHttpResponse;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

import javax.annotation.Resource;
import java.nio.charset.StandardCharsets;
import java.util.Arrays;
import java.util.List;

@Component
public class CustomFilter implements GlobalFilter, Ordered {
    @Resource
    private RedisTemplate<String,Object> redisTemplate;

    /**
     * 过滤器执行过滤代码
     */
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        //准备数据
        ResponseResult<Object> result = new ResponseResult<>(500,"error",null);
        byte[] data = null;

        //获取request对象
        ServerHttpRequest request = exchange.getRequest();
        //获取response对象
        ServerHttpResponse response = exchange.getResponse();

        //获取请求的URL
        String path = request.getPath().toString();

        //判断是否需要认证
        if (requireAuth(path)){
            //获取请求头
            List<String> refreshTokens = request.getHeaders().get("refreshToken");
            //获取token
            List<String> tokens = request.getHeaders().get("authorization");

            //判断是否有refreshToken
            if (refreshTokens == null || !redisTemplate.hasKey(refreshTokens.get(0)) || tokens == null || JWTUtil.verify(tokens.get(0))== TokenEnum.TOKEN_BAD) {
                //没登录
                try {
                    data = new ObjectMapper().writeValueAsString(result).getBytes(StandardCharsets.UTF_8);
                } catch (Exception e) {
                    e.printStackTrace();
                }
                //创建数据缓存对象
                DataBuffer buffer = response.bufferFactory().wrap(data);
                //设置响应头
                response.getHeaders().add("Content-Type", "application/json;charset=utf-8");
                //中断请求
                //response.setComplete();
                //返回数据
                return response.writeWith(Mono.just(buffer));
            }
            System.out.println(JWTUtil.verify(tokens.get(0)));
            //如果token过期
            if (JWTUtil.verify(tokens.get(0)) == TokenEnum.TOKEN_EXPIRE){
                //生成新的token，并设置到响应头
                String token = JWTUtil.generateToken(JWTUtil.getUname(tokens.get(0)));
                response.getHeaders().add("authorization",token);
                //暴露头
                response.getHeaders().add("Access-Control-Expose-Headers", "authentication");

            }
        }
        //
        //继续执行
        return chain.filter(exchange);
    }

    /*
     * 指定过滤器的顺序，值越小越先执行
     */
    @Override
    public int getOrder() {
        return 0;
    }

    //验证请求的URL是否需要认证
    private boolean requireAuth(String path){
        //需要认证操作的url
        List<String> paths = Arrays.asList(
                "/goods",
                "/order"
        );
        for (String url: paths){
            if (path.startsWith(url)) return true;
        }
        return false;
    }
}
```



#### 在commons中添加自定义注解

```java
package com.commons.annotations;

import java.lang.annotation.*;

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface RequirePerms {
    String value();
}
```



##### 在commons中引入spring boot web依赖

```xml
<!--web-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.3.7.RELEASE</version>
</dependency>
```



##### 在commons中自定义拦截器

```java
package com.commons.interceptors;

import com.commons.annotations.RequirePerms;
import com.commons.result.ResponseResult;
import com.commons.service.AuthService;
import com.commons.utils.JWTUtil;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.stereotype.Component;
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;

import javax.annotation.Resource;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.lang.reflect.Method;


@Component
public class Authinterceptor extends HandlerInterceptorAdapter {

    @Resource
    private AuthService authService;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //得到token
        String token = request.getHeader("authorization");
        System.out.println("拦截器:"+token);

        //方法
        if (handler instanceof HandlerMethod){
            HandlerMethod handlerMethod = (HandlerMethod)handler;
            //
            Method method = handlerMethod.getMethod();
            //
            if (method.isAnnotationPresent(RequirePerms.class)){
                //得到权限信息
                RequirePerms requirePerms = method.getDeclaredAnnotation(RequirePerms.class);
                String perms = requirePerms.value();
                
                //得到账号
                String account = JWTUtil.getUname(token);
                
                //使用token和perms请求auth模块的验证权限方法
                ResponseResult<String> result = authService.isPerms(account,perms);
                if (result.getMessage().equals("success")){
                    //有权限，放行
                    return true;
                }else {
                    //没有权限   返回结果
                    response.setContentType("application/json;charset=utf-8");
                    try {
                        response.getWriter().write(new ObjectMapper().writeValueAsString(result));
                    }catch (Exception e){
                        e.printStackTrace();
                    }
                    return false;
                }
            }
        }
        return super.preHandle(request, response, handler);
    }
}
```



##### 在commons中添加AuthService接口

```java
package com.commons.service;

import com.commons.result.ResponseResult;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;


@FeignClient(name = "AUTH")
public interface AuthService {
    //校验权限
    @GetMapping("/auth/isPerms/{account}/{perms}")
    public ResponseResult<String> isPerms(@PathVariable("account") String account, @PathVariable("perms") String perms);

}
```



#### 在auth的AuthController中添加对应方法

```java
@RestController
@RequestMapping("/auth")
public class AuthController {
	//.....其它代码....
    //校验权限
    @GetMapping("/isPerms/{account}/{perms}")
    public ResponseResult<String> isPerms(@PathVariable("account") String account,@PathVariable("perms") String perms){
        //校验权限
        System.out.println("校验权限"+account+","+perms);
        if (new Random().nextInt(2) == 1){
            //
            return new ResponseResult<>(200,"success",null);
        }
        //
        return new ResponseResult<>(500,"你没有权限",null);
    }
}
```



#### 在commons中添加拦截器配置类

```java
package com.commons.interceptors.configuration;

import com.commons.interceptors.Authinterceptor;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

import javax.annotation.Resource;

@Configuration
public class InterceptorConfiguration implements WebMvcConfigurer {
    @Resource
    private Authinterceptor authinterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(authinterceptor).addPathPatterns("/**");
    }
}
```



#### 在provider的controller对应方法上添加注解，指定访问的权限

```java
@RequirePerms("goods:all")
@GetMapping("/all")
@HystrixCommand(fallbackMethod = "fallback")
public List<Goods> all(){
    System.out.println(port);
    return Arrays.asList(
            new Goods(1001,"手机"),
            new Goods(1002,"电脑")
    );
}
```



##### 在provider的主启动类上开启相关注解

```java
@SpringBootApplication(scanBasePackages = "com")//扫描到拦截器
@EnableEurekaClient
@EnableCircuitBreaker
@EnableFeignClients(basePackages = "com.commons.service")//得到openfeign对象
public class ProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(ProviderApplication.class, args);
    }

}
```



#### gateway的commons包中去掉web模块

```xml
<dependency>
    <groupId>com.woniuxy</groupId>
    <artifactId>commons</artifactId>
    <version>1.0</version>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```



依次运行eureka、provider、auth、gateway模块进行测试

#### gateway排除拦截器

```java
@SpringBootApplication//(scanBasePackages = "com.woniuxy")
@EnableEurekaClient
@EnableCircuitBreaker
@ComponentScan(basePackages = "com.woniuxy",
        excludeFilters = @ComponentScan.Filter(
                type = FilterType.REGEX,
                pattern = "com.woniuxy.commons.interceptor.*")
)
public class GatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }

}
```







