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
## Spring Security
### 相关函数
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