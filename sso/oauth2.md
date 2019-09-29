# OAuth2

## OAuth2简介

在认证与授权的过程中，主要包含以下3种角色：

l   服务提供方 Provider Server

Provider的角色被分为 Authorization Server（授权服务）和 Resource Service（资源服务）

l   资源持有者，即用户

l   客户端 Client

OAuth2的认证流程如图所示，具体如下：

![](https://github.com/xubinlee/Notes/blob/master/assets/oauth2.png?raw=true)                                                  

（1）用户（资源持有者）打开客户端 ，客户端询问用户授权。

（2）用户同意授权。

（3）客户端向授权服务器申请授权。

（4）授权服务器对客户端进行认证，也包括用户信息的认证，认证成功后授权给予令牌。

（5）客户端获取令牌后，携带令牌向资源服务器请求资源。

（6）资源服务器确认令牌正确无误，向客户端释放资源。

&emsp;&emsp;OAuth2 Provider 的角色被分为 Authorization Server（授权服务）和 Resource Service（资源服务），通常它们不在同一个服务中，可能一个 Authorization Service 对应多个 Resource Service。Spring OAuth2 需配合 Spring Security 一起使用，所有的请求由 Spring MVC 控制器处理，并经过一系列的Spring Security过滤器。

&emsp;&emsp;在Spring Security过滤器链中有以下两个节点，这两个节点是向 Authorization Service 获取验证和授权的。

l   授权节点：默认为 /oauth/authorize。

l   获取Token节点：默认为 /oauth/token。

## OAuth2快速开始

1. 新建本地数据库

&emsp;&emsp;客户端信息可以存储在数据库中，这样就可以通过更改数据库来实时更新客户端信息的数据。Spring OAuth2 已经设计好了数据库的表，且不可变。https://www.cnblogs.com/yueshutong/p/10272528.html#3857860655

2. 编写授权服务（Uaa）

   **1)**         **配置Spring Security**

   @EnableWebSecurity

   public class WebSecurityConfig extends WebSecurityConfigurerAdapter{}

   @Service

   public class UserServiceDetail implements UserDetailsService{}

   public class User implements UserDetails, Serializable {}

   public class Role implements GrantedAuthority {}

   **2)**          **配置Authorization Server**

   @EnableAuthorizationServer //开启授权服务的功能

   public class OAuth2AuthorizationConfig extends AuthorizationServerConfigurerAdapter {}

   **3)**         **暴露Remote Token Service接口**

   &emsp;&emsp;采用 RemoteTokenService 这种方式对 Token 进行验证，如果其他资源服务需要验证 Token，则需要远程调用授权服务暴露的验证 Token 的API接口

3. 编写资源服务

   **1)         配置Resource Server**

   @EnableResourceServer

   public class ResourceServerConfigurer extends ResourceServerConfigurerAdapter {}

   **2)         配置OAuth2 Client**

   @EnableOAuth2Client //开启OAuth2Client

   @EnableConfigurationProperties

   public class OAuth2ClientConfig {}

   **3)         编写用户注册接口**

 

## OAuth2授权方式

1. OAuth2在“客户端”与“服务提供商”之间设置了一个授权层（authorization layer）。“客户端”不能直接登录“服务提供商”，只能登录授权层，以此将用户和客户端分离。“客户端”登录需要OAuth提供的令牌

2. 令牌获取方式：OAuth2提供了四种授权方式

   一、  授权码模式（authorization code）

   步骤：

   a)          用户访问客户端，后者将前者导向认证服务器

   b)         用户选择是否给与客户端授权

   c)          假设用户给予授权，认证服务器将用户导向客户端事先指定的“重定向URI”，同时附上一个授权码

   d)         客户端收到授权码，附上早先的“重定向URI”向认证服务器申请令牌，这一步是在客户端的后台服务器上完成的，对用户不可见

   e)          认证服务器核对授权码和重定向URI，确认无误后向客户端发送令牌和更新令牌

   二、  简化模式（implicit）

   &emsp;&emsp;这种模式不通过服务器端程序来完成，直接由浏览器发送请求获取令牌，令牌是完全暴露在浏览器中的，这种模式极力不推崇。步骤：

   a)          客户端将用户导向认证服务器

   b)         用户决定是否给予客户端授权

   c)          假设用户给予授权，认证服务器将用户导向客户端指定的“重定向URI”，并在URI的Hash部分包含了访问令牌

   d)         浏览器向资源服务器发出请求，其中不包括上一步收到的Hash值

   e)          资源服务器返回一个网友，其中包含了代码可以获取Hash值中的令牌

   f)          浏览器执行上一步获得的脚本，获取令牌

   g)         浏览器将令牌发给客户端

   三、  密码模式（resource owner password credentials）

   &emsp;&emsp;用户向客户端提供自己的用户名和密码，客户端使用这些信息向“服务提供商”索要授权。步骤：

   a)          用户向客户端提供用户名和密码

   b)         客户端将用户名密码发送给认证服务器，请求令牌

   c)          认证服务器确认无误后，向客户端提供访问令牌

   四、  客户端模式（client credentials）

   &emsp;&emsp;客户端以自己的名义向“服务提供商”进行认证，在这种模式中，用户直接向客户端注册，客户端以自己名义要求“服务提供商”提供服务。步骤：

   a)          客户端向认证服务器进行身份认证，并要求一个访问令牌

   b)         认证服务器确认无误后，向客户端提供访问令牌