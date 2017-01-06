# 错误处理

大多数Parse Javascript函数都使用回调进行报告它们成功或者失败的对象，类似于一个Backbone的"options"对象。这两种主要的回调函数是`success`和`error`。    
在一个操作完成且没有错误的时候，`success`就会被调用。
一般来说，它的参数将会是`Parse.Object`中的`save`或者`get`，或者一个`Parse.Object`的`find`数组。

<!-- Most Parse JavaScript functions report their success or failure using an object with callbacks, similar to a Backbone "options" object.  The two primary callbacks used are `success` and `error`.  `success` is called whenever an operation completes without errors.  Generally, its parameter will be either the `Parse.Object` in the case of `save` or `get`, or an array of `Parse.Object` for `find`. -->


`error`将会在与`Parse Cloud`服务器进行交互时发生任何错误的时候被调用。
这些错误与连接到服务器或执行请求操作出现问题时有关。让我们来看另一个例子。
在下面的代码中，我们尝试获取一个不存在的`objectId`对象。 Parse 云服务器将返回一个错误 - 所以这里是如何在回调中正确处理它：
<!-- `error` is called for any kind of error that occurs when interacting with the Parse Cloud over the network. These errors are either related to problems connecting to the cloud or problems performing the requested operation. Let's take a look at another example.  In the code below, we try to fetch an object with a non-existent `objectId`. The Parse Cloud will return an error - so here's how to handle it properly in your callback: -->

<pre><code class="javascript">
var query = new Parse.Query(Note);
query.get("aBcDeFgH", {
  success: function(results) {
    // This function will *not* be called.
    alert("Everything went fine!");
  },
  error: function(model, error) {
    // This will be called.
    // error is an instance of Parse.Error with details about the error.
    if (error.code === Parse.Error.OBJECT_NOT_FOUND) {
      alert("Uh oh, we couldn't find the object!");
    }
  }
});
</code></pre>

查询也可能失败，因为设备无法连接到解析云。 这里是相同的回调，但有一点额外的代码来处理这种情况：
<!-- The query might also fail because the device couldn't connect to the Parse Cloud. Here's the same callback but with a bit of extra code to handle that scenario: -->

<pre><code class="javascript">
var query = new Parse.Query(Note);
query.get("thisObjectIdDoesntExist", {
  success: function(results) {
    // This function will *not* be called.
    alert("Everything went fine!");
  },
  error: function(model, error) {
    // This will be called.
    // error is an instance of Parse.Error with details about the error.
    if (error.code === Parse.Error.OBJECT_NOT_FOUND) {
      alert("Uh oh, we couldn't find the object!");
    } else if (error.code === Parse.Error.CONNECTION_FAILED) {
      alert("Uh oh, we couldn't even connect to the Parse Cloud!");
    }
  }
});
</code></pre>

对于影响特定`Parse.Object`的`save`和`signUp`方法，错误函数的第一个参数是对象本身，第二个参数是`Parse.Error`。 这是为了与骨干类型框架兼容。
<!-- For methods like `save` and `signUp` that affect a particular `Parse.Object`, the first argument to the error function will be the object itself, and the second will be the `Parse.Error` object.  This is for compatibility with Backbone-type frameworks. -->

对于所有可能的`Parse.Error`代码的列表，向下滚动到[错误代码](#errors)，或参见[JavaScript API `Parse.Error`](https://parse.com/docs/js/api/symbols/Parse.Error.html)。
<!-- For a list of all possible `Parse.Error` codes, scroll down to [Error Codes](#errors), or see the `Parse.Error` section of the  [JavaScript API `Parse.Error`](https://parse.com/docs/js/api/symbols/Parse.Error.html).
 -->
