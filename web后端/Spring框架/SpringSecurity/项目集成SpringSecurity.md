# 项目集成SpringSecurity

## 1.依赖导入

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

## 2.定义认证过滤器  `JwtLoginAuthenticationFilter`

```java
public class JwtLoginAuthenticationFilter extends AbstractAuthenticationProcessingFilter {

    private RedisTemplate redisTemplate;
    
    public void setRedisTemplate(RedisTemplate redisTemplate) {
        this.redisTemplate = redisTemplate;
    }
    
    public JwtLoginAuthenticationFilter(String loginUrl) {
        super(loginUrl);
    }
    
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException, IOException, ServletException {
        if (!request.getMethod().equals("POST") ||
                ! (request.getContentType().equalsIgnoreCase(MediaType.APPLICATION_JSON_VALUE) || request.getContentType().equalsIgnoreCase(MediaType.APPLICATION_JSON_UTF8_VALUE))) {
            throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
        }
        ServletInputStream in = request.getInputStream();
        LoginReqVo vo = new ObjectMapper().readValue(in, LoginReqVo.class);
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.setCharacterEncoding("utf-8");
        ...
        UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(username, password);
        return this.getAuthenticationManager().authenticate(authRequest);
    }
}
```

## 3.定义UserDetail认证详情信息类

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class LoginUserDetail implements UserDetails {
    ...
}
```

## 4.定义UserDetailsService实现

```java
@Component
public class LoginUserDetailService implements UserDetailsService {
    // 当用户登录认证是，底层会自动调用MyUserDetailService#loadUserByUsername（）把登录的账户名称传入
    // 根据用户名称获取用户的详情信息：用户名 加密密码 权限集合，还包含前端需要的侧边栏树 、前端需要的按钮权限标识的集合等
    @Override
    public UserDetails loadUserByUsername(String loginName) throws UsernameNotFoundException {
        LoginUserDetail loginUserDetail = new LoginUserDetail();
        // TODO getter() and setter()
        return loginUserDetail;
    }
}
```

## 5.认证成功后响应Token实现 `JwtLoginAuthenticationFilter`

```java
public class JwtLoginAuthenticationFilter extends AbstractAuthenticationProcessingFilter {
    private RedisTemplate redisTemplate;

    public void setRedisTemplate(RedisTemplate redisTemplate) {
        this.redisTemplate = redisTemplate;
    }

    public JwtLoginAuthenticationFilter(String loginUrl) {
        super(loginUrl);
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException, IOException, ServletException {
    }
    
    @Override
    protected void successfulAuthentication(HttpServletRequest request,
                                            HttpServletResponse response,
                                            FilterChain chain,
                                            Authentication authResult) throws IOException, ServletException {
        //获取用户的详情信息
        LoginUserDetail userDetail = (LoginUserDetail) authResult.getPrincipal();
        //组装LoginRespVoExt
        //获取用户名称
        String username = userDetail.getUsername();
        //获取权限集合对象
        List<GrantedAuthority> authorities = userDetail.getAuthorities();
        String auStrList = authorities.toString();
        //复制userDetail属性值到LoginRespVoExt对象即可
        LoginRespVoExt resp = new LoginRespVoExt();
        BeanUtils.copyProperties(userDetail,resp);
        //生成token字符串:将用户名称和权限信息价格生成token字符串
        String tokenStr = JwtTokenUtil.createToken(username, auStrList);
        resp.setAccessToken(tokenStr);
        //封装统一响应结果
        R<Object> r = R.ok(resp);
        String respStr = new ObjectMapper().writeValueAsString(r);
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.setCharacterEncoding("UTF-8");
        response.getWriter().write(respStr);
    }
    
    @Override
    protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response, AuthenticationException failed) throws IOException, ServletException {
        R<Object> r = R.error(ResponseCode.SYSTEM_PASSWORD_ERROR);
        String respStr = new ObjectMapper().writeValueAsString(r);
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.setCharacterEncoding("UTF-8");
        response.getWriter().write(respStr);
    }
```

## 6.定义Security配置类

```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig  extends WebSecurityConfigurerAdapter {

    @Autowired
    private RedisTemplate redisTemplate;
    
    private String[] getPubPath(){
        //公共访问资源
        String[] urls = {
                "/**/*.css","/**/*.js","/favicon.ico","/doc.html",
                "/druid/**","/webjars/**","/v2/api-docs","/api/captcha",
                "/swagger/**","/swagger-resources/**","/swagger-ui.html"
        };
        return urls;
    }
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //登出功能
        http.logout().logoutUrl("/api/logout").invalidateHttpSession(true);
        //开启允许iframe 嵌套。security默认禁用ifram跨域与缓存
        http.headers().frameOptions().disable().cacheControl().disable();
        //session禁用
        http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);
        http.csrf().disable();//禁用跨站请求伪造
        http.authorizeRequests()//对资源进行认证处理
                .antMatchers(getPubPath()).permitAll()//公共资源都允许访问
                .anyRequest().authenticated();  //除了上述资源外，其它资源，只有 认证通过后，才能有权访问
        //自定义的过滤器
        http.addFilterBefore(jwtLoginAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
    }
    
    @Bean
    public JwtLoginAuthenticationFilter jwtLoginAuthenticationFilter() throws Exception {
        JwtLoginAuthenticationFilter filter = new JwtLoginAuthenticationFilter("/api/login");
        filter.setAuthenticationManager(authenticationManagerBean());
        filter.setRedisTemplate(redisTemplate);
        return filter;
    }
    
    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
}
```
## 7.定义授权过滤器 `JwtAuthorizationFilter`

### 作用
- 过滤请求，获取请求头中的token字符串
- 解析token字符串，并获取token中信息：username role
- 将用户名和权限信息封装到UsernamePassowrdAuthentionToken票据对象下
- 将票据对象放入安全上下文，方便校验权限时，随时获取

```java
public class JwtAuthorizationFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        //1.从request对象下获取token数据，约定key：Authorization
        String tokenStr = request.getHeader(StockConstant.TOKEN_HEADER);
        //判断token字符串是否存在
        if (tokenStr==null) {
            //如果票据为null，可能用户还没有认证，正准备去认证,所以放行请求
            //放行后，会不会访问当受保护的资源呢？不会，因为没有生成UsernamePassowrdAuthentionToken
            filterChain.doFilter(request,response);
            return;
        }
        //2.解析tokenStr,获取用户详情信息
        Claims claims = JwtTokenUtil.checkJWT(tokenStr);
        //token字符串失效的情况
        if (claims==null) {
            //说明 票据解析出现异常，票据就失效了
            R<Object> r = R.error(ResponseCode.TOKEN_NO_AVAIL);
            String respStr = new ObjectMapper().writeValueAsString(r);
            response.setContentType(MediaType.APPLICATION_JSON_VALUE);
            response.setCharacterEncoding("UTF-8");
            response.getWriter().write(respStr);
            return;
        }
        //获取用户名和权限信息
        String userName = JwtTokenUtil.getUsername(tokenStr);
        //生成token时，权限字符串的格式是：[P8,ROLE_ADMIN]
        String perms = JwtTokenUtil.getUserRole(tokenStr);
        //生成权限集合对象
        String strip = StringUtils.strip(perms, "[]");
        List<GrantedAuthority> authorityList = AuthorityUtils.commaSeparatedStringToAuthorityList(strip);
        //将用户名和权限信息封装到token对象下
        UsernamePasswordAuthenticationToken token=new UsernamePasswordAuthenticationToken(userName,null,authorityList);
        //将token对象存入安全上限文，这样，线程无论走到哪里，都可以获取token对象，验证当前用户访问对应资源是否被授权
        SecurityContextHolder.getContext().setAuthentication(token);
        //资源发行
        filterChain.doFilter(request,response);
    }
}
```

## 8. 配置授权过滤器 

```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig  extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        ...
        // 认证过滤器
        http.addFilterBefore(jwtLoginAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
        // 授权过滤器
        http.addFilterBefore(jwtAuthorizationFilter(), UsernamePasswordAuthenticationFilter.class);
    }
    
    @Bean
    public JwtLoginAuthenticationFilter jwtLoginAuthenticationFilter() throws Exception {
        JwtLoginAuthenticationFilter filter = new JwtLoginAuthenticationFilter("/api/login");
        filter.setAuthenticationManager(authenticationManagerBean());
        filter.setRedisTemplate(redisTemplate);
        return filter;
    }

    @Bean
    public JwtAuthorizationFilter jwtAuthorizationFilter(){
        return new JwtAuthorizationFilter();
    }
}
```

## 9.定义权限拒绝处理器 `StockAccessDenyHandler` 和  `StockAuthenticationEntryPoint`

### 作用
- 定义没有权限，访问拒绝的处理器 
- 用户认证成功，但是没有访问的权限，则会除非拒绝处理器 
- 如果是匿名用户访问被拒绝则使用匿名拒绝的处理器

> 定义用户认证成功无权限访问处理器
```java
@Slf4j
public class StockAccessDenyHandler implements AccessDeniedHandler {
    @Override
    public void handle(HttpServletRequest request,
                       HttpServletResponse response,
                       AccessDeniedException ex) throws IOException, ServletException {
        R<Object> r = R.error(ResponseCode.NOT_PERMISSION);
        String respStr = new ObjectMapper().writeValueAsString(r);
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.setCharacterEncoding("UTF-8");
        response.getWriter().write(respStr);
    }
}
```

> 匿名用户(未认证用户)访问拒绝处理器
```java
public class StockAuthenticationEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest request,
                         HttpServletResponse response,
                         AuthenticationException authException) throws IOException, ServletException {
        R<Object> r = R.error(ResponseCode.NOT_PERMISSION);
        String respStr = new ObjectMapper().writeValueAsString(r);
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.setCharacterEncoding("UTF-8");
        response.getWriter().write(respStr);
    }
}
```

> 在SecurityConfig类配置处理器
```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig  extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        ...
        http.exceptionHandling().accessDeniedHandler(new StockAccessDenyHandler())
                                .authenticationEntryPoint(new StockAuthenticationEntryPoint());
    }
}
```

## 10.配置资源访问权限注解Controller层

### 作用
- 定义Controller层的资源访问权限注解
- 定义用户认证成功，但是没有访问的权限，则会除非拒绝处理器 
- 如果是匿名用户访问被拒绝则使用匿名拒绝的处理器

```java
@RestController
@RequestMapping("/api")
public class LogController {
    @PreAuthorize("hasAuthority('sys:log:list')")
    @PostMapping("/logs")
    public R<PageResult> logPageQuery(@RequestBody LogPageReqVo vo){
        ...
    }

    @DeleteMapping("/log")
    @PreAuthorize("hasAuthority('sys:log:delete')")
    public R<String> deleteBatch(@RequestBody List<Long> logIds){
        ...
    }
}
```


