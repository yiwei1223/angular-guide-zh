@ngdoc overview
@name 开发指南 - 模块
@description

翻译：[@GrahamLe](https://github.com/grahamle)

# 何为模块？

大多数应用程序都有个 `main` 函数来初始化、连接以及启动整个应用。ng 中虽然没有 `main` 函数，但它用模块来描述应用将如何启动。这种策略有如下几种优势：

  * 整个过程是声明式的，更容易理解
  * 在单元测试中，没有必要加载所有模块，这样有利于单元测试的代码书写
  * 在场景测试中，额外的模块可以被加载进来，进而重写一些配置，这样有助于实现应用的端到端的测试
  * 第三方代码可以很容易被打包成可重用的模块
  * 模块可以用任意顺序或并行顺序加载（得益于模块执行的延迟性）

# 模块ABC

你会说，别废话一箩筐了，说个 Hello World 的模块示例吧，让它跑起来才是王道呀。

可以，在此之前，有几件重要的事情必须记住：

  * {@link api/angular.Module 模块(Module)} API
  * 在 `<html ng-app="myApp">` ，注意对于 `myApp` 模块的引用，这是用模块的方法启动应用的触发点

<doc:example module='myApp'>
  <doc:source>
    <script>
      // 声明一个模块
      var myAppModule = angular.module('myApp', []);

      // 模块配置
      // 在这个例子中，我们创建一个过滤器
      myAppModule.filter('greet', function() {
       return function(name) {
          return 'Hello, ' + name + '!';
        };
      });

    </script>
    <div>
      {{ 'World' | greet }}
    </div>
  </doc:source>
</doc:example>



# 推荐配置

上面的例子太过简单了，对于大型应用显然不适用。对于大型应用，我们建议把它像这样分成多个模块：

  * 服务模块
  * 指令模块
  * 过滤器模块
  * 一个应用的模块，依赖于上述的三个模块，而且包含应用的初始化及启动代码

这样划分模块的原因主要是在你的测试中，经常需要忽略难以测试的初始化的代码，而且这样测试时可以单独加载模块进行相关的测试。

上述模块划分仅仅是一种建议性的方案，你可以根据自己的应用去调整。下面的代码显示了上面所述的模块划分：

<doc:example module='xmpl'>
  <doc:source>
    <script>
      angular.module('xmpl.service', []).
        value('greeter', {
          salutation: 'Hello',
          localize: function(localization) {
            this.salutation = localization.salutation;
          },
          greet: function(name) {
            return this.salutation + ' ' + name + '!';
          }
        }).
        value('user', {
          load: function(name) {
            this.name = name;
          }
        });

      angular.module('xmpl.directive', []);

      angular.module('xmpl.filter', []);

      angular.module('xmpl', ['xmpl.service', 'xmpl.directive', 'xmpl.filter']).
        run(function(greeter, user) {
          // 这相当于 main 函数
          greeter.localize({
            salutation: 'Bonjour'
          });
          user.load('World');
        })


      // 应用所需的一个控制器
      var XmplController = function($scope, greeter, user) {
        $scope.greeting = greeter.greet(user.name);
      }
    </script>
    <div ng-controller="XmplController">
      {{ greeting }}!
    </div>
  </doc:source>
 </doc:example>



# 模块加载及依赖

模块是配置代码块和运行代码块的集合，在启动阶段被执行。最简单的模块中包含下面两种代码块：

  1. **配置代码块** - 在 `provider` 注册和配置阶段执行（注：provider 是 ng 服务的一种）。只有 `provider` 和 `constant` 可以被注入配置代码块。这是为了防止服务在完全配置好之前被意外地初始化。
  2. **执行代码块** - 在 `injector` 被创建后执行，被用来启动整个应用。只有服务的实例对象以及 `constant` 可以被注入到执行代码块。这是为了防止在应用执行期间系统的更进一步的配置。

<pre>
angular.module('myModule', []).
  config(function(injectables) { // provider型注入器
    // 这是配置(config)代码块的范例，你可以有任意多个配置代码块
    // 配置块中你只能注入Provider类（注意：不是由Provider类生成的实例）以及`constant`
  }).
  run(function(injectables) { // instance型注入器
    // 这是运行(run)代码块的范例，你可以有任意个运行代码块
    // 运行块中你只能注入Provider实例（注意：不是Provider类）
  });
</pre>

## 配置代码块

有一些快捷方法可供模块调用，效果等同于配置代码块，比如：

<pre>
angular.module('myModule', []).
  value('a', 123).
  factory('a', function() { return 123; }).
  directive('directiveName', ...).
  filter('filterName', ...);

// 等同于

angular.module('myModule', []).
  config(function($provide, $compileProvider, $filterProvider) {
    $provide.value('a', 123);
    $provide.factory('a', function() { return 123; });
    $compileProvider.directive('directiveName', ...);
    $filterProvider.register('filterName', ...);
  });
</pre>

配置语句的执行顺序就是根据它们注册的顺序而定的。唯一的例外是 `constant` 的定义，它会被调整到所有配置块的最前面执行。

## 执行代码块

执行代码块是 ng 中最接近 `main` 函数的一个东西。执行代码块是应用启动时运行的代码。它在所有的服务被配置好以及 `注入器(injector)` 被创建好之后执行。通常，执行代码块包含的代码都很难进行单元测试，正因为如此，它通常应该被丢在一个单独的模块中，这样我们可以在单元测试时忽略它。

## 模块依赖

模块声明时可以列出它所需要依赖的其它模块。一个模块依赖某模块，意味着这个被依赖的模块需要在模块被加载之前加载完毕。更具体些，假设模块A依赖于模块B，那么模块A的配置代码块的执行，必须发生在模块B的配置代码块之后；模块A的执行代码块亦同理，也在模块B的执行代码块之后被执行。每个模块只能被加载一次，即使有多个别的模块依赖它。

## 异步加载

模块是一种管理 `$injector` 配置的方式，它和将脚本加载到JavaScript虚拟机（VM）没有半毛钱关系。现在已经有很多项目用来在 ng 中处理脚本的动态加载。由于模块在加载期什么都不做，所以它们可以按任何顺序载入到虚拟机中，脚本加载器也可以充分利用这一特性来并行地加载模块和脚本。

## 创建模块 vs 获取模块

注意，使用 `angular.module('myModule', [])` 将创建名为 `myModule` 的模块并重写已有的同名模块。而使用 `angular.module('myModule')` 则只会获取已有的模块实例。

<pre>
  var myModule = angular.module('myModule', []);
  
  // 添加一些指令和服务
  myModule.service('myService', ...);
  myModule.directive('myDirective', ...);

  // 创建一个新模块将覆盖掉这些指令和服务
  var myModule = angular.module('myModule', []);

  // 由于myOtherModule模块还没有定义，所以会抛出一个异常
  var myModule = angular.module('myOtherModule');
</pre>

# 单元测试

单元测试最简单的形式是在测试容器中初始化应用的一小部分对象然后进行模拟运行。意识到每个模块在每个 `injector` 创建时只能被加载一次是非常重要的。通常每个应用中只有一个 `注入器(injector)`（译注：即全局单例），但是在测试中，每个测试容器都有自己的独立`injector` ，这意味着对于JS虚拟机，模块都会被多次加载。用正确的形式组织模块将有助于我们进行单元测试，如下面例子所示：

以下例子中，我们都会用到的模块定义如下：

<pre>
  angular.module('greetMod', []).

    factory('alert', function($window) {
      return function(text) {
        $window.alert(text);
      }
    }).

    value('salutation', 'Hello').

    factory('greet', function(alert, salutation) {
      return function(name) {
        alert(salutation + ' ' + name + '!');
      }
    });
</pre>

让我们写一些测试代码：

<pre>
describe('myApp', function() {
  // 加载相关的应用模块，然后加载指定的测试模块，它将把$window改写为一个mock版
  // 所以，调用window.alert()将不会弹出一个提示框而阻塞测试的运行。这就是一个在测试容器中改写配置信息的例子。
  beforeEach(module('greetMod', function($provide) {
    $provide.value('$window', {
      alert: jasmine.createSpy('alert')
    });
  }));

  // inject()函数将创建一个注入器，并且把greet服务和$window服务注入到测试容器中。
  // 测试代码不需要关心如何获取应用，只管测试它就行了。
  it('should alert on $window', inject(function(greet, $window) {
    greet('World');
    expect($window.alert).toHaveBeenCalledWith('Hello World!');
  }));

  // 这是让测试容器改写配置的另一种方案：使用内嵌模块和依赖注入
  it('should alert using the alert service', function() {
    var alertSpy = jasmine.createSpy('alert');
    module(function($provide) {
      $provide.value('alert', alertSpy);
    });
    inject(function(greet) {
      greet('World');
      expect(alertSpy).toHaveBeenCalledWith('Hello World!');
    });
  });
});
</pre>
