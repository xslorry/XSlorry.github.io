# SpirngSecurity


# **SpirngSecurity**

**用户认证**

**用户授权**

与Shiro对比

+ SSM+Shiro
+ Spirng Boot/SpirngCloud +Spirng Security

SpringSecurity本质上是一个过滤器链：有很多过滤器

**FilterSecurityInterceptor**：是一个方法级的权限过滤器，基本位于过滤链的最底部

**ExceptionTranslationFilter**:是一个异常过滤器，用来处理在认证授权过程汇总抛出的异常

**UsernamePasswordAuthenticationFilter**：对/login的POST请求做拦截，校验表单中用户名，密码。

# 过滤器是如何进行加载的？

1. 使用SpringSecurity配置过滤器
   * DelegatingFilterProxy

# UserDetailsService接口 

查询数据库用户名和密码的过程

+ 创建类继承UsernamePasswordAuthenticationFilter，重写三个方法

+ 创建类实现UserDetailService，编写查询数据过程，返回User对象，这个User对象是安全框架提供对象

# PasswordEncoder接口

数据加密接口，用于返回User对象里面密码加密

**web权限方案**

(1)认证

(2)授权

1、设置登录的用户名和密码

第一种方式：通过配置文件

```properties
spring.security.user.name=lorry
spring.security.user.password=123456
```

第二种方式：通过配置类

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
        String encode = passwordEncoder.encode("123");
        auth.inMemoryAuthentication().withUser("lorry2").password(encode).roles("admin");

    }
    @Bean
    PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
}
  
```

第三种方式：自定义编写实现类

+ 第一步 创建配置类，设置使用哪个UserDetailsService实现

```java
@Configuration
public class SecurityConfig1 extends WebSecurityConfigurerAdapter {

    @Autowired
    private UserDetailsService userDetailsService;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {

        auth.userDetailsService(userDetailsService).passwordEncoder(passwordEncoder());
    }
    @Bean
    PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
}

```

+ 第二步 编写实现类，返回User对象，User对象有用户名密码和操作权限

```java
@Service("userDetailsService")
public class MyUserDetailsService implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        List<GrantedAuthority> role = AuthorityUtils.commaSeparatedStringToAuthorityList("role");
        return new User("lorry3",new BCryptPasswordEncoder().encode("123"),role);
    }
}
```

**查询数据库完成用户认证**

+ 整合mybatisplus完成数据库操作

  + 引入相关依赖

  ```java
  <dependency>
  	<groupId>mysql</groupId>
  	<artifactId>mysql-connector-java</artifactId>
  	</dependency>
  <dependency>
  	<groupId>org.projectlombok</groupId>
  	<artifactId>lombok</artifactId>
  </dependency>
  <dependency>
  	<groupId>com.baomidou</groupId>
  	<artifactId>mybatis-plus-boot-starter</artifactId>
  	<version>3.4.3.1</version>
  </dependency>
  ```

  + 创建数据库和表
  + 创建实体类
  + 配置数据源
  + mybatisplus配置

  ```java
  @Mapper
  public interface AdminMapper extends BaseMapper<Admin> {
  }
  ```

  + 在MyAdminDetailsService调用mapper里面的方法查询数据库进行用户认证

  ```java
  @Service("userDetailsService")
  public class MyUserDetailsService implements UserDetailsService {
  
      @Autowired
      private AdminMapper adminMapper;
  
      @Override
      public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
          //调用adminMapper方法查询数据库
          QueryWrapper<Admin> wrapper = new QueryWrapper();
          wrapper.eq("username",username);
          Admin admin = adminMapper.selectOne(wrapper);
          //判断
          if (admin==null){
              throw new UsernameNotFoundException("用户名不存在");
          }
  
          List<GrantedAuthority> role = AuthorityUtils.commaSeparatedStringToAuthorityList("role");
          return new User(admin.getUsername(),new BCryptPasswordEncoder().encode(admin.getPassword()),role);
      }
  }
  ```

  

# 自定义设置登录页面

编写配置类重写configure(HttpSecurity http)方法

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.formLogin() //自定义登录页面
        .loginPage("/login.html")
        .loginProcessingUrl("/user/login") //登录访问路径
        .defaultSuccessUrl("/test/index").permitAll() //登录成功之后，跳转路径
        .and().authorizeRequests()
        .antMatchers("/","/test/hello","/user/login").permitAll() //设置哪些路径可以直接访问，不需要认证
        .anyRequest().authenticated()
        .and().csrf().disable(); //关闭csrf防护
}
```

静态页面name必须要设置成username,password

```html
<form action="/user/login" method="post">    用户名：<input type="text" name="username"/>    <br/>    密码：<input type="text" name="password"/>    <br/>    <input type="submit" value="login"/></form>
```

# 基于角色或者权限进行访问控制

## hasAuthority方法

​	如果当前的主体具有指定的权限，则返回true，否则返回false

1. 在配置类设置当前访问地址有哪些权限

```java
.antMatchers("/test/index").hasAnyAuthority("admins") //当前登录用户，只有具有admins权限才能访问这个路径
```

2. 在UserDetailsService，给返回User对象设置权限

```java
List<GrantedAuthority> role =                AuthorityUtils.commaSeparatedStringToAuthorityList("admins");
```

## hasAnyAuthority方法

​	针对多个权限

```java
 //hasAnyAuthority方法            .antMatchers("/test/index").hasAnyAuthority("admins","manager")
```

​	第二步勿漏

## hasRole方法

底层与前面两个方法不一样

```java
private static String hasRole(String role) {        Assert.notNull(role, "role cannot be null");        Assert.isTrue(!role.startsWith("ROLE_"), () -> {            return "role should not start with 'ROLE_' since it is automatically inserted. Got '" + role + "'";        });        return "hasRole('ROLE_" + role + "')";    }
```

返回的是'ROLE_'+传入的role

1. 

```java
//hasRole方法 返回ROLE_+hasRole的传入值.antMatchers("/test/index").hasRole("sale")
```

2. 

```java
List<GrantedAuthority> role =                AuthorityUtils.commaSeparatedStringToAuthorityList("admins,ROLE_sale");
```

## hasAnyRole

 表示用户具有任一条件就可以访问，和hasAnyAuthority方法类似

# 自定义403界面（没有权限访问）

1. 写静态页面
2. 在configure(HttpSecurity http)里面配置

```java
//配置没有权限访问跳转自定义页面http.exceptionHandling().accessDeniedPage("/unauth.html");
```

# 认证授权注解使用

## @Secured

判断是否具有**角色**，另外需注意的是这里匹配的字符串要添加前缀“ROLE_”

使用注解前要开启注解功能！

@EnableGlobalMethodSecurity(securedEnabled=true)

1. 启动类(配置类)开启注解

```java
@EnableGlobalMethodSecurity(securedEnabled=true)
```

2. 在controller的方法上面使用注解，设置角色

```java
@GetMapping("/update")@Secured({"ROLE_sale","ROLE_manager"})public String update(){    return "hello update";}
```

3. userDetailsService设置用户角色

## @PreAuthorize

@PreAuthorize ：注解适合**进入方法前的权限验证**，@PreAuthorize可以将登陆用户的roles/permissions参数传到方法中。

1. 启动类(配置类)开启注解

```java
@EnableGlobalMethodSecurity(securedEnabled=true,prePostEnabled = true)
```

2. 在controller的方法上面使用注解，设置角色

```java
@GetMapping("/update")//@Secured({"ROLE_sale","ROLE_manager"})@PreAuthorize("hasAnyAuthority('admins')")public String update(){    return "hello update";}
```

3. userDetailsService设置用户角色

## @PostAuthorize

@PostAuthorize：注解适合**进入方法后的权限验证**

```java
@GetMapping("/update")//@Secured({"ROLE_sale","ROLE_manager"})//    @PreAuthorize("hasAnyAuthority('admins')")@PostAuthorize("hasAnyAuthority('admin')")public String update(){    System.out.println("update....");    return "hello update";}
```

## @PostFilter

@PostFilter 方法返回数据进行过滤

```java
@GetMapping("getAll")@PreAuthorize("hasAnyAuthority('admins')")@PostFilter("filterObject.username == 'admin2'")public List<Admin> getAllAdmin(){    ArrayList<Admin> list = new ArrayList<>();    list.add(new Admin(1,"admin1","666"));    list.add(new Admin(2,"admin2","888"));    System.out.println(list);    return list;}
```

response返回[{"id":2,"username":"admin2","password":"888"}]

## @PreFilter

@PreFilter 进入控制器之前对数据进行过滤

首先在service层定义 方法，声明只有admins权限和username为admins的会过滤下来

```java
@PreFilter("hasAnyAuthority('admins') and filterObject.username=='admins'")public List<Admin> save(List<Admin> admins){    return admins;}
```

在controller层下写案例，调用service下的save方法

```java
@GetMapping("pre")@PreAuthorize("hasAnyAuthority('admins')")public List<Admin> save(){    ArrayList<Admin> admins = new ArrayList<>();    admins.add(new Admin(1,"zhangsan","111"));    admins.add(new Admin(2,"lisi","222"));    admins.add(new Admin(3,"admins","333"));    return myUserDetailsService.save(admins);}
```

浏览器返回

```html
[{"id":3,"username":"admins","password":"333"}]
```

# 用户注销

## 在登录界面添加一个退出

```java
.defaultSuccessUrl("/success.html").permitAll() //登录成功之后，跳转路径
```

```html
//success.html 界面登录成功！<a href="/logout">退出</a>
```

## 在配置类中添加一个退出映射

```java
//退出http.logout().logoutUrl("/logout").logoutSuccessUrl("/login.html").permitAll();
```

# 自动登录

## cookie技术

## 安全框架机制

### 实现原理

![image-20211007190437047](../posts/SpirngSecurity.assets/image-20211007190437047-16336046781901.png)

![image-20211007190505379](../posts/SpirngSecurity.assets/image-20211007190505379-16336047064442.png)

SCP17讲原理

### 具体实现

1. 创建数据库
2. 配置类，注入数据源，配置操作数据库对象

```java
//注入数据源@Autowiredprivate DataSource dataSource;@Beanpublic PersistentTokenRepository persistentTokenRepository(){    JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();    jdbcTokenRepository.setDataSource(dataSource);    //自动创建表    //        jdbcTokenRepository.setCreateTableOnStartup(true);    return jdbcTokenRepository;}
```

3. 配置类配置自动登录

```java
   //设置记住我.and().rememberMe().tokenRepository(persistentTokenRepository()).tokenValiditySeconds(60) //设置有效时长，单位秒.userDetailsService(userDetailsService)
```

4. 在登录页面绑定复选框

```html
记住我：<input type="checkbox" name="remember-me" title="记住密码"/>
```

# CSRF

跨站请求伪造

# SpringSecurity微服务权限方案

1.什么是微服务

2.微服务认证和授权实现过程

![image-20211007210137315](../../../../SpirngSecurity.assets/image-20211007210137315-16336116982773.png)

3.完成基于SpringSecurity认证授权案例

![image-20211007210151836](../../../../SpirngSecurity.assets/image-20211007210151836-16336117128874.png)


