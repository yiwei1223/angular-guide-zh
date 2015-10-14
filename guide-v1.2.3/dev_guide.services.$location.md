@ngdoc overview
@name Developer Guide: Angular Services: Using $location
@description

翻译者：[@asnowwolf](https://github.com/asnowwolf)

# $location是做什么的？

`$location`服务负责解析浏览器地址栏中的URL（基于{@link
https://developer.mozilla.org/en/window.location window.location}），以便你的应用可以访问它。
这是一个双向同步机制 —— 对地址栏URL的任何修改都会被映射到$location服务中，对$location的任何修改也同样会被映射到地址栏。

**$location服务：**

- 导出在浏览器地址栏中的当前地址，以便你
  - 监视与观察此地址的变更
  - 修改URL
- 当用户做下列操作时，维持`$location`和浏览器当前URL之间的同步
  - 在浏览器地址栏中修改地址
  - 在浏览器中点击前进、后退按钮，或点击“历史”中的链接
  - 在页面上点击一个链接
- 把URL对象表示成一组方法（protocol, host, port, path, search, hash）

## $location和window.location的对比

<table class="table">
<thead>

  <tr>
    <th class="empty-corner-lt"></th>
    <th>window.location</th>
    <th>$location服务</th>
  </tr>

</thead>
<tbody>

  <tr>
    <td class="head">用途</td>
    <td>允许你访问浏览器的当前地址</td>
    <td>相同</td>
  </tr>

  <tr>
    <td class="head">API</td>
    <td>导出带有属性的“原始”对象，它可以被直接修改</td>
    <td>导出jQuery风格的setter和getter方法</td>
  </tr>

  <tr>
    <td class="head">和Angular应用的生命周期整合在一起</td>
    <td>无</td>
    <td>了解所有Angular生命周期中的各个阶段，与$watch集成 ...</td>
  </tr>

  <tr>
    <td class="head">与HTML5 API无缝整合</td>
    <td>否</td>
    <td>是（对老浏览器，具有退回机制，译注：新式使用Push State API，老式使用#hashbang方式）</td>
  </tr>

  <tr>
    <td class="head">带有应用宿主页面的上下文(context)路径</td>
    <td>否 - window.location.path将返回"/docroot/actual/path"</td>
    <td>是 - $location.path()返回"/actual/path"</td>
  </tr>

</tbody>
</table>

## 我该什么时候使用$location？

只要你的应用想更改当前地址，或者你想要修改浏览器的当前地址，就用它吧！

## 什么时候不该用它？

在浏览器地址变化的时候，它不会导致全页面刷新。要想在地址变化之后重载整个页面，请使用底层API`$window.location.href`。

# API的通用概览

`$location`服务可能具有不同的行为，这取决于它被实例化时候你提供给它的配置。默认的配置适用于很多应用，而自定义的配置可以让你启用一些新特性。

一旦`$location`服务被实例化了，你能通过jQuery风格的getter或setter方法与它互动，这样你就能获取或修改浏览器中的当前URL了。

## $location服务配置

要想配置`$location`服务，应该获取{@link api/ng.$locationProvider $locationProvider}对象，然后按下面的方法设置它的参数：

- **html5Mode(模式)**: {boolean}<br />
  `true` - 参见下文中的“HTML5模式”<br />
  `false` - 参见下文中的“Hashbang模式”<br />
  默认值: `false`

- **hashPrefix(前缀)**: {string}<br />
  Hashbang的URL前缀（用于Hashbang模式或者使用Html5模式但运行在老式浏览器中）<br />
  默认值: `""`（空字符串）

### 范例配置
<pre>
$locationProvider.html5Mode(true).hashPrefix('!');
</pre>

## Getter和setter方法

`对于地址中的只读部分（absUrl, protocol, host,
port），$location`服务提供getter方法，对于可变部分（url, path, search, hash），则同时提供getter和setter方法。
<pre>
// 或的当前path
$location.path();

// 修改当前path
$location.path('/newValue')
</pre>

所有的setter方法都返回同一个`$location`对象，以便你进行链式调用。例如，要想一次性修改地址中的几个字段，可以用这样的语法把几个setter串起来：
<pre>$location.path('/newValue').search({key: value});</pre>

## replace方法

有一个特殊的`replace`方法，用来告诉$location服务：你下次和浏览器同步的时候，应该用新地址“替代”最后一条历史记录而不是“追加”新纪录。这在你实现重定向的时候很有用，否则它将“玩死”浏览器的“后退”按钮（“后退”立刻触发一次重定向，我胡汉三又回来了！）。要想改变当前地址却不创建新历史记录，你可以这样调用：

<pre>
  $location.path('/someNewPath');
  $location.replace();
  // 你也可以像这样把它们串起来：$location.path('/someNewPath').replace();
</pre>

注意，setter方法不会立刻更新`window.location`。反之，`$location`服务是挂在与{@link api/ng.$rootScope.Scope 作用域(scope)}的生命周期中的，它会在作用域的`$digest`阶段把多个`$location`调用合并到一起，然后“提交”到`window.location`对象中。由于对$location的状态的多次修改会被合并成一次修改通知浏览器，在一个生命周期内你只要调用了一次`replace`操作就可以“提交”一个replace操作(“替换”而不是“追加”历史记录)。浏览器地址一旦被更新，$location服务器就会重置`replace()`方法设置的标识，将来的修改将会追加新的历史记录，除非再次调用`replace()`。

### setter和字符编码

你可以给`$location`服务传入特殊字符，它将根据{@link http://www.ietf.org/rfc/rfc3986.txt RFC 3986}中指定的规则进行编码。
这会在执行下列方法时自动触发：

- 所有通过`$location`的setter方法（`path()`, `search()`, `hash()`）传入的值都将被编码。
- getter方法（不带任何参数调用`path()`, `search()`, `hash()`方法）返回的值将被解码。
- 调用`absUrl()`方法的返回值是一个编码过的字符串。
- 调用`url()`方法返回的值是path，search和hash，形如`/path?search=a&b=c#hash`，各个组成部分也都是编码过的。

# Hashbang和HTML5模式

`$location`服务有两种配置模式，它将控制浏览器地址栏中的地址显示格式：**Hashbang模式**（默认）和**HTML5模式**，
后者基于HTML5的{@link http://www.w3.org/TR/html5/history.html 历史(history)API}。无论哪种模式，应用层都使用相同的API，
`$location`服务会自己找出合适的URL格式和浏览器API，来完成修改浏览器地址和进行历史管理的工作。

<img src="img/guide/hashbang_vs_regular_url.jpg">

<table class="table">
<thead>

  <tr>
    <th class="empty-corner-lt"></th>
    <th>Hashbang模式</th>
    <th>HTML5模式</th>
  </tr>

</thead>
<tbody>

  <tr>
    <td class="head">配置项</td>
    <td>默认</td>
    <td>{ html5Mode: true }</td>
  </tr>

  <tr>
    <td class="head">URL格式</td>
    <td>在所有浏览器中使用hashbang格式</td>
    <td>现代浏览器中使用标准URL格式，老式浏览器中使用hashbang格式</td>
  </tr>

  <tr>
    <td class="head">&lt;a href=""&gt; link重写</td>
    <td>否</td>
    <td>是</td>
  </tr>

  <tr>
    <td class="head">需要服务端配置</td>
    <td>否</td>
    <td>是</td>
  </tr>
</tbody>
</table>

## Hashbang模式(缺省模式)

在这种模式中，`$location`在所有浏览器中都使用Hashbang地址

### 范例

<pre>
it('显示范例', inject(
  function($locationProvider) {
    $locationProvider.html5Mode(false);
    $locationProvider.hashPrefix('!');
  },
  function($location) {
    // 打开http://example.com/base/index.html#!/a
    $location.absUrl() == 'http://example.com/base/index.html#!/a'
    $location.path() == '/a'

    $location.path('/foo')
    $location.absUrl() == 'http://example.com/base/index.html#!/foo'

    $location.search() == {}
    $location.search({a: 'b', c: true});
    $location.absUrl() == 'http://example.com/base/index.html#!/foo?a=b&c'

    $location.path('/new').search('x=y');
    $location.absUrl() == 'http://example.com/base/index.html#!/new?x=y'
  }
));
</pre>

### 把应用献给爬虫

要想让你的Ajax应用能够被搜索引擎找到，你必须在HTML文档的head部分添加一个特殊的meta标记：
<pre><meta name="fragment" content="!" /></pre>

这将导致爬虫机器人使用`_escaped_fragment_`参数请求你的链接，以便你的服务器能识别出爬虫，并且给他一个HTML快照。
更多信息，参见{@link http://code.google.com/web/ajaxcrawling/docs/specification.html 让Ajax应用也能被抓取}。

## HTML5模式

在HTML5模式中，`$location`服务的getter和setter方法通过HTML5 history API与浏览器地址栏互动，
它允许你使用标准的URL path和search格式来代替等价的hashbang模式。如果浏览器不支持HTML5 history API，
那么`$location`服务将自动退回到使用hashbang URL。这将把你从担心用户的浏览器是否支持history API中解放出来，
`$location`服务将会透明的帮你选择最佳的可用形式。

- 在一个老式浏览器中使用标准url格式 -> 重定向到hashbang URL
- 在一个现代浏览器中使用hashbang格式 -> 用标准URL形式进行重写

### 范例

<pre>
it('显示范例', inject(
  function($locationProvider) {
    $locationProvider.html5Mode(true);
    $locationProvider.hashPrefix('!');
  },
  function($location) {
    // 在支持HTML5历史API的浏览器中:
    // 打开 http://example.com/#!/a -> 将被重写为 http://example.com/a
    // (替换 http://example.com/#!/a 的历史记录)
    $location.path() == '/a'

    $location.path('/foo');
    $location.absUrl() == 'http://example.com/foo'

    $location.search() == {}
    $location.search({a: 'b', c: true});
    $location.absUrl() == 'http://example.com/foo?a=b&c'

    $location.path('/new').search('x=y');
    $location.url() == 'new?x=y'
    $location.absUrl() == 'http://example.com/new?x=y'

    // 在不支持HTML5历史API的浏览器中:
    // 打开 http://example.com/new?x=y -> 将被重定向到 http://example.com/#!/new?x=y
    // (替换 http://example.com/new?x=y 的历史记录)
    $location.path() == '/new'
    $location.search() == {x: 'y'}

    $location.path('/foo/bar');
    $location.path() == '/foo/bar'
    $location.url() == '/foo/bar?x=y'
    $location.absUrl() == 'http://example.com/#!/foo/bar?x=y'
  }
));
</pre>

### 退回到老式浏览器

对于支持HTML5 history API的浏览器，`$location`使用HTML5 history API来修改path和search部分。
如果浏览器不支持history API，`$location`提供hashbang URL。这会把你从不得不担心用户的浏览器是否支持history API中解放出来，
`$location`服务为你透明的完成这一切。

### HTML链接重写

如果你正在使用HTML5 history API模式，你需要在不同的浏览器中使用不同的链接，但是你要做的只是制定一个标准URL链接形式，
比如：`<a href="/some?foo=bar">某链接</a>`

当用户点击这个链接时，

- 在老式浏览器中，地址变为`/index.html#!/some?foo=bar`
- 在现代浏览器中，地址变为`/some?foo=bar`

在下面的例子中，链接不会被重写，而是会在当前页面中执行一次全页面刷新。

- 链接包含`target`属性<br>
  例如： `<a href="/ext/link?a=b" target="_self">某链接</a>`
- 指向其他域名的绝对链接<br>
  例如: `<a href="http://angularjs.org/">某链接</a>`
- 当通过base元素定义了页面基地址时，以'/'开头，但是指向其他基地址的链接<br>
  例如: `<a href="/not-my-base/link">某链接</a>`
  
当在域名的根路径下运行Angular时，可能还有其他普通应用在同一个目录下，"otherwise"路由将会尝试处理所有URL，
甚至包括映射到静态文件的那些。

要想阻止这种行为，你可以把你的base元素的href属性到`<base href=".">`，然后，给所有应该由你处理的链接加上`.`前缀。
这样，那些没有用`.`做前缀的地址将不会再被Angular路由，也就不会再被你的`$routeProvider`中配置的`otherwise`规则拦截到了。

### 服务端

使用HTML5模式需要在服务端重写URL，通常，你要把所有链接都转给应用的入口点（比如index.html）。

### 让爬虫抓取你的应用

如果你想你的AJAX应用被web爬虫索引到，你需要添加下列meta标签到你文档中的HEAD区：
<pre><meta name="fragment" content="!" /></pre>

这个语句会导致爬虫在请求链接时带上一个空的`_escaped_fragment_`参数，以便你的服务器可以识别出爬虫，
并且给它提供一个HTML快照。更多信息，参见{@link http://code.google.com/web/ajaxcrawling/docs/specification.html 让AJAX应用可以被抓取}

### 相对链接

注意检查所有相对链接、图片、脚本等。你或者要在主页面的head区通过`<base href="/my-base">`指定一个url基地址，或者到处使用绝对路径（用`/`开头的）。
因为相对路径将根据当前页面的初始绝对地址解析为绝对路径，而这通常与此应用的根路径是不同的。

强烈建议在根路径下运行使用HTML5历史API模式的Angular应用，它会处理好所有关于相对路径的问题。

### 在不同的浏览器中使用链接

因为HTML5模式下的路径重写能力，你的用户也可以在老式浏览器中打开标准url链接，或者在现代浏览器中打开hashbang链接：

- 现代浏览器将会使用标准URL形式重写hashbang地址。
- 老式浏览器将会把标准URL重定向到hashbang URL。

### 范例

接下来你将看到两个`$location`实例。全都使用**HTML5模式**但运行于不同的浏览器,你将看到它们的区别。
这两个`$location`服务连接到虚拟浏览器中,每个input输入框表示一个浏览器的地址栏。

注意，你在第一个浏览器中输入hashbang url的时候（标准URL也一样）,它不会重写/重定向到标准或hashbang形式，
因为这种转换只会发生在页面加载时解析初始URL的那一步。

在这个例子中，我们使用`<base href="/base/index.html" />`
<doc:example>
<doc:source source="false">

<div ng-non-bindable class="html5-hashbang-example">
  <div id="html5-mode" ng-controller="Html5Cntl">
    <h3>带有历史API的浏览器</h3>
    <div ng-address-bar browser="html5"></div><br><br>
    $location.protocol() = {{$location.protocol()}}<br>
    $location.host() = {{$location.host()}}<br>
    $location.port() = {{$location.port()}}<br>
    $location.path() = {{$location.path()}}<br>
    $location.search() = {{$location.search()}}<br>
    $location.hash() = {{$location.hash()}}<br>
    <a href="http://www.example.com/base/first?a=b">/base/first?a=b</a> |
    <a href="http://www.example.com/base/sec/ond?flag#hash">sec/ond?flag#hash</a> |
    <a href="/other-base/another?search">外部链接</a>
  </div>

  <div id="hashbang-mode" ng-controller="HashbangCntl">
    <h3>没有历史API的浏览器</h3>
    <div ng-address-bar browser="hashbang"></div><br><br>
    $location.protocol() = {{$location.protocol()}}<br>
    $location.host() = {{$location.host()}}<br>
    $location.port() = {{$location.port()}}<br>
    $location.path() = {{$location.path()}}<br>
    $location.search() = {{$location.search()}}<br>
    $location.hash() = {{$location.hash()}}<br>
    <a href="http://www.example.com/base/first?a=b">/base/first?a=b</a> |
    <a href="http://www.example.com/base/sec/ond?flag#hash">sec/ond?flag#hash</a> |
    <a href="/other-base/another?search">外部链接</a>
  </div>
</div>

<script>
  function FakeBrowser(initUrl, baseHref) {
    this.onUrlChange = function(fn) {
      this.urlChange = fn;
    };

    this.url = function() {
      return initUrl;
    };

    this.defer = function(fn, delay) {
      setTimeout(function() { fn(); }, delay || 0);
    };

    this.baseHref = function() {
      return baseHref;
    };

    this.notifyWhenOutstandingRequests = angular.noop;
  }

  var browsers = {
    html5: new FakeBrowser('http://www.example.com/base/path?a=b#h', '/base/index.html'),
    hashbang: new FakeBrowser('http://www.example.com/base/index.html#!/path?a=b#h', '/base/index.html')
  };

  function Html5Cntl($scope, $location) {
    $scope.$location = $location;
  }

  function HashbangCntl($scope, $location) {
    $scope.$location = $location;
  }

  function initEnv(name) {
    var root = angular.element(document.getElementById(name + '-mode'));
    // 我们必须移除与这个元素相关的注入器($injector)，否则angular会认为它已经启动(bootstrap)过了
    root.data('$injector', null);
    angular.bootstrap(root, [function($compileProvider, $locationProvider, $provide){
      $locationProvider.html5Mode(true).hashPrefix('!');

      $provide.value('$browser', browsers[name]);
      $provide.value('$sniffer', {history: name == 'html5'});

      $compileProvider.directive('ngAddressBar', function() {
        return function(scope, elm, attrs) {
          var browser = browsers[attrs.browser],
              input = angular.element('<input type="text" style="width: 400px">').val(browser.url()),
              delay;

          input.on('keypress keyup keydown', function() {
            if (!delay) {
              delay = setTimeout(fireUrlChange, 250);
            }
          });

          browser.url = function(url) {
            return input.val(url);
          };

          elm.append('Address: ').append(input);

          function fireUrlChange() {
            delay = null;
            browser.urlChange(input.val());
          }
        };
      });
    }]);
    root.on('click', function(e) {
      e.stopPropagation();
    });
  }

  initEnv('html5');
  initEnv('hashbang');
</script>

</doc:source>
</doc:example>


# 附加说明

## 页面重载与导航

`$location`服务只允许你修改URL，却不会帮你重新加载页面。如果你需要修改地址并重新加载页面，或者需要导航到其他页面，
请使用底层API：{@link api/ng.$window $window.location.href}。

## 在作用域(scope)生命周期之外使用$location

`$location`知道Angular的{@link api/ng.$rootScope.Scope 作用域(scope)}的生命周期。当浏览器中的地址变化时，
它会更新`$location`对象，并且调用`$apply`，以便所有的$watchers和$observers都能得到通知。

当你在`$digest`阶段中修改`$location`的时候，一切如常：`$location`会把这些修改反映给浏览器，
并且通知所有$watchers / $observers。
如果你想在Angular之外（比如，通过DOM事件或者在测试容器中）修改`$location`，你必须调用`$apply`来告知这项改动。

## $location.path()与!或/前缀

Path永远会以正斜杠（`/`）开头，`$location.path()`作为setter方法调用时，如果没有用正斜杠开头，它会给加上一个。

注意，在hashbang模式中，`!`前缀并不是`$location.path()`的一部分；它只是hash“前缀”。

# 带有$location服务的测试

如果在测试期间使用`$location`服务，你将处于Angular的{@link
api/ng.$rootScope.Scope 作用域(scope)}之外。这就意味着你有责任调用`scope.$apply()`方法。

<pre>
describe('serviceUnderTest', function() {
  beforeEach(module(function($provide) {
    $provide.factory('serviceUnderTest', function($location){
      // 随便做点什么...
    });
  });

  it('should...', inject(function($location, $rootScope, serviceUnderTest) {
    $location.path('/new/path');
    $rootScope.$apply();

    // 验证这个服务应该做的事...

  }));
});
</pre>


# 从AngularJS的早期版本中迁移过来

在Angular的早期版本中，`$location`使用`hashPath`或`hashSearch`来处理path和search方法。
而这个版本中，`$location`使用`path`和`search`方法，然后，在需要时，它会根据自己获得的信息来组合出
hashbang URL（比如`http://server.com/#!/path?search=a`）

## 你代码中的改动

<table class="table">
  <thead>
    <tr class="head">
      <th>在应用程序范围内导航</th>
      <th>改为</th>
    </tr>
  </thead>

  <tbody>
    <tr>
      <td>$location.href = value<br />$location.hash = value<br />$location.update(value)<br
/>$location.updateHash(value)</td>
      <td>$location.path(path).search(search)</td>
    </tr>

    <tr>
      <td>$location.hashPath = path</td>
      <td>$location.path(path)</td>
    </tr>

    <tr>
      <td>$location.hashSearch = search</td>
      <td>$location.search(search)</td>
    </tr>

    <tr class="head">
      <td>导航到应用程序外部</td>
      <td>使用底层API</td>
    </tr>

    <tr>
      <td>$location.href = value<br />$location.update(value)</td>
      <td>$window.location.href = value</td>
    </tr>

    <tr>
      <td>$location[protocol | host | port | path | search]</td>
      <td>$window.location[protocol | host | port | path | search]</td>
    </tr>

    <tr class="head">
      <td>读取</td>
      <td>改为</td>
    </tr>

    <tr>
      <td>$location.hashPath</td>
      <td>$location.path()</td>
    </tr>

    <tr>
      <td>$location.hashSearch</td>
      <td>$location.search()</td>
    </tr>

    <tr>
      <td>$location.href<br />$location.protocol<br />$location.host<br />$location.port<br
/>$location.hash</td>
      <td>$location.absUrl()<br />$location.protocol()<br />$location.host()<br />$location.port()<br
/>$location.path() + $location.search()</td>
    </tr>

    <tr>
      <td>$location.path<br />$location.search</td>
      <td>$window.location.path<br />$window.location.search</td>
    </tr>
  </tbody>
</table>

## $location的双向绑定

Angular不再支持到“方法”的双向绑定（参见{@link
https://github.com/angular/angular.js/issues/404 issue}）
如果你确实需要到$location的双向绑定（在一个输入框中使用{@link api/ng.directive:input.text
ngModel}指令），你需要制定一个额外的中介model（比如自定义一个`locationPath`属性），
并且使用两个监视器，把$location的变动通知彼此。例如：

<example>
<file name="index.html">
<div ng-controller="LocationController">
  <input type="text" ng-model="locationPath" />
</div>
</file>
<file name="script.js">
function LocationController($scope, $location) {
  $scope.$watch('locationPath', function(path) {
    $location.path(path);
  });
  $scope.$watch(function() {
    return $location.path();
  }, function(path) {
    $scope.locationPath = path;
  });
}
</file>
</example>

# 相关API

* {@link api/ng.$location $location API}

译者注:

  1. 关于angularSEO，可以看见[prerender](http://prerender.io/),以及blog[prerender-SPA程序的SEO优化策略](http://greengerong.github.io/blog/2013/12/08/prerender-seo-for-single-page-application/)
  2. 对url的局部改变路由使用$lcoation,如果重新加载页面或者导航到其他页面请使用$window.location.href.
