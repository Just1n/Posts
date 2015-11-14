<!--
Title|极简化实现AngularJS单页面通过Asp.Net WebApi token登陆二
Id|angular-minimal-login-with-token-by-aspnetwebapi-part2
Date|2015-11-14 10:22:00
Status|Publish
Type|Post
Tags|tech,AngularJS,WebApi
Excerpt|上一篇用最简单的一个方法实现了WebApi服务端代码，这篇我们依然用最简单的AngularJS代码实现客户端。
-->

本文源码：[源码下载][1]

Part1：[极简化实现AngularJS单页面通过Asp.Net WebApi token登陆一][2]

AngularJS是Google推出的一个优秀的MVVM JS框架，它支持模块化、自动化双向数据绑定、语义化标签、依赖注入等等。WebApi是微软推出的一个HTTP服务解决方案，它可以给客户端提供标准化的restful接口服务。OAuth是一个关于授权（authorization）的开放网络标准，在全世界得到广泛应用。本文就是用这三者实现的一个最简单登陆模型。

阅读本文，需要一些基本的[Asp.Net WebApi][3],[AngularJS][4]和[OAuth2.0][5]知识。

## 二、AngularJS客户端
### 1、下载第三方的库

 - AngularJS：我们用中国科技大学提供的[AngularJS CDN][6]镜像，版本是1.4.7
 - [Loading Bar][7]
 - [Bootstrap][8]

### 2、准备客户端
新建一个空的网站，取名`AngularAuthDemo.Web`，如图：
![此处输入图片的描述][9]
是项目的结构如图所示：
![此处输入图片的描述][10]

### 3、添加Layout（index.html)
代码如下：

    <!DOCTYPE html>
    <html data-ng-app="AngularAuthApp">
    <head>
        <meta content="IE=edge, chrome=1" http-equiv="X-UA-Compatible" />
        <title>AngularJS 权限验证</title>
        <link href="content/css/bootstrap.min.css" rel="stylesheet" />
        <link href="content/css/site.css" rel="stylesheet" />
        <link href="content/css/loading-bar.css" rel="stylesheet" />
        <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1" />
    </head>
    <body>
        <div class="navbar navbar-inverse navbar-fixed-top" role="navigation" data-ng-controller="indexController">
            <div class="container">
                <div class="navbar-header">
                    <button class="btn btn-success navbar-toggle" data-ng-click="navbarExpanded = !navbarExpanded">
                        <span class="glyphicon glyphicon-chevron-down"></span>
                    </button>
                    <a class="navbar-brand" href="#/">Home</a>
                </div>
                <div class="collapse navbar-collapse" data-collapse="!navbarExpanded">
                    <ul class="nav navbar-nav navbar-right">
                        <li data-ng-hide="!authentication.isAuth"><a href="#">Welcome {{authentication.userName}}</a></li>
                        <li data-ng-hide="!authentication.isAuth"><a href="#/orders">My Orders</a></li>
                        <li data-ng-hide="!authentication.isAuth"><a href="" data-ng-click="logOut()">Logout</a></li>
                        <li data-ng-hide="authentication.isAuth"> <a href="#/login">Login</a></li>
                        <li data-ng-hide="authentication.isAuth"> <a href="#/signup">Sign Up</a></li>
                    </ul>
                </div>
            </div>
        </div>
        <div class="jumbotron">
            <div class="container">
                <div class="page-header text-center">
                    <h1>AngularJS 登陆验证</h1>
                </div>
                <p>这是用AngularJS写的一个SPA（单页应用），通过bearer token与后台的Asp.Net Web Api验证。</p>
            </div>
        </div>
        <div class="container">
            <div data-ng-view="">
            </div>
        </div>
        <hr />
        <div id="footer">
            <div class="container">
                <div class="row">
                    <div class="col-md-6">
                        <p class="text-muted">Created by Just1n. Weibo: <a target="_blank" href="http://weibo.com/ccanochen">@Just1n</a></p>
                    </div>
                    <div class="col-md-6">
                        <p class="text-muted">Just1n's Post: <a target="_blank" href="http://just1n.net/">Just1n.net</a></p>
                    </div>
                </div>
            </div>
        </div>
        <!-- 3rd party libraries -->
        <script src="//ajax.lug.ustc.edu.cn/ajax/libs/angularjs/1.4.7/angular.min.js"></script>
        <script src="//ajax.lug.ustc.edu.cn/ajax/libs/angularjs/1.4.7/angular-route.min.js"></script>
        <script src="scripts/angular-local-storage.min.js"></script>
        <script src="scripts/loading-bar.min.js"></script>
        <!-- Load app main script -->
        <script src="app/app.js"></script>
        <!-- Load services -->
        <script src="app/services/authInterceptorService.js"></script>
        <script src="app/services/authService.js"></script>
        <script src="app/services/ordersService.js"></script>
        <!-- Load controllers -->
        <script src="app/controllers/indexController.js"></script>
        <script src="app/controllers/homeController.js"></script>
        <script src="app/controllers/loginController.js"></script>
        <script src="app/controllers/signupController.js"></script>
        <script src="app/controllers/ordersController.js"></script>
    </body>
    </html>
