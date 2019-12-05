##### 服务器搭建

###### 1.安装javaSE 9

![Q3TGQg.png](https://s2.ax1x.com/2019/12/05/Q3TGQg.png)

###### 2.安装tomcat,推荐tomcat8.5.49

浏览器输入http://127.0.0.1:8080查看tomcat

![Q3TJyQ.png](https://s2.ax1x.com/2019/12/05/Q3TJyQ.png)

###### 3.github上下载cas 6.0.2版本

 https://github.com/apereo/cas/releases/tag/v6.0.2 

![Q3TUwn.png](https://s2.ax1x.com/2019/12/05/Q3TUwn.png)

###### 4.cas服务端其实就是一个war包

将下载的包解压后在moudles目录下找到 cas-server-webapp-4.0.0.war ，将其复制到tomcat目录下的webapps下。启动tomcat自动解压war包。浏览器输入http://127.0.0.1:8080/cas/login就可以看到登录页面

登录成功后跳转提示登录成功

![Q3jKGn.png](https://s2.ax1x.com/2019/12/05/Q3jKGn.png)
![Q3juPs.png](https://s2.ax1x.com/2019/12/05/Q3juPs.png)

  

##### 客户端配置

###### 1.创建Asp.Net .Net Framework 4.5项目

###### 2.安装DotNetCasClient包

###### 3.PM> install-package DotNetCasClient

###### 4.web.config配置 

```C#
configuration节点下
<configSections>
<section name="casClientConfig" type="DotNetCasClient.Configuration.CasClientConfiguration, DotNetCasClient"/>
</configSections>
<!--cas 配置-->
<casClientConfig
casServerLoginUrl="http://125.65.115.248:60003/cas/login"
casServerUrlPrefix="http://125.65.115.248:60003/cas"
serverName="http://118.112.183.197:8076"
notAuthorizedUrl="~/NotAuthorized.aspx"
cookiesRequiredUrl="~/CookiesRequired.aspx"
redirectAfterValidation="true"
renew="false"
singleSignOut="true"
ticketValidatorName="Cas20"
    proxyTicketManager="CacheProxyTicketManager" 
serviceTicketManager="CacheServiceTicketManager" />
system.web节点下
<httpModules>
<add name="DotNetCasClient" type="DotNetCasClient.CasAuthenticationModule,DotNetCasClient"/>
</httpModules>
 <!-- 解决多次重定向的问题-->  
<sessionState mode="StateServer" stateConnectionString="tcpip=127.0.0.1:42424" timeout="20" cookieless="false"/>
<authentication mode="Forms">
<forms loginUrl="http://125.65.115.248:60003/cas/login" timeout="30" defaultUrl="/Home/Index" cookieless="UseCookies" slidingExpiration="true" path="/"/>
</authentication>
system.webServer节点下
<modules runAllManagedModulesForAllRequests="true">
<remove name="DotNetCasClient"/>
<add name="DotNetCasClient" type="DotNetCasClient.CasAuthenticationModule,DotNetCasClient"/>
</modules>
```

###### 5.对于需要控制的action添加AuthorizationAttribute

这里是全局添加

```C#
 public class FilterConfig
    {
        public static void RegisterGlobalFilters(GlobalFilterCollection filters)
        {
            filters.Add(new HandleErrorAttribute());
            filters.Add(new AuthorizeAttribute());
        }
    }
```

###### 6.启动项目，输入用户名密码登录

![Q3TwF0.png](https://s2.ax1x.com/2019/12/05/Q3TwF0.png)

跳转

![Q3TBWT.png](https://s2.ax1x.com/2019/12/05/Q3TBWT.png)



###### 7.网上推荐的去除https方法，亲鉴没什么用。

cas默认使用https协议，需要向特定的机构申请或购买ssl安全证书。如果对安全性要求不高，或者在开发测试环境下就可以通过配置修改，让cas使用http协议

1）修改cas的WEB-INF/deployerConfigContext.xml
找到下面的配置

<bean class="org.jasig.cas.authentication.handler.support.HttpBasedServiceCredentialsAuthenticationHandler"
p:httpClient-ref="httpClient"/>

这里需要增加参数p:requireSecure="false"，requireSecure属性意思为是否需要安全验证，即HTTPS，false为不采用

2）修改cas的/WEB-INF/spring-configuration/ticketGrantingTicketCookieGenerator.xml
找到下面配置

<bean id="ticketGrantingTicketCookieGenerator" class="org.jasig.cas.web.support.CookieRetrievingCookieGenerator"

      p:cookieSecure="true"
    
      p:cookieMaxAge="-1"
    
      p:cookieName="CASTGC"
    
      p:cookiePath="/cas" />

参数p:cookieSecure="true"，同理为HTTPS验证相关，TRUE为采用HTTPS验证，FALSE为不采用https验证。

参数p:cookieMaxAge="-1"，是COOKIE的最大生命周期，-1为无生命周期，即只在当前打开的窗口有效，关闭或重新打开其它窗口，仍会要求验证。可以根据需要修改为大于0的数字，比如3600等，意思是在3600秒内，打开任意窗口，都不需要验证。

我们这里将cookieSecure改为false ,  cookieMaxAge 改为3600

3）修改cas的WEB-INF/spring-configuration/warnCookieGenerator.xml

找到下面配置

<bean id="warnCookieGenerator" class="org.jasig.cas.web.support.CookieRetrievingCookieGenerator"
p:cookieSecure="true"
p:cookieMaxAge="-1"
p:cookieName="CASPRIVACY"
p:cookiePath="/cas" />

 

我们这里将cookieSecure改为false ,  cookieMaxAge 改为3600
