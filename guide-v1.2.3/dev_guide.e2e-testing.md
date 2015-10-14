@workInProgress
@ngdoc overview
@name 开发指南：端到端测试(E2E Testing)
@description

翻译者：[@asnowwolf](https://github.com/asnowwolf)

** 如果你正准备新建一个Angular项目，建议你使用{@link https://github.com/angular/protractor Protractor}，它将在不久之后取代现在的模块成为端到端测试的内置模块。 **

现在的应用程序在大小和复杂度方面与日俱增，依靠人工测试来验证新特性、查找bug和进行回归测试已经变得不现实了。

为了解决这个问题，我们创建了Angular场景测试工具（Angular Scenario Runner），它将模拟用户的交互，以帮助你为Angular应用程序进行“体检”。

# 概览

你可以用JavaScript来写场景测试，它描述你的应用程序的工作方式，在各种特定的状态下定义出正确的互动结果。
一个场景由一个或多个`it`块语句组成（可以把这些看做应用程序的“需求”），这些语句由**命令（command）**和**期望(expectation)**组成。
命令负责告诉测试工具（runner）应用程序去做点什么（比如访问一个页面或者点击一个按钮），而期望则告诉测试工具，这个状态下哪些断言（assertion）必须成立（比如某个输入框的值或者当前页面的URL）。
如果某些期望落空了，测试工具就会把相应的`it`语句标记为“失败(failed)”，然后继续执行下一个。
场景（scenarios）还可能包含**beforeEach**和**afterEach**语句，它们将在每个`it`语句之前（或之后）运行 —— 无论这些语句是执行成功了还是失败了。

<img src="img/guide/scenario_runner.png">

除了上面这些内容之外，场景还可能包含一些辅助函数，用来消除各个`it`语句中的重复代码。

下面是一个简单的场景范例：

<pre>
describe('Buzz客户端', function() {
it('应该对结果进行过滤', function() {
  input('user').enter('jacksparrow');
  element(':button').click();
  expect(repeater('ul li').count()).toEqual(10);
  input('filterText').enter('Bees');
  expect(repeater('ul li').count()).toEqual(1);
});
});
</pre>

注意：[`input('user')`](https://github.com/angular/angular.js/blob/master/docs/content/guide/dev_guide.e2e-testing.ngdoc#L119)语句会查找具有`ng-model="user"`属性的`<input>`元素，而不会查找具有`name="user"`属性的！

这个场景描述了一个Buzz客户端的需求，特别是，他应该能够根据用户的输入进行过滤。它先模拟把一个值输入到带有ng-model="user"属性的输入框中，然后，点击页面中唯一的一个按钮，然后，检查是否列出了10条结果。接下来，它在带有ng-model="filterText"属性的输入框中输入文本：'Bees'，最后验证一下这个列表是否已经被过滤得只剩下一条结果。

下面列出了在测试工具(runner)的命令(command)和期望(expectation)语句中可以使用的API。

#API
源码：{@link https://github.com/angular/angular.js/blob/master/src/ngScenario/dsl.js}

## pause()
暂停测试代码的执行，直到你在控制台中调用了`resume()`函数（或者在测试工具的UI中点击了“恢复(resume)”链接）

## sleep(seconds)
让测试代码的执行暂停`seconds`秒。

## browser().navigateTo(url)
把指定的`url`加载到测试框架（译注：测试框架是指用来运行测试的浏览器环境，比如chrome浏览器或phantomjs）中。

## browser().navigateTo(url, fn)
把fn返回的URL加载到测试框架中。此处指定的`url`参数对代码没有实际影响，只用于测试输出。如果目标URL是动态生成的，这种形式会非常有用（即：目标URL在我们写测试的时候是预先无法确定的）。

## browser().reload()
刷新测试框架中的当前页。

## browser().window().href()
返回测试框架中当前页的window.location.href值。

## browser().window().path()
返回测试框架中当前页的window.location.pathname值。

## browser().window().search()
返回测试框架中当前页的window.location.search值。

## browser().window().hash()
返回测试框架中当前页的window.location.hash值（不包括`#`）。

## browser().location().url()
返回测试框架中当前页的{@link api/ng.$location $location.url()}值。

## browser().location().path()
返回测试框架中当前页的{@link api/ng.$location $location.path()}值。

## browser().location().search()
返回测试框架中当前页的{@link api/ng.$location $location.search()}值。

## browser().location().hash()
返回测试框架中当前页的{@link api/ng.$location $location.hash()}值。

## expect(future).{matcher}
断言`future`参数（译注：future就是异步执行模式中的通知对象，会在异步执行完毕时触发回调，类似于$q中的promise）的“值(value)”符合`匹配器(matcher)`的期望。所有API语句都会返回`future`对象，它被执行后会返回一个“值”。`匹配器`是通过`angular.scenario.matcher`定义的，并且通过求出这个`future`对象的“值”来验证是否符合期望。比如：`expect(browser().location().href()).toEqual('http://www.google.com')`。后面的文档中将深入讲解各种可用的匹配器。

## expect(future).not().{matcher}
断言`future`的值不满足`matcher`的要求

## using(selector, label)
限定接下来的语句中元素选择器的所属范围。
（译注：这里的选择器 - `selector`都是指jQuery选择器，参见[jQuery选择器](http://api.jquery.com/category/selectors/)）

## binding(name)
返回匹配指定`name`的第一个绑定对象(binding)的值。
（译注：什么是binding？比如假设模板中有'<div id="element" ng-bind="object"></div>'，则object称为#element的“绑定对象”）

## input(name).enter(value)
在ng-model值是`name`的文本框中输入指定的`value`。

## input(name).check()
选中或反选ng-model值是`name`的检查框。

## input(name).select(value)
在ng-model值是`name`的单选组中选中值为`value`的那个。

## input(name).val()
返回ng-model值是`name`的输入框的当前值。

## repeater(selector, label).count()
返回`selector`选定的repeater（译注：可以简单的理解为ng-repeat）的行数。`label`参数对代码没有实际影响，只用做测试输出。

## repeater(selector, label).row(index)
返回`selector`选定的repeater中，绑定到第`index`行的所有绑定对象(译注：各个绑定对象的值，参见binding函数，一行一般都有多个绑定表达式)构成的数组。`label`参数对代码没有实际影响，只用做测试输出。

## repeater(selector, label).column(binding)
返回`selector`选定的repeater中，由所有绑定到`binding`（译注：传入绑定对象的表达式，比如模板中是{{name + "a"}}，此处就应该用'name + "a"'作为参数，表达式中+前后的空格会被忽略）的列内容构成的数组。`label`参数对代码没有实际影响，只用做测试输出。

## select(name).option(value)
从ng-model值是`name`的select元素中，选择指定`value`值的option。

## select(name).options(value1, value2...)
从ng-model值是`name`的select元素中，选择所有存在于`values`（即：value1, value2...）参数中的option。


## element(selector, label).count()
返回`selector`选定的元素的数量。`label`参数对代码没有实际影响，只用做测试输出。

## element(selector, label).click()
模拟点击`selector`选定的元素。`label`参数对代码没有实际影响，只用做测试输出。

## element(selector, label).query(fn)
用`fn(selectedElements, done)`的形式调用fn函数，`selectedElements`是匹配指定选择器的所有元素，`done`是一个函数，供`fn`结束时调用。`label`参数对代码没有实际影响，只用做测试输出。

## element(selector, label).{method}()
返回在`selector`选定的元素上调用`method()`的结果。`method`可以是下列jquery方法之一：`val`, `text`, `html`, `height`,
`innerHeight`, `outerHeight`, `width`, `innerWidth`, `outerWidth`, `position`, `scrollLeft`,
`scrollTop`, `offset`。`label`参数对代码没有实际影响，只用做测试输出。

## element(selector, label).{method}(value)
在`selector`选定的元素上执行`method(value)`函数。`method`可以是下列jquery方法之一：`val`, `text`, `html`, `height`,
`innerHeight`, `outerHeight`, `width`, `innerWidth`, `outerWidth`, `position`, `scrollLeft`,
`scrollTop`, `offset`。`label`参数对代码没有实际影响，只用做测试输出。

## element(selector, label).{method}(key)
在`selector`选定的元素上执行`method(key)`函数。`method`可以是下列jquery方法之一：`attr`, `prop`, `css`。`label`参数对代码没有实际影响，只用做测试输出。

## element(selector, label).{method}(key, value)
在`selector`选定的元素上执行`method(key, value)`函数。`method`可以是下列jquery方法之一：`attr`, `prop`, `css`。`label`参数对代码没有实际影响，只用做测试输出。

# Matchers
匹配器(matcher)用于和`expect(...)`函数组合起来构成断言，并且可以和`not()`连用来表示否定。例如：`expect(element('h1').text()).not().toEqual('Error')`。

源码: {@link https://github.com/angular/angular.js/blob/master/src/ngScenario/matchers.js}

<pre>
// 值和对象的比较使用与angular.equals相同的规则
expect(value).toEqual(value)

// 简单类型的比较使用===运算符进行精确比较
expect(value).toBe(value)

// 检查value当前是否具有任何已定义的类型（即value !== undefined）
expect(value).toBeDefined()

// 这两个匹配器使用JavaScript标准的真值规则进行判断
expect(value).toBeTruthy()
expect(value).toBeFalsy()

// 检查value是否符合指定的正则表达式。正则表达式既可以用字符串的形式传入，也可以用正则表达式对象的形式（如new RegExp('.*')或/.*/）传入。
expect(value).toMatch(expectedRegExp)

// 使用===精确检查null值
expect(value).toBeNull()

// 内部用Array.indexOf(...)函数检查指定的元素是否包含在当前数组中。
expect(value).toContain(expected)

// 使用`<`和`>`运算符进行数值比较。
expect(value).toBeLessThan(expected)
expect(value).toBeGreaterThan(expected)
</pre>

# 范例
参见{@link https://github.com/angular/angular-seed Angular种子项目}项目中的例子。

## 通过element(...).query(fn)执行有条件的动作

Angular场景化的端到端(E2E)测试，高度支持异步特性，它通过将动作和期望存入队列来隐藏了处理异步结果(future)时的很多复杂度。
你可能需要使用一些有条件的断言或元素选取规则，或者需要某些通用的机制来消除重复代码（重复代码往往表示测试代码中有“坏味道”），这时，你可以通过`element(...).query(fn)`来添加一些有条件的行为。
下列代码将演示这个函数如何通过应用程序的web界面来删除附加的实体（这里的实体是一些领域对象）。

假设应用程序是由两个视图组成的：

 1. *列表视图*列出表中添加的所有实体。
 2. *详情视图*现实实体的细节，还有一个“删除”按钮。点击“删除”按钮的时候，用户重定向回*列表视图*。

<pre>
beforeEach(function () {
  var deleteEntry = function () {
    browser().navigateTo('/entries');

    // 我们需要选择<tbody>元素，他现在没有实体（即：没有<tr>元素）。如果选择器没有匹配到结果，则本测试直接失败。
    element('table tbody').query(function (tbody, done) {
      // ngScenario传给我们的是一个jQuery lite包装之后的元素。我们可以调用它的children()函数获取tbody的所有<tr>元素。
      var children = tbody.children();

      if (children.length > 0) {
        // 如果表格中至少有一个实体，点击链接，转到这个实体的详情页
        element('table tbody a').click();
        // 路由变化之后，点击“删除”按钮。
        element('.btn-danger').click();
      }

      // 如果表格中显示了不止一个实体，则把其他的删除操作排入队列。
      if (children.length > 1) {
        deleteEntry();
      }

      // 别忘了调用`done()`函数，这样ngScenario才会继续执行测试，否则会出现超时错误。
      done();
    });

  };

  // 开始删除实体
  deleteEntry();
});
</pre>

// 为了帮助理解它的工作原理，我们要强调一句：ngScenario的调用不是立刻执行的，而是先排入队列（按照ngScenario中的术语，我们称之为添加“未来动作(future action)”）。如果在表格中我们只有一个实体，那么下列“未来动作”将被排入队列：

<pre>
// 删除实体1
browser().navigateTo('/entries');
element('table tbody').query(function (tbody, done) { ... });
element('table tbody a');
element('.btn-danger').click();
</pre>

对于两个实体，ngScenario将产生下列队列：

<pre>
// 删除实体1
browser().navigateTo('/entries');
element('table tbody').query(function (tbody, done) { ... });
element('table tbody a');
element('.btn-danger').click();

    // 删除实体2
    // 这里的缩进排版用来表示递归调用的层数
    browser().navigateTo('/entries');
    element('table tbody').query(function (tbody, done) { ... });
    element('table tbody a');
    element('.btn-danger').click();
</pre>

# 警告

ngScenario不能和通过调用angular.bootstrap来实现的手动初始化协同工作。你必须使用ng-app指令来启动应用程序。
