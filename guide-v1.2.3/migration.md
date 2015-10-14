@ngdoc overview
@name Angular 1.0到1.2迁移指南
@description

翻译者：[@asnowwolf](https://github.com/asnowwolf)

这次AngularJS 1.2的升级破坏了API兼容性，你可能需要修改应用程序的部分源代码。

虽然我们已经尽力去消除破坏性更改，却仍有一些地方没法消除。
AngularJS 1.2进行了一次关于安全性的彻底复查，来让应用程序在默认配置下更加安全，这导致了很多破坏性更改。
一系列新增的特性，特别是关于动画方面的，也无法避免此类更改。
最后，有一些突出的bug也最好通过修改现有API来纠正。

<div class="alert alert-warning">
<p>**注意:** AngularJS的1.1.x系列版本是实验性版本，所以它的API在小版本之间会出现破坏性更改。
而1.2版本则是一个在1.1的一系列分支基础上推出的成果，它的API是稳定的。</p>

<p>如果你有一个应用程序需要从1.1迁移到1.2，本指南中的内容仍然是有用的，不过你最好同时参考
[changelog](https://github.com/angular/angular.js/blob/master/CHANGELOG.md)。</p>
</div>

<ul class="nav nav-list">
  <li class="nav-header">变更汇总表</li>
  <li>{@link guide/migration#ngRoute移到了独立的模块中 ngroute移到了独立的模块中}</li>
  <li>{@link guide/migration#模板不再支持promise自动解包 模板不再支持promise自动解包}</li>
  <li>{@link guide/migration#修改了中的命名通配符语法 修改了<code>$route</code>中的命名通配符语法}</li>
  <li>{@link guide/migration#在属性中只允许出现一个插值表达式 <code>*[src]</code>和<code>*[ng-src]</code>属性中不再允许出现多个表达式}</li>
  <li>{@link guide/migration#dom事件处理器中不再允许出现插值表达式 DOM事件处理器中不再允许出现插值表达式}</li>
  <li>{@link guide/migration#指令名不再允许以-start或-end结尾 指令名不再允许以-start或-end结尾}</li>
  <li>{@link guide/migration#$q中的promisealways改名为promisefinally $q中的promise.always改名为promise.finally}</li>
  <li>{@link guide/migration#ngmobile改名为ngtouch ngMobile改名为ngTouch}</li>
  <li>{@link guide/migration#移除了resource$then 移除了resource.$then}</li>
  <li>{@link guide/migration#resource调用将返回promise对象 Resource调用将返回promise对象}</li>
  <li>{@link guide/migration#resource的promise将以resource实例为参数触发 Resource的promise将以resource实例为参数触发}</li>
  <li>{@link guide/migration#$locationsearch支持多重关键字 $location.search支持多重关键字}</li>
  <li>{@link guide/migration#移除了ngbindhtmlunsafe，由ngbindhtml代替 移除了ngBindHtmlUnsafe，由ngBindHtml代替}</li>
  <li>{@link guide/migration#支持以表达式作为表单的name属性 支持以表达式作为表单的name属性}</li>
  <li>{@link guide/migration#hasownproperty不再允许作为input元素的name属性 hasOwnProperty不再允许作为input元素的name属性}</li>
  <li>{@link guide/migration#指令：逆转了postlink函数的调用顺序 指令：逆转了postLink函数的调用顺序}</li>
  <li>{@link guide/migration#调整了指令优先级 调整了指令优先级}</li>
  <li>{@link guide/migration#修改了ngscenario模块 修改了ngScenario模块}</li>
  <li>{@link guide/migration#nginclude和ngview在更新后会替换掉它所在的整个元素 ngInclude和ngView在更新后会替换掉它所在的整个元素}</li>
  <li>{@link guide/migration#url改为根据白名单进行“消毒” URL改为根据白名单进行“消毒”}</li>
  <li>{@link guide/migration#独立作用域只能通过属性导出给指令 独立作用域只能通过<code>scope</code>属性导出给指令}</li>
  <li>{@link guide/migration#修改了插值表达式的优先级 修改了插值表达式的优先级}</li>
  <li>{@link guide/migration#以下划线开头或结尾的属性不会再被绑定 以下划线开头或结尾的属性不会再被绑定}</li>
  <li>{@link guide/migration#不再支持绑定到多选框 不再支持绑定到多选框(select[multiple])}</li>
  <li>{@link guide/migration#从i18n中移除了不常用的本地化文件 从i18n中移除了不常用的本地化文件}</li>
</ul>

## ngRoute移到了独立的模块中

像`ngResource`一样，`ngRoute`也有自己的模块了。

使用`$route`, `ngView`, `$routeParams`的应用程序现在需要主动加载`angualr-route.js`文件，并且让应用程序显式依赖`ngRoute`模块。

以前：

```html
<script src="angular.js"></script>
```

```javascript
var myApp = angular.module('myApp', ['someOtherModule']);
```

现在：

```html
<script src="angular.js"></script>
<script src="angular-route.js"></script>
```

```javascript
var myApp = angular.module('myApp', ['ngRoute', 'someOtherModule']);
```

参见 [5599b55b](https://github.com/angular/angular.js/commit/5599b55b04788c2e327d7551a4a699d75516dd21)。

## 模板不再支持promise自动解包

`$parse`和模板不再支持promise自动解包。

以前：

```javascript
$scope.foo = $http({method: 'GET', url: '/someUrl'});
```

```html
<p>{{foo}}</p>
```

现在：

```javascript
$http({method: 'GET', url: '/someUrl'})
  .success(function(data) {
    $scope.foo = data;
  });
```

```html
<p>{{foo}}</p>
```

自动支持promise解包这个特性已经过时了。如果确实需要，你可以通过调用`$parseProvider.unwrapPromises(true)` API来启用它。

参见 [5dc35b52](https://github.com/angular/angular.js/commit/5dc35b527b3c99f6544b8cb52e93c6510d3ac577),
[b6a37d11](https://github.com/angular/angular.js/commit/b6a37d112b3e1478f4d14a5f82faabf700443748)。


## 修改了<code>$route</code>中的命名通配符语法

要迁移此类代码，请参考下面的范例。注意，这里的`*highlight`变为了`:highlight*`。

以前：

```javascript
$routeProvider.when('/Book1/:book/Chapter/:chapter/*highlight/edit',
          {controller: noop, templateUrl: 'Chapter.html'});
```

现在：

```javascript
$routeProvider.when('/Book1/:book/Chapter/:chapter/:highlight*/edit',
        {controller: noop, templateUrl: 'Chapter.html'});
```

参见 [04cebcc1](https://github.com/angular/angular.js/commit/04cebcc133c8b433a3ac5f72ed19f3631778142b)。


## 在`*[src]`或`*[ng-src]`属性中只允许出现一个插值表达式

除了`<a>`和`<img>`元素之外，不允许再绑定多个{{}}表达式到`src`或`ng-src`属性上。

这是Angular 1.2进行的一系列安全性增强之一。

复合表达式让Angular难以判断哪种表达式组合出的值是安全的，可能隐含XSS跨站脚本攻击风险。
为了简化防范XSS风险的工作（简称“消毒”），我们限制了`*[src/ng-src]`中只能出现一个表达式，比如在`iframe[src]`, `object[src]`等。

<table class="table table-bordered code-table">
<thead>
<tr>
  <th>范例</th>
</tr>
</thead>
<tbody>
<tr>
  <td><code>&lt;img src="{{a}}/{{b}}"&gt;</code></td>
  <td class="success">正确，img元素的src属性允许出现多个表达式</td>
</tr>
<tr>
  <td><code>&lt;iframe src="{{a}}/{{b}}"&gt;&lt;/iframe&gt;</code></td>
  <td class="error">错误，iframe元素的src属性不允许出现多个表达式</td>
</tr>
<tr>
  <td><code>&lt;iframe src="{{a}}"&gt;&lt;/iframe&gt;</code></td>
  <td class="success">正确</td>
</tr>
</tbody>
</table>


要想迁移你的代码，你需要在你的作用域中定义一个方法来组合多个表达式。

以前：

```javascript
scope.baseUrl = 'page';
scope.a = 1;
scope.b = 2;
```

```html
<!-- 这里的a和b属性被转义了吗？baseUrl会不会受到用户（特别是潜藏的攻击者）的控制？ -->
<iframe src="{{baseUrl}}?a={{a}&b={{b}}">
```

现在：

```javascript
var baseUrl = "page";
scope.getIframeSrc = function() {

  // 有人可能希望进行特殊处理和相应的“消毒”工作
  var qs = ["a", "b"].map(function(value, name) {
      return encodeURIComponent(name) + "=" +
             encodeURIComponent(value);
    }).join("&");

  // `baseUrl`不会暴露在用户的控制之下，所以我们不用担心忘了进行转义。
  return baseUrl + "?" + qs;
};
```

```html
<iframe src="{{getIframeSrc()}}">
```

参见 [38deedd6](https://github.com/angular/angular.js/commit/38deedd6e3d806eb8262bb43f26d47245f6c2739)。

## DOM事件处理器中不再允许出现插值表达式

DOM事件处理器可以执行任意Javascript代码。在这里使用插值表达式，意味着表达式的结果会作为一个JS字符串而被执行。存储或生成这样一个字符串，是主动向XSS攻击者招手的节奏。
另一方面，`ngClick`和其他Angular专用的事件处理器在Scope作用域而非Window(全局)作用域下执行Angular表达式 —— 这显然要安全得多。

要迁移代码，参见下面的范例：

以前：

```
JS:   scope.foo = 'alert(1)';
HTML: <div onclick="{{foo}}">
```

现在：

```
JS:   scope.foo = function() { alert(1); }
HTML: <div ng-click="foo()">
```

参见 [39841f2e](https://github.com/angular/angular.js/commit/39841f2ec9b17b3b2920fd1eb548d444251f4f56)。

## 指令名不再允许以-start或-end结尾

这个更改的目的是为了支持多元素(multi-element)指令。最好的修改方案是重命名现有的指令，让他们不再以这些后缀结尾。

参见 [e46100f7](https://github.com/angular/angular.js/commit/e46100f7097d9a8f174bdb9e15d4c6098395c3f2)。

## $q中的promise.always改名为promise.finally

做这项更改的原因是要让`$q`和[Q promise
library](https://github.com/kriskowal/q)保持一致，即使这项修改事实上为兼容非ES5的浏览器（比如IE8）制造了一些困难。

`finally`也可以很好地与最近加入到`$q`中的`catch` api协同工作，并且它还是[DOM promises标准](http://dom.spec.whatwg.org/)的一部分。

要迁移这种代码，参阅下面的范例：

以前：

```javascript
$http.get('/foo').always(doSomething);
```

现在：

```javascript
$http.get('/foo').finally(doSomething);
```

或者用兼容IE8的方式：

```javascript
$http.get('/foo')['finally'](doSomething);
```

参见 [f078762d](https://github.com/angular/angular.js/commit/f078762d48d0d5d9796dcdf2cb0241198677582c)。

## ngMobile改名为ngTouch

很多支持触控的设备并不是移动设备，所以，我们决定把这个模块改名，来更好地体现它的实际内涵。

要做这项迁移，只要把所有对`ngMobile`的引用改为`ngTouch`，并且把`angular-mobile.js`替换为`angular-touch.js`就可以了。

参见 [94ec84e7](https://github.com/angular/angular.js/commit/94ec84e7b9c89358dc00e4039009af9e287bbd05)。

## 移除了resource.$then

Resource的实例不再有一个$then函数。请用`$promise.then`代替。

以前：

```javascript
Resource.query().$then(callback);
```

现在:

```javascript
Resource.query().$promise.then(callback);
```

参见 [05772e15](https://github.com/angular/angular.js/commit/05772e15fbecfdc63d4977e2e8839d8b95d6a92d)。

## Resource调用将返回promise对象

resource实例的方法返回一个promise对象，而不再是它自身。

以前：

```javascript
resource.$save().chaining = true;
```

现在：

```javascript
resource.$save();
resource.chaining = true;
```

参见 [05772e15](https://github.com/angular/angular.js/commit/05772e15fbecfdc63d4977e2e8839d8b95d6a92d)。

## Resource的promise将以resource实例为参数触发

调用成功时，resource的promise将以resource实例为参数触发，而不再是HTTP response对象。

通过自定义拦截器(interceptor)API来访问HTTP response对象。

以前：

```javascript
Resource.query().$then(function(response) {...});
```

现在：

```javascript
var Resource = $resource('/url', {}, {
  get: {
    method: 'get',
    interceptor: {
      response: function(response) {
        // expose response
        return response;
      }
    }
  }
});
```

参见 [05772e15](https://github.com/angular/angular.js/commit/05772e15fbecfdc63d4977e2e8839d8b95d6a92d)。


## $location.search支持多重关键字

{@link api/ng.$location#methods_search `$location.search`}现在支持多重关键字，相同关键字的值将被存储到一个数组里。

在这项修正之前：

* `parseKeyValue`用最后一个关键字的值，覆盖以前所有的值。
* `toKeyValue`把这些关键字组合成一个逗号分隔的字符串。

这种行为显然是错误的。如果你的服务器依赖这种错误的行为，那么或者修改服务器端的代码，或者在前端把这些值进行序列化，然后传给`$location`。

参见 [80739409](https://github.com/angular/angular.js/commit/807394095b991357225a03d5fed81fea5c9a1abe)。

## 移除了ngBindHtmlUnsafe，由ngBindHtml代替

`ngBindHtml`从`ngSanitize`模块中移到了核心的`ng`模块。

`ngBindHtml`可以提供和`ngBindHtmlUnsafe`类似的行为（计算表达式，然后通过innerHTML把结果放到DOM中） —— 只要把值用`$sce.trustAsHtml(string)`进行包装就可以了。
如果绑定到一个普通字符串，这个字符串会在被innerHTML之前交给`$santitize`进行“消毒”。
如果`$santitize`服务不可用（`ngSantitize`模块没有被加载），并且绑定表达式的结果不安全，就会触发一个异常。

参见 [dae69473](https://github.com/angular/angular.js/commit/dae694739b9581bea5dbc53522ec00d87b26ae55)。

## 支持以表达式作为表单的name属性

如果你有一个表单的名字是一个表达式：

```
<form name="ctrl.form">
```

如果你要从控制器中访问这个表单：

以前：

```javascript
function($scope) {
  $scope['ctrl.form'] // form controller的实例
}
```

现在：

```javascript
function($scope) {
  $scope.ctrl.form // form controller的实例
}
```

这种形式允许你从控制器中通过新的"controller as"语法风格访问表单。沿用以前的风格没什么好处。

参见 [8ea802a1](https://github.com/angular/angular.js/commit/8ea802a1d23ad8ecacab892a3a451a308d9c39d7)。

## hasOwnProperty不再允许作为input元素的name属性

在form或ngForm的指令中，不再允许以`hasOwnProperty`作为input的name。

以前的版本，名字为"hasOwnProperty"的input会被悄悄地忽略，不会被加入到作用域中。现在则会触发一个badname异常。
使用"hasOwnProperty"作为input的名字是个罕见而且很差的做法。要想迁移，改下input的名字吧。

参见 [7a586e5c](https://github.com/angular/angular.js/commit/7a586e5c19f3d1ecc3fefef084ce992072ee7f60)。

## 指令：逆转了postLink函数的调用顺序

逆转了postLink函数的执行顺序，以便让它和preLink和compile函数的执行顺序相对称。（译注：后进先出 —— 最后执行preLink的指令会最先执行postLink）

以前的compile/link函数都是按照优先级(priority)的顺序依次执行的。

<table class="table table-bordered table-striped code-table">
<thead>
<tr>
  <th>#</th>
  <th>步骤</th>
  <th align="center">原调用顺序</th>
  <th align="center">新调用顺序</th>
</tr>
</thead>
<tbody>
<tr>
  <td>1</td>
  <td>Compile函数</td>
  <td align="center" colspan="2">高 → 低</td>
</tr>
<tr>
  <td>2</td>
  <td colspan="3">Compile各个子节点</td>
</tr>
<tr>
  <td>3</td>
  <td>PreLink函数</td>
  <td align="center" colspan="2">高 → 低</td>
</tr>
<tr>
  <td>4</td>
  <td colspan="3">Link各个子节点</td>
</tr>
<tr>
  <td>5</td>
  <td>PostLink函数</td>
  <td align="center">高 → 低</td>
  <td align="center">**低 → 高**</td>
</tr>
</tbody>
</table>

<small>"高 → 低" 是针对指令的 `priority`(优先级)属性而言的。</small>

现实中只有极少量指令是是依赖postLink函数的执行顺序的（不像compile函数对顺序的依赖那么严重），所以这项改动很少会影响到现有的指令，它可能需要移到preLink函数中，或者为它指定一个负的优先级。

你可以访问[这次提交的diff](https://github.com/angular/angular.js/commit/31f190d4d53921d32253ba80d9ebe57d6c1de82b)来看看指令内部的解释过程修改了什么。

参见 [31f190d4](https://github.com/angular/angular.js/commit/31f190d4d53921d32253ba80d9ebe57d6c1de82b)。

## 调整了指令优先级

ngRepeat, ngSwitchWhen, ngIf, ngInclude 和 ngView 这几个指令的优先级被修改了。这将更明确的规定出它们之间的相对优先级。

为了让ngRepeat, ngSwitchWhen, ngIf, ngInclude 和 ngView指令在通常的场景下都能正常工作，它们被调整到了下列优先级：

指令        | 原优先级 | 新优先级
-----------------|--------------|-------------
ngRepeat         | 1000         | 1000
ngSwitchWhen     | 500          | 800
ngIf             | 1000         | 600
ngInclude        | 1000         | 400
ngView           | 1000         | 400

参见 [b7af76b4](https://github.com/angular/angular.js/commit/b7af76b4c5aa77648cc1bfd49935b48583419023)。

## 修改了ngscenario模块

browserTrigger函数使用eventData对象代替了以前的原生鼠标事件event对象。
做这项迁移，只要把`key`,`x`,`y`这几个参数放到一个对象中，然后作为browserTrigger函数的第三个参数传进去就可以了。

参见 [28f56a38](https://github.com/angular/angular.js/commit/28f56a383e9d1ff378e3568a3039e941c7ffb1d8)。

## ngInclude和ngView在更新后会替换掉它所在的整个元素

以前的`ngInclude`和`ngView`指令只是更新它所在元素的内容。现在，这些指令将会在每次加载了新内容的时候重建这个元素本身。

这确保所有被包括进来的内容都始终存在一个单一的根元素，这可以让定义与动画效果有关的css更为容易。

参见 [7d69d52a](https://github.com/angular/angular.js/commit/7d69d52acff8578e0f7d6fe57a6c45561a05b182)，
[aa2133ad](https://github.com/angular/angular.js/commit/aa2133ad818d2e5c27cbd3933061797096356c8a)。

## URL改为根据白名单进行“消毒”

通过`compileProvider`配置一个白名单，可用来指定想把哪些url视为安全的。默认情况下，所有通用的前缀被视为安全的，包括带有`image/*`这种mime头的`data:`协议。这种修改不会干扰哪些未包含恶意图片的正常应用。

参见 [1adf29af](https://github.com/angular/angular.js/commit/1adf29af13890d61286840177607edd552a9df97)，
[3e39ac7e](https://github.com/angular/angular.js/commit/3e39ac7e1b10d4812a44dad2f959a93361cd823b)。

## 独立作用域(Isolate scope)只能通过<code>scope</code>属性导出给指令

没有独立作用域的指令不允许再访问同一个元素上其它带有独立作用域的指令的作用域。如果你的代码依赖这种行为（即“非独立作用域指令”需要访问同一个元素上另一个指令的“独立作用域”），那么，就要把它改为独立作用域指令，把所需要的变量显式传进去。

** 以前 **

```
<input ng-model="$parent.value" ng-isolate>

.directive('ngIsolate', function() {
  return {
    scope: {},
    template: '{{value}}'
  };
});
```

** 现在 **

```
<input ng-model="value" ng-isolate>

.directive('ngIsolate', function() {
  return {
    scope: {value: '=ngModel'},
    template: '{{value}}
  };
});
```

参见 [909cabd3](https://github.com/angular/angular.js/commit/909cabd36d779598763cc358979ecd85bb40d4d7),
[#1924](https://github.com/angular/angular.js/issues/1924) 和
[#2500](https://github.com/angular/angular.js/issues/2500)。

## 修改了插值表达式`{{}}`的优先级

1.2.0-rc.2版本中，插值表达式的优先级是`-100`，更早的版本中则是`100`，而且，绑定发生在链接后(post-linking)阶段。

现在这个正式版中，属性中的插值表达式则相当于一个优先级是100的指令，而且，绑定发生在链接前(pre-linking)阶段。

参见 [79223eae](https://github.com/angular/angular.js/commit/79223eae5022838893342c42dacad5eca83fabe8),
[#4525](https://github.com/angular/angular.js/issues/4525),
[#4528](https://github.com/angular/angular.js/issues/4528), 和
[#4649](https://github.com/angular/angular.js/issues/4649)

## 以下划线开头或结尾的属性不会再被绑定

<div class="alert alert-info">
<p>**它已经被改回去了**: 这个改动已经在1.2.1中改回了以前的样子，所以如果你在使用**1.2.1或更高**的版本，请忽略这一条。</p>
</div>

这个改动准备用来在作用域链上实现“私有”属性（即名字用下划线开头和/或结尾的属性）。
这些属性将不能被模板或字符串中的Angular表达式使用。但是它们仍然可以被JavaScript代码任意使用（就像以前那样）。

**修改动机**

Angular表达式执行在一个受限的语境中。它们不能直接访问全局作用域，比如`window`，`document`或者Function的构造函数。
但是他们仍然可以直接访问作用域链上的各个“名称/属性”等。
一直以来，把敏感的API保护在作用域链之外都是一个好主意（比如定义在一个闭包或者你的控制器中）。
这主要出于两个原因：

1. JavaScript没有私有属性的概念，所以如果你准备把数据放到作用域链上供JavaScript使用，你就同时也把它们开放给了Angular表达式，让它们任意使用。
2. 新的`controller as`语法，导出了整个控制器，这也极大地增加了作用域链中这些变量的可见性。

尽管Angular表达式是由开发人员书写和控制的，但他们：

1. 通常需要接受用户输入。
2. JavaScript代码的测试覆盖率往往不够

这次提交的版本提供了一种机制，通过命名约定，来允许在控制器/作用域这一层对这些属性的访问进行限制。
这意味着可以让模板中的表达式只能访问到自己确实需要的那些属性。

参见 [3d6a89e8](https://github.com/angular/angular.js/commit/3d6a89e8888b14ae5cb5640464e12b7811853c7e)。


## 不再支持绑定到多选框(select[multiple])

在`select[single]`和`select[multiple]`形态之间切换经常触犯浏览器的怪癖（不兼容特性）。
这个特性在双向数据绑定中从来没有正常工作过，所以也不用担心谁会用到它。

如果你有兴趣亲自动手添加这个特性，请到Github上给我们提交一个pull request。

参见 [d87fa004](https://github.com/angular/angular.js/commit/d87fa0042375b025b98c40bff05e5f42c00af114)。

## 从i18n中移除了不常用的本地化文件

AngularJS使用的是Google Closure Library中的本地化文件。下列本地化文件已经从Closure中移除了，因此Angular也无法再继续支持：

`chr`, `cy`, `el-polyton`, `en-zz`, `fr-rw`, `fr-sn`, `fr-td`, `fr-tg`, `haw`, `it-ch`, `ln-cg`,
`mo`, `ms-bn`, `nl-aw`, `nl-be`, `pt-ao`, `pt-gw`, `pt-mz`, `pt-st`, `ro-md`, `ru-md`, `ru-ua`,
`sr-cyrl-ba`, `sr-cyrl-me`, `sr-cyrl`, `sr-latn-ba`, `sr-latn-me`, `sr-latn`, `sr-rs`, `sv-fi`,
`sw-ke`, `ta-lk`, `tl-ph`, `ur-in`, `zh-hans-hk`, `zh-hans-mo`, `zh-hans-sg`, `zh-hans`,
`zh-hant-hk`, `zh-hant-mo`, `zh-hant-tw`, `zh-hant`

虽然这些本地化文件已经从AngularJS的库中移除了，你仍然可以加载和使用你自己的本地化文件，不过就得你去自己维护它们了。

参见 [6382e21f](https://github.com/angular/angular.js/commit/6382e21fb28541a2484ac1a241d41cf9fbbe9d2c)。
