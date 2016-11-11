#AngularJS学习笔记

#MVC
##特点
 - 模块划分
 - 复用性
 - 维护方便：功能独立 --> 解耦 依赖 

----------
###典型的controller实现方式：
把通用的逻辑做成一个**服务**，然后用控制器调用而不是继承这个公共的东西
![][1]


----------

##controller使用过程中的注意点：
1. 不要试图去**复用controller**,一个控制器一般只负责一小块视图
2. 不要在controller中**操作dom**，这不是控制器的职责（控制器中操作dom会导致浏览器重绘或重新布局，是非常昂贵的；dom操作应该放在**指令（的link方法）**里面）
3. 不要在controller里面做**数据格式化**，ng有很好的表单控件
4. 不要在controller中做**数据过滤操作**，ng有$filter服务
5. 一般来说，controller是不会互相调用的，控制器之间的交互通过scope或者数据模型上的事件进行交互，controller在内部监控事件完成


----------


##$scope

- *\$scope是一个POJP （Plain Old JavaScript Object）
- \$scope提供了一些工具方法\$watch()/\$apply()
- \$scope是表达式的执行环境(或者叫作用域)
- \$scope是一个树形结构，**与DOM标签平行**
- 可以用angular.element(\$0).scope()进行调试
- 子\$scope对象会继承父\$scope上的**属性和方法**
- 每一个Angular应用只有一根\$scope对象(一般位于ng-app上)
- \$scope可以传播事件，类似DOM事件，可以向上也可以向下
- \$scope不仅是MVC的基础，也是后面实现双向数据绑定的基础
- angular.element(\$0).scope()调试scope

----------


#angularJS

##AngularJS四大核心特性
1. MVC
2. 模块化和依赖注入
3. 双向数据绑定
4. 指令


----------
###MVC: AngularJS的 MVC 是借助于 $scope 实现的

###模块化：划分module
```
var myModule1 = angular.module("MyModule1", []);
var myModule2 = angular.module("MyModule2", []);
...
```
####什么是Module
- ⼤大部分应⽤用都有⼀一个主⽅方法(main)⽤用来实例化、组织、启动应⽤用。
- AngularJS应⽤用没有主⽅方法，⽽而是使⽤用模块来声明应⽤用应该如何启动。
- 模块允许通过声明的⽅方式来描述应⽤用中的依赖关系，以及如何进⾏行组装和启动
- 模块是组织业务的一个框框，在一个模块当中定义多个服务。当引入了一个模块的时候，就可以使⽤用这个模块提供的一种或多种服务了。
- AngularJS 本⾝身的⼀一个默认模块叫做ng ，它提供了\$http ，\$scope等等服务
- 服务只是模块提供的多种机制中的⼀一种，其它的还有指令（directive ），过滤器（ filter ），及其它配置信息。
- 模块可以以任何先后或者并⾏行的顺序加载（因为模块的执⾏行本⾝身是延迟的）。

###依赖注入：
```
var myModule = angular.module("MyModule", ['MyModule1','MyModule2',...]);
```
MyModule依赖于MyModule1,MyModule2...
> MyModule内部功能的实现需要依赖于后续的模块所提供的内容

反过来，MyModule1,MyModule2...被注入了MyModule
> 可以理解为MyModule1,MyModule2...作为参数传递进入了MyModule

*Angular的依赖注⼊只是简单的获取它所需要的东⻄，⽽不需要创建那些他们所依赖的东⻄*

##$provide

- angular 是用\$provide对象来实现自动依赖注入机制，注入机制通过调用一个 provider的\$get()方法，把返回的对象作为被注入函数的参数进行相关调用

    ```
    var myApp = angular.module('myApp',[],function($provide){
        // 自定义服务
        $provide.provider('CustomService',function(){
            this.$get = function(){
                return {
                    message : 'CustomService Message'
                }
            }
        });
    });
    
    myApp.controller('firstController',function(CustomService,$scope,CustomService){//注入CustomService
    $scope.name = '张三';
    console.log(CustomService);// {message:'CustomService Message'
});
    ```
    
