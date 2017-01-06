# 入门

如果您尚未设置项目，请转到[快速开始指南]({{page.quickstart}}＃js/native/blank)启动并运行。 您还可以查看我们的[API参考](/Parse-SDK-JS/api/)有关我们的SDK的更多详细信息。
<!-- If you haven't set up your project yet, please [head over to the QuickStart guide]({{ page.quickstart }}#js/native/blank) to get up and running. You can also check out our [API Reference](/Parse-SDK-JS/api/) for more detailed information about our SDK. -->

Parse平台为您的移动应用程序提供完整的后端解决方案。 我们的目标是完全消除编写服务器代码或维护服务器的需要。
<!-- The Parse platform provides a complete backend solution for your mobile application. Our goal is to totally eliminate the need for writing server code or maintaining servers. -->

<div class='tip info'><div>
  如果你想用Parse构建一个React应用程序，我们提供一个特殊的库。 所有文档都可以在
  <!-- If you're looking to build a React application with Parse, we provide a special library for that. All of the documentation is available at the --> <a href="https://github.com/ParsePlatform/ParseReact">GitHub 仓库</a>.
</div></div>

我们的JavaScript SDK最初基于流行的[Backbone.js](http://backbonejs.org/)框架，但它提供了灵活的API，允许它与您喜欢的JS库配对。 我们的目标是最小化配置，让您快速开始在Parse上构建JavaScript和HTML5应用程序。
<!-- Our JavaScript SDK is originally based on the popular [Backbone.js](http://backbonejs.org/) framework, but it provides flexible APIs that allow it to be paired with your favorite JS libraries. Our goal is to minimize configuration and let you quickly start building your JavaScript and HTML5 app on Parse. -->

我们的SDK支持Firefox 23+，Chrome 17+，Safari 5+和IE 10.只有使用HTTPS托管的应用程序才支持IE 9。
<!-- Our SDK supports Firefox 23+, Chrome 17+, Safari 5+, and IE 10. IE 9 is supported only for apps that are hosted with HTTPS. -->

要用Javascript初始化你自己的Parse-Server，你应该用这个替换你当前的初始化代码
<!-- To initialize your own Parse-Server with Javascript, you should replace your current initialization code with this -->

``` javascript
Parse.initialize("YOUR_APP_ID");
Parse.serverURL = 'http://YOUR_PARSE_SERVER:1337/parse'
```
