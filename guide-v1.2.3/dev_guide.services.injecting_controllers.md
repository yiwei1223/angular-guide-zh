@ngdoc overview
@name 开发指南: Angular服务: 把服务注入到控制器中
@description

翻译者：[@asnowwolf](https://github.com/asnowwolf)

把服务作为依赖注入到控制器中，和把服务作为依赖注入到其他服务中的方式非常相似。

由于JavaScript是一门动态语言，DI不能像静态类型语言一样根据类型判断该注入哪个服务，而只能根据名字。
所以，你可以通过`$inject`属性来实现注入，这个属性是一个数组，包含了一组准备注入的服务名称。
这些名称必须是已经注册到angular中的服务标示。在`$inject`数组中， **这些服务标示的顺序是很重要的**：
它们在数组中的顺序将被同样用作往工厂函数中注入参数的顺序。在工厂函数中，这些参数的名字并不重要，
但是按照惯例，它们应该和服务标示保持一致，下面将详细论述这项惯例的优点所在。

<pre>
function myController($loc, $log) {
  this.firstMethod = function() {
    // 使用$location服务
    $loc.setHash();
  };
  this.secondMethod = function() {
    // 使用$log服务
    $log.info('...');
  };
}
// 这里指定哪些服务将被注入
myController.$inject = ['$location', '$log'];
</pre>

<doc:example module="MyServiceModule">
<doc:source>
<script>
angular.
 module('MyServiceModule', []).
 factory('notify', ['$window', function(win) {
    var msgs = [];
    return function(msg) {
      msgs.push(msg);
      if (msgs.length == 3) {
        win.alert(msgs.join("\n"));
        msgs = [];
      }
    };
  }]);

function myController(scope, notifyService) {
  scope.callNotify = function(msg) {
    notifyService(msg);
  };
}

myController.$inject = ['$scope','notify'];
</script>

<div ng-controller="myController">
  <p>让我们简单的试验一下这个注入到控制器中的notify服务</p>
  <input ng-init="message='test'" ng-model="message" >
  <button ng-click="callNotify(message);">NOTIFY</button>
  <p>(你必须点三次才能看到一个alert对话框)</p>
</div>
</doc:source>
<doc:scenario>
  it('should test service', function() {
    expect(element(':input[ng\\:model="message"]').val()).toEqual('test');
  });
</doc:scenario>
</doc:example>

## 隐式依赖注入

Angular依赖注入(DI)的一项新特性是从参数名中查找依赖。我们重写上面的范例，来看看如何用隐式注入的方式注入`$window`，`$scope`，和我们自己的`notify`服务

<doc:example module="MyServiceModuleDI">
<doc:source>
<script>
angular.
 module('MyServiceModuleDI', []).
 factory('notify', function($window) {
    var msgs = [];
    return function(msg) {
      msgs.push(msg);
      if (msgs.length == 3) {
        $window.alert(msgs.join("\n"));
        msgs = [];
      }
    };
  });

function myController($scope, notify) {
  $scope.callNotify = function(msg) {
    notify(msg);
  };
}
</script>
<div ng-controller="myController">
  <p>我们来试试notify服务，它被隐式的注入到本控制器中</p>
  <input ng-init="message='test'" ng-model="message">
  <button ng-click="callNotify(message);">NOTIFY</button>
  <p>(你必须点击三次才能看到一个alert对话框)</p>
</div>
</doc:source>
</doc:example>

然而，如果你打算{@link http://en.wikipedia.org/wiki/Minification_(programming) 最小化(minify)}你的代码，
你的变量名（包括参数名）将被改成尽可能短的形式，这种情况下，你仍然需要使用`$inject`属性来显式的指定这些服务的名字。

## 相关主题

* {@link dev_guide.services.understanding_services 理解Angular服务}
* {@link dev_guide.services.creating_services 创建Angular服务}
* {@link dev_guide.services.managing_dependencies 管理服务依赖}
* {@link dev_guide.services.testing_services 测试Angular服务}

## 相关APIAPI

{@link api/ng Angular服务API}
