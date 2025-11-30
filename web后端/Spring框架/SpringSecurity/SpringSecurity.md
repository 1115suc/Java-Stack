# SpringSecurity

## SpringSecurity 使用

### 引入依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### SpringSecurity 配置

#### 自定义认证配置类

##### 步骤

- `@EnableWebSecurity` 开启web安全设置生效
- 继承`WebSecurityConfigurerAdapter`类
- 注入`UserDetailsService`服务类

```java
@Configuration
@EnableWebSecurity//开启web安全设置生效
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    /**
     * 构建认证服务，并将对象注入spring IOC容器，用户登录时，会调用该服务进行用户合法信息认证
     * @return
     */
    @Bean
    public UserDetailsService userDetailsService(){
        //从内存获取用户认证信息的服务类（了解）后期用户的信息要从表中获取
        InMemoryUserDetailsManager inMemoryUserDetailsManager = new InMemoryUserDetailsManager();
        //构建用户,真实开发中用户信息要从数据库加载构建
        UserDetails u1 = User
                .withUsername("1115suc")
                .password("{noop}123456")//{noop}:no operration--》表示登录时对避免不做任何操作，说白了就是明文比对
                .authorities("P5", "ROLE_ADMIN")//用户的权限信息
                .build();
        UserDetails u2 = User
                .withUsername("1115suc")
                .password("{noop}123456")
                .authorities("P7", "ROLE_ADMIN", "ROLE_SELLER")//如果角色也作为一种权限资源，则角色名称的前缀必须加ROLE_
                .build();
        inMemoryUserDetailsManager.createUser(u1);
        inMemoryUserDetailsManager.createUser(u2);
        return inMemoryUserDetailsManager;
    }
}
```
> 在userDetailsService()方法中 返回了一个UserDetailsService对象给spring容器管理，当用户发生登录认证行为时，Spring Security底层会自动调用UserDetailsService类型bean提供的用户信息进行合法比对，如果比对成功则资源放行，否则就认证失败；

---

#### 自定义授权配置

##### 步骤

- `@EnableWebSecurity` 开启web安全设置生效
- 继承`WebSecurityConfigurerAdapter`类
- 重写`configure(HttpSecurity http)`方法

```java
@Configuration
@EnableWebSecurity
//@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {
	
    @Bean
    @Override
    public UserDetailsService userDetailsService(){
        InMemoryUserDetailsManager inMemoryUserDetailsManager = new InMemoryUserDetailsManager();
        inMemoryUserDetailsManager.createUser(User.withUsername("1115suc").password("{noop}123456").authorities("P1","ROLE_ADMIN").build());
        inMemoryUserDetailsManager.createUser(User.withUsername("1115suc").password("{noop}123456").authorities("O1","ROLE_SELLER").build());
        return inMemoryUserDetailsManager;
    }
	
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin()//开启默认form表单登录方式
                .and()
                .logout()//登出用默认的路径登出 /logout
                .permitAll()//允许所有的用户访问登录或者登出的路径
                .and()
                  .csrf().disable()//启用CSRF,防止CSRF攻击
                .authorizeRequests()//授权方法，该方法后有若干子方法进行不同的授权规则处理
                //允许所有账户都可访问（不登录即可访问）,同时可指定多个路径
                .antMatchers("/register").permitAll()
                //开发方式1：基于配置
                .antMatchers("/a1","/a2").hasRole("seller")//拥有seller角色的用户可访问a1和a2资源
                .antMatchers("/b1").hasAnyRole("manager1","manager2")
                .antMatchers("/c1").hasAnyAuthority("aa","bb")
                .antMatchers("/d").denyAll()//拒绝任意用户访问
                .antMatchers("/e").anonymous()//允许匿名访问
                .antMatchers("/f").hasIpAddress("localhost/82")
                .antMatchers("/hello").hasAuthority("P5") //具有P5权限才可以访问
                .antMatchers("/say").hasRole("ADMIN") //具有ROLE_ADMIN 角色才可以访问
                .anyRequest().authenticated(); //除了上边配置的请求资源，其它资源都必须授权才能访问
    }
}
```
---

#### 自定义注解授权配置

##### 步骤
- 开启全局方法授权注解生效
```java
@EnableGlobalMethodSecurity(prePostEnabled = true)
```
- 调整资源配置类
```java
@Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin()//定义认证时使用form表单的方式提交数据
                .and()
                .logout()//登出用默认的路径登出 /logout
                .permitAll()//允许所有的用户访问登录或者登出的路径,如果 .anyRequest().authenticated()注释掉，则必须添加permitAll()，否则就不能正常访问登录或者登出的路径
                .and()
                .csrf().disable()
                .authorizeRequests();//授权方法，该方法后有若干子方法进行不同的授权规则处理
    }
```
- 注解配置资源权限
```java
 /**
     *  @PreAuthorize:指在注解作用的方法执行之前，做权限校验
     *  @PostAuthorize:指在注解作用的方法执行之后，做权限校验
     *  @return
     */
    @PreAuthorize("hasAuthority('P5')")
//    @PostAuthorize("hasAuthority('P4')")
    @GetMapping("/hello")
    public String hello(){
        return "hello security";
    }
    
    @PreAuthorize("hasRole('ADMIN')")
    @GetMapping("/say")
    public String say(){
        return "say security";
    }
    
    @PermitAll//等价于antMatchers("/register").permitAll()//任何用户都可访问
    //@PreAuthorize("isAnonymous()")
    @GetMapping("/register")
    public String register(){
        return "register security";
    }
```

---

#### 自定义Security认证过滤器

![1646452649158.png](img/1646452649158.png)

##### 1.1 定义获取用户权限UserDetailsService

```java
@Service("userDetailsService")
public class MyUserDetailServiceImpl implements UserDetailsService {

    @Autowired
    private TbUserMapper tbUserMapper;

    //使用security当用户认证时，会自动将用户的名称注入到该方法中
    //然后我们可以自己写逻辑取加载用户的信息，然后组装成UserDetails认证对象
    @Override
    public UserDetails loadUserByUsername(String userName) throws UsernameNotFoundException {
        TbUser dbUser=tbUserMapper.findUserInfoByName(userName);
        if (dbUser==null) {
            throw new UsernameNotFoundException("用户名输入错误！");
        }
        //2.组装UserDetails对象
        //获取当前用户对应的权限集合（自动将以逗号间隔的权限字符串封装到权限集合中）
        List<GrantedAuthority> list = AuthorityUtils.commaSeparatedStringToAuthorityList(dbUser.getRoles());
        UserDetails userDetails = new User(dbUser.getUsername(), dbUser.getPassword(), list);
        return userDetails;
    }
}
```

#### 1.2 自定义认证过滤器AbstractAuthenticationProcessingFilter

```java
public class MyUserNamePasswordAuthenticationFilter extends AbstractAuthenticationProcessingFilter {
    private static final String USER_NAME = "username";
    private static final String PASSWORD = "password";
    
    // 设置构造，传入自定义登录url地址
    public MyUserNamePasswordAuthenticationFilter(String defaultFilterProcessesUrl) {
        super(defaultFilterProcessesUrl);
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException, IOException, ServletException {
        //判断请求方法必须是post提交，且提交的数据的内容必须是application/json格式的数据
        if (!request.getMethod().equals("POST") || !(request.getContentType().equalsIgnoreCase(MediaType.APPLICATION_JSON_VALUE) || request.getContentType().equalsIgnoreCase(MediaType.APPLICATION_JSON_UTF8_VALUE))) {
            throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
        }
        //获取请求参数
        //获取reqeust请求对象的发送过来的数据流
        ServletInputStream in = request.getInputStream();
        //将数据流中的数据反序列化成Map
        HashMap<String,String> loginInfo = new ObjectMapper().readValue(in, HashMap.class);
        String username = loginInfo.get(USER_NAME);
        username = (username != null) ? username : "";
        username = username.trim();
        String password = loginInfo.get(PASSWORD);
        password = (password != null) ? password : "";
        //将用户名和密码信息封装到认证票据对象下
        UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(username, password);
        // Allow subclasses to set the "details" property
		//setDetails(request, authRequest);
        //调用认证管理器认证指定的票据对象
        return this.getAuthenticationManager().authenticate(authRequest);
    }
    
    @Override
    protected void successfulAuthentication(HttpServletRequest request,
                                            HttpServletResponse response,
                                            FilterChain chain,
                                            Authentication authResult) throws IOException, ServletException {
        response.getWriter().write("login success 666.....");
    }
    
    @Override
    protected void unsuccessfulAuthentication(HttpServletRequest request,
                                              HttpServletResponse response,
                                              AuthenticationException failed) throws IOException, ServletException {
        response.getWriter().write("login failue 999");
    }
}
```

#### 1.3 定义SecurityConfig类

```java
    // 配置授权策略
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable();//禁用跨站请求伪造
  		http.authorizeRequests()//对资源进行认证处理
    	.antMatchers("/myLogin").permitAll()//登录路径无需拦截
    	.anyRequest().authenticated();  //除了上述资源外，其它资源，只有 认证通过后，才能有权访问
        //坑-过滤器要添加在默认过滤器之前，否则，登录失效
     	http.addFilterBefore(myUserNamePasswordAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
    }

    @Bean
    public MyUserNamePasswordAuthenticationFilter myUserNamePasswordAuthenticationFilter() throws Exception {
      	//设置默认登录路径
        MyUserNamePasswordAuthenticationFilter myUserNamePasswordAuthenticationFilter =
                new MyUserNamePasswordAuthenticationFilter("/myLogin");
        myUserNamePasswordAuthenticationFilter.setAuthenticationManager(authenticationManagerBean());
        return myUserNamePasswordAuthenticationFilter;
    }
```

#### 1.4 基于JWT实现无状态认证

```java
@Override
    protected void successfulAuthentication(HttpServletRequest request,
                                            HttpServletResponse response,
                                            FilterChain chain,
                                            Authentication authResult) throws IOException, ServletException {
        //认证主体信息，此时以填充用户权限信息
        UserDetails principal = (UserDetails) authResult.getPrincipal();
        //组装响应前端的信息
        String username = principal.getUsername();
        Collection<? extends GrantedAuthority> authorities = principal.getAuthorities();
        //构建JwtToken 加入权限信息是为了将来访问时，jwt解析获取当前用户对应的权限，做授权的过滤
      	//权限字符串格式：[P5, ROLE_ADMIN]
        String token = JwtTokenUtil.createToken(username, authorities.toString());
        HashMap<String, String> info = new HashMap<>();
        info.put("name",username);
        info.put("token",token);
        //设置响应格式
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        //内容编码格式
        response.setCharacterEncoding("utf-8");
        response.getWriter().write(new ObjectMapper().writeValueAsString(info));
    }
```

#### 1.5 自定义Security授权过滤
![renzheng.png](img/renzheng.png)
![授权校验流程.png](img/shouquan.png)

```java
public class AuthenticationFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        //1.获取http请求头中携带的jwt票据字符串（注意：如果用户尚未认证，则jwt票据字符串不存在）
        String jwtToken = request.getHeader(JwtTokenUtil.TOKEN_HEADER);
        //2.判断请求中的票据是否存在
        if (StringUtils.isBlank(jwtToken)) {
            //如果票据为空，可能用户准备取认证，所以不做拦截，但是此时UsernamePasswordAuthenticationToken对象未生成，那么即使放行本次请求
            //后续的过滤器链中也会校验认证票据对象
            filterChain.doFilter(request,response);
            return;
        }
        //3.校验票据
        Claims claims = JwtTokenUtil.checkJWT(jwtToken);
        //票据失效
        if (claims==null) {
            //票据失效则提示前端票据已经失效，需要重新认证
            response.setContentType(MediaType.APPLICATION_JSON_VALUE);
            response.setContentType("utf-8");
            response.getWriter().write("jwt token failure!!");
            return;
        }
        //4.从合法的票据中获取用户名和权限信息
        //用户名
        String username = JwtTokenUtil.getUsername(jwtToken);
        //权限信息 [P5, ROLE_ADMIN]
        String roles = JwtTokenUtil.getUserRole(jwtToken);
        //将数组格式的字符串转化成权限对象集合
        String comStr = StringUtils.strip(roles, "[]");
        List<GrantedAuthority> authorityList = AuthorityUtils.commaSeparatedStringToAuthorityList(comStr);
        //5.组装认证成功的票据对象（认证成功时，密码位置null）
        UsernamePasswordAuthenticationToken token = new UsernamePasswordAuthenticationToken(username, null, authorityList);
        //6.将认证成功的票据对象保存到安全上下文中，方便后续的过滤器直接获取权限信息
        SecurityContextHolder.getContext().setAuthentication(token);
        //7.发行过滤器
        filterChain.doFilter(request,response);
    }
}
```

#### 1.6 配置自定义授权过滤器

```java
//  给访问的资源配置权 限过滤
@Override
protected void configure(HttpSecurity http) throws Exception {
  http.csrf().disable();//禁用跨站请求伪造
  http.authorizeRequests()//对资源进行认证处理
    .antMatchers("/myLogin").permitAll()//登录路径无需拦截
    .anyRequest().authenticated();  //除了上述资源外，其它资源，只有 认证通过后，才能有权访问
  //添加自定义的认证过滤器：UsernamePasswordAuthenticationFilter是默认的登录认证过滤器，而我们重写了该过滤器，所以访问时，应该先走我们
  //自定义的过滤器
  http.addFilterBefore(myUsernamePasswordAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
  //配置授权过滤器，过滤一切资源
  http.addFilterBefore( authenticationFilter(),MyUsernamePasswordAuthenticationFilter.class);
}  

// 自定义授权过滤器，过滤一切被访问的资源
@Bean
public AuthenticationFilter authenticationFilter(){
  AuthenticationFilter filter = new AuthenticationFilter();
  return filter;
}
```

#### 1.7 自定义权限拒绝处理

- 通过实现AccessDeniedHandler接口实现：
```java
@Override
    protected void configure(HttpSecurity http) throws Exception {
		//......
        http.exceptionHandling()
                .accessDeniedHandler(new AccessDeniedHandler() {
                    @Override
                    public void handle(HttpServletRequest request,
                                       HttpServletResponse response,
                                       AccessDeniedException e) throws IOException, ServletException {
                        //认证用户访问资源时权限拒绝处理策略
                        response.getWriter().write("no permission......reject....");
                    }
                });
    }
```

- 通过实现AuthenticationEntryPoint接口实现：
```java
@Override
    protected void configure(HttpSecurity http) throws Exception {
		//......
				http.exceptionHandling()
                .accessDeniedHandler(new AccessDeniedHandler() {
                    @Override
                    public void handle(HttpServletRequest request,
                                       HttpServletResponse response,
                                       AccessDeniedException e) throws IOException, ServletException {
                        //认证用户访问资源时权限拒绝处理策略
                        response.getWriter().write("no permission......reject....");
                    }
                })
          		.authenticationEntryPoint(new AuthenticationEntryPoint(){
                    @Override
    				public void commence(HttpServletRequest request,
                         HttpServletResponse response,
                         AuthenticationException authException) throws IOException, ServletException {
        				 response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        				 response.setCharacterEncoding("UTF-8");
        				 response.getWriter().write("匿名用户无权访问");
          });
    }
}        
```

### SpringSecurity 认证原理分析
- Spring Security所解决的问题就是安全访问控制，而安全访问控制功能其实就是对所有进入系统的请求进行拦截，校验每个请求是否能够访问它所期望的资源。根据前边知识的学习，可以通过Filter或AOP等技术来实现；
- Spring Security对Web资源的保护是基于Filter过滤器+AOP实现的，所以我们从Filter来入手，逐步深入Spring Security原理；
- 当初始化Spring Security时，会创建一个名为 SpringSecurityFilterChain的Servlet过滤器，类型为org.springframework.security.web.FilterChainProxy，它实现了javax.servlet.Filter，因此外部的请求会经过该类；
![image-20210314174535436.png](img/image-20210314174535436.png)

FilterChainProxy是一个代理，真正起作用的是FilterChainProxy中SecurityFilterChain所包含的各个Filter，同时 这些Filter作为Bean被Spring管理，它们是Spring Security核心，各有各的职责，但他们并不直接处理用户的认 证，也不直接处理用户的授权，而是把它们交给了认证管理器
（AuthenticationManager）和决策管理器 （AccessDecisionManager）进行处理。

下面介绍过滤器链中主要的几个过滤器及其作用：

- SecurityContextPersistenceFilter 这个Filter是整个拦截过程的入口和出口（也就是第一个和最后一个拦截器），会在请求开始时从配置好的 SecurityContextRepository 中获取SecurityContext，然后把它设置给 SecurityContextHolder。在请求完成后将SecurityContextHolder 持有的 SecurityContext 再保存到配置好 的SecurityContextRepository，同时清除 securityContextHolder 所持有的 SecurityContext；
- UsernamePasswordAuthenticationFilter用于处理来自表单提交的认证。该表单必须提供对应的用户名和密码，其内部还有登录成功或失败后进行处理的 AuthenticationSuccessHandler 和 AuthenticationFailureHandler，这些都可以根据需求做相关改变；
- FilterSecurityInterceptor 是用于保护web资源的，使用AccessDecisionManager对当前用户进行授权访问；
- ExceptionTranslationFilter 能够捕获来自 FilterChain 所有的异常，并进行处理。但是它只会处理两类异常： AuthenticationException 和 AccessDeniedException，其它的异常它会继续抛出。

核心代码说明：