- \$provider.provider 是一种定义服务的方法, $provider还提供了很多很简便的方法,这些简便的方法还直接被module所引用


###\$provider.factory
- factory 方法直接把一个函数当成是一个对象的 $get() 方法
- 返回的内容可以是**任何类型**

###\$provider.service
- 和factory类似，但返回的东西**必须是对象**
```
//自定义工厂
$provide.factory('CustomFactory',function(){
    return [1,2,3,4,5,6,7];
});

// 自定义服务
$provide.service('CustomService2',function(){
    return {};
})
```
*\*也可以在module内直接调用factory和service*
```
myApp.factory('CustomFactory',function(){
    return [1,2,3,4,5,6,7];
});
myApp.service('CustomService2',function(){
    return {};
});
```

###双向绑定
在修改输入域的值时， AngularJS 属性的值也将修改：
```
<div ng-app="myApp" ng-controller="myCtrl">
    名字: <input ng-model="name">
    <h1>你输入了: {{name}}</h1>
</div>
```
####angular是怎么知道变量发⽣了改变：使⽤**脏检查**
- angular的策略
    1. 不会脏检查所有的对象，当对象被绑定到html中，这个
    对象添加为检查对象（watcher）。
    2. 不会脏检查所有的属性，同样当属性被绑定后，这个
    属性会被列为检查的属性。

- 在angular程序初始化时，会将绑定的对象的属性添加为
监听对象（watcher），也就是说⼀个对象绑定了N个属
性，就会添加N个watcher。

- angular 所系统的⽅法中都会触发⽐较事件，⽐如：
controller 初始化的时候，所有以ng-开头的事件执⾏后，都会触发脏检查
- ⼿动触发脏检查
    - \$apply仅仅只是进⼊angular context ,然后通过\$digest去
触发脏检查 
    - \$digest()所属的scope和其所有⼦scope的脏检查，脏检查⼜会触发\$watch()，整个Angular双向绑定机制就活了起来
    - \$watch
        - 在digest执⾏时，如果watch观察的value与上次执⾏时不⼀
    样时，就会被触发
        - AngularJS内部的watch实现了⻚⾯随model的及时更新
        - $watch(watchFn, watchAction, deepWatch)
            watchFn:angular表达式或函数的字符串，需要被watch的目标
            watchAction(newValue,oldValue,scope):watch目标发⽣变化会被调⽤
            deepWacth:可选的布尔值命令检查被监控的Object（如果被watch的目标是Object时）的每个属性是否发⽣变化

----------
##AngularJS中的$scope
###AngularJS 应用组成如下：
- View(视图), 即 HTML。
- Model(模型), 当前视图中可用的数据。
- Controller(控制器), 即 JavaScript 函数，可以添加或修改属性。

###scope 是模型
- Scope(作用域) 是应用在 HTML (视图) 和 JavaScript (控制器)之间的纽带。
- Scope 是一个对象，有可用的方法和属性。
- Scope 可应用在视图和控制器上

###根作用域
- 所有的应用都有一个 \$rootScope，它可以作用在 ng-app 指令包含的所有 HTML 元素中。
- \$rootScope 可作用于整个应用中。是各个 controller 中 scope 的桥梁。用 rootscope 定义的值，可以在各个 controller 中使用。


----------



