@ngdoc overview
@name 理解Angular服务
@description
翻译者:[@lightma](https://github.com/lightma)

## 什么是服务？

服务是单例对象或者函数，用来实现web应用里常见的特定任务。
Angular有很多内置的服务，比如[$http service服务](http://docs.angularjs.org/api/ng.$http)，它提供了对浏览器`XMLHttpRequest`对象的访问，用以实现向服务器发出请求。
就像其他Angular核心的变量和标识符一样，内置的服务名都以`$`开头(比如上面提到的`$http`)。当然，你也能创建自定义的服务。

## 使用服务

为了使用某个服务，你需要指定其他组件(例如控制器，其他服务，过滤器或指令)依赖于该服务，剩下的就交给Angular的依赖注入子系统吧。Angular依赖注入子系统负责服务实例化、依赖解析并按声明向组件提供所依赖对象。

Angular的依赖注入使用{@link http://misko.hevery.com/2009/02/19/constructor-injection-vs-setter-injection/ "构造函数注入"}，在这种注入方式中，依赖是被当作参数传入组件的工厂方法或构造函数里。由于JavaScript是一种动态类型语言，Angular的依赖注入子系统就不能使用静态类型来指定依赖的服务。因此一个组件必须使用{@link di 依赖注入注解}方法中的一种来声明它的依赖。

例如，提供`$inject`属性：

 		var MyController = function($location) { ... };
        MyController.$inject = ['$location'];
        myModule.controller('MyController', MyController);

或者提供"内联"注入注解：

        var myService = function($http) { ... };
        myModule.factory('myService', ['$http', myService]);


## 定义服务

应用开发者可通过在Angular的模块中注册服务的名字和**服务工厂方法**来自由的定义服务。

**服务工厂方法**的目的就是生成单实例对象或者函数，这些单实例对象或者函数对于应用的其他部分来说就代表着该服务。当任何组件声明将该服务作为依赖时，这个对象或者函数就会被注入到该组件中(包括控制器、其他服务、过滤器或指令)。


Angular工厂方法是延迟执行的，也就是说，仅当需要满足依赖声明时才会被执行，并且只会被执行一次。依赖于该服务的所有对象都共享**服务工厂方法**生成的单实例对象的引用。


## 相关主题

* {@link di 关于Angular的依赖注入}
* {@link dev_guide.services.creating_services 创建服务}
* {@link dev_guide.services.managing_dependencies 管理服务依赖}
* {@link dev_guide.services.testing_services 测试Angular服务}

## 相关API

* {@link api/ng Angular服务API}
* {@link api/angular.injector 注入器API}

译者注：

  1. angular中所有的service都将是单例的
  2. 更多的了解angular的service以及区别:[Angular.js Services](http://www.cnblogs.com/whitewolf/p/angular-services.html)
  