@ngdoc overview
@name Developer Guide
@description

# 关于Angular Guide翻译的说明

一次偶然的机会，我们在Angular.js中文社区群里相遇，更是一次巧妙的交谈，大家对于Angular官方的Guide最新版本没有中文版本表示无助，
所以，本着为社区做贡献的心态，我们临时组织了一个Angular 开发指南翻译团队。

我们是一群技术的热爱者，更是一群Angular的使用者，但是我们的翻译不代表官方，如有任何问题随时联系我们。

目前参加翻译工作的人员有9位，分别是：

ckken，grahamle，NigelYao，asnowwolf，lightma，joeylin，FrankyYang，lrrluo， why520crazy，破狼


负责审核的成员目前有：二当家

# AngularJS 文档指南

你需要知道AngularJS

* {@link guide/introduction 什么是AngularJS?}
* {@link guide/concepts 概述}

## 教程

* {@link tutorial/index AngularJS官方教程}
* [选择Angular的十个理由](http://www.sitepoint.com/10-reasons-use-angularjs/)
* [AngularJS的设计原则(视频)](https://www.youtube.com/watch?v=HCR7i5F5L8c)
* [60分钟入门教程(视频)](http://www.youtube.com/watch?v=i9MHigUZKEM)
* [jQuery程序员学习Angular必读](http://stackoverflow.com/questions/14994391/how-do-i-think-in-angularjs-if-i-have-a-jquery-background)

## 核心概念

### 模板

在Angular应用中，你把通过数据填充页面模板的工作从后端移到了前端，这导致动态更新的页面有了更好的代码结构。下面是你将用到的一些核心概念。

* {@link guide/databinding 数据绑定(Data Binding)}
* {@link guide/expression 表达式(Expression)}
* {@link guide/directive 指令(Directive)}
* {@link api/ngRoute.$route 视图(View)和路由(Route) (参见范例)}
* {@link guide/filter 过滤器(Filter)}
* {@link guide/forms 表单(Form)} 和 [AngularJS的表单概念](http://mrbool.com/the-concepts-of-angularjs-forms/29117)

### 程序结构

* **博客：**[什么时候使用指令(Directive)、控制器(Controller)或服务(Service)](http://kirkbushell.me/when-to-use-directives-controllers-or-services-in-angular/)
* **程序装载：** {@link guide/di 依赖注入(DI, Dependency injection)}
* **把数据模型(Model)导出给视图(View)：** {@link guide/scope 作用域(Scope)}
* **和服务器通讯：** {@link api/ng.$http $http}, {@link api/ngResource.$resource $resource}

### 其他 AngularJS 功能

* **动画：** {@link guide/animations 核心概念}, {@link api/ngAnimate ngAnimate API}, 以及 [AngularJS 1.2中的动画](http://www.yearofmoo.com/2013/08/remastered-animation-in-angularjs-1-2.html)
* **安全：** {@link api/ng.$sce 受限场景转义(SCE)}, {@link api/ng.directive:ngCsp 内容安全策略(CSP)}, {@link api/ngSanitize.$sanitize $sanitize}, [视频](https://www.youtube.com/watch?v=18ifoT-Id54)
* **国际化(i18n)和本地化(l10n)：** {@link guide/i18n Angular的国际化(i18n)和本地化(l10n)指南}, {@link api/ng.filter:date 日期过滤器}, {@link api/ng.filter:currency 货币过滤器}, [创建多语种支持](http://www.novanet.no/blog/hallstein-brotan/dates/2013/10/creating-multilingual-support-using-angularjs/)
* **移动开发：** {@link api/ngTouch 触控(Touch)事件}

### 测试

* **单元测试：** [使用Karma（视频）](http://www.youtube.com/watch?v=YG5DEzaQBIc), {@link guide/dev_guide.unit-testing 单元测试}, {@link guide/dev_guide.services.testing_services 测试服务}, [在WebStorm中使用Karma](http://blog.jetbrains.com/webstorm/2013/10/running-javascript-tests-with-karma-in-webstorm-7/)
* **场景测试：** [Protractor](https://github.com/angular/protractor)

## 具体的主题

* **登录：**[Google范例](https://developers.google.com/+/photohunt/python), [Facebook范例](http://blog.brunoscopelliti.com/facebook-authentication-in-your-angularjs-web-app), [认证策略](http://blog.brunoscopelliti.com/deal-with-users-authentication-in-an-angularjs-web-app), [unix风格的认证](http://frederiknakstad.com/authentication-in-single-page-applications-with-angular-js/)
* **移动开发：** [Angular移动开发指南](http://www.ng-newsletter.com/posts/angular-on-mobile.html), [PhoneGap](http://devgirl.org/2013/06/10/quick-start-guide-phonegap-and-angularjs/)
* **其他语言：** [CoffeeScript](http://www.coffeescriptlove.com/2013/08/angularjs-and-coffeescript-tutorials.html), [Dart](https://github.com/angular/angular.dart.tutorial/wiki)
* **实时应用：**[Socket.io](http://www.creativebloq.com/javascript/angularjs-collaboration-board-socketio-2132885), [OmniBinder](https://github.com/jeffbcross/omnibinder)
* **可视化：** [SVG](http://gaslight.co/blog/angular-backed-svgs), [D3.js](http://www.ng-newsletter.com/posts/d3-on-angular.html)

## 工具

* **调试：** [Batarang](https://chrome.google.com/webstore/detail/angularjs-batarang/ighdmehidhipcmcojjgiloacoafjmpfk?hl=en)
* **测试：** [Karma](http://karma-runner.github.io), [Protractor](https://github.com/angular/protractor)
* **编辑器：** [Webstorm](http://plugins.jetbrains.com/plugin/6971) (以及 [视频](http://www.youtube.com/watch?v=LJOyrSh1kDU)), [Sublime Text](https://github.com/angular-ui/AngularJS-sublime-package), [Visual Studio](http://madskristensen.net/post/angularjs-intellisense-in-visual-studio-2012)
* **工作流：** [Yeoman.io](https://github.com/yeoman/generator-angular) 以及 [Angular Yeoman 指南](http://www.sitepoint.com/kickstart-your-angularjs-development-with-yeoman-grunt-and-bower/)

## 补充 类库

这里是部分在Angular方面提供了明确支持和文档的外部库的简短清单。你可以到[ngmodules.org](http://ngmodules.org/)上找到已知Angular外部库的完整列表。

* **国际化(i18n)：** [angular-translate](http://pascalprecht.github.io/angular-translate/), [angular-gettext](http://angular-gettext.rocketeer.be/)
* **RESTful服务：** [Restangular](https://github.com/mgonto/restangular)
* **SQL和NoSQL后端：** [BreezeJS](http://www.breezejs.com/), [AngularFire](http://angularfire.com/)
* **UI部件：**[KendoUI](http://kendo-labs.github.io/angular-kendo/#/), [UI Bootstrap](http://angular-ui.github.io/bootstrap/), [Wijmo](http://wijmo.com/tag/angularjs-2/)

## 部署

### 概括

* **Javascript最小化：**[背景](http://thegreenpizza.github.io/2013/05/25/building-minification-safe-angular.js-applications/), [ngmin自动处理工具](http://www.thinkster.io/pick/XlWneEZCqY/angularjs-ngmin)
* **跟踪：** [分析(Google Analytics)](http://ngmodules.org/modules/angularytics), [记录前端错误](http://www.bennadel.com/blog/2542-Logging-Client-Side-Errors-With-AngularJS-And-Stacktrace-js.htm)
* **SEO:** [手动](http://www.yearofmoo.com/2012/11/angularjs-and-seo.html), [prerender.io](http://prerender.io/), [Brombone](http://www.brombone.com/), [SEO.js](http://getseojs.com/), [SEO4Ajax](http://www.seo4ajax.com/)

关于prerender的中文资料，详见[破狼的博客](http://greengerong.github.io/blog/2013/12/08/prerender-seo-for-single-page-application/)

### 服务-指定

* **Django：** [入门](http://blog.mourafiq.com/post/55034504632/end-to-end-web-app-with-django-rest-framework), [集成AngularJS与Django](http://django-angular.readthedocs.org/en/latest/integration.html)
* **FireBase：** [AngularFire](http://angularfire.com/), [使用AngularJS和FireBase开发实时应用 (视频)](http://www.youtube.com/watch?v=C7ZI7z7qnHU)
* **Google云平台：**[使用Cloud Endpoints](https://cloud.google.com/resources/articles/angularjs-cloud-endpoints-recipe-for-building-modern-web-applications), [使用Go](https://github.com/GoogleCloudPlatform/appengine-angular-gotodos)
* **Hood.ie:** [Angular 60分钟强力秀](http://www.roberthorvick.com/2013/06/30/todomvc-angularjs-hood-ie-60-minutes-to-awesome/)
* **MEAN开发栈：**[博客](http://blog.mongodb.org/post/49262866911/the-mean-stack-mongodb-expressjs-angularjs-and), [起步](http://thecodebarbarian.wordpress.com/2013/07/22/introduction-to-the-mean-stack-part-one-setting-up-your-tools/), [Google GDL视频](https://developers.google.com/live/shows/913996610) 
* **Rails：**[入门](http://coderberry.me/blog/2013/04/22/angularjs-on-rails-4-part-1/), [集成AngularJS与Rails4](https://shellycloud.com/blog/2013/10/how-to-integrate-angularjs-with-rails-4), [angularjs-rails](https://github.com/hiravgandhi/angularjs-rails)
* **PHP：**[构建RESTful的Web服务](http://blog.brunoscopelliti.com/building-a-restful-web-service-with-angularjs-and-php-more-power-with-resource), [Laravel 4 端到端(视频)](http://www.youtube.com/watch?v=hqAyiqUs93c)

## 学习资源

###书籍
* [AngularJS](http://www.amazon.com/AngularJS-Brad-Green/dp/1449344852) 作者：Brad Green 和 Shyam Seshadri
* [Mastering Web App Development](http://www.amazon.com/Mastering-Web-Application-Development-AngularJS/dp/1782161821) 作者：Pawel Kozlowski 和 Pete Bacon Darwin
* [AngularJS Directives（AngularJS指令详解）](http://www.amazon.com/AngularJS-Directives-Alex-Vanston/dp/1783280336) 作者：Alex Vanston
* [Recipes With AngularJS（AngularJS秘诀）](http://www.amazon.co.uk/Recipes-Angular-js-Frederik-Dietz-ebook/dp/B00DK95V48) 作者：Frederik Dietz
* [Developing an AngularJS Edge](http://www.amazon.com/Developing-AngularJS-Edge-Christopher-Hiller-ebook/dp/B00CJLFF8K) 作者：Christopher Hiller
* [ng-book: The Complete Book on AngularJS（AngularJS大全）](http://ng-book.com/) 作者：Ari Lerner

###视频:
* [egghead.io](http://egghead.io/)
* [Youtube上的Angular专题](http://youtube.com/angularjs)

### 课程
* **免费在线教程：**
  [thinkster.io](http://thinkster.io),
  [CodeAcademy](http://www.codecademy.com/courses/javascript-advanced-en-2hJ3J/0/1)
* **收费在线教程：**
  [Pluralsite (3门课)](http://www.pluralsight.com/training/Courses/Find?highlight=true&searchTerm=angularjs),
  [Tuts+](https://tutsplus.com/course/easier-js-apps-with-angular/),
  [lynda.com](http://www.lynda.com/AngularJS-tutorials/Up-Running-AngularJS/133318-2.html)
* **收费现场培训：**
  [angularbootcamp.com](http://angularbootcamp.com/)

## 帮助

如果你有个问题想得到帮助，最好在[Plunker](http://plnkr.co/), [JSFiddle](http://jsfiddle.net/), 或者类似的站点创建一个示例，并提交给下面的网站：

* [Stackoverflow.com](http://stackoverflow.com/search?q=angularjs)
* [AngularJS邮件列表](https://groups.google.com/forum/#!forum/angular)
* [AngularJS IRC频道](http://webchat.freenode.net/?channels=angularjs&uio=d4)

## 社交渠道

* **日常更新：** [Google+](https://plus.google.com/u/0/+AngularJS) 或 [Twitter](https://twitter.com/angularjs)
* **每周新闻：** [ng-newsletter](http://www.ng-newsletter.com/)
* **Meetups: **[meetup.com](http://www.meetup.com/find/?keywords=angularJS&radius=Infinity&userFreeform=San+Francisco%2C+CA&mcId=z94108&mcName=San+Francisco%2C+CA&sort=member_count&eventFilter=mysugg)
* **官方新闻与发布：**[AngularJS Blog](http://blog.angularjs.org/)

## 贡献给 AngularJS

虽然Angular团队中有好多核心成员都在Google工作，但Angular是一个凝聚了100位贡献者的开源项目，如果你已经准备好成为其中的一员，请阅读{@link misc/contribute AngularJS贡献指南}


## 中文资源

[本Angular业余翻译组的“家”，欢迎加盟](https://github.com/jingyanjiaoliu/angular-guide-zh)

[Angular中文文档的预览站（包括非正式发布的部分）](http://angular.duapp.com/guide)

[angularjs中文社区](http://angularjs.cn) 技术资料、中文文档，这个网站本身也是用angular + nodejs开发的

[破狼Blog](http://greengerong.github.io) [破狼cnBlog](http://www.cnblogs.com/whitewolf/)  一位angular爱好者的博客

[github AngularJS-Learning](https://github.com/jmcunningham/AngularJS-Learning) 包含AngularJS很多资料的github库

QQ交流群1 278252889(满)

QQ交流群2 305739270

## 最后一点

没有找到你想要的？访问[AngularJS-Learning](https://github.com/jmcunningham/AngularJS-Learning)可以找到更多的视频、教程、博客的列表。

如果你认为某些优秀的AngularJS资源应该放在本页面，请在[Google+](https://plus.google.com/u/0/+AngularJS)或[Twitter](https://twitter.com/angularjs)告诉我们。

