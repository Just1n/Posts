<!--
Title|极简化实现AngularJS单页面通过Asp.Net WebApi token登陆一
Id|angular-minimal-login-with-token-by-aspnetwebapi-part1
Date|2015-11-14 20:12:00
Status|Publish
Type|Post
Tags|tech,AngularJS,WebApi
Excerpt|AngularJS是Google推出的一个优秀的MVVM JS框架，它支持模块化、自动化双向数据绑定、语义化标签、依赖注入等等。WebApi是微软推出的一个HTTP服务解决方案，它可以给客户端提供标准化的restful接口服务。OAuth是一个关于授权（authorization）的开放网络标准，在全世界得到广泛应用。本文就是用这三者实现的一个最简单登陆模型。
-->

AngularJS是Google推出的一个优秀的MVVM JS框架，它支持模块化、自动化双向数据绑定、语义化标签、依赖注入等等。WebApi是微软推出的一个HTTP服务解决方案，它可以给客户端提供标准化的restful接口服务。OAuth是一个关于授权（authorization）的开放网络标准，在全世界得到广泛应用。本文就是用这三者实现的一个最简单登陆模型。

阅读本文，需要一些基本的[Asp.Net WebApi][1],[AngularJS][2]和[OAuth2.0][3]知识。

## 一、WebApi服务端
### 1、新建一个空的WebApi项目
本文所用的环境是VS2015和SQL Server2014，所选的.Net Framework版本是4.5。
新建一个空解决方案，取名`AngularAuthenticDemo`,再在此解决方案下新建一个空的WebApi项目，取名`AngularAuthDemo.Api`：
![此处输入图片的描述][4]
### 2、安装需要的Nuget包

    Install-Package Microsoft.AspNet.WebApi.Owin
    Install-Package Microsoft.Owin.Host.SystemWeb
前者是设置Owin服务所需要的，后者是把我们这个Api服务Host在IIS Express上。
### 3、在根目录下添加`Startup`类
每个OWIN应用程序都需要一个Startup类作为OWIN管道中的配置类，`Startup`代码如下：

    using System.Web.Http;
    using Microsoft.Owin;
    using Owin;

    [assembly: OwinStartup(typeof(AngularAuthDemo.Api.Startup))]
    namespace AngularAuthDemo.Api
    {
        public class Startup
        {
            public void Configuration(IAppBuilder app)
            {
                HttpConfiguration config = new HttpConfiguration();
                WebApiConfig.Register(config);
                app.UseWebApi(config);
            }
        }
    }
此时的`App_Start`文件夹下的`WebApiConfig`类的代码应如下所示：

    using System.Linq;
    using System.Net.Http.Formatting;
    using System.Web.Http;
    using Newtonsoft.Json.Serialization;
    
    namespace AngularAuthDemo.Api
    {
        public static class WebApiConfig
        {
            public static void Register(HttpConfiguration config)
            {
                // Web API 配置和服务
    
                // Web API 路由
                config.MapHttpAttributeRoutes();
    
                config.Routes.MapHttpRoute(
                    name: "DefaultApi",
                    routeTemplate: "api/{controller}/{id}",
                    defaults: new { id = RouteParameter.Optional }
                );
    
                var jsonFormatter = config.Formatters.OfType<JsonMediaTypeFormatter>().First();
                jsonFormatter.SerializerSettings.ContractResolver = new CamelCasePropertyNamesContractResolver();
            }
        }
    }
后面两行代码是让服务返回驼峰语义化的JSON格式。
此时，就可以把原来的`Global.asax`文件删除了。
### 4、添加ASP.NET Identity验证系统
添加程序包：

    Install-Package Microsoft.AspNet.Identity.Owin
    Install-Package EntityFramework
    Install-Package Microsoft.AspNet.Identity.EntityFramework
这两个包是为了让我们用微软的EntityFramework来操作数据库。

在根目录下添加`DbContext`类，取名`AuthContext`，代码如下：

    using Microsoft.AspNet.Identity.EntityFramework;

    namespace AngularAuthDemo.Api
    {
        public class AuthContext : IdentityDbContext<IdentityUser>
        {
            public AuthContext() : base("AuthConnString")
            {
    
            }
        }
    }
这个类继承`IdentityDbContext`，提供了大部分`ASP.NET Identity`需要的函数。
下面要定义一个用户的模型，在`Models`文件夹下新建类`User`，代码如下：

    using System.ComponentModel.DataAnnotations;

    namespace AngularAuthDemo.Api.Models
    {
        public class User
        {
            [Required]
            [Display(Name = "User name")]
            public string UserName { get; set; }
    
            [Required]
            [StringLength(100, ErrorMessage = "The {0} must be at least {2} characters long.", MinimumLength = 6)]
            [DataType(DataType.Password)]
            [Display(Name = "Password")]
            public string Password { get; set; }
    
            [DataType(DataType.Password)]
            [Display(Name = "Confirm password")]
            [Compare("Password", ErrorMessage = "The password and confirmation password do not match.")]
            public string ConfirmPassword { get; set; }
        }
    }
定义`connectionStrings`：

    <connectionStrings>
        <add name="AuthConnString" connectionString="Data Source=.\;Initial Catalog=AngularAuthDemo;Integrated Security=SSPI;User ID=sa;Password=password;" providerName="System.Data.SqlClient" />
    </connectionStrings>
这里我连的是本机的SQL Server，可以换成Sql Express或者Sql LocalDb。
### 5、添加ASP.NET Identity的Repository类
在根目录下新建一个`AuthRepository`类，代码如下：

    using System;
    using System.Threading.Tasks;
    using AngularAuthDemo.Api.Models;
    using Microsoft.AspNet.Identity;
    using Microsoft.AspNet.Identity.EntityFramework;
    
    namespace AngularAuthDemo.Api
    {
        public class AuthRepository : IDisposable
        {
            private readonly AuthContext _ctx;
    
            private readonly UserManager<IdentityUser> _userManager;
    
            public AuthRepository()
            {
                _ctx = new AuthContext();
                _userManager = new UserManager<IdentityUser>(new UserStore<IdentityUser>(_ctx));
            }
    
            public async Task<IdentityResult> RegisterUser(User userModel)
            {
                IdentityUser user = new IdentityUser
                {
                    UserName = userModel.UserName
                };
    
                var result = await _userManager.CreateAsync(user, userModel.Password);
    
                return result;
            }
    
            public async Task<IdentityUser> FindUser(string userName, string password)
            {
                IdentityUser user = await _userManager.FindAsync(userName, password);
    
                return user;
            }
    
            public void Dispose()
            {
                _ctx.Dispose();
                _userManager.Dispose();
    
            }
        }
    }
### 6、添加“Account” Controller
在`Controllers`文件夹下新建一个空的Web Api2控制器，名字叫`AccountController`：

    using System.Threading.Tasks;
    using System.Web.Http;
    using AngularAuthDemo.Api.Models;
    using Microsoft.AspNet.Identity;
    
    namespace AngularAuthDemo.Api.Controllers
    {
        [RoutePrefix("api/Account")]
        public class AccountController : ApiController
        {
            private readonly AuthRepository _repo;
    
            public AccountController()
            {
                _repo = new AuthRepository();
            }
    
            // POST api/Account/Register
            [AllowAnonymous]
            [Route("Register")]
            public async Task<IHttpActionResult> Register(User userModel)
            {
                if (!ModelState.IsValid)
                    return BadRequest(ModelState);
    
                var result = await _repo.RegisterUser(userModel);
    
                var errorResult = GetErrorResult(result);
    
                return errorResult ?? Ok();
            }
    
            protected override void Dispose(bool disposing)
            {
                if (disposing)
                    _repo.Dispose();
    
                base.Dispose(disposing);
            }
    
            private IHttpActionResult GetErrorResult(IdentityResult result)
            {
                if (result == null)
                    return InternalServerError();
    
                if (result.Succeeded) return null;
                if (result.Errors != null)
                {
                    foreach (var error in result.Errors)
                    {
                        ModelState.AddModelError("", error);
                    }
                }
    
                if (ModelState.IsValid)
                {
                    // 如果没有错误，就直接返回BadRequest
                    return BadRequest();
                }
    
                return BadRequest(ModelState);
            }
        }
    }
`AccountController`提供了一个`Register`Action，供用户注册。`AllowAnonymous`属性，确保任何人都可以通过`http://localhost:port/api/account/register`注册。**现在已经可以用[Postman][5]来测试了**（略）。如果返回code 200，那么会自动在数据库里创建如下表：
![此处输入图片的描述][6]
### 7、添加一个必须由已登陆用户才能访问的Orders Controller
在`Controllers`文件夹下新建空的Controller，取名`OrdersController`，代码如下：

    using System;
    using System.Collections.Generic;
    using System.Web.Http;
    
    namespace AngularAuthDemo.Api.Controllers
    {
        [RoutePrefix("api/Orders")]
        public class OrdersController : ApiController
        {
            [Authorize]
            [Route("")]
            public IHttpActionResult Get()
            {
                return Ok(Order.CreateOrders());
            }
        }
    
        public class Order
        {
            public int OrderId { get; set; }
            public string CustomerName { get; set; }
            public string ShipperCity { get; set; }
            public Boolean IsShipped { get; set; }
    
            public static List<Order> CreateOrders()
            {
                var orderList = new List<Order>
                {
                    new Order {OrderId = 10248, CustomerName = "Han Meimei", ShipperCity = "Shanghai", IsShipped = true },
                    new Order {OrderId = 10249, CustomerName = "Li Lei", ShipperCity = "Beijing", IsShipped = false},
                    new Order {OrderId = 10250,CustomerName = "Zhang San", ShipperCity = "Nanjing", IsShipped = false },
                    new Order {OrderId = 10251,CustomerName = "Wang Ermazi", ShipperCity = "Suzhou", IsShipped = false},
                    new Order {OrderId = 10252,CustomerName = "Li Si", ShipperCity = "Nanchang", IsShipped = true}
                };
    
                return orderList;
            }
        }
    }
`Authorize`特性确保了必须只有已经登陆验证的用户才能访问`http://localhost:port/api/orders`，否则会返回Http status code 401。
### 8、添加OAuth Bearer Tokens功能
目前为止我们已经完成了最基本的注册功能，下面我们添加token登陆功能。安装Nuget包：

    Install-Package Microsoft.Owin.Security.OAuth
安装完之后，再在`Startup`里增加一个`ConfigureOAuth`函数，全部代码如下：

    
    [assembly: OwinStartup(typeof(AngularAuthDemo.Api.Startup))]
    namespace AngularAuthDemo.Api
    {
        public class Startup
        {
            public void Configuration(IAppBuilder app)
            {
                ConfigureOAuth(app);
                var config = new HttpConfiguration();
                WebApiConfig.Register(config);
                app.UseWebApi(config);
            }
    
            public void ConfigureOAuth(IAppBuilder app)
            {
                var oAuthServerOptions = new OAuthAuthorizationServerOptions()
                {
                    AllowInsecureHttp = true,
                    TokenEndpointPath = new PathString("/token"),
                    AccessTokenExpireTimeSpan = TimeSpan.FromMinutes(30),
                    Provider = new SimpleAuthorizationServerProvider()
                };
                app.UseOAuthAuthorizationServer(oAuthServerOptions);
                app.UseOAuthBearerAuthentication(new OAuthBearerAuthenticationOptions());
            }
        }
    }
上述代码实现功能如下：

 - 通过`http://localhost:port/token`获取一个token。
 - token的有效时间是30分钟。
 - 通过`SimpleAuthorizationServerProvider`来验证用户信息。
 
### 9、实现SimpleAuthorizationServerProvider
新建文件夹`Providers`,在该文件夹下新建类`SimpleAuthorizationServerProvider`，代码如下：

        using System.Security.Claims;
        using System.Threading.Tasks;
        using Microsoft.Owin.Security.OAuth;
    
        namespace AngularAuthDemo.Api.Providers
        {
            public class SimpleAuthorizationServerProvider : OAuthAuthorizationServerProvider
            {
                public override async Task ValidateClientAuthentication(OAuthValidateClientAuthenticationContext context) => context.Validated();
        
                public override async Task GrantResourceOwnerCredentials(OAuthGrantResourceOwnerCredentialsContext context)
                {
        
                    context.OwinContext.Response.Headers.Add("Access-Control-Allow-Origin", new[] { "*" });
        
                    using (var repo = new AuthRepository())
                    {
                        var user = await repo.FindUser(context.UserName, context.Password);
        
                        if (user == null)
                        {
                            context.SetError("invalid_grant", "The user name or password is incorrect.");
                            return;
                        }
                    }
        
                    var identity = new ClaimsIdentity(context.Options.AuthenticationType);
                    identity.AddClaim(new Claim("sub", context.UserName));
                    identity.AddClaim(new Claim("role", "user"));
        
                    context.Validated(identity);
        
                }
            }
        }
第一个函数，是验证客户端，在这个例子中，我们只有一个客户端（Angular），所以我们直接validated。
第二个函数，我们用AuthRepository来验证用户的用户名和密码，一旦验证成功，就新建一个`ClaimsIdentity`类，把一些验证信息放里面（这里我们后面将会用`bearer token`），至于`ClaimsIdentity`是什么东东，可以看这篇[文章][7]
`context.OwinContext.Response.Headers.Add("Access-Control-Allow-Origin", new[] { "*" });`语句是为了给[CORS][8]使用。

### 10、允许跨域访问（CORS）ASP.NET Web API
安装Nuget包：

    Install-Package Microsoft.Owin.Cors
把`app.UseCors(Microsoft.Owin.Cors.CorsOptions.AllowAll);`加在`Startup`的`Configuration`函数里：

    public void Configuration(IAppBuilder app)
        {
            HttpConfiguration config = new HttpConfiguration();

            ConfigureOAuth(app);

            WebApiConfig.Register(config);
            app.UseCors(Microsoft.Owin.Cors.CorsOptions.AllowAll);
            app.UseWebApi(config);

        }

OK,至此我们已经完成了Asp.Net WebApi服务端的开发，可以通过[Postman][5]来测试是否成功。
下一篇，是用AngularJS写客户端。

  [1]: http://www.asp.net/web-api
  [2]: https://angularjs.org/
  [3]: http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html
  [4]: http://ww3.sinaimg.cn/large/655f1de0gw1ey0n78ji2bj20rc0lbn23.jpg
  [5]: https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?utm_source=chrome-app-launcher-info-dialog
  [6]: http://ww3.sinaimg.cn/large/655f1de0gw1ey0r3iiio1j207305y0tn.jpg
  [7]: http://www.cnblogs.com/jesse2013/p/aspnet-identity-claims-based-authentication-and-owin.html
  [8]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS
