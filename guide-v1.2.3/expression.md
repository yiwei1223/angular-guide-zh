@ngdoc overview
@name 开发人员指南: 表达式(Expressions)
@description
翻译者:[@NigelYao](https://github.com/NigelYao)

"表达式"是一种类似JavaScript的代码片段，通常在视图中以'{{ expression }}'的形式使用。表达式由{@link api/ng.$parse $parse}服务解析，而解析之后经常会使用{@link filter 过滤器}来格式化成一种更加用户友好的形式。

例如，下面这些都是Angular中合法的表达式：
  
  * `1+2`
  * `user.name`
  * `user.name`
  * `items[index]`

## Angular表达式与JS表达式 

可能会有人认为Angular视图表达式就是JavaScript表达式，但这不完全正确，因为Angular并没有使用JavaScript中的'eval()'来解析表达式。你可以认为Angular表达式与JavaScript表达式有如下的区别：

  * **属性解析：** 所有的属性的解析都是相对于作用域(scope)的，而不像JavaScript中的表达式解析那样是相对于全局'window'对象的。

  * **容错性：** 表达式的解析对'undefined'和'null'具有容错性，这不像在JavaScript中，试图解析未定义的属性时会抛出`ReferenceError`或`TypeError`错误.

  * **禁止控制流语句：** 表达式中不允许包括下列语句：条件判断(if)，循环(for/while)，抛出异常(throw)。

另一方面，如果你想执行特定的JavaScript代码，你应该在一个控制器里导出一个方法，然后在模板中调用这个方法。如果你想在JavaScript中解析一个Angular表达式，使用{@link api/ng.$rootScope.Scope#methods_$eval `$eval()`}方法。

## 例子

<doc:example>
<doc:source>
 1+2={{1+2}}
</doc:source>
<doc:scenario>
 it('should calculate expression in binding', function() {
   expect(binding('1+2')).toEqual('3');
 });
</doc:scenario>
</doc:example>

你可以在此处试一试解析其他的表达式：

<example module="expressionExample">
  <file name="index.html">
    <div ng-controller="ExampleController" class="expressions">
      Expression:
      <input type='text' ng-model="expr" size="80"/>
      <button ng-click="addExp(expr)">Evaluate</button>
      <ul>
       <li ng-repeat="expr in exprs track by $index">
         [ <a href="" ng-click="removeExp($index)">X</a> ]
         <tt>{{expr}}</tt> => <span ng-bind="$parent.$eval(expr)"></span>
        </li>
      </ul>
    </div>
  </file>

  <file name="script.js">
    angular.module('expressionExample', [])
      .controller('ExampleController', ['$scope', function($scope) {
        var exprs = $scope.exprs = [];
        $scope.expr = '3*10|currency';
        $scope.addExp = function(expr) {
          exprs.push(expr);
        };

        $scope.removeExp = function(index) {
          exprs.splice(index, 1);
        };
      }]);
  </file>

  <file name="protractor.js" type="protractor">
    it('should allow user expression testing', function() {
      element(by.css('.expressions button')).click();
      var lis = element(by.css('.expressions ul')).all(by.repeater('expr in exprs'));
      expect(lis.count()).toBe(1);
      expect(lis.get(0).getText()).toEqual('[ X ] 3*10|currency => $30.00');
    });
  </file>
</example>


# 作用域

表达式的作用域是scope上。不像在JavaScript中，将默认的作用域放在全局的window对象中，Angular表达式必须使用{@link api/ng.$window
`$window`}来指向全局的'window'对象。例如，如果你想调用在'window'上定义的'alert()'方法，在表达式中，你必须使用'$window.alert()'。作者故意这样设定，就是为了防止对全局状态非正常的访问（一些奇怪bug的常见来源）。

<example module="expressionExample">
  <file name="index.html">
    <div class="example2" ng-controller="ExampleController">
      Name: <input ng-model="name" type="text"/>
      <button ng-click="greet()">Greet</button>
      <button ng-click="window.alert('Should not see me')">Won't greet</button>
    </div>
  </file>

  <file name="script.js">
    angular.module('expressionExample', [])
      .controller('ExampleController', ['$window', '$scope', function($window, $scope) {
        $scope.name = 'World';

        $scope.greet = function() {
          $window.alert('Hello ' + $scope.name);
        };
      }]);
  </file>

  <file name="protractor.js" type="protractor">
    it('should calculate expression in binding', function() {
      if (browser.params.browser == 'safari') {
        // Safari can't handle dialogs.
        return;
      }
      element(by.css('[ng-click="greet()"]')).click();

      var alertDialog = browser.switchTo().alert();

      expect(alertDialog.getText()).toEqual('Hello World');

      alertDialog.accept();
    });
  </file>
</example>

## 容错性

表达式的解析对undefined和null具有容错性。在JavaScript中，解析'a,b,c'时，如果'a'不是一个对象会抛出异常。在一些语言中这可能会有用，但表达式的解析在Angular中主要用在数据绑定上，我们会像下面这样使用表达式：

        {{a.b.c}}

此时如果'a'是未定义的（也许我们正等着服务器响应数据，在不久的将来其会被定义），解析抛出异常将不再合理。如果表达式不具有容错性的话，我们的代码会变的繁杂且影响阅读，例如：`{{((a||{}).b||{}).c}}`。

类似的，在undefined和null的对象上调用方法'a.b.c()'时会返回undefined。

## 禁止控制流语句

在表达式中禁止写控制流语句。这背后的原因在于，Angular设计哲学的核心认为，应用逻辑应该在控制器中，而不是在视图中控制。如果你需要条件表达式，循环或者抛出异常出现在你的视图表达式中，请将其委托到JavaScript方法中执行。

## `$event`

指令 {@link ng.directive:ngClick `ngClick`} and {@link ng.directive:ngFocus `ngFocus`}向当前表达式的作用域提供了一个 `$event` 的对象。

<example module="eventExampleApp">
  <file name="index.html">
    <div ng-controller="EventController">
      <button ng-click="clickMe($event)">Event</button>
      <p><code>$event</code>: <pre> {{$event | json}}</pre></p>
      <p><code>clickEvent</code>: <pre>{{clickEvent | json}}</pre></p>
    </div>
  </file>

  <file name="script.js">
    angular.module('eventExampleApp', []).
      controller('EventController', ['$scope', function($scope) {
        /*
         * expose the event object to the scope
         */
        $scope.clickMe = function(clickEvent) {
          $scope.clickEvent = simpleKeys(clickEvent);
          console.log(clickEvent);
        };

        /*
         * return a copy of an object with only non-object keys
         * we need this to avoid circular references
         */
        function simpleKeys (original) {
          return Object.keys(original).reduce(function (obj, key) {
            obj[key] = typeof original[key] === 'object' ? '{ ... }' : original[key];
            return obj;
          }, {});
        }
      }]);
  </file>
</example>

请注意，在上面的示例中，我们是传递`$event` 的参数给`clickMe`，而不是通过 `{{$event}}`这种绑定的方式来传递参数的。这是因为，在数据绑定的时候，`$event` 不在该次绑定的作用域。

## 单次绑定

一个以`::`开头的表达式会被当作单次绑定来处理。单次绑定会在数据趋于稳定之后停止对其解析计算，如果表达式的结果不是未定义(non-undefined)的话，这个过程发生在首次digest(具体请查看下面的数据稳定算法)。

<example module="oneTimeBidingExampleApp">
  <file name="index.html">
    <div ng-controller="EventController">
      <button ng-click="clickMe($event)">Click Me</button>
      <p id="one-time-binding-example">One time binding: {{::name}}</p>
      <p id="normal-binding-example">Normal binding: {{name}}</p>
    </div>
  </file>
  <file name="script.js">
    angular.module('oneTimeBidingExampleApp', []).
      controller('EventController', ['$scope', function($scope) {
        var counter = 0;
        var names = ['Igor', 'Misko', 'Chirayu', 'Lucas'];
        /*
         * expose the event object to the scope
         */
        $scope.clickMe = function(clickEvent) {
          $scope.name = names[counter % names.length];
          counter++;
        };
      }]);
  </file>
  <file name="protractor.js" type="protractor">
    it('should freeze binding after its value has stabilized', function() {
      var oneTimeBiding = element(by.id('one-time-binding-example'));
      var normalBinding = element(by.id('normal-binding-example'));

      expect(oneTimeBiding.getText()).toEqual('One time binding:');
      expect(normalBinding.getText()).toEqual('Normal binding:');
      element(by.buttonText('Click Me')).click();

      expect(oneTimeBiding.getText()).toEqual('One time binding: Igor');
      expect(normalBinding.getText()).toEqual('Normal binding: Igor');
      element(by.buttonText('Click Me')).click();

      expect(oneTimeBiding.getText()).toEqual('One time binding: Igor');
      expect(normalBinding.getText()).toEqual('Normal binding: Misko');

      element(by.buttonText('Click Me')).click();
      element(by.buttonText('Click Me')).click();

      expect(oneTimeBiding.getText()).toEqual('One time binding: Igor');
      expect(normalBinding.getText()).toEqual('Normal binding: Lucas');
    });
  </file>
</example>


### 为什么使用这个特性

单次绑定的主要意图是提供一种在数据绑定稳定之后，注销绑定并且释放资源。
减少表达式被监察的次数，加快digest循环，允许同时显示更多的数据信息。

### 数据稳定算法

单次绑定表达式在值未定义的情况下，于digest循环结束的时候获得表达式的值。如果表达式的值在digest循环执行过程中就被赋值了，在同一个digest循环中会再次将其设置为为未定义的。此时表达式未被填充，在digest循环结束之前仍将会被监察。

  1、若指定的表达式是以`::`开头，在进入digest循环的时候，对表达式进行脏数据检验，并将值存为V

  2、如果V的值是未定义的，将这个表达式的值标记为稳定的并执行一个任务调度，在退出digest循环的时候，注销对此表达式的监察。


  3、按正常的方式执行digest

  4、当digest循环完成后，所有已经被赋值的表达式会注销监察任务。对其中每一个在注销时会检查值是否已经定义并有值。如果是，那么注销监察。否则会在以后的digest循环中，从算法的第一步开始，对被监察的表达式进行脏数据检查。


## 何时更好的利用单次绑定

在插入绑定文字和属性的时候。如果表达式在赋值后不会修改，那么是适合使用单次绑定的。

```html
  <div name="attr: {{::color}}">text: {{::name}}</div>
```

当使用双向绑定指令，参数不需要改变的情况下。

```js
someModule.directive('someDirective', function() {
  return {
    scope: {
      name: '=',
      color: '@'
    },
    template: '{{name}}: {{color}}'
  };
});
```

```html
  <div some-directive name=“::myName” color=“My color is {{::myColor}}”></div>
```


在使用的指令以表达式为参数的情况下。

```html
<ul>
  <li ng-repeat="item in ::items">{{item.name}};</li>
</ul>
```