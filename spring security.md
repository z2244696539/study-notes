[TOC]



## Spring Security原理

+ 本质是一个过滤器链
+ FilterSecurityInterceptor 是一个方法级的权限过滤器，基本位于过滤器最低部

## 默认密码配置方式

#### 配置yal文件设置

~~~yaml
  security:
    user:
      name: user
      password: user
~~~

#### 配置文件设置

+ 必须设置BCryptPasswordEncoder对象

~~~
/**
 * Created with IntelliJ IDEA.
 *
 * @Author: 爱吃山楂的小天
 * @Date: 2022/08/20/16:29
 * @Description:
 */
@Configuration
public class SpringSecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Bean
    BCryptPasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();

        auth.inMemoryAuthentication() // 内存验证
                .withUser("lucy") // 创建用户
                .password(encoder.encode("123456")) // 密码
                .roles("USER"); // 角色
    }
}

~~~



#### 自定义实现类设置

#### 不使用数据库

+ 创建配置类 设置使用那个userDetailService实现类

+ 编写实现类 返回user对象，user对象有用户名密码和操作权限

+ ~~~
  @Configuration
  public class MySpringSecurityConfig extends WebSecurityConfigurerAdapter {
  
      @Autowired
      private UserDetailsService userDetailsService;
  
      @Bean
      PasswordEncoder password() {
          return new BCryptPasswordEncoder();
      }
  
      @Override
      protected void configure(AuthenticationManagerBuilder auth) throws Exception {
          auth.userDetailsService(userDetailsService) //设置用户详情服务
                  .passwordEncoder(password()); //设置密码加密器
  
      }
  }
  ~~~

+ ~~~java
  /**
   * Created with IntelliJ IDEA.
   *
   * @Author: 爱吃山楂的小天
   * @Date: 2022/08/21/10:23
   * @Description:
   */
  @Service("userDetailsService")
  public class MyUserDetailService implements UserDetailsService {
      @Override
      public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
           return new User("lucy", new BCryptPasswordEncoder().encode("123456"),
                  AuthorityUtils.createAuthorityList("USER"));
      }
  }
  
  ~~~

##### 使用数据库

+ 实体类

+ ~~~java
  /**
   * <p>
   * 用户
   * </p>
   *
   * @author qy
   * @since 2019-11-08
   */
  @Data
  @ApiModel(description = "用户")
  @TableName("acl_user")
  public class User extends BaseEntity {
  
  	private static final long serialVersionUID = 1L;
  
  	@ApiModelProperty(value = "用户名")
  	@TableField("username")
  	private String username;
  
  	@ApiModelProperty(value = "密码")
  	@TableField("password")
  	private String password;
  
  	@ApiModelProperty(value = "昵称")
  	@TableField("nick_name")
  	private String nickName;
  
  	@ApiModelProperty(value = "用户头像")
  	@TableField("salt")
  	private String salt;
  
  	@ApiModelProperty(value = "用户签名")
  	@TableField("token")
  	private String token;
  }
  ~~~

+ 继承类

+ ~~~java
  package com.example.springsecurity.entity;
  
  import com.baomidou.mybatisplus.annotation.IdType;
  import com.baomidou.mybatisplus.annotation.TableField;
  import com.baomidou.mybatisplus.annotation.TableId;
  import com.baomidou.mybatisplus.annotation.TableLogic;
  import com.fasterxml.jackson.annotation.JsonFormat;
  import io.swagger.annotations.ApiModelProperty;
  import lombok.Data;
  
  import java.io.Serializable;
  import java.util.Date;
  import java.util.HashMap;
  import java.util.Map;
  
  @Data
  public class BaseEntity implements Serializable {
  
      @ApiModelProperty(value = "id")
      @TableId(type = IdType.AUTO)
      private Long id;
  
      @ApiModelProperty(value = "创建时间")
      @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
      @TableField("create_time")
      private Date createTime;
  
      @ApiModelProperty(value = "更新时间")
      @TableField("update_time")
      private Date updateTime;
  
      @ApiModelProperty(value = "逻辑删除(1:已删除，0:未删除)")
      @TableLogic
      @TableField("is_deleted")
      private Integer isDeleted;
  
      @ApiModelProperty(value = "其他参数")
      @TableField(exist = false)
      private Map<String,Object> param = new HashMap<>();
  }
  
  ~~~

+ UserDetailSerive实现类

+ ~~~java
  package com.example.springsecurity.service;
  
  import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
  import com.example.springsecurity.entity.User;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.security.core.GrantedAuthority;
  import org.springframework.security.core.authority.AuthorityUtils;
  import org.springframework.security.core.userdetails.UserDetails;
  import org.springframework.security.core.userdetails.UserDetailsService;
  import org.springframework.security.core.userdetails.UsernameNotFoundException;
  import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
  import org.springframework.stereotype.Service;
  import sun.util.calendar.Gregorian;
  
  import java.util.List;
  
  /**
   * Created with IntelliJ IDEA.
   *
   * @Author: 爱吃山楂的小天
   * @Date: 2022/08/21/10:23
   * @Description:
   */
  @Service("userDetailsService")
  public class MyUserDetailService implements UserDetailsService {
  
      @Autowired
      private UserService userService;
      @Override
      public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
  //         return new User("lucy", new BCryptPasswordEncoder().encode("123456"),
  //                AuthorityUtils.createAuthorityList("USER"));
          // 根据用户名查询用户信息
          LambdaQueryWrapper<com.example.springsecurity.entity.User> queryWrapper = new LambdaQueryWrapper<>();
          queryWrapper.eq(User::getUsername, s);
          User user = userService.getOne(queryWrapper);
          // 判断用户是否存在
          if (null == user){
              throw new UsernameNotFoundException("用户名不存在！");
          }
          // 返回UserDetails实现类
          return new org.springframework.security.core.userdetails.User(user.getUsername(), new BCryptPasswordEncoder().encode(user.getPassword()),
                  AuthorityUtils.createAuthorityList("USER"));
      }
  }
  
  ~~~

+ 配置类

+ ~~~java
  package com.example.springsecurity.config;
  
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
  import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
  import org.springframework.security.core.userdetails.UserDetailsService;
  import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
  import org.springframework.security.crypto.password.PasswordEncoder;
  
  /**
   * Created with IntelliJ IDEA.
   *
   * @Author: 爱吃山楂的小天
   * @Date: 2022/08/20/16:29
   * @Description:
   */
  @Configuration
  public class MySpringSecurityConfig extends WebSecurityConfigurerAdapter {
  
      @Autowired
      private UserDetailsService userDetailsService;
  
      @Bean
      PasswordEncoder password() {
          return new BCryptPasswordEncoder();
      }
  
      @Override
      protected void configure(AuthenticationManagerBuilder auth) throws Exception {
          auth.userDetailsService(userDetailsService) //设置用户详情服务
                  .passwordEncoder(password()); //设置密码加密器
  
      }
  }
  
  ~~~

## 设置自定义登录页面

+ 在配置类设置

+ ~~~java
      @Override
      protected void configure(HttpSecurity http) throws Exception {
          http.formLogin()
                  .loginPage("/login.html") //设置登录页面
                  .loginProcessingUrl("/user/login")  //设置登录请求处理的url
                  .defaultSuccessUrl("/test/index") //登录成功后访问的路径
                  .permitAll() //允许所有用户访问登录页面
                  .and()
                  .authorizeRequests() //配置权限
                  .antMatchers("/test/hello","/user/login").permitAll() //允许所有用户访问
                  .anyRequest().authenticated() //其他的都需要认证
                  .and().csrf().disable(); //关闭csrf
  
  
      }
  ~~~

+ ~~~html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
  </head>
  <body>
  <form action="/user/login" method="post">
      用户名:<input type="text" name="username" placeholder="username">
      <br>
      密码:<input type="password" name="password" placeholder="password">
      <br>
      <input type="submit" value="login">
  </form>
  </body>
  </html>
  ~~~

  

## 设置访问权限

#### hasAuthority

+ ~~~
                  .antMatchers("/test/index").hasAuthority("admin") //需要admin权限才能访问
  ~~~

#### hasAnyAuthority

~~~
                .antMatchers("/test/index").hasAnyAuthority("admin,manage,USER") //有其中角色一个就可以访问
~~~

#### hasRole

~~~java
                // hasRole 会给角色加上ROLE_ ROLE_USER
                .antMatchers("/test/index").hasRole("USER") //需要USER权限才能访问
~~~

#### hanAnyRole

同上

## 注解使用

#### **@Secured**

**使用注解前需要开启注解功能 @EnableGlobalMethodSecurity(securedEnabled = true)**

判断用户是否有这个角色，没有不能访问，

```java
@SpringBootApplication
@EnableGlobalMethodSecurity(securedEnabled = true)
public class SpringSecurityApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringSecurityApplication.class, args);
    }

}
```

在controller方法上设置角色

```java
@GetMapping("update")
@Secured({"ROLE_ADMIN","ROLE_MANAGE"})
public String update() {
    return "update";
}
```

####  @PreAuthorize

**使用注解前需要开启注解功能 @EnableGlobalMethodSecurity(securedEnabled = true，prePostEnabled = true)**

方法执行前进行权限验证

```java
    @GetMapping("index")
    @PreAuthorize("hasAuthority('USER')")
    public String index() {
        return "index";
    }
```

#### @PostAuthorize

方法执行后验证

没有权限也能访问

方法会执行但是执行完成后会验证权限

```
 @GetMapping("delete")
    @PostAuthorize("hasAuthority('user')")
    public String delete() {
        System.out.println("delete");
        return "delete";
    }
```

```
2022-08-22 10:39:39.742  INFO 20080 --- [nio-8080-exec-3] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
delete
```

![image-20220822104128671](C:\Users\22446\AppData\Roaming\Typora\typora-user-images\image-20220822104128671.png)

#### @PreFilter

传入数据过滤

```java
@GetMapping("getAll")
@Secured({"ROLE_ADMIN","ROLE_MANAGE"})
@PostFilter("filterObject.username=='a1'")
public List<User> getAll() {

    ArrayList<User> list = new ArrayList<>();
    User user = new User();
    user.setUsername("a1");
    User user1 = new User();
    user1.setUsername("a2");
    list.add(user);
    list.add(user1);
    return  list;
}
```

```json
[{"id":null,"createTime":null,"updateTime":null,"isDeleted":null,"param":{},"username":"a1","password":null,"nickName":null,"salt":null,"token":null}]
```

#### PostFilter

返回数据过滤

同上

## 退出登录

```java
http.logout() //注销
        .logoutUrl("/logout") //设置退出登录的url
        .logoutSuccessUrl("/test/hello") //退出成功后跳转的页面
        .permitAll(); //允许所有用户访问
```

## 自动登录

创建数据库

~~~sql
CREATE TABLE `persistent_logins`(

	`username` VARCHAR(64) NOT NULL,

	`series` VARCHAR(64) NOT NULL,

	`token` VARCHAR(64) NOT NULL,

	`last_used` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

	PRIMARY KEY(`series`)

) ENGINE=INNODB DEFAULT CHARSET=utf8;
~~~

配置数据源

```
//数据源
@Autowired
private DataSource dataSource;

@Bean
public PersistentTokenRepository repository(){

    JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();
    jdbcTokenRepository.setDataSource(dataSource);
        //   jdbcTokenRepository.setCreateTableOnStartup(true); 可自动生成表
    return jdbcTokenRepository;
}
```

添加配置

```
.and()
.rememberMe()
.tokenRepository(repository())
.tokenValiditySeconds(60) //设置有效时长
.userDetailsService(userDetailsService) 
```

加入复选框

name必须叫remember-me

```
<input type="checkbox" name="remember-me">自动登录
```