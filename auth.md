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
### 相关配置
#### Protocol Endpoints
##### OAuth2 Authorization Endpoint
用于与资源拥有者(用户)交互并获取授权的节点，必须先对用户的身份进行验证。
在`OAuth2AuthorizationEndpointConfigurer`中定义了扩展点，可以自定义OAuth2 authorization requests的预处理、后处理和核心处理逻辑，其主要配置如下：
```java
@Bean
public SecurityFilterChain authorizationServerSecurityFilterChain(HttpSecurity http) throws Exception {
	OAuth2AuthorizationServerConfigurer authorizationServerConfigurer =
		new OAuth2AuthorizationServerConfigurer();
	http.apply(authorizationServerConfigurer);

	authorizationServerConfigurer
		.authorizationEndpoint(authorizationEndpoint ->
			authorizationEndpoint
				.authorizationRequestConverter(authorizationRequestConverter)  // (1)
				.authorizationRequestConverters(authorizationRequestConvertersConsumer) //(2)
				.authenticationProvider(authenticationProvider) //(3)
				.authenticationProviders(authenticationProvidersConsumer)  // (4)
				.authorizationResponseHandler(authorizationResponseHandler) //(5)
				.errorResponseHandler(errorResponseHandler) //(6)
				.consentPage("/oauth2/v1/authorize")    //(7)
		);

	return http.build();
}
```
1. 添加一个`AuthenticationConverter`，用于从`HttpServletRequest`中将一个Oauth2 authorization请求(或同意)提取为`OAuth2AuthorizationCodeRequestAuthenticationToken`或`OAuth2AuthorizationConsentAuthenticationToken`的实例时。
2. 设置提供对一系列默认和（可选）额外添加的 `AuthenticationConverter` 的访问的 Consumer，其允许添加、删除或自定义特定 `AuthenticationConverter`。
3. 添加用于验证 `OAuth2AuthorizationCodeRequestAuthenticationToken` 或 `OAuth2AuthorizationConsentAuthenticationToken` 的 `AuthenticationProvider` （核心处理器）
4. 设置提供对一系列默认和（可选）额外添加的 `AuthenticationProvider` 的访问的 Consumer，其允许添加、删除或自定义特定 `AuthenticationProvider`。
5. `AuthenticationSuccessHandler`（后处理器）用于处理“经过身份验证的”`OAuth2AuthorizationCodeRequestAuthenticationToken` 并返回 [OAuth2AuthorizationResponse](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1.2)。
6. `AuthenticationFailureHandler`（后处理器）用于处理 `OAuth2AuthorizationCodeRequestAuthenticationException` 并返回 OAuth2Error response
7. 如果在授权请求流程期间需要同意，则将资源所有者重定向到的自定义同意页面的 URI。

OAuth2AuthorizationEndpointConfigurer配置OAuth2AuthorizationEndpointFilter并将其注册到 OAuth2 授权服务器SecurityFilterChain @Bean。 OAuth2AuthorizationEndpointFilter是Filter处理 OAuth2 授权请求（和同意）的。
`OAuth2AuthorizationEndpointFilter`配置有以下默认值：

* `AuthenticationConverter` — 一个由`OAuth2AuthorizationCodeRequestAuthenticationConverter`和`OAuth2AuthorizationConsentAuthenticationConverter`组成的`DelegatingAuthenticationConverter`。
* `AuthenticationManager` — 一个由`OAuth2AuthorizationCodeRequestAuthenticationProvide`和`OAuth2AuthorizationConsentAuthenticationProvider`组成的`AuthenticationManager`。
* `AuthenticationSuccessHandler` — 处理“已验证”`OAuth2DeviceAuthorizationRequestAuthenticationToken`并返回`OAuth2DeviceAuthorizationResponse`.
* `AuthenticationFailureHandler` — 使用`OAuth2Error`与 `OAuth2AuthenticationException`关联并返回`OAuth2Error`响应的内部实现。

###### Customizing Authorization Request Validation
`OAuth2AuthorizationCodeRequestAuthenticationValidator`是用于验证授权代码授予中使用的特定 OAuth2 授权请求参数的默认验证器。默认实现验证redirect_uri和scope参数。如果验证失败，`OAuth2AuthorizationCodeRequestAuthenticationException`则会抛出异常。
`OAuth2AuthorizationCodeRequestAuthenticationProvider`提供通过向 `setAuthenticationValidator()` 提供 `Consumer<OAuth2AuthorizationCodeRequestAuthenticationContext>` 类型的自定义身份验证验证器的方式覆盖默认的验证器。
官方例子：允许在`redirect_uri`参数中使用`localhost`
```java
@Bean
public SecurityFilterChain authorizationServerSecurityFilterChain(HttpSecurity http) throws Exception {
	OAuth2AuthorizationServerConfigurer authorizationServerConfigurer =
		new OAuth2AuthorizationServerConfigurer();
	http.apply(authorizationServerConfigurer);

	authorizationServerConfigurer
		.authorizationEndpoint(authorizationEndpoint ->
			authorizationEndpoint
				.authenticationProviders(configureAuthenticationValidator())
		);

	return http.build();
}

private Consumer<List<AuthenticationProvider>> configureAuthenticationValidator() {
	return (authenticationProviders) ->
		authenticationProviders.forEach((authenticationProvider) -> {
			if (authenticationProvider instanceof OAuth2AuthorizationCodeRequestAuthenticationProvider) {
				Consumer<OAuth2AuthorizationCodeRequestAuthenticationContext> authenticationValidator =
					// Override default redirect_uri validator
					new CustomRedirectUriValidator()
						// Reuse default scope validator
						.andThen(OAuth2AuthorizationCodeRequestAuthenticationValidator.DEFAULT_SCOPE_VALIDATOR);

				((OAuth2AuthorizationCodeRequestAuthenticationProvider) authenticationProvider)
					.setAuthenticationValidator(authenticationValidator);
			}
		});
}

static class CustomRedirectUriValidator implements Consumer<OAuth2AuthorizationCodeRequestAuthenticationContext> {

	@Override
	public void accept(OAuth2AuthorizationCodeRequestAuthenticationContext authenticationContext) {
		OAuth2AuthorizationCodeRequestAuthenticationToken authorizationCodeRequestAuthentication =
			authenticationContext.getAuthentication();
		RegisteredClient registeredClient = authenticationContext.getRegisteredClient();
		String requestedRedirectUri = authorizationCodeRequestAuthentication.getRedirectUri();

		// Use exact string matching when comparing client redirect URIs against pre-registered URIs
		if (!registeredClient.getRedirectUris().contains(requestedRedirectUri)) {
			OAuth2Error error = new OAuth2Error(OAuth2ErrorCodes.INVALID_REQUEST);
			throw new OAuth2AuthorizationCodeRequestAuthenticationException(error, null);
		}
	}
}
```
**注意：**`OAuth2AuthorizationCodeRequestAuthenticationContext`持有`OAuth2AuthorizationCodeRequestAuthenticationToken`, 其中包括 OAuth2 authorization request 的参数.
##### OAuth2 Token Endpoint
用于让客户端利用授权或者refresh token获取access token的节点。
在`OAuth2TokenEndpointConfigurer`中定义了扩展点，可以自定义OAuth2 authorization requests的预处理、后处理和核心处理逻辑，其主要配置如下：
```java
@Bean
public SecurityFilterChain authorizationServerSecurityFilterChain(HttpSecurity http) throws Exception {
	OAuth2AuthorizationServerConfigurer authorizationServerConfigurer =
		new OAuth2AuthorizationServerConfigurer();
	http.apply(authorizationServerConfigurer);

	authorizationServerConfigurer
		.tokenEndpoint(tokenEndpoint ->
			tokenEndpoint
				.accessTokenRequestConverter(accessTokenRequestConverter)  // (1)
				.accessTokenRequestConverters(accessTokenRequestConvertersConsumer)// (2)
				.authenticationProvider(authenticationProvider) //(3)
				.authenticationProviders(authenticationProvidersConsumer)  // (4)
				.accessTokenResponseHandler(accessTokenResponseHandler) //(5)
				.errorResponseHandler(errorResponseHandler) //(6)
		);

	return http.build();
}
```
其配置说明与`OAuth2 Authorization Endpoint`基本一致。
##### OAuth2 Token Introspection Endpoint
参数是token，返回一个Json表示与token相关的元信息，包括当前是否有效（未过期，未撤销且有在受保护资源上introspection的权限）
```java
authorizationServerConfigurer
		.tokenIntrospectionEndpoint(tokenIntrospectionEndpoint ->
			tokenIntrospectionEndpoint
				.introspectionRequestConverter(introspectionRequestConverter)   
				.introspectionRequestConverters(introspectionRequestConvertersConsumer) 
				.authenticationProvider(authenticationProvider) 
				.authenticationProviders(authenticationProvidersConsumer)   
				.introspectionResponseHandler(introspectionResponseHandler) 
				.errorResponseHandler(errorResponseHandler) 
		);

```
##### OAuth2 Token Revocation Endpoint
用于对已授权token或者refresh token的撤销
```java
authorizationServerConfigurer
		.tokenRevocationEndpoint(tokenRevocationEndpoint ->
			tokenRevocationEndpoint
				.revocationRequestConverter(revocationRequestConverter) 
				.revocationRequestConverters(revocationRequestConvertersConsumer)   
				.authenticationProvider(authenticationProvider) 
				.authenticationProviders(authenticationProvidersConsumer)   
				.revocationResponseHandler(revocationResponseHandler)   
				.errorResponseHandler(errorResponseHandler) 
		);
```
##### OAuth2 Authorization Server Metadata Endpoint
用于获取认证服务器的相关参数,并以Json的形式返回
```java
authorizationServerConfigurer
		.authorizationServerMetadataEndpoint(authorizationServerMetadataEndpoint ->
			authorizationServerMetadataEndpoint
				.authorizationServerMetadataCustomizer(authorizationServerMetadataCustomizer));  
```
#### Core Model / Components
##### RegisteredClient
表示在授权服务器上注册的客户端。客户端必须先向授权服务器注册，然后才能启动授权流程。在客户端注册期间，客户端被分配一个唯一的客户端标识符client id、（可选）客户端密钥client secret（取决于客户端类型）以及与其唯一客户端标识符关联的元数据。客户端的元数据范围可以从向人类展示的显示字符串（例如客户端名称）到特定于协议流的项目（例如有效重定向 URI 的列表）。
* 与Spring Security’s OAuth2 Client中的`ClientRegistration`对应
```java
RegisteredClient registeredClient = RegisteredClient.withId(UUID.randomUUID().toString())
	.clientId("client-a")
	.clientSecret("{noop}secret") //(1)
	.clientAuthenticationMethod(ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
	.authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
	.redirectUri("http://127.0.0.1:8080/authorized")
	.scope("scope-a")
	.clientSettings(ClientSettings.builder().requireAuthorizationConsent(true).build())
	.build();
```
1. {noop} 代表Spring Security中`NoOpPasswordEncoder`的PasswordEncoder id ，其采用了明文密码
其包括以下属性
```java
public class RegisteredClient implements Serializable {
	private String id;  
	private String clientId;    
	private Instant clientIdIssuedAt;   
	private String clientSecret;    
	private Instant clientSecretExpiresAt;  
	private String clientName;  
	private Set<ClientAuthenticationMethod> clientAuthenticationMethods;    
	private Set<AuthorizationGrantType> authorizationGrantTypes;    
	private Set<String> redirectUris;   
	private Set<String> postLogoutRedirectUris; 
	private Set<String> scopes; 
	private ClientSettings clientSettings;  
	private TokenSettings tokenSettings;    
	...
}
```
##### RegisteredClientRepository
是可以注册新客户端，并查询已存在客户端的中心组件。当遵循特定的协议流时，它被其他组件使用，例如客户端身份验证、授权授予处理、令牌内省、动态客户端注册等。在开发过程中，常用`JdbcRegisteredClientRepository`，其使用JDBC持久存储实例。
该组件是必须的组件。
有两种注册方法,一个是通过bean的形式
```java
@Bean
public RegisteredClientRepository registeredClientRepository() {
	List<RegisteredClient> registrations = ...
	return new InMemoryRegisteredClientRepository(registrations);
}
```
或者通过`OAuth2AuthorizationServerConfigurer`配置
```java
@Bean
public SecurityFilterChain authorizationServerSecurityFilterChain(HttpSecurity http) throws Exception {
	OAuth2AuthorizationServerConfigurer authorizationServerConfigurer =
		new OAuth2AuthorizationServerConfigurer();
	http.apply(authorizationServerConfigurer);

	authorizationServerConfigurer
		.registeredClientRepository(registeredClientRepository);

	...

	return http.build();
}
```
##### OAuth2Authorization
`OAuth2Authorization` 是 OAuth2 授权的表示，它保存与资源所有者授予客户端的授权相关的状态，在 `client_credentials` 授权授予类型的情况下，它是资源所有者或自身的 OAuth2 授权的表示。
* 与Spring Security’s OAuth2 Client中的` OAuth2AuthorizedClient`对应
[当前进度](https://docs.spring.io/spring-authorization-server/docs/current/reference/html/core-model-components.html#registered-client-repository)








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
8. `DaoAuthenticationProvider`：`AuthenticationProvider`最最最常用的一个实现便是`DaoAuthenticationProvider`，它获取用户提交的用户名和密
码，比对其正确性，如果正确，返回一个数据库中的用户信息（假设用户信息被保存在数据库中）。
#### 注意事项
1. `Authentication`的`getCredentials()`与`UserDetails`中的`getPassword()`需要被区分对待，前者是用户提交的密码凭证，后者是用户正确的密码，认证器其实就是对这两者的比对。
### 常用注解
* `@PreAuthorize`:表示访问方法或类在执行之前先判断权限，方法或类级注解。参数为权限表达式
* `@PostAuthorize`：表示方法或类执行结束后判断权限，同上。
* 以上两个注解需要在启动类上开启`@EnableGlobalMethodSecurity(prePostEnabled = true)`
### 使用
#### 自定义用户及密码获取方式
1. 创建配置类，指定使用的userDetailsService实现类
2. 变现对应的实现类，实现以指定的用户名和密码的获取方式返回user
#### 自定义配置类
定义配置类继承`AuthorizationServerConfigurerAdapter`方法，并重写其三个configure方法，实现`ClientDetailsServiceConfigurer`,`AuthorizationServerSecurityConfigurer`和`AuthorizationServerEndpointsConfigurer`的配置。
![配置流程](res/auth_res/%E9%85%8D%E7%BD%AE%E6%B5%81%E7%A8%8B.png)
#### 注意事项
1. 基于Role访问，传入时需要前缀`ROLE_`

### 源码流程分析(https://blog.csdn.net/luoxiaomei999/category_11653125.html)