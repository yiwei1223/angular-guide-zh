@ngdoc overview
@name 过滤器
@description
翻译者:[@NigelYao](https://github.com/NigelYao)

过滤器用来格式化表达式中的值。它可以用在视图模板(templates)、控制器(controllers)、服务(services)和指令(directives)中。我们很容易就能自定义过滤器。

过滤器的API可以在{@link api/ng.$filterProvider 'filterProvider'}中找到。

## 在模板中使用过滤器

过滤器可以应用在视图模板中的表达式中，按如下的格式：

        {{ 表达式 | 过滤器名 }}

例如，在"{{ 12 | currency }}"标记中格式化了数字12作为一种货币的形式来显示，它使用了{@link api/ng.filter:currency `currency`}过滤器。格式化之后的结果是"$12.00"。


过滤器可以应用在另外一个过滤器的结果之上。这叫做“链式”使用，按如下格式：

        {{ 表达式 | 过滤器1 | 过滤器2 | ... }}

过滤器可以拥有（多个）参数，按如下格式：

        {{ 表达式 | 过滤器:参数1:参数2:... }}

例如，在“{{ 1234 | number:2 }}”的标记中格式化显示了数字1234为小数点后两位，使用了{@link api/ng.filter:number `number`}过滤器。显示的结果为"1,234.00"。

##在控制器和服务中使用过滤器

你同样可以在控制器和服务中使用过滤器。在这种情况下，在你的控制器或者服务中添加以“<过滤器名>Filter”为名的依赖。例如，使用"numberFilter"为依赖时，会相应的注入number过滤器。

下面的例子展示了一个名为filter的过滤器。
这个过滤器根据不同的参数将一个数组拆分成多个子数组。这个过滤器可以在模板中以{{ctrl.array | filter:'a'}}`的方式来使用，这会以'a'作为查询字符串来进行过滤。但是，在视图模板中使用过滤器会在每次的更新中重新调用过滤器，当数组很大的时候，开销会很大。

紧接着下面的例子在控制器中直接调用了这个过滤器。
使用这种方式，控制器可以在需要的时候手动调用（例如在从后台获取数据或者过滤器中的表达式有改变的时候）。

<doc:example module="FilterInControllerModule">
<doc:source>
<script>
  angular.module('FilterInControllerModule', []).
    controller('FilterController', ['filterFilter', function(filterFilter) {
      this.array = [
        {name: 'asnowwolf'},
        {name: 'why520crazy'},
        {name: 'joe'},
        {name: 'ckken'},
        {name: 'lightma'},
        {name: 'FrankyYang'}
      ];
      this.filteredArray = filterFilter(this.array, 'a');
    }]);
</script>

<div ng-controller="FilterController as ctrl">
  <div>
    All entries:
    <span ng-repeat="entry in ctrl.array">{{entry.name}} </span>
  </div>
  <div>
    Entries that contain an "a":
    <span ng-repeat="entry in ctrl.filteredArray">{{entry.name}} </span>
  </div>
</div>
</doc:source>
</doc:example>



## 创建自定义过滤器

创建自定义过滤器的过程很简单:仅仅需要在模块中注册一个新的过滤器工厂方法。其中使用了{@link api/ng.$filterProvider `filterProvider`}。这个工厂方法应该返回一个以输入值为第一个参数的新过滤方法，过滤器中的参数都会作为附加参数传递给它。

下面的示例过滤器倒转显示一个字符串。另外，它可以根据参数把字母全部大写。

<example module="myReverseModule">
  <file name="index.html">
    <div ng-controller="Controller">
      <input ng-model="greeting" type="text"><br>
      未添加过滤器: {{greeting}}<br>
      倒转: {{greeting|reverse}}<br>
      倒转 + 大写: {{greeting|reverse:true}}<br>
    </div>
  </file>

  <file name="script.js">
    angular.module('myReverseModule', [])
      .filter('reverse', function() {
        return function(input, uppercase) {
          input = input || '';
          var out = "";
          for (var i = 0; i < input.length; i++) {
            out = input.charAt(i) + out;
          }
          // 根据可选参数处理
          if (uppercase) {
            out = out.toUpperCase();
          }
          return out;
        };
      })
      .controller('Controller', ['$scope', function($scope) {
        $scope.greeting = 'hello';
      }]);
  </file>
</example>

译者注：

  1. 建议filter命名为小写驼峰命名.
  2. filter是个无状态的工具函数，并且尽量少的逻辑(性能问题，filter的触发频繁度)
