## [Spring Authorization Server](https://docs.spring.io/spring-authorization-server/docs/current/reference/html/)
提供了OAuth2.1和 OpenID Connect 1.0规范的实现。
### OAuth
[参考](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)
其参与者分为：
* RO (resource owner): 资源所有者，对资源具有授权能力的人。也就是登录用户。
* RS (resource server): 资源服务器，它存储资源，并处理对资源的访问请求。
* Client: 第三方应用，它获得RO的授权后便可以去访问RO的资源。
* AS (authorization server): 授权服务器，它认证RO的身份，为RO提供授权审批流程，并最终颁发授权令牌(Access Token)。
在物理上，AS与RS的功能可以由同一个服务器来提供服务

#### 授权模式
1. 授权码模式：通过客户端的后台服务器，与"服务提供商"的认证服务器进行互动。相关参数使用JSON发送，且HTTP头信息中明确指定不得缓存。
2. 简化模式：不通过第三方应用程序的服务器，直接在浏览器中向认证服务器申请令牌，跳过了"授权码"这个步骤。所有步骤在浏览器中完成，令牌对访问者是可见的，且客户端不需要认证。
3. 密码模式： 用户向客户端提供自己的用户名和密码。客户端使用这些信息，向"服务商提供商"索要授权。
4. 客户端模式：指客户端以自己的名义，而不是以用户的名义，向"服务提供商"进行认证。严格地说，客户端模式并不属于OAuth框架所要解决的问题

### 使用
[参考](https://www.appsdeveloperblog.com/spring-authorization-server-tutorial/)
1. 配置客户端凭证
```java
 @Bean
    public RegisteredClientRepository registeredClientRepository() {
        RegisteredClient oidcClient = RegisteredClient.withId(UUID.randomUUID().toString())
                .clientId("oidc-client")//Spring will use it to identify which client is trying to access the resource
                .clientSecret("{noop}secretSigen")//a secret known to the client and server that provides trust between the two
                .clientAuthenticationMethod(ClientAuthenticationMethod.CLIENT_SECRET_BASIC)//
                .authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
                .authorizationGrantType(AuthorizationGrantType.REFRESH_TOKEN)//a token to renew the authorization code
                .redirectUri("http://127.0.0.1:8080/login/oauth2/code/oidc-client")
                .postLogoutRedirectUri("http://127.0.0.1:8080/")
                .scope(OidcScopes.OPENID)
                .scope(OidcScopes.PROFILE)//this parameter defines authorizations that the client may have
                .clientSettings(ClientSettings.builder().requireAuthorizationConsent(true).build())
                .build();
        //对于返回值还有一种JDBCRegiteredClientRepository，可以使用save()方法保存客户端信息
        return new InMemoryRegisteredClientRepository(oidcClient);
    }
```
* ClientId、ClientSecret: 用于客户端向服务器进行身份验证，配置ClientSecret意味着将这个客户端作为confidential client，如果客户端是JS应用，可以不用ClientSecret，使用PKCE。
* clientAuthenticationMethod: 
    - CLIENT_SECRET_BASIC：使用ClientID和Client Secret，这两项存于HTTP请求中，被合并为一个字符串且base64编码
    - CLIENT_SECRET_POST：ClientID和Client Secret，这两项存于HTTP的post请求中
* authorizationGrantType：
    - AUTHORIZATION_CODE：授权码模式
* redirectUri：
* scope：取决于应用需要的功能，如read、write、prfile等
* clientSettings: 设置注册到服务上的客户端
    - requireAuthorizationConsent: 服务是否需要用户同意每个权限请求，选择false时，会跳过选择同意的界面，服务器会自动授予权限
    - requireProofKey: 服务是否需要为每个授权请求提供秘钥拥有证明，选择false时，服务器不会强制执行PKCE (Proof Key for Code Exchange)验证
2. 签署凭证的方法
```java
 @Bean
    public JWKSource<SecurityContext> jwkSource() {
        KeyPair keyPair = generateRsaKey();
        RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();
        RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();
        RSAKey rsaKey = new RSAKey.Builder(publicKey)
                .privateKey(privateKey)
                .keyID(UUID.randomUUID().toString())
                .build();
        JWKSet jwkSet = new JWKSet(rsaKey);
        return new ImmutableJWKSet<>(jwkSet);
    }

    private static KeyPair generateRsaKey() {
        KeyPair keyPair;
        try {
            KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("RSA");
            keyPairGenerator.initialize(2048);
            keyPair = keyPairGenerator.generateKeyPair();
        }
        catch (Exception ex) {
            throw new IllegalStateException(ex);
        }
        return keyPair;
    }
    
```
## [Spring Security](https://www.springcloud.cc/spring-security.html#overall-architecture)
### 相关成员
1. HttpSecurity
```java
http
    .authorizeHttpRequests(authorize ->//该方法用于制定哪些请求需要什么样的认证或授权
                    authorize
                    //其链式规则为：url匹配规则1+权限控制规则1  url匹配规则2+权限控制规则2 ...
                    //因此下述规则表示，匹配到的url任何人都可以访问， 其他的所有请求都需要验证
                        .requestMatchers("/assets/**", "/webjars/**", "/login")//传入待匹配的数组
                        .permitAll()//所匹配的url任何人都可以访问
                        .anyRequest()
                        .authenticated()//表示所匹配的URL都需要被认证才能访问
                )
                .formLogin(formLogin ->//登陆表单配置
                    formLogin
                        .loginPage("/login")
                );
```
* 权限控制方法
![Alt text](res/auth_res/%E6%9D%83%E9%99%90%E6%8E%A7%E5%88%B6%E6%96%B9%E6%B3%95.png)
* 登录登出配置
![Alt text](res/auth_res/%E7%99%BB%E5%BD%95%E7%99%BB%E5%87%BA%E9%85%8D%E7%BD%AE.png)
### [核心组件](https://baijiahao.baidu.com/s?id=1711889305762686065&wfr=spider&for=pc)
1. `SecurityContextHolder`：它持有的是安全上下文（security context）的信息。当前操作的用户是谁，该用户是否已经被认证，他拥有哪些角色权等等，这些都被保存在`SecurityContextHolde`r中。S`ecurityContextHolde`r默认使用ThreadLocal 策略来存储认证信息。看到ThreadLocal 也就意味着，这是一种与线程绑定的策略。
2. `SecurityContext`：安全上下文，主要持有Authentication对象，如果用户未鉴权，那Authentication对象将会是空的，可以使用SecurityContextHolder.getContext静态方法获取
3. `Authentication`：鉴权对象，该对象主要包含了用户的详细信息（UserDetails）和用户鉴权时所需要的信息，如用户提交的用户名密码、Remember-me Token，或者digest hash值等
4. `GrantedAuthority`：该接口表示了当前用户所拥有的权限（或者角色）信息。鉴权时并不会使用到该对象。
5. `UserDetails`：这个接口规范了用户详细信息所拥有的字段，譬如用户名、密码、账号是否过期、是否锁定等。获取当前登录的用户的信息,一般情况是需要在这个接口上面进行扩展，用来对接自己系统的用户
6. `UserDetailsService`：这个接口只提供一个接口`loadUserByUsername(String username)`，一般情况我们都是通过扩展这个接口来显示获取我们的用户信息，用户登录时传递的用户名和密码也是通过这里这查找出来的用户名和密码进行校验，但是真正的校验不在这里，而是由`AuthenticationManager`以及`AuthenticationProvider`负责的，需要强调的是，如果用户不存在，不应返回NULL，而要抛出异常UsernameNotFoundException.框架中自带一个 User 实现, 但是一般我们需要对 UserDetails 进行定制, 内置的 User 太过简单实际项目无法满足需要。
7. `AuthenticationManager`：是认证相关的核心接口，也是发起认证的出发点，一般不直接认证，其常用实现类`ProviderManager` 内部会维护一个`List<AuthenticationProvider>`列表，存放多种认证方式
8. `DaoAuthenticationProvider`：`AuthenticationProvider`最最最常用的一个实现便是`DaoAuthenticationProvider`，它获取用户提交的用户名和密码，比对其正确性，如果正确，返回一个数据库中的用户信息（假设用户信息被保存在数据库中）。
#### 注意事项
1. `Authentication`的`getCredentials()`与`UserDetails`中的`getPassword()`需要被区分对待，前者是用户提交的密码凭证，后者是用户正确的密码，认证器其实就是对这两者的比对。
### 常用注解
* `@PreAuthorize`:表示访问方法或类在执行之前先判断权限，方法或类级注解。参数为权限表达式
* `@PostAuthorize`：表示方法或类执行结束后判断权限，同上。
* 以上注解需要在启动类上开启`@EnableGlobalMethodSecurity(prePostEnabled = true)`
#### 自定义用户及密码获取方式
1. 创建配置类，指定使用的userDetailsService实现类
2. 变现对应的实现类，实现以指定的用户名和密码的获取方式返回user