### 4、编辑app.js
代码如下：

    var app = angular.module('AngularAuthApp', ['ngRoute', 'LocalStorageModule', 'angular-loading-bar']);

    app.config(function ($routeProvider) {
    
        $routeProvider.when("/home", {
            controller: "homeController",
            templateUrl: "/app/views/home.html"
        });
    
        $routeProvider.when("/login", {
            controller: "loginController",
            templateUrl: "/app/views/login.html"
        });
    
        $routeProvider.when("/signup", {
            controller: "signupController",
            templateUrl: "/app/views/signup.html"
        });
    
        $routeProvider.when("/orders", {
            controller: "ordersController",
            templateUrl: "/app/views/orders.html"
        });
    
        $routeProvider.otherwise({ redirectTo: "/home" });
    });
    
    app.config(function ($httpProvider) {
        $httpProvider.interceptors.push('authInterceptorService');
    });
    
    app.run(['authService', function (authService) {
        authService.fillAuthData();
    }]);
代码里定义了4个route，分别对应不同的view（后面再加）。
### 5、添加Authentication服务
目前为止，我们已经有了最基本的view，但是还需要一个和Web Api交互的服务，正常情况下AngularJS里的服务大多是用来做这些事。在`services`文件夹下新建一个js文件，取名`authService`：

    'use strict';
    app.factory('authService', ['$http', '$q', 'localStorageService', function ($http, $q, localStorageService) {
    
        var serviceBase = 'http://localhost:yourport/';
        var authServiceFactory = {};
    
        var authentication = {
            isAuth: false,
            userName: ""
        };
    
        var saveRegistration = function (registration) {
    
            logOut();
    
            return $http.post(serviceBase + 'api/account/register', registration).then(function (response) {
                return response;
            });
    
        };
    
        var login = function (loginData) {
    
            var data = "grant_type=password&username=" + loginData.userName + "&password=" + loginData.password;
    
            var deferred = $q.defer();
    
            $http.post(serviceBase + 'token', data, { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } }).success(function (response) {
    
                localStorageService.set('authorizationData', { token: response.access_token, userName: loginData.userName });
    
                authentication.isAuth = true;
                authentication.userName = loginData.userName;
    
                deferred.resolve(response);
    
            }).error(function (err, status) {
                logOut();
                deferred.reject(err);
            });
    
            return deferred.promise;
    
        };
    
        var logOut = function () {
    
            localStorageService.remove('authorizationData');
    
            authentication.isAuth = false;
            authentication.userName = "";
    
        };
    
        var fillAuthData = function () {
    
            var authData = localStorageService.get('authorizationData');
            if (authData) {
                authentication.isAuth = true;
                authentication.userName = authData.userName;
            }
    
        }
    
        authServiceFactory.saveRegistration = saveRegistration;
        authServiceFactory.login = login;
        authServiceFactory.logOut = logOut;
        authServiceFactory.fillAuthData = fillAuthData;
        authServiceFactory.authentication = authentication;
    
        return authServiceFactory;
    }]);
我们把获取到的token用html5的localStorageService存储到本地。
### 6、添加注册、登录以及查看Orders的controller和view
此处略，可以查看源码。
### 7、添加拦截器
`Interceptor`是AngularJS提供的一个拦截器，它会抓取每一个XHR request，然后经过一番处理，再返回给后端API，在此处我们用来验证用户是否登陆，以及相应的业务逻辑。
在services文件夹下新建`authInterceptorService.js`：

    'use strict';
    app.factory('authInterceptorService', ['$q', '$location', 'localStorageService', function ($q, $location, localStorageService) {
    
        var authInterceptorServiceFactory = {};
    
        var request = function (config) {
    
            config.headers = config.headers || {};
    
            var authData = localStorageService.get('authorizationData');
            if (authData) {
                config.headers.Authorization = 'Bearer ' + authData.token;
            }
    
            return config;
        }
    
        var responseError = function (rejection) {
            if (rejection.status === 401) {
                $location.path('/login');
            }
            return $q.reject(rejection);
        }
    
        authInterceptorServiceFactory.request = request;
        authInterceptorServiceFactory.responseError = responseError;
    
        return authInterceptorServiceFactory;
    }]);
我们在每一个request上加上`bearer token`。如果出现错误，就返回到登陆页面。
### 8、添加index和home控制器以及视图
此处略，可看源码。
OK，至此，我们已经完成了全部的代码，可以看一下成果了：
![首页][11]
![登陆页][12]
![注册页][13]
![已登陆][14]


  [1]: https://github.com/Just1n/AngularAuthByAspNetWebApiDemo
  [2]: http://just1n.net/2015/11/angular-minimal-login-with-token-by-aspnetwebapi-part1
  [3]: http://www.asp.net/web-api
  [4]: https://angularjs.org/
  [5]: http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html
  [6]: http://ajax.lug.ustc.edu.cn/ajax/libs/angularjs/1.4.7/angular.min.js
  [7]: http://chieffancypants.github.io/angular-loading-bar/
  [8]: http://getbootstrap.com/
  [9]: http://ww3.sinaimg.cn/large/655f1de0gw1ey0uad2tp1j20rc0lbwj9.jpg
  [10]: http://ww3.sinaimg.cn/large/655f1de0gw1ey0ugnnpmxj20c00hhtb8.jpg
  [11]: http://ww1.sinaimg.cn/large/655f1de0gw1ey0w50jg7wj20x00fagnr.jpg
  [12]: http://ww3.sinaimg.cn/large/655f1de0gw1ey0w59qpvij20t20heabl.jpg
  [13]: http://ww3.sinaimg.cn/large/655f1de0gw1ey0w5iqc1cj20sq0g3abp.jpg
  [14]: http://ww3.sinaimg.cn/large/655f1de0gw1ey0w5trbgyj20zy0ifjtz.jpg
