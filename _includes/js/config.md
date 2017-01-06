# 配置

`Parse.Config`是通过在Parse上存储单个配置对象来远程配置应用程序的好方法。 它使您能够添加功能选通或简单的“每日消息”。 要开始使用Parse.Config，您需要在Parse Config Dashboard上添加一些键/值对（参数）到您的应用程序。
<!-- `Parse.Config` is a great way to configure your applications remotely by storing a single configuration object on Parse. It enables you to add things like feature gating or a simple "Message of the Day". To start using `Parse.Config` you need to add a few key/value pairs (parameters) to your app on the Parse Config Dashboard. -->

之后，您将能够在客户端上获取“Parse.Config”，如下例所示：
<!-- After that you will be able to fetch the `Parse.Config` on the client, like in this example: -->

<pre><code class="javascript">
Parse.Config.get().then(function(config) {
  var winningNumber = config.get("winningNumber");
  var message = "Yay! The number is " + winningNumber + "!";
  console.log(message);
}, function(error) {
  // Something went wrong (e.g. request timed out)
});
</code></pre>

## 检索配置

`ParseConfig'被建立为尽可能的鲁棒和可靠，即使面对不良的互联网连接。 默认情况下使用缓存，以确保最新成功获取的配置始终可用。 在下面的例子中，我们使用`get`从服务器检索最新版本的config，如果fetch失败了，我们可以简单地回到我们通过`current`获取的版本。
<!-- `ParseConfig` is built to be as robust and reliable as possible, even in the face of poor internet connections. Caching is used by default to ensure that the latest successfully fetched config is always available. In the below example we use `get` to retrieve the latest version of config from the server, and if the fetch fails we can simply fall back to the version that we successfully fetched before via `current`. -->

<pre><code class="javascript">
Parse.Config.get().then(function(config) {
  console.log("Yay! Config was fetched from the server.");

  var welcomeMessage = config.get("welcomeMessage");
  console.log("Welcome Message = " + welcomeMessage);
}, function(error) {
  console.log("Failed to fetch. Using Cached Config.");

  var config = Parse.Config.current();
  var welcomeMessage = config.get("welcomeMessage");
  if (welcomeMessage === undefined) {
    welcomeMessage = "Welcome!";
  }
  console.log("Welcome Message = " + welcomeMessage);
});
</code></pre>

## 当前配置

你得到的每个`Parse.Config`实例总是不可变的。 当你从网络中检索一个新的`Parse.Config`时，它不会修改任何现有的`Parse.Config`实例，而是会创建一个新的Parse.Config，并通过`Parse.Config.current（） `。 因此，你可以安全地传递任何`current（）`对象，并且安全地假设它不会自动改变。
<!-- Every `Parse.Config` instance that you get is always immutable. When you retrieve a new `Parse.Config` in the future from the network, it will not modify any existing `Parse.Config` instance, but will instead create a new one and make it available via `Parse.Config.current()`. Therefore, you can safely pass around any `current()` object and safely assume that it will not automatically change. -->

从服务器检索配置每次你想使用它可能是麻烦的。 你可以通过简单地使用缓存的`current()`对象，并在一段时间内只获取一次配置来避免这种情况。
<!-- It might be troublesome to retrieve the config from the server every time you want to use it. You can avoid this by simply using the cached `current()` object and fetching the config only once in a while. -->

<pre><code class="javascript">
// Fetches the config at most once every 12 hours per app runtime
var refreshConfig = function() {
  var lastFetchedDate;
  var configRefreshInterval = 12 * 60 * 60 * 1000;
  return function() {
    var currentDate = new Date();
    if (lastFetchedDate === undefined ||
        currentDate.getTime() - lastFetchedDate.getTime() > configRefreshInterval) {
      Parse.Config.get();
      lastFetchedDate = currentDate;
    }
  };
}();
</code></pre>

## 参数

`ParseConfig`支持`Parse.Object`支持的大多数数据类型：
<!-- `ParseConfig`  supports most of the data types supported by `Parse.Object`: -->

*   string
*   number
*   Date
*   Parse.File
*   Parse.GeoPoint
*   JS Array
*   JS Object

我们目前在配置中允许最多** 100 **个参数，所有参数的总大小为** 128KB **。
<!-- We currently allow up to **100** parameters in your config and a total size of **128KB** across all parameters. -->
