@ngdoc overview
@name 开发人员指南：单元测试
@description

翻译者：[@asnowwolf](https://github.com/asnowwolf)

JavaScript是一个动态类型的语言，具有强大的表达能力，但同时，你几乎没法从编译器获得任何帮助。
因此，我们深切体会到：任何JavaScript程序都需要伴随着一组强大的测试。
我们在angular中引入了很多特性来让你更轻松的测试应用程序。
所以，想不写测试？不，没有任何借口！

# 一切的关键是：不要把不同的任务揉成一团

单元测试，顾名思义就是用于测试代码中不可分割的单元。单元测试试图回答下列问题：“我所认为的逻辑正确吗？”或者“我的sort函数是否按照正确的顺序排序了这个列表？”

为了回答上述问题，最重要的事情就是：我们要能把这个单元的代码隔离到一个独立的test模块中。
这是因为我们测试sort函数的时候，绝不会希望也被迫创建相关的部分 —— 比如DOM元素，或者先发起一个XHR调用来获取完数据才能测试sort函数。

虽然这看起来无所谓，但在一个典型的项目中，想要调用一个独立函数确实是非常困难的。
原因在于，开发人员经常会把不同的任务揉成一团，导致一部分代码中往往会做很多很多事：发起XHR请求，对返回的数据进行排序，然后更新到DOM。

在Angular中，我们尝试进行简化，来确保你总能“做正确的事”，所以，我们提供了依赖注入（DI），来让你自由使用XHR（这样你就能mock它了）；
我们创建了抽象层，让你可以排序你的模型(Model)，而不用去管DOM。
最终的结果就是：很容易写出这样一个对一些数据进行排序的sort函数，你的测试程序可以创建一组数据、调用sort函数，然后验证这些数据是否被按照正确的顺序排列了。
这个测试不用等XHR返回结果，不用想方设法创建正确类型的测试用DOM，也不用验证DOM节点是否根据排好的顺序发生了变化。

## Angular确实做了很多，但你也不能偷懒

写Angular的核心思想之一就是“可测试性”，但是它仍然需要你按照正确的方式使用它。
你当然希望轻而易举的把事情做对，但Angular不是魔术。如果你不遵循下列指导原则，你仍然很可能得到一个不可测试的程序。

## 依赖注入(DI)

你可以有很多种方式获得所依赖的对象。比如：

1. 通过`new`运算符创建一个。
2. 在一个众所周知的地方找一个现成的，比如全局性的单例(singleton)对象。
3. 从一个注册表(registry)（比如服务注册表）中找一个现成的。（但是你如何找到一个注册表的引用呢？通常要从一个众所周知的地方寻找。参见#2。）
4. 等别人把它“递”到你手里。

在这四种方案中，只有最后一种是可测试的。让我们分析一下这是为什么：

### 使用`new`运算符

本质上，使用`new`运算符没有错，问题出在从构造函数中调用`new`运算符的时候。
这种情况下，调用者被永久性的和它要`new`的这个类型绑定在一起。比如，如果为了从服务器获得数据而对XHR进行实例化会导致什么后果？

<pre>
function MyClass() {
  this.doWork = function() {
    var xhr = new XHR();
    xhr.open(method, url, true);
    xhr.onreadystatechange = function() {...}
    xhr.send();
  }
}
</pre>

在测试时，问题表现在：当我们想要实例化一个`MockXHR` —— 我们需要它来返回模拟数据，并且模拟网络异常。
如果我们调用`new XHR()`来获得实例，我们就永久性的和实际的XHR（而不是Mock的！）绑定在一起，并且没有任何办法替换它。
固然，我们可以使用猴子补丁（译注：monkey patch —— 见下面例子），但这绝对是个坏注意，理由很多，不过本文档中不展开论述。

下面是一个例子，可以看出为何即使借助于猴子补丁仍然不是个好办法。
<pre>
var oldXHR = XHR;
XHR = function MockXHR() {};
var myClass = new MyClass();
myClass.doWork();
// 确保MockXHR按照正确的参数进行了调用
XHR = oldXHR; // 如果你忘了写这句，就糟了
</pre>

### 全局查找：

另一个方法是从一个众所周知的地方查找此服务。

<pre>
function MyClass() {
  this.doWork = function() {
    global.xhr({
      method:'...',
      url:'...',
      complete:function(response){ ... }
    })
  }
}
</pre>

虽然这次没有直接创建新的依赖对象，问题和`new`方案仍是一样的：测试方无法拦截对`global.xhr`的调用 —— 除非通过猴子补丁。
对测试来说，根本问题在于全局变量应该允许被测试方修改，以便能替换它，并且调用一个mock函数。
关于“这种方式为什么不好”的详细论述请参见：
{@link http://misko.hevery.com/code-reviewers-guide/flaw-brittle-global-state-singletons/ 孤僻的全局状态和单例对象}

上面这个类之所以难于测试，原因就在于我们不得不修改全局状态：
<pre>
var oldXHR = global.xhr;
global.xhr = function mockXHR() {};
var myClass = new MyClass();
myClass.doWork();
// 确保mockXHR使用正确的参数调用了
global.xhr = oldXHR; // 如果你忘了写这句，就糟了
</pre>


### 服务注册表

粗看起来似乎有一个好办法解决这个问题：创建一个注册表，它保存着所有服务，那么测试方就可以替换这些服务了。

<pre>
function MyClass() {
  var serviceRegistry = ????;
  this.doWork = function() {
    var xhr = serviceRegistry.get('xhr');
    xhr({
      method:'...',
      url:'...',
      complete:function(response){ ... }
    })
}
</pre>

问题在于，serviceRegistry从哪里来呢？如果：

* 是 new 出来的，那么测试方没有机会重定义这些服务以供测试。
* 来自全局查找，那么所返回的服务也是全局的（但是重定义比较容易，因为需要重定义的只有一个全局变量）

上面的这个类仍然难于测试，因为我们还是不得不修改全局状态：
<pre>
var oldServiceLocator = global.serviceLocator;
global.serviceLocator.set('xhr', function mockXHR() {});
var myClass = new MyClass();
myClass.doWork();
// 确保mockXHR被使用正确的参数调用
global.serviceLocator = oldServiceLocator; // 如果你忘了写这句，就糟了
</pre>


### 传入依赖对象
最后，可以被动接收所依赖的对象。

<pre>
function MyClass(xhr) {
  this.doWork = function() {
    xhr({
      method:'...',
      url:'...',
      complete:function(response){ ... }
    })
}
</pre>

这是首选方案！因为这段代码让我们不用对`xhr`从哪里来作出任何假设，而只要知道谁负责创建这个类并且传给我们就够了。
因为类的创建者和类的使用者一般不是同一段代码，这里把创建类的职责从应用逻辑里分离出去。这就是依赖注入的简易原理。

上面这个类是可测试的，在测试代码中我们可以这样写：
<pre>
function xhrMock(args) {...}
var myClass = new MyClass(xhrMock);
myClass.doWork();
// 确保xhrMock使用正确的参数调用
</pre>

注意，这个测试中我们不用写任何全局变量。

Angular内建了{@link di 依赖注入}机制，让你可以很容易的“做正确的事”，但是如果你希望在可测试性方面更进一步，你还需要了解更多。 

## 控制器(Controller)

让应用程序与众不同的地方在于它的“逻辑”，而“逻辑”正是我们想要测试的对象。
如果你的应用逻辑中包含了DOM操作，它就很难被测试了。参见下面的例子：

<pre>
function PasswordCtrl() {
  // 获得DOM元素的引用
  var msg = $('.ex1 span');
  var input = $('.ex1 input');
  var strength;

  this.grade = function() {
    msg.removeClass(strength);
    var pwd = input.val();
    password.text(pwd);
    if (pwd.length > 8) {
      strength = 'strong';
    } else if (pwd.length > 3) {
      strength = 'medium';
    } else {
      strength = 'weak';
    }
    msg
     .addClass(strength)
     .text(strength);
  }
}
</pre>

上述代码在可测试性方面的问题在于，它需要你的测试代码在执行被测代码时提供正确类型的DOM。测试代码看起来将是这样的：

<pre>
var input = $('<input type="text"/>');
var span = $('<span>');
$('body').html('<div class="ex1">')
  .find('div')
    .append(input)
    .append(span);
var pc = new PasswordCtrl();
input.val('abc');
pc.grade();
expect(span.text()).toEqual('weak');
$('body').html('');
</pre>

在angular的设计中，控制器和DOM操作被严密的隔离开，其效果就是可以更轻易的提供可测试性，如下所示：

<pre>
function PasswordCtrl($scope) {
  $scope.password = '';
  $scope.grade = function() {
    var size = $scope.password.length;
    if (size > 8) {
      $scope.strength = 'strong';
    } else if (size > 3) {
      $scope.strength = 'medium';
    } else {
      $scope.strength = 'weak';
    }
  };
}
</pre>

测试代码也立即变得整洁了：

<pre>
var $scope = {};
var pc = $controller('PasswordCtrl', { $scope: $scope });
$scope.password = 'abc';
$scope.grade();
expect($scope.strength).toEqual('weak');
</pre>

注意，测试代码不仅仅是变短了，也能更简明的体现出发生了什么。我们看到这段代码“描述了一个故事”，而不只是一组看起来互不相关的“点”。

## 过滤器(Filter)

{@link api/ng.$filterProvider 过滤器}是一个函数，用来把数据转换成用户可读的格式。
它们的重要性在于把数据格式化方面的职责从应用逻辑中移除了，从而简化了应用逻辑。
 
<pre>
myModule.filter('length', function() {
  return function(text){
    return (''+(text||'')).length;
  }
});

var length = $filter('length');
expect(length(null)).toEqual(0);
expect(length('abc')).toEqual(3);
</pre>

## 指令(Directive)

Angular中的指令，用于通过自定义HTML标记(Tag)、属性(Attribute)、类(Class)或注释(Comment)的形式封装复杂的功能。
对于指令来说，单元测试是非常重要的，因为你创建的指令有可能被用于你的整个应用程序中，甚至被用在很多不同的环境中。

### 简单HTML型元素指令

我们先定义一个不依赖其他模块的angular应用。

<pre>
var app = angular.module('myApp', []);
</pre>

然后在我们的应用中添加一个指令。

<pre>
app.directive('aGreatEye', function () {
    return {
        restrict: 'E',
        replace:  true,
        template: '<h1>lidless, wreathed in flame, {{1 + 1}} times</h1>'
    };
});
</pre>

这个这个指令作为标记(tag)时的用法是`<a-great-eye></a-great-eye>`。
这个标记会被模板`<h1>lidless, wreathed in flame, {{1 + 1}} times</h1>`代替。
另外，这里的`{{1 + 1}}`表达式在渲染内容时也将被计算。
接下来，我们将写一个jasmine（一种单元测试框架）单元测试，来验证这个功能。

<pre>
describe('单元测试集', function() {
    var $compile;
    var $rootScope;

    // 加载myApp模块，它包含着指令
    beforeEach(module('myApp'));

    // 保存$rootScope和$compile的引用，以便它们能被这里的所有测试使用
    beforeEach(inject(function(_$compile_, _$rootScope_){
      // 注射器匹配的时候会去掉参数名两端的下划线再匹配
      $compile = _$compile_;
      $rootScope = _$rootScope_;
    }));
    
    it('用适当的内容替换元素', function() {
        // 编译一块包含指令的HTML
        var element = $compile("<a-great-eye></a-great-eye>")($rootScope);
	// 触发所有的监听(watch)，以便在作用域中计算表达式{{1 + 1}}
        $rootScope.$digest();
	// 检查编译后的元素中包含了模板中的内容
        expect(element.html()).toContain("lidless, wreathed in flame, 2 times");
    });
});
</pre>

我们在每个jasmine测试中注入了$compile服务和$rootScope对象。
$compile服务用于渲染aGreatEye指令。
渲染这个指令后我们确保指令已经把内容替换成了 "lidless, wreathed in flame, 2 times" 

## 范例工程

范例工程参见 {@link https://github.com/angular/angular-seed Angular种子工程}。
