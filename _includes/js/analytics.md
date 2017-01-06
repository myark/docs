# 分析

Parse为你获取你的程序的内部细节提供了一些钩子。我们知道那对你了解你应用做了什么，以及一些状况，比如使用频率、时间非常重要。
<!-- Parse provides a number of hooks for you to get a glimpse into the ticking heart of your app. We understand that it's important to understand what your app is doing, how frequently, and when. -->

虽然本节将介绍不同的方法来测试您的应用程序，以充分利用Parse的分析后端，但使用Parse存储和检索数据的开发人员已经可以利用Parse上的指标优势。
<!-- While this section will cover different ways to instrument your app to best take advantage of Parse's analytics backend, developers using Parse to store and retrieve data can already take advantage of metrics on Parse. -->

无需实施任何客户端逻辑，您可以在应用的信息中心中查看API请求的实时图表和细分（按设备类型，解析类名或REST动词），并保存这些图表过滤器，以便快速访问 您感兴趣的数据。
<!-- Without having to implement any client-side logic, you can view real-time graphs and breakdowns (by device type, Parse class name, or REST verb) of your API Requests in your app's dashboard and save these graph filters to quickly access just the data you're interested in. -->


## 自定义分析

`Parse.Analytics`也允许你跟踪自由形式的事件，用一些`string`键和值。 这些额外的维度允许您通过应用的信息中心对自定义事件进行细分。
<!-- `Parse.Analytics` also allows you to track free-form events, with a handful of `string` keys and values. These extra dimensions allow segmentation of your custom events via your app's Dashboard. -->

假设您的应用程式提供公寓资讯的搜寻功能，而且您想追踪使用这项功能的频率，以及一些额外的中继资料。
<!-- Say your app offers search functionality for apartment listings, and you want to track how often the feature is used, with some additional metadata. -->

<pre><code class="javascript">
var dimensions = {
  // Define ranges to bucket data points into meaningful segments
  priceRange: '1000-1500',
  // Did the user filter the query?
  source: 'craigslist',
  // Do searches happen more often on weekdays or weekends?
  dayType: 'weekday'
};
// Send the dimensions to Parse along with the 'search' event
Parse.Analytics.track('search', dimensions);
</code></pre>

`Parse.Analytics`甚至可以用作一个轻量级的错误跟踪器＆mdash; 只需调用以下内容，您就可以访问应用程序中错误的速率和频率（按错误代码细分）的概述：
<!-- `Parse.Analytics` can even be used as a lightweight error tracker &mdash; simply invoke the following and you'll have access to an overview of the rate and frequency of errors, broken down by error code, in your application: -->

<pre><code class="javascript">
var codeString = '' + error.code;
Parse.Analytics.track('error', { code: codeString });
</code></pre>

注意，Parse当前只存储每次调用`Parse.Analytics.track（）`的前八个维度对。
<!-- Note that Parse currently only stores the first eight dimension pairs per call to `Parse.Analytics.track()`. -->