----------
##路由
AngularJS 路由 通过 **# + 标记** 帮助我们区分不同的逻辑页面并将不同的页面绑定到对应的控制器上。
###实例
```
<body ng-app='routingDemoApp'>
    <ul>
        <li><a href="#/">首页</a></li>
        <li><a href="#/computers">电脑</a></li>
        <li><a href="#/printers">打印机</a></li>
        <li><a href="#/blabla">其他</a></li>
    </ul>
    <!-- 使用 ngView 指令。该 div 内的 HTML 内容会根据路由的变化而变化 -->
    <div ng-view></div>
    
    <script src="http://apps.bdimg.com/libs/angular.js/1.4.6/angular.min.js"></script>
    <!-- 载入实现路由的 js 文件：angular-route.js。-->
    <script src="http://apps.bdimg.com/libs/angular-route/1.3.13/angular-route.js"></script>
    
    <script>
        //包含 ngRoute 模块作为主应用模块的依赖模块。
        angular.module('routingDemoApp',['ngRoute'])
        //配置 $routeProvider，AngularJS $routeProvider 用来定义路由规则。
        .config(['$routeProvider', function($routeProvider){
            $routeProvider
            .when('/',{template:'这是首页页面'})
            .when('/computers',{template:'这是电脑分类页面'})
            .when('/printers',{template:'这是打印机页面'})
            .otherwise({redirectTo:'/'});
        }]);
    </script>
</body>
```
- AngularJS 模块的 config 函数用于配置路由规则。通过使用 configAPI，我们请求把\$routeProvider **注入**到我们的配置函数并且使用\$routeProvider.whenAPI来定义我们的路由规则。
- \$routeProvider 为我们提供了 when(path,object) & otherwise(object) 函数按顺序定义所有路由，函数包含两个参数:
    - 第一个参数是 URL 或者 URL 正则规则。
    - 第二个参数是**路由配置对象**。

- 路由设置对象
AngularJS 路由也可以通过不同的模板来实现。
路由配置对象语法规则如下：
```
$routeProvider.when(url, {
    template: string,
    templateUrl: string,
    controller: string, function 或 array,
    controllerAs: string,
    redirectTo: string, function,
    resolve: object<key, function>
});
```

- 参数说明：
    - template:
    如果我们只需要在 ng-view 中插入简单的 HTML 内容，则使用该参数：
    `.when('/computers',{template:'这是电脑分类页面'})`
    - templateUrl:
    如果我们只需要在 ng-view 中插入 HTML 模板文件，则使用该参数：
    ```
    $routeProvider.when('/computers', {
        templateUrl: 'views/computers.html',
    });
    ```
    
    以上代码会从服务端获取 views/computers.html 文件内容插入到 ng-view 中。
    - controller:
    function、string或数组类型，在当前模板上执行的controller函数，生成新的scope。
    - controllerAs:
    string类型，为controller指定别名。
    - redirectTo:
    重定向的地址。
    - resolve:
    指定当前controller所依赖的其他模块。
    
----------


##ui-router的使用
与ngRoute的区别是：
1. 可实现路由分开控制多模块页面的各个模块
2. html中的视图部分为`<div ui-view></div>`
3. 需要依赖ui.router，函数需要引入\$stateProvider及\$urlRouterProvider:
```
var myUIRoute = angular.module('MyUIRoute', ['ui.router', 'ngAnimate']);
myUIRoute.config(function($stateProvider, $urlRouterProvider) {
    $urlRouterProvider.otherwise("/state1");
    $stateProvider
        .state('state1', {
            url: "/state1",
            templateUrl: "tpls/state1.html"
        })
        .state('state1.list', {
            url: "/list",
            templateUrl: "tpls/state1.list.html",
            controller: function($scope) {
                ...
            }
        })
        .state('state2', {
            ...
        })
        .state('state2.list', {
            ...
        });
});
```


----------


##指令directive
###AngularJS内置的指令
- ng-app 指令初始化一个 AngularJS 应用程序。
- *ng-init 指令初始化应用程序数据。
- ng-model 指令把HTML元素值（比如输入域的值）绑定到应用程序数据。
- ng-repeat 指令对于集合中（数组中）的每个项会**克隆一次**HTML 元素。

###自定义指令
- 可以使用 .directive 函数来添加自定义的指令。
- 要调用自定义指令，HTML 元素上需要添加自定义指令名。
- 使用驼峰法来命名一个指令， runoobDirective, 但在使用它时需要以 - 分割, runoob-directive:
```
<runoob-directive></runoob-directive>

var app = angular.module("myApp", []);
app.directive("runoobDirective", function() {
    return {
        template : "<h1>自定义指令!</h1>"
    };
});
```





```
<hello>
	<div>这里是指令内部的内容。</div>
</hello>
<div hello></div>
<div class="hello"></div>
<!-- directive:hello -->
<div></div>

```
```
var myModule = angular.module("MyModule", []);
myModule.directive("hello", function() {
    return {
    	restrict:"AE",
    	template:"<div>Hello everyone!</div>",
    	//templateUrl: 'hello.html',
    	template:"<div>Hello everyone!<div ng-transclude></div></div>",
    	transclude:true,//transclude有保留地替换指令中的内容
    	replace:true,//replace完全替换指令中的内容，二者相斥
    } 
});
```


----------


####restrict--匹配模式
通过添加 restrict 属性, 并设置值, 来设置指令只能通过相应的方式来调用:

 - A：attribute 属性（默认） `<div my-menu="Products"></div>`
 - E：element 元素 `<my-menu title="Products"></my-menu>`
 - *M：comment 注释 `<!-- directive: my-menu Products -->`
 - *C：class css样式类 `<div class="my-menu:Products"></div>`


----------

####template--替换模板
- template
- templateUrl
- transclude: 指令的transclude功能非常重要，他可以保留原本的内嵌html内容，和实现指令嵌套的能力;在template里面自定义的元素div内部使用<div ng-transclude></div>可以使html里面自定义嵌套的内容显示出来，而不被替换
    ```
    <hello>
    	<div>这里是指令内部的内容。</div>
    </hello>
    transclude:true,
    template:"<div>Hello everyone!<div ng-transclude></div></div>",
    ```
    
- templateCache: 缓存这个模版，使用run方法执行templateCache，在其他指令使用，使用时候用get
    ```
    //run:注射器加载完所有模块时，此方法执行一次
    myModule.run(function($templateCache){
    	$templateCache.put("hello.html","<div>Hello everyone!!!!!!</div>");
    });
    
    myModule.directive("hello", function($templateCache) {
        return {
            restrict: 'AECM',
            template: $templateCache.get("hello.html"),
            replace: true
        }
    });
    ```


----------

####指令执行机制
1. 加载阶段：加载angular.js，找到ng-app指令，确定应用的边界
2. 编译阶段：
    1. 遍历DOM、找到所有指令
    2. 根据指令代码中的template、replace、transclude转换DOM结构；
    3. 如果存在compile函数调用；
3. 链接阶段：
    1. 对每一条指令运行link函数
    2. link函数一般用来操作DOM、绑定事件监听器


----------
####指令内部
```
<superman strength></superman>
<superman strength speed></superman>

var myModule = angular.module("MyModule", []);
myModule.directive("superman", function() {
    return {
        scope: {},//创建独立作用域
        restrict: 'AE',
        controller: function($scope) {//暴露一组公共函数给外部调用
            $scope.abilities = [];
            this.addStrength = function() {
                $scope.abilities.push("strength");
            };
            ...
        },
        link: function(scope, element, attrs) {
        //directive中link方法用来处理dom元素，事件和监听行为。
        //directive中link方法有4个参数，3个常用的为scope,element,attr
            element.addClass('btn btn-primary');
            ...
        }
    }
});
myModule.directive("strength", function() {
    return {
        require: '^superman',//require 表示依赖。
        //在使用 require:'^superman'后 ng 会自动把 superman 的 controller 注入到 link 函数的第四个参数supermanCtrl里。
        link: function(scope, element, attrs, supermanCtrl) {
            supermanCtrl.addStrength();
        }
    }
});
```
- controller 与 link 之间的逻辑选择：
    - controller：想要让指令暴露出一些方法给外部调用
    - link：处理指令内部事物，给元素绑定事件、绑定数据等。


















    
  [1]: http://img.mukewang.com/581dd52f000110a712800720.jpg