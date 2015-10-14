@ngdoc overview
@name Controllers
@description

Translated by [@GrahamLe](https://github.com/grahamle)

# 理解控制器

在Angular中，控制器就像 JavaScript 中的**构造函数**一般，是用来增强 {@link scope Angular作用域(scope)} 的。

当一个控制器通过 {@link api/ng.directive:ngController ng-controller} 指令被添加到DOM中时，ng 会调用该控制器的**构造函数**来生成一个控制器对象，这样，就创建了一个新的**子级 作用域(scope)**。在这个构造函数中，作用域(scope)会作为`$scope`参数注入其中，并允许用户代码访问它。

一般情况下，我们使用控制器做两件事：

- 初始化 `$scope` 对象
- 为 `$scope` 对象添加行为（方法）

# 初始化 `$scope` 对象

当我们创建应用程序时，我们通常要为Angular的 `$scope` 对象设置初始状态，这是通过在 `$scope` 对象上添加属性实现的。这些属性就是供在视图中展示用的**视图模型**（**view model**），它们在与此控制器相关的模板中均可以访问到。

下面的例子中定义了一个非常简单的控制器构造函数：`GreetingCtrl`，我们在该控制器所创建的 scope 中添加一个 `greeting` 属性：

<pre>
    function GreetingCtrl($scope) {
        $scope.greeting = 'Hola!';
    }
</pre>

如上所示，我们有了一个控制器，它初始化了一个 `$scope`对象，并且有一个`greeting`属性。当我们把该控制器关联到DOM节点上，模板就可以通过数据绑定来读取它：

<pre>
    <div ng-controller="GreetingCtrl">
      {{ greeting }}
    </div>
</pre>

**注意**：虽然Angular允许我们在全局作用域下(window)定义控制器函数，但**建议不要**用这种方式。在一个实际的应用程序中，推荐在 {@link module Angular模块} 下通过 `.controller` 为你的应用创建控制器，如下所示：

<pre>
    var myApp = angular.module('myApp',[]);

    myApp.controller('GreetingCtrl', ['$scope', function($scope) {
        $scope.greeting = 'Hola!';
    }]);
</pre>

在上面例子中，我们使用**内联注入**的方式声明 `GreetingCtrl` 依赖于Angular提供的 `$scope` 服务。更多详情，参阅 {@link di 依赖注入} 。


# 为 `$scope` 对象添加行为

为了对事件作出响应，或是在视图中执行计算，我们需要为 scope 提供相关的行为操作的逻辑。上面一节中，我们为 scope 添加属性来让模板可以访问数据模型，现在，我们为 `$scope` 添加方法来让它提供相关的交互逻辑。添加完之后，这些方法就可以在**模板/视图**中被调用了。

下面的例子将演示为控制器的 scope 添加方法，它用来使一个数字翻倍：

<pre>
    var myApp = angular.module('myApp',[]);

    myApp.controller('DoubleCtrl', ['$scope', function($scope) {
        $scope.double = function(value) { return value * 2; };
    }]);
</pre>

当上述控制器被添加到DOM之后，`double` 方法即可被调用，如在模板中的一个Angular表达式中：

<pre>
    <div ng-controller="DoubleCtrl">
      <input ng-model="num"> 翻倍后等于 {{ double(num) }}
    </div>
</pre>

如 {@link concepts 概述} 部分所指出的一样，任何对象（或者原生类型的变量）被添加到 scope 后都将成为 scope 的属性，作为数据模型供模板/视图调用。任何方法被添加到 scope 后，也能在模板/视图中通过Angular表达式或是Angular的事件处理器（如：{@link api/ng.directive:ngClick ngClick}）调用。

# 正确使用控制器

通常情况下，控制器不应被赋予太多的责任和义务，它只需要负责一个单一视图所需的业务逻辑。

最常见的保持控制器“纯度”的方法是将那些不属于控制器的逻辑都封装到服务（services）中，然后在控制器中通过依赖注入调用相关服务。详见指南中的 {@link di 依赖注入} {@link dev_guide.services 服务} 这两部分。

注意，下面的场合**千万不要用控制器**：

- 任何形式的DOM操作：控制器只应该包含业务逻辑。DOM操作则属于应用程序的表现层逻辑操作，向来以测试难度之高闻名于业界。把任何表现层的逻辑放到控制器中将会大大增加业务逻辑的测试难度。ng 提供数据绑定 （{@link databinding 数据绑定}） 来实现自动化的DOM操作。如果需要手动进行DOM操作，那么最好将表现层的逻辑封装在 {@link guide/directive 指令} 中
- 格式化输入：使用 {@link forms angular表单控件} 代替
- 过滤输出：使用 {@link filter angular过滤器} 代替
- 在控制器间复用有状态或无状态的代码：使用{@link dev_guide.services angular服务} 代替
- 管理其它部件的生命周期（如手动创建 service 实例）


# 将控制器与 scope 对象关联

通过两种方法可以实现控制器和 scope 对象的关联：

- {@link api/ng.directive:ngController ngController指令} 这个指令就会创建一个新的 scope
- {@link api/ngRoute.$route $route路由服务}


## 简单的控制器范例

为了更深入地阐释Angular的控制器是如何工作的，我们用以下几个部件来构建一个小型应用：

- 一个由两个按钮和一条简单反馈构成的{@link templates 模板}
- 一个名为 `spice` 的数据模型对象，是一个字符串
- 一个拥有两个方法的控制器，可以设置`spice` 的值

模板中的消息包含了一个到数据模型 `spice` 的绑定，默认值为 `very`。之后，取决于哪个按钮被点击，`spice` 的值会被置为 `chili` 或是 `jalapeño` ，受益于数据绑定，模板中的这个消息会在 `spice` 变化时自动更新。

<doc:example module="spicyApp1">
  <doc:source>
    <div ng-app="spicyApp1" ng-controller="SpicyCtrl">
     <button ng-click="chiliSpicy()">Chili</button>
     <button ng-click="jalapenoSpicy()">Jalapeño</button>
     <p>The food is {{spice}} spicy!</p>
    </div>
    <script>
      var myApp = angular.module('spicyApp1', []);

      myApp.controller('SpicyCtrl', ['$scope', function($scope){
          $scope.spice = 'very';
          
          $scope.chiliSpicy = function() {
              $scope.spice = 'chili';
          };
          
          $scope.jalapenoSpicy = function() {
              $scope.spice = 'jalapeño';
          };
      }]);
    </script>
  </doc:source>
</doc:example>

上面的例子中有几个值得注意的地方：

- `ng-controller` 指令用来为我们的模板创建一个 scope ，而且它受到 `SpicyCtrl` 控制器的管理
- `SpicyCtrl` 就是一个普通的 JavaScript 函数，只是命名上以首字母大写，以 "Ctrl" 或 "Controller" 结尾
- 把一个属性指定给 `$scope` 这样会创建或更新一个数据模型
- 控制器的方法可以通过在 scope 中添加函数来创建，如 `chiliSpicy` 方法
- 控制器的方法和属性在模板/视图中都是可以获得的，在上例中的 `<div>` 元素及其子节点

## 控制器范例扩展--带参数

控制器方法可以带参数，我们看一下如下范例（是上面例子的变种）：

<doc:example module="spicyApp2">
  <doc:source>
  <div ng-app="spicyApp2" ng-controller="SpicyCtrl">
   <input ng-model="customSpice">
   <button ng-click="spicy('chili')">Chili</button>
   <button ng-click="spicy(customSpice)">Custom spice</button>
   <p>The food is {{spice}} spicy!</p>
  </div>
  <script>
    var myApp = angular.module('spicyApp2', []);

    myApp.controller('SpicyCtrl', ['$scope', function($scope){
        $scope.customSpice = "wasabi";
        $scope.spice = 'very';
        
        $scope.spicy = function(spice){
            $scope.spice = spice;
        };
    }]);
  </script>
</doc:source>
</doc:example>

注意上面的 `SpicyCtrl` 控制器现在只定义了一个 `spicy` 方法，带一个 `spice` 参数。然后在模板中，第一个按钮调用 `spicy` 方法的时候传进一个字符串常量 `'chili'` ；第二个按钮则传进一个与`<input>`进行了双向绑定的数据模型 `customSpice`（初始值在 scope 中设置为了 `'wasabi'`）。（译者注：这样在 `<input>` 输入框输入什么，点击第二个按钮时，`<p>` 标签就会显示 `<input>` 中的当前值。）

## Scope 继承范例

我们常常会在不同层级的DOM结构中添加控制器。由于 {@link api/ng.directive:ngController ng-controller} 指令会创建新的子级 scope ，这样我们就会获得一个与DOM层级结构相对应的的基于继承关系的 scope 层级结构。（译者注：由于 Js 是基于原型的继承，所以）底层（内层）控制器的 `$scope` 能够访问在高层控制器的 scope 中定义的属性和方法。详情参见 {@link https://github.com/angular/angular.js/wiki/Understanding-Scopes 理解“作用域”} 。

*译者注*：下面是一个拥有三层div结构，也就对应有三层 scope 继承关系的层级结构（不包括 rootScope 的话），demo中的蓝色边框很清晰的展现了 scope 的层级和DOM层级的对应关系。它还展示了“scope 是由 `ng-controller` 指令创建并由其对应的控制器所管理”这个概念。

<doc:example module="scopeInheritance">
  <doc:source>
    <div ng-app="scopeInheritance" class="spicy">
      <div ng-controller="MainCtrl">
        <p>Good {{timeOfDay}}, {{name}}!</p>

        <div ng-controller="ChildCtrl">
          <p>Good {{timeOfDay}}, {{name}}!</p>

          <div ng-controller="GrandChildCtrl">
            <p>Good {{timeOfDay}}, {{name}}!</p>
          </div>
        </div>
      </div>
    </div>
    <style>
      div.spicy div {
        padding: 10px;
        border: solid 2px blue;
      }
    </style>
    <script>
      var myApp = angular.module('scopeInheritance', []);
      myApp.controller('MainCtrl', ['$scope', function($scope){
        $scope.timeOfDay = 'morning';
        $scope.name = 'Nikki';
      }]);
      myApp.controller('ChildCtrl', ['$scope', function($scope){
        $scope.name = 'Mattie';
      }]);
      myApp.controller('GrandChildCtrl', ['$scope', function($scope){
        $scope.timeOfDay = 'evening';
        $scope.name = 'Gingerbreak Baby';
      }]);
    </script>
  </doc:source>
</doc:example>

注意，上面例子中我们在HTML模板中嵌套了三个 `ng-controller` 指令，这导致我们的视图中有4个 scope：

- root scope，所有作用域的“根”
- `MainCtrl` 控制器管理的 scope （简称 `MainCtrl` scope），拥有 `timeOfDay` 和 `name` 两个属性
- `ChildCtrl` 控制器管理的 scope （简称 `ChildCtrl` scope），继承了 `MainCtrl` scope 中的 `timeOfDay` 属性，但重写了它的 `name` 属性
- `GrandChildCtrl` 控制器管理的 scope （简称 `GrandChildCtrl` scope），重写了 `MainCtrl` scope 中的 `timeOfDay` 属性和 `ChildCtrl` scope 中的 `name` 属性

控制器中，方法继承和属性继承的工作方式是一样的，所以，上面例子中的所有属性，我们也可以改写成能够返回字符串值的方法，同样有效。

## 控制器的单元测试

虽然我们有很多方法可以对控制器进行测试，但在这里，我们仅展示最常见的一种，包括注入 {@link api/ng.$rootScope $rootScope} 以及 {@link api/ng.$controller $controller} ：

**控制器定义**：
<pre>
    var myApp = angular.module('myApp',[]);

    myApp.controller('MyController', function($scope) {
      $scope.spices = [{"name":"pasilla", "spiciness":"mild"},
                       {"name":"jalapeno", "spiceiness":"hot hot hot!"},
                       {"name":"habanero", "spiceness":"LAVA HOT!!"}];
      $scope.spice = "habanero";
    });
</pre>

**控制器测试**：
<pre>
describe('myController function', function() {

  describe('myController', function() {
    var $scope;

    beforeEach(module('myApp'));

    beforeEach(inject(function($rootScope, $controller) {
      $scope = $rootScope.$new();
      $controller('MyController', {$scope: $scope});
    }));

    it('should create "spices" model with 3 spices', function() {
      expect($scope.spices.length).toBe(3);
    });

    it('should set the default value of spice', function() {
      expect($scope.spice).toBe('habanero');
    });
  });
});
</pre>


如果有需要测试嵌套关系的控制器，那么在你的测试代码中，你也得创建对应于 scope 层级结构的测试代码：

<pre>
describe('state', function() {
    var mainScope, childScope, grandChildScope;

    beforeEach(module('myApp'));

    beforeEach(inject(function($rootScope, $controller) {
        mainScope = $rootScope.$new();
        $controller('MainCtrl', {$scope: mainScope});
        childScope = mainScope.$new();
        $controller('ChildCtrl', {$scope: childScope});
        grandChildScope = childScope.$new();
        $controller('GrandChildCtrl', {$scope: grandChildScope});
    }));

    it('should have over and selected', function() {
        expect(mainScope.timeOfDay).toBe('morning');
        expect(mainScope.name).toBe('Nikki');
        expect(childScope.timeOfDay).toBe('morning');
        expect(childScope.name).toBe('Mattie');
        expect(grandChildScope.timeOfDay).toBe('evening');
        expect(grandChildScope.name).toBe('Gingerbreak Baby');
    });
});
</pre>

译者注：

    1. controller命名建议使用首字母大写的驼峰命名，并以Ctrl或者Controller结束.
    2. 在1.2版本过后对于controller有新的controller as的语法，具体可以参见博客(Angular Controller as Syntax vs Scope)[http://greengerong.github.io/blog/2013/12/24/angular-controller-as-syntax-vs-scope/]
    3. scope最为一个视图和模型的胶合层,最好保持scope的干净,所有出现在scope上的属性/方法尽量将是会被视图模板所使用.还有格式化和过滤器，建议使用angular的filter做格式化，对输入的限制过滤使用ngModelController.[Angular Ng-model类型格式转化](http://greengerong.github.io/blog/2013/12/15/angular-model--format/)和[Angularjs组件之input Mask](http://greengerong.github.io/blog/2013/12/15/jqueryinputmarsk4angular/)。
    4. 如上所述对于controller的scope会根据DOM结构有继承关系，所以最好在scope上加入vm保持对象的引用([Learning Javascript with Object Graphs](http://howtonode.org/object-graphs))
    5. 对于scope分为独立scope，原型继承scope，以及部分继承scope(出现在指令中)，这需要更好的理解。一般我们推荐独立scope为最好，部分次之.
    6. 对于多个controller必备数据，为了控制异步操作，在route情况下，建议使用resolve解决。
    7. 对于controller之间通信，如果是强耦合建议使用factory实现([Angular.js为什么如此火呢？-福利节](http://www.cnblogs.com/whitewolf/p/angularjs-start.html))，弱引用建议使用事件机制见[Angularjs Controller 间通信机制](http://greengerong.github.io/blog/2013/04/16/Angularjs-Controller-jian-tong-xin-ji-zhi/)

