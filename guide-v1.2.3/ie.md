@ngdoc overview
@name Developer Guide: Internet Explorer 兼容性
@description

翻译者：[@greengerong](https://github.com/greengerong)

# 概述

这篇文档主要描述Internet Explorer (IE) 在处理自动以html attributes和tag的一些特立独行的行为。如果你希望让
你的应用程序工作在IE 8 或者更早的IE版本，那么你就需要了解本文的内容。


Angular项目当前致力于支持和修复一些IE 8 及以上版本的bug(译者注：angular 1.3已经不再支持IE 8 {@link http://blog.angularjs.org/2013/12/angularjs-13-new-release-approaches.html})。项目的持续集成服务器会在IE 8下运行所有的测试,可以参见{@link http://ci.angularjs.org }


测试不会运行在IE7 及以下版本,并且我们不会保证Angular将会工作在这些版本上。AngularJS的部分功能子集仍然可以工作，
你可以针对你的项目应用自由选择。


针对IE7 或者更早版本的issues core team也不会给予更多的时间去处理这些问题.[GitHub](https://github.com/angular/angular.js/issues/4974)

# 简化版本

让你的Angular程序能在IE上很好的工作，首先需要确保:

  1. 在IE7或者以下版本需要JSON.stringify兼容包。你可以引入[JSON2](https://github.com/douglascrockford/JSON-js)或者
     [JSON3](http://bestiejs.github.com/json3/)来解决这个问题.

     <pre>
       <!doctype html>
       <html xmlns:ng="http://angularjs.org">
         <head>
           <!--[if lte IE 8]>
             <script src="/path/to/json2.js"></script>
           <![endif]-->
         </head>
         <body>
           ...
         </body>
       </html>
     </pre>

  2. 添加`id="ng-app"` 在根元素节点，使其能和`ng-app` attribute结合使用.

     <pre>
       <!doctype html>
       <html xmlns:ng="http://angularjs.org" id="ng-app" ng-app="optionalModuleName">
         ...
       </html>
     </pre>

  3. 尽量不要使用自定义节点例如`<ng:view>`（用属性Attribute方式代替）

  4. 如果你由于语义或者第三方的Angular组件需要使用tag的方式的话,那么你必须按照如下步骤 make IE happy(译者注:或者[ie-shv](https://github.com/angular-ui/ui-utils/tree/master/modules/ie-shiv)):

     <pre>
       <!doctype html>
       <html xmlns:ng="http://angularjs.org" id="ng-app" ng-app="optionalModuleName">
         <head>
           <!--[if lte IE 8]>
             <script>
               document.createElement('ng-include');
               document.createElement('ng-pluralize');
               document.createElement('ng-view');
     
               // Optionally these for CSS
               document.createElement('ng:include');
               document.createElement('ng:pluralize');
               document.createElement('ng:view');
             </script>
           <![endif]-->
         </head>
         <body>
           ...
         </body>
       </html>
     </pre>

重要的部分:
  
  * `xmlns:ng`- *命名空间* - 你需要为每一个将使用的自定义tag注册一个命名空间(译者注:IE作为严格xml模式解析).

  * `document.createElement(yourTagName)` - *自定义节点创建* - 由于这只是老版本的IE issues，所以你需要按条件加载这些脚本(IE低版本特有的条件注释)。对于每一个需要使用的没有注册命名空间以及非HTML定义的tag你需要利用它来预申明来make IE happy。


# 详尽版本

IE在处理关于非标准HTML tag 的问题主要由两类，每种类型又其自己的修复方式.

  * If the tag name starts with `my:` prefix then it is considered an XML namespace and must
    have corresponding namespace declaration on `<html xmlns:my="ignored">`

  * 以`my:`为前缀的tag 考虑到严格的XML命名空间，你必须有相应的命名空间申明,如`<html xmlns:my="ignored">`。

  * If the tag has no `:` but it is not a standard HTML tag, then it must be pre-created using
    `document.createElement('my-tag')`

  * 没有`:`的非标准HTML tag, 你需要使用`document.createElement('my-tag')`来预申明改节点(译者注:ie-shv)。

  * If you are planning on styling the custom tag with CSS selectors, then it must be
    pre-created using `document.createElement('my-tag')` regardless of XML namespace.

  * 如果你希望采用CSS选择器的方式，那么你需要使用`document.createElement('my-tag')`预申明，忽略XML命名空间。

## 幸存的佳音


庆幸的是这些问题只会出现在自定义节点上，attribute的方式还能工作。所以`<div my-tag your:tag></div>`在IE中不需要特殊的任何处理.

## IE的DOM解析

假如你拥有位置的tag `mytag`(`my:tag` 或者 `my-tag` ):

<pre>
  <html>
    <body>
      <mytag>some text</mytag>
    </body>
  </html>
</pre>

正确的DOM解析为：

<pre>
#document
  +- HTML
     +- BODY
        +- mytag
           +- #text: some text
</pre>


期望的结果是在`BODY`节点下有一个`mytag`的子节点，并且在其下有一个`some text`的文本节点。


但是IE却不是这样工作的(假设并未做任何上面的修复方法):

<pre>
#document
  +- HTML
     +- BODY
        +- mytag
        +- #text: some text
        +- /mytag
</pre>

在IE中解析结果是在`BODY`下有两个子节点：


  1. 一个是自关闭的`mytag`。形如自关闭的<br/> tag。对于‘/’是可选的。但是`<br>` 不允许包含任何的子节点，浏览器会把`<br>some
  text</br>`解析为3个节点而不是两个`<br>`和`some text`子节点。


  2. 一个是`some text`的文本节点.正确的解析应该是`mytag`的子节点而不是兄弟节点。


  3. 一个不合法的自关闭的`/mytag`。因为在DOM节点名称不允许使用`/`字符。而且正确的解析结果这不该出现在DOM中，
    是因为它只是描述DOM的结构。
  
## CSS Styling of 自定义节点

使得自定义节点能工作在CSS选择器上，自定义节点必须使用`document.createElement('my-tag')`预创建申明忽略XML命名空间.

<pre>
  <html xmlns:ng="needed for ng: namespace">
    <head>
      <!--[if lte IE 8]>
        <script>
          // needed to make ng-include parse properly
          document.createElement('ng-include');

          // needed to enable CSS reference
          document.createElement('ng:view');
        </script>
      <![endif]-->
      <style>
        ng\:view {
          display: block;
          border: 1px solid red;
        }

        ng-include {
          display: block;
          border: 1px solid blue;
        }
      </style>
    </head>
    <body>
      <ng:view></ng:view>
      <ng-include></ng-include>
      ...
    </body>
  </html>
</pre>


译者注：

 1. 以上的IE兼容性只是针对IE 中关于DOM解析的issues。在Angular开发中大多数情况你还需要引入JavaScript的兼容包，如[es5-shim]（https://github.com/es-shims/es5-shim）来处理String.trim、Array等一些JavaScriptAPI。

 2. 如果你希望在IE6中工作建议使用angular.bootstrap方法来手动启动Angular app。

 3. 部分兼容问题参见[Angularjs开发一些经验总结](http://www.cnblogs.com/whitewolf/archive/2013/03/24/2979344.html)


