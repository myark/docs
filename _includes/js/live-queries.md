<h1 id="live-queries">实时查询</h1>

<h2 id="standard-api">标准API</h2>

正如我们在[LiveQuery协议](https://github.com/ParsePlatform/parse-server/wiki/Parse-LiveQuery-Protocol-Specification)中讨论的，我们维护一个WebSocket连接来与Parse LiveQuery服务器通信。 
当使用服务器端时，我们使用[`ws`](https://www.npmjs.com/package/ws)包，在浏览器中我们使用[`window.WebSocket`](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)。 
我们认为在大多数情况下，没有必要直接处理WebSocket连接。 
因此，我们开发了一个简单的API，让您专注于自己的业务逻辑。
<!-- As we discussed in our [LiveQuery protocol](https://github.com/ParsePlatform/parse-server/wiki/Parse-LiveQuery-Protocol-Specification), we maintain a WebSocket connection to communicate with the Parse LiveQuery server. When used server side we use the [`ws`](https://www.npmjs.com/package/ws) package and in the browser we use [`window.WebSocket`](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API). We think in most cases it isn't necessary to deal with the WebSocket connection directly. Thus, we developed a simple API to let you focus on your own business logic. -->

注意：只能在[Parse Server](https://github.com/ParsePlatform/parse-server)中使用[JS SDK〜1.8](https://github.com/ParsePlatform/Parse-SDK-JS)来支持Live Queries.
<!-- Note: Live Queries is supported only in [Parse Server](https://github.com/ParsePlatform/parse-server) with [JS SDK ~1.8](https://github.com/ParsePlatform/Parse-SDK-JS).
 -->

<h2 id="create-a-subscription">创建订阅</h2>

<pre><code class="javascript">
let query = new Parse.Query('Game');
let subscription = query.subscribe();
</code></pre>
您获得的订阅实际上是一个事件发射器。 有关事件发射器的更多信息，请选中[这里](https://nodejs.org/api/events.html)。 
你将通过这个`subscription`获得LiveQuery事件。 第一次调用subscribe时，我们将尝试为您打开与LiveQuery服务器的WebSocket连接。
<!-- The subscription you get is actually an event emitter. For more information on event emitter, check [here](https://nodejs.org/api/events.html). You'll get the LiveQuery events through this `subscription`. The first time you call subscribe, we'll try to open the WebSocket connection to the LiveQuery server for you. -->

<h2 id="event-handling">事件处理</h2>
我们定义了通过订阅对象获得的几种类型的事件：
<!-- We define several types of events you'll get through a subscription object: -->

<h3 id="open-event">打开事件</h3>
<pre><code class="javascript">
subscription.on('open', () => {
 console.log('subscription opened');
});
</code></pre>
当调用`query.subscribe()`时，我们向LiveQuery服务器发送一个订阅请求。 
当我们从LiveQuery服务器获得确认时，将会发出此事件。
<!-- When you call `query.subscribe()`, we send a subscribe request to the LiveQuery server. When we get the confirmation from the LiveQuery server, this event will be emitted. -->

当客户端丢失与LiveQuery服务器的WebSocket连接时，我们将尝试自动重新连接LiveQuery服务器。 
如果我们重新连接LiveQuery服务器并成功重新订阅`ParseQuery`，您还将获得此事件。
<!-- When the client loses WebSocket connection to the LiveQuery server, we'll try to auto reconnect the LiveQuery server. If we reconnect the LiveQuery server and successfully resubscribe the `ParseQuery`, you'll also get this event. -->

<br/>

<h3 id="create-event">创建事件</h3>
<pre><code class="javascript">
subscription.on('create', (object) => {
  console.log('object created');
});
</code></pre>
当一个新的`ParseObject`被创建并且它满足你所订阅的`ParseQuery`时，你会得到这个事件。 
`object`是创建的`ParseObject`。
<!-- When a new `ParseObject` is created and it fulfills the `ParseQuery` you subscribe, you'll get this event. The `object` is the `ParseObject` which was created.
 -->
<br/>

<h3 id="update-event">更新事件</h3>

<pre><code class="javascript">
subscription.on('update', (object) => {
  console.log('object updated');
});
</code></pre>
当一个现有的`ParseObject`满足你的订阅`ParseQuery`被更新时（`ParseObject`在更改之前和之后满足`ParseQuery`），你将得到这个事件。 
`object`是被更新的`ParseObject`。 它的内容是`ParseObject`的最新值。
<!-- When an existing `ParseObject` which fulfills the `ParseQuery` you subscribe is updated (The `ParseObject` fulfills the `ParseQuery` before and after changes), you'll get this event. The `object` is the `ParseObject` which was updated. Its content is the latest value of the `ParseObject`.
 -->
<br/>

<h3 id="enter-event">输入事件</h3>
<pre><code class="javascript">
subscription.on('enter', (object) => {
  console.log('object entered');
});
</code></pre>
当一个现有的`ParseObject`的旧值不满足`ParseQuery`，但它的新值满足`ParseQuery`，你会得到这个事件。 
`object`是`ParseObject`，它进入`ParseQuery`。 它的内容是`ParseObject`的最新值。
<!-- When an existing `ParseObject`'s old value does not fulfill the `ParseQuery` but its new value fulfills the `ParseQuery`, you'll get this event. The `object` is the `ParseObject` which enters the `ParseQuery`. Its content is the latest value of the `ParseObject`.
 -->
<br/>

<h3 id="leave-event">离开事件</h3>

<pre><code class="javascript">
subscription.on('leave', (object) => {
  console.log('object left');
});
</code></pre>
当现有的`ParseObject`的旧值满足`ParseQuery`，但是它的新值不满足`ParseQuery`，你会得到这个事件。 
`object`是`ParseObject`，留下`ParseQuery`。 它的内容是`ParseObject`的最新值。
<!-- When an existing `ParseObject`'s old value fulfills the `ParseQuery` but its new value doesn't fulfill the `ParseQuery`, you'll get this event. The `object` is the `ParseObject` which leaves the `ParseQuery`. Its content is the latest value of the `ParseObject`.
 -->
<br/>

<h3 id="delete-event">删除事件</h3>

<pre><code class="javascript">
subscription.on('delete', (object) => {
  console.log('object deleted');
});
</code></pre>
当一个现有的满足`ParseQuery`的`ParseObject`被删除时，你会得到这个事件。 
`object`是`ParseObject`，它被删除。
<!-- When an existing `ParseObject` which fulfills the `ParseQuery` is deleted, you'll get this event. The `object` is the `ParseObject` which is deleted.
 -->
<br/>

<h3 id="close-event">关闭事件</h3>

<pre><code class="javascript">
subscription.on('close', () => {
  console.log('subscription closed');
});
</code></pre>
当客户端丢失到LiveQuery服务器的WebSocket连接，并且我们不能再获得任何事件时，您将收到此事件。
<!-- When the client loses the WebSocket connection to the LiveQuery server and we can't get anymore events, you'll get this event.
 -->
<br/>

<h2 id="unsubscribe">取消订阅</h2>

<pre><code class="javascript">
subscription.unsubscribe();
</code></pre>
如果你想停止接收来自`ParseQuery`的事件，你可以取消订阅`subscription`。 之后，你不会从`subscription`对象获得任何事件。
<!-- If you would like to stop receiving events from a `ParseQuery`, you can just unsubscribe the `subscription`. After that, you won't get any events from the `subscription` object. -->


<h2 id="close">关闭</h2>

<pre><code class="javascript">
Parse.LiveQuery.close();
</code></pre>
使用LiveQuery完成后，您可以调用`Parse.LiveQuery.close()`。 此函数将关闭与LiveQuery服务器的WebSocket连接，取消自动重新连接，并取消订阅基于它的所有订阅。 如果在此之后调用`query.subscribe()`，我们将创建一个到LiveQuery服务器的新WebSocket连接。
<!-- When you are done using LiveQuery, you can call `Parse.LiveQuery.close()`. This function will close the WebSocket connection to the LiveQuery server, cancel the auto reconnect, and unsubscribe all subscriptions based on it. If you call `query.subscribe()` after this, we will create a new WebSocket connection to the LiveQuery server.
 -->

## 设置服务器地址
<pre><code class="javascript">
Parse.liveQueryServerURL = 'ws://XXXX'
</code></pre>
大多数时候你不需要手动设置这个。 如果你设置了你的`Parse.serverURL`，我们将尝试提取主机名，并使用`ws://hostname`作为默认`liveQueryServerURL`。 然而，如果你想定义你自己的`liveQueryServerURL`或使用不同的协议，如`wss`，你应该自己设置它。
<!-- Most of the time you do not need to manually set this. If you have setup your `Parse.serverURL`, we will try to extract the hostname and use `ws://hostname` as the default `liveQueryServerURL`. However, if you want to define your own `liveQueryServerURL` or use a different protocol such as `wss`, you should set it by yourself.
 -->

## WebSocket状态
我们公开了三个事件来帮助您监视WebSocket连接的状态：
<!-- We expose three events to help you monitor the status of the WebSocket connection: -->
### 打开事件
<pre><code class="javascript">
Parse.LiveQuery.on('open', () => {
  console.log('socket connection established');
});
</code></pre>
当我们建立到LiveQuery服务器的WebSocket连接时，您将收到此事件。
<!-- When we establish the WebSocket connection to the LiveQuery server, you'll get this event. -->

<br/>

### 关闭事件
<pre><code class="javascript">
Parse.LiveQuery.on('close', () => {
  console.log('socket connection closed');
});
</code></pre>
当我们失去与LiveQuery服务器的WebSocket连接时，您会收到此事件。
<!-- When we lose the WebSocket connection to the LiveQuery server, you'll get this event.
 -->
<br/>

### 错误事件
<pre><code class="javascript">
Parse.LiveQuery.on('error', (error) => {
  console.log(error);
});
</code></pre>
当发生某些网络错误或LiveQuery服务器错误时，您会收到此事件。
<!-- When some network error or LiveQuery server error happens, you'll get this event.
 -->
<br/>

***
## 高级API
在我们的标准API中，我们为您管理一个全局WebSocket连接，这适用于大多数情况。 
但是，在某些情况下，例如，当您有多个LiveQuery服务器并想要连接到所有这些服务器时，单个WebSocket连接是不够的。 
我们为这些场景公开了`LiveQueryClient`。
<!-- In our standard API, we manage a global WebSocket connection for you, which is suitable for most cases. However, for some cases, like when you have multiple LiveQuery servers and want to connect to all of them, a single WebSocket connection isn't enough. We've exposed the `LiveQueryClient` for these scenarios. -->

## LiveQueryClient
`LiveQueryClient`是标准WebSocket客户端的包装器。 我们添加了几个有用的方法来帮助您连接/断开与LiveQueryServer和订阅/取消订阅一个`ParseQuery`容易。
<!-- A `LiveQueryClient` is a wrapper of a standard WebSocket client. We add several useful methods to help you connect/disconnect to LiveQueryServer and subscribe/unsubscribe a `ParseQuery` easily. -->

### 初始化
<pre><code class="javascript">
let Parse = require('parse/node');
let LiveQueryClient = Parse.LiveQueryClient;
let client = new LiveQueryClient({
  applicationId: '',
  serverURL: '',
  javascriptKey: '',
  masterKey: ''
});

</code></pre> 
* `applicationId` 是必选的，它是您的Parse应用程序的`applicationId`
* `liveQueryServerURL` 是必选的，它是您的LiveQuery服务器的URL
* `javascriptKey` and `masterKey` 是可选的，它们用于在尝试连接到LiveQuery服务器时验证`LiveQueryClient`。 如果你设置它们，他们应该匹配您的Parse应用程序。 您可以检查LiveQuery协议[这里](https://github.com/ParsePlatform/parse-server/wiki/Parse-LiveQuery-Protocol-Specification)了解更多详情。

### 开启
<pre><code class="javascript">
client.open();
</code></pre>
调用此命令后，`LiveQueryClient`将尝试向LiveQuery服务器发送连接请求。
<!-- After you call this, the `LiveQueryClient` will try to send a connect request to the LiveQuery server.
 -->
<br/>

### 订阅
<pre><code class="javascript">
let query = new Parse.Query('Game');
let subscription = client.subscribe(query, sessionToken); 
</code></pre>
* `query` is mandatory, it is the `ParseQuery` you want to subscribe
* `sessionToken` is optional, if you provide the `sessionToken`, when the LiveQuery server gets `ParseObject`'s updates from parse server, it'll try to check whether the `sessionToken` fulfills the `ParseObject`'s ACL. The LiveQuery server will only send updates to clients whose sessionToken is fit for the `ParseObject`'s ACL. You can check the LiveQuery protocol [here](https://github.com/ParsePlatform/parse-server/wiki/Parse-LiveQuery-Protocol-Specification) for more details.
The `subscription` you get is the same `subscription` you get from our Standard API. You can check our Standard API about how to use the `subscription` to get events.

### 取消订阅
<pre><code class="javascript">
client.unsubscribe(subscription); 
</code></pre>
* `subscription` 是强制性的，它是您要取消订阅的订阅。
调用此命令后，将不会从预订对象获取任何事件。
<br/>

### 关闭
<pre><code class="javascript">
client.close();
</code></pre>
此函数将关闭与此`LiveQueryClient`的WebSocket连接，取消自动重新连接，并取消订阅基于它的所有订阅。
<!-- This function will close the WebSocket connection to this `LiveQueryClient`, cancel the auto reconnect, and unsubscribe all subscriptions based on it.
 -->
<br/>

## 事件处理
我们公开了三个事件来帮助你监视`LiveQueryClient`的状态。

### 打开事件
<pre><code class="javascript">
client.on('open', () => {
  console.log('connection opened');
});
</code></pre>
当我们建立到LiveQuery服务器的WebSocket连接时，您将收到此事件。
<!-- When we establish the WebSocket connection to the LiveQuery server, you'll get this event. -->

<br/>

### 关闭事件
<pre><code class="javascript">
client.on('close', () => {
  console.log('connection closed');
});
</code></pre>
当我们失去与LiveQuery服务器的WebSocket连接时，您会收到此事件。
<!-- When we lose the WebSocket connection to the LiveQuery server, you'll get this event.
 -->
<br/>

### 错误事件
<pre><code class="javascript">
client.on('error', (error) => {
  console.log('connection error');
});
</code></pre>
当发生某些网络错误或LiveQuery服务器错误时，您会收到此事件。
<!-- When some network error or LiveQuery server error happens, you'll get this event.
 -->
<br/>

***
## 重新连接
由于整个LiveQuery功能依赖于到LiveQuery服务器的WebSocket连接，因此我们总是尝试维护一个打开的WebSocket连接。 因此，当我们发现我们失去了与LiveQuery服务器的连接时，我们将尝试自动重新连接。 我们在引擎盖下做指数回退。 但是，如果WebSocket连接由于`Parse.LiveQuery.close()`或`client.close()`关闭，我们将取消自动重新连接。
<!-- Since the whole LiveQuery feature relies on the WebSocket connection to the LiveQuery server, we always try to maintain an open WebSocket connection. Thus, when we find we lose the connection to the LiveQuery server, we will try to auto reconnect. We do exponential back off under the hood. However, if the WebSocket connection is closed due to `Parse.LiveQuery.close()` or `client.close()`, we'll cancel the auto reconnect. -->


***
## SessionToken
当你订阅一个`ParseQuery`时，我们发送`sessionToken`给LiveQuery服务器。 对于标准API，我们默认使用当前用户的`sessionToken`。 对于高级API，您可以在订阅`ParseQuery`时使用任何`sessionToken`。 一个重要的事情要注意的是当你注销或你使用的`sessionToken`是无效的，你应该取消订阅订阅并再次订阅`ParseQuery`。 否则，您可能会面临安全问题，因为您将收到不应发送给您的事件。
<!-- We send `sessionToken` to the LiveQuery server when you subscribe to a `ParseQuery`. For the standard API, we use the `sessionToken` of the current user by default. For the advanced API, you can use any `sessionToken` when you subscribe a `ParseQuery`. An important thing to be aware of is when you log out or the `sessionToken` you are using is invalid, you should unsubscribe the subscription and subscribe to the `ParseQuery` again. Otherwise you may face a security issue since you'll get events which shouldn't be sent to you. -->
