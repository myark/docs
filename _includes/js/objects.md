# 对象

## Parse.Object

在Parse中存储数据是围绕`Parse.Object`构建的。 每个`Parse.Object`包含JSON兼容数据的键值对。 此数据是无模式的，这意味着您不需要提前在每个`Parse.Object`上存在什么键。 你只需设置你想要的键值对，我们的后端会存储它。
<!-- Storing data on Parse is built around `Parse.Object`. Each `Parse.Object` contains key-value pairs of JSON-compatible data. This data is schemaless, which means that you don't need to specify ahead of time what keys exist on each `Parse.Object`. You simply set whatever key-value pairs you want, and our backend will store it. -->

例如，假设您要跟踪游戏的高分。 单个`Parse.Object`可以包含：
<!-- For example, let's say you're tracking high scores for a game. A single `Parse.Object` could contain: -->

<pre><code class="javascript">
score: 1337, playerName: "Sean Plott", cheatMode: false
</code></pre>

键必须是字母数字字符串。 值可以是字符串，数字，布尔值，甚至数组和字典 - 任何可以JSON编码的东西。
<!-- Keys must be alphanumeric strings. Values can be strings, numbers, booleans, or even arrays and dictionaries - anything that can be JSON-encoded. -->

每个`Parse.Object`是一个具有类名的特定子类的实例，您可以使用它来区分不同类型的数据。 例如，我们可以将高分对象称为`GameScore`。 我们建议你`NameYourClassesLikeThis`和n`ameYourKeysLikeThis`，只是为了保持你的代码看起来漂亮。
<!-- Each `Parse.Object` is an instance of a specific subclass with a class name that you can use to distinguish different sorts of data. For example, we could call the high score object a `GameScore`. We recommend that you NameYourClassesLikeThis and nameYourKeysLikeThis, just to keep your code looking pretty. -->

要创建一个新的子类，使用`Parse.Object.extend`方法。 任何`Parse.Query`将返回任何具有相同类名的`Parse.Object`的新类的实例。 如果你熟悉`Backbone.Model`，那么你已经知道如何使用`Parse.Object`。 它的设计是以相同的方式创建和修改。
<!-- To create a new subclass, use the `Parse.Object.extend` method.  Any `Parse.Query` will return instances of the new class for any `Parse.Object` with the same classname.  If you're familiar with `Backbone.Model`, then you already know how to use `Parse.Object`.  It's designed to be created and modified in the same ways. -->

<pre><code class="javascript">
// Simple syntax to create a new subclass of Parse.Object.
var GameScore = Parse.Object.extend("GameScore");

// Create a new instance of that class.
var gameScore = new GameScore();

// Alternatively, you can use the typical Backbone syntax.
var Achievement = Parse.Object.extend({
  className: "Achievement"
});
</code></pre>

您可以向`Parse.Object`的子类添加其他方法和属性。
<!-- You can add additional methods and properties to your subclasses of `Parse.Object`. -->

<pre><code class="javascript">

// A complex subclass of Parse.Object
var Monster = Parse.Object.extend("Monster", {
  // Instance methods
  hasSuperHumanStrength: function () {
    return this.get("strength") > 18;
  },
  // Instance properties go in an initialize method
  initialize: function (attrs, options) {
    this.sound = "Rawr"
  }
}, {
  // Class methods
  spawn: function(strength) {
    var monster = new Monster();
    monster.set("strength", strength);
    return monster;
  }
});

var monster = Monster.spawn(200);
alert(monster.get('strength'));  // Displays 200.
alert(monster.sound); // Displays Rawr.
</code></pre>

要创建任何Parse Object类的单个实例，您还可以直接使用`Parse.Object`构造函数。 `new Parse.Objec(className)`将创建一个具有该类名的单个Parse对象。
<!-- To create a single instance of any Parse Object class, you can also use the `Parse.Object` constructor directly. `new Parse.Object(className)` will create a single Parse Object with that class name. -->

如果你已经在你的代码库中使用ES6，好消息！ 从1.6.0开始，JavaScript SDK与ES6类兼容。 你可以用`extend`关键字来继承`Parse.Object`：
<!-- If you’re already using ES6 in your codebase, good news! From version 1.6.0 onwards, the JavaScript SDK is compatible with ES6 classes. You can subclass `Parse.Object` with the `extends` keyword: -->

<pre><code class="javascript">
class Monster extends Parse.Object {
  constructor() {
    // Pass the ClassName to the Parse.Object constructor
    super('Monster');
    // All other initialization
    this.sound = 'Rawr';
  }

  hasSuperHumanStrength() {
    return this.get('strength') > 18;
  }

  static spawn(strength) {
    var monster = new Monster();
    monster.set('strength', strength);
    return monster;
  }
}
</code></pre>

但是，当使用`extend`时，SDK不会自动知道你的子类。 如果你想要从查询返回的对象使用你的子类`Parse.Object`，你将需要注册子类，类似于我们在其他平台上做的。
<!-- However, when using `extends`, the SDK is not automatically aware of your subclass. If you want objects returned from queries to use your subclass of `Parse.Object`, you will need to register the subclass, similar to what we do on other platforms. -->

<pre><code class="javascript">
// After specifying the Monster subclass...
Parse.Object.registerSubclass('Monster', Monster);
</code></pre>

## 保存对象

假设你想把上面描述的`GameScore`保存到Parse Cloud中。 该接口类似于一个`Backbone.Model`，包括`save`方法：
<!-- Let's say you want to save the `GameScore` described above to the Parse Cloud. The interface is similar to a `Backbone.Model`, including the `save` method: -->

<pre><code class="javascript">
var GameScore = Parse.Object.extend("GameScore");
var gameScore = new GameScore();

gameScore.set("score", 1337);
gameScore.set("playerName", "Sean Plott");
gameScore.set("cheatMode", false);

gameScore.save(null, {
  success: function(gameScore) {
    // Execute any logic that should take place after the object is saved.
    alert('New object created with objectId: ' + gameScore.id);
  },
  error: function(gameScore, error) {
    // Execute any logic that should take place if the save fails.
    // error is a Parse.Error with an error code and message.
    alert('Failed to create new object, with error code: ' + error.message);
  }
});
</code></pre>

在这段代码运行后，你可能会想知道是否真的发生了。 为了确保数据已保存，您可以在Parse的应用程序中查看数据浏览器。 你应该看到这样的：
<!-- After this code runs, you will probably be wondering if anything really happened. To make sure the data was saved, you can look at the Data Browser in your app on Parse. You should see something like this: -->

<pre><code class="json">
objectId: "xWMyZ4YEGZ", score: 1337, playerName: "Sean Plott", cheatMode: false,
createdAt:"2011-06-10T18:33:42Z", updatedAt:"2011-06-10T18:33:42Z"
</code></pre>

这里有两件事要注意。 在运行此代码之前，您不必配置或设置一个名为`GameScore`的新类。 你的Parse应用程序懒洋洋地为你创建这个类，当它第一次遇到它。
<!-- There are two things to note here. You didn't have to configure or set up a new Class called `GameScore` before running this code. Your Parse app lazily creates this Class for you when it first encounters it. -->

还有一些字段，您不需要指定为方便提供。 `objectId`是每个保存对象的唯一标识符。 `createdAt`和`updatedAt`表示每个对象在云中创建和最后修改的时间。 这些字段中的每一个都由Parse填充，因此在保存操作完成之前，它们不存在于`Parse.Object`中。
<!-- There are also a few fields you don't need to specify that are provided as a convenience. `objectId` is a unique identifier for each saved object. `createdAt` and `updatedAt` represent the time that each object was created and last modified in the cloud. Each of these fields is filled in by Parse, so they don't exist on a `Parse.Object` until a save operation has completed. -->

如果你喜欢，你可以直接在调用中设置属性`save`。
<!-- If you prefer, you can set attributes directly in your call to `save` instead. -->

<pre><code class="javascript">
var GameScore = Parse.Object.extend("GameScore");
var gameScore = new GameScore();

gameScore.save({
  score: 1337,
  playerName: "Sean Plott",
  cheatMode: false
}, {
  success: function(gameScore) {
    // The object was saved successfully.
  },
  error: function(gameScore, error) {
    // The save failed.
    // error is a Parse.Error with an error code and message.
  }
});
</code></pre>

## 获取对象

将数据保存到云是很有趣，但它再更有趣的数据。 如果你有`objectId`，你可以使用`Parse.Query`检索整个`Parse.Object`：
<!-- Saving data to the cloud is fun, but it's even more fun to get that data out again. If you have the `objectId`, you can retrieve the whole `Parse.Object` using a `Parse.Query`: -->

<pre><code class="javascript">
var GameScore = Parse.Object.extend("GameScore");
var query = new Parse.Query(GameScore);
query.get("xWMyZ4YEGZ", {
  success: function(gameScore) {
    // The object was retrieved successfully.
  },
  error: function(object, error) {
    // The object was not retrieved successfully.
    // error is a Parse.Error with an error code and message.
  }
});
</code></pre>

要从`Parse.Object`获取值，使用`get`方法。
<!-- To get the values out of the `Parse.Object`, use the `get` method. -->

<pre><code class="javascript">
var score = gameScore.get("score");
var playerName = gameScore.get("playerName");
var cheatMode = gameScore.get("cheatMode");
</code></pre>

三个特殊的保留值作为属性提供，不能使用`get`方法检索，也不能使用`set`方法修改：
<!-- The three special reserved values are provided as properties and cannot be retrieved using the 'get' method nor modified with the 'set' method: -->

<pre><code class="javascript">
var objectId = gameScore.id;
var updatedAt = gameScore.updatedAt;
var createdAt = gameScore.createdAt;
</code></pre>

如果你需要刷新一个已经拥有最新数据的对象
     在Parse Cloud中，您可以调用`fetch`方法，如下所示：
<!-- If you need to refresh an object you already have with the latest data that
    is in the Parse Cloud, you can call the `fetch` method like so: -->

<pre><code class="javascript">
myObject.fetch({
  success: function(myObject) {
    // The object was refreshed successfully.
  },
  error: function(myObject, error) {
    // The object was not refreshed successfully.
    // error is a Parse.Error with an error code and message.
  }
});
</code></pre>

## 更新对象

更新对象很简单。 只需在其上设置一些新数据，并调用保存方法。 例如：
<!-- Updating an object is simple. Just set some new data on it and call the save method. For example: -->

<pre><code class="javascript">
// Create the object.
var GameScore = Parse.Object.extend("GameScore");
var gameScore = new GameScore();

gameScore.set("score", 1337);
gameScore.set("playerName", "Sean Plott");
gameScore.set("cheatMode", false);
gameScore.set("skills", ["pwnage", "flying"]);

gameScore.save(null, {
  success: function(gameScore) {
    // Now let's update it with some new data. In this case, only cheatMode and score
    // will get sent to the cloud. playerName hasn't changed.
    gameScore.set("cheatMode", true);
    gameScore.set("score", 1338);
    gameScore.save();
  }
});
</code></pre>

解析自动计算出哪些数据已更改，因此只有"垃圾"字段将发送到解析云。 你不需要担心压缩你不想更新的数据。
<!-- Parse automatically figures out which data has changed so only "dirty" fields will be sent to the Parse Cloud. You don't need to worry about squashing data that you didn't intend to update. -->

### 计数器

上面的例子包含一个常见的用例。 `score`字段是一个计数器，我们需要持续更新玩家的最新得分。 使用上面的方法工作，但它是繁琐的，如果有多个客户端尝试更新相同的计数器，可能会导致问题。
<!-- The above example contains a common use case. The "score" field is a counter that we'll need to continually update with the player's latest score. Using the above method works but it's cumbersome and can lead to problems if you have multiple clients trying to update the same counter. -->

为了帮助存储计数器类型数据，Parse提供了原子地增加（或减少）任何数字字段的方法。 因此，相同的更新可以重写为：
<!-- To help with storing counter-type data, Parse provides methods that atomically increment (or decrement) any number field. So, the same update can be rewritten as: -->

<pre><code class="javascript">
gameScore.increment("score");
gameScore.save();
</code></pre>

你也可以通过传递第二个参数到`increment`来增加任何数量。 当未指定数量时，默认使用1。
<!-- You can also increment by any amount by passing in a second argument to `increment`. When no amount is specified, 1 is used by default. -->

### 数组

为了帮助存储数组数据，有三个操作可用于原子地改变与给定键相关联的数组：
<!-- To help with storing array data, there are three operations that can be used to atomically change an array associated with a given key: -->

*   `add` 将给定的对象附加到数组字段的末尾。
*   `addUnique` 只有在给定对象尚未包含在数组字段中时才添加该对象。 不保证插入件的位置。
*   `remove` 从数组字段中删除给定对象的所有实例。

例如，我们可以添加项目到类似set的"技能"字段，如：
<!-- For example, we can add items to the set-like "skills" field like so: -->

<pre><code class="javascript">
gameScore.addUnique("skills", "flying");
gameScore.addUnique("skills", "kungfu");
gameScore.save();
</code></pre>

注意，当前不可能在同一保存中从数组中原子地添加和删除项目。 你必须在每种不同类型的数组操作之间调用`save`。
<!-- Note that it is not currently possible to atomically add and remove items from an array in the same save. You will have to call `save` in between every different kind of array operation. -->

## 销毁对象

从云中删除对象：
<!-- To delete an object from the cloud: -->

<pre><code class="javascript">
myObject.destroy({
  success: function(myObject) {
    // The object was deleted from the Parse Cloud.
  },
  error: function(myObject, error) {
    // The delete failed.
    // error is a Parse.Error with an error code and message.
  }
});
</code></pre>

您可以使用`unset`方法从对象中删除单个字段：
<!-- You can delete a single field from an object with the `unset` method: -->

<pre><code class="javascript">
// After this, the playerName field will be empty
myObject.unset("playerName");

// Saves the field deletion to the Parse Cloud.
// If the object's field is an array, call save() after every unset() operation.
myObject.save();
</code></pre>

请注意，不建议使用`object.set(null)`从对象中删除字段，这将导致意外的功能。
<!-- Please note that use of object.set(null) to remove a field from an object is not recommended and will result in unexpected functionality.
 -->

## 关系数据

对象可以具有与其他对象的关系。 例如，在博客应用程序中，`Post`对象可能有许多`Comment`对象。 Parse支持所有类型的关系，包括一对一，一对多和多对多。
<!-- Objects may have relationships with other objects. For example, in a blogging application, a `Post` object may have many `Comment` objects. Parse supports all kind of relationships, including one-to-one, one-to-many, and many-to-many. -->

### 一对一和一对多关系

一对一和一对多关系通过将`Parse.Object`保存为另一个对象中的值来建模。 例如，博客应用中的每个`Comment`可能对应一个`Post`。
<!-- One-to-one and one-to-many relationships are modeled by saving a `Parse.Object` as a value in the other object. For example, each `Comment` in a blogging app might correspond to one `Post`. -->

要创建一个带有`Comment`的`Post`，你可以这样写：
<!-- To create a new `Post` with a single `Comment`, you could write: -->

<pre><code class="javascript">
// Declare the types.
var Post = Parse.Object.extend("Post");
var Comment = Parse.Object.extend("Comment");

// Create the post
var myPost = new Post();
myPost.set("title", "I'm Hungry");
myPost.set("content", "Where should we go for lunch?");

// Create the comment
var myComment = new Comment();
myComment.set("content", "Let's do Sushirrito.");

// Add the post as a value in the comment
myComment.set("parent", myPost);

// This will save both myPost and myComment
myComment.save();
</code></pre>

在内部，Parse框架将在一个地方存储引用对象，以保持一致性。 你也可以使用它们的`objectId`来链接对象，像这样：
<!-- Internally, the Parse framework will store the referred-to object in just one place, to maintain consistency. You can also link objects using just their `objectId`s like so: -->

<pre><code class="javascript">
var post = new Post();
post.id = "1zEcyElZ80";

myComment.set("parent", post);
</code></pre>

默认情况下，当提取对象时，不会提取相关的`Parse.Object`。 在获取这些对象的值之前，无法检索这些值：
<!-- By default, when fetching an object, related `Parse.Object`s are not fetched.  These objects' values cannot be retrieved until they have been fetched like so: -->

<pre><code class="javascript">
var post = fetchedComment.get("parent");
post.fetch({
  success: function(post) {
    var title = post.get("title");
  }
});
</code></pre>

### 多对多关系

多对多关系使用`Parse.Relation`建模。 这类似于在一个键中存储一个`Parse.Object`s数组，除了你不需要同时获取关系中的所有对象。 此外，这允许`Parse.Relation`扩展到比`Parse.Object`方法的数组更多的对象。 例如，一个`User`可能有很多`Posts`，她可能会喜欢。 在这种情况下，你可以存储`User`喜欢使用关系的`Posts`的集合。 为了在`User`的`likes`列表中添加一个`Post`，你可以这样做：
<!-- Many-to-many relationships are modeled using `Parse.Relation`. This works similar to storing an array of `Parse.Object`s in a key, except that you don't need to fetch all of the objects in a relation at once.  In addition, this allows `Parse.Relation` to scale to many more objects than the array of `Parse.Object` approach.  For example, a `User` may have many `Posts` that she might like. In this case, you can store the set of `Posts` that a `User` likes using `relation`.  In order to add a `Post` to the "likes" list of the `User`, you can do: -->

<pre><code class="javascript">
var user = Parse.User.current();
var relation = user.relation("likes");
relation.add(post);
user.save();
</code></pre>

你可以从`Parse.Relation`中删除一个帖子：
<!-- You can remove a post from a `Parse.Relation`: -->

<pre><code class="javascript">
relation.remove(post);
user.save();
</code></pre>

在调用save之前，你可以多次调用`add`和`remove`
<!-- You can call `add` and `remove` multiple times before calling save: -->

<pre><code class="javascript">
relation.remove(post1);
relation.remove(post2);
user.save();
</code></pre>

你也可以把一个`Parse.Object`数组传递给`add`和`remove`：
<!-- You can also pass in an array of `Parse.Object` to `add` and `remove`: -->

<pre><code class="javascript">
relation.add([post1, post2, post3]);
user.save();
</code></pre>

默认情况下，不会下载此关系中的对象列表。 您可以通过使用`query`返回的`Parse.Query`获取用户喜欢的帖子的列表。 代码如下所示：
<!-- By default, the list of objects in this relation are not downloaded.  You can get a list of the posts that a user likes by using the `Parse.Query` returned by `query`.  The code looks like: -->

<pre><code class="javascript">
relation.query().find({
  success: function(list) {
    // list contains the posts that the current user likes.
  }
});
</code></pre>

如果你只想要一个子集的帖子，你可以添加额外的约束到`Parse.Query`通过查询返回如下：
<!-- If you want only a subset of the Posts, you can add extra constraints to the `Parse.Query` returned by query like this: -->

<pre><code class="javascript">
var query = relation.query();
query.equalTo("title", "I'm Hungry");
query.find({
  success:function(list) {
    // list contains post liked by the current user which have the title "I'm Hungry".
  }
});
</code></pre>

有关`Parse.Query`的更多细节，请查看本指南的查询部分。 `Parse.Relation`的行为类似于`Parse.Object`的数组，用于查询目的，所以任何查询你可以做一个对象数组，你可以做一个`Parse.Relation`。
<!-- For more details on `Parse.Query`, please look at the query portion of this guide. A `Parse.Relation` behaves similar to an array of `Parse.Object` for querying purposes, so any query you can do on an array of objects, you can do on a `Parse.Relation`. -->

## 数据类型

到目前为止，我们使用类型为`String`，`Number`和`Parse.Object`的值。 Parse还支持`Date`s和`null`。 你可以嵌套`JSON对象`和`JSON数组`来存储更多的结构化数据在一个`Parse.Object`中。 总的来说，对象中的每个字段都允许使用以下类型：
<!-- So far we've used values with type `String`, `Number`, and `Parse.Object`. Parse also supports `Date`s and `null`. You can nest `JSON Object`s and `JSON Array`s to store more structured data within a single `Parse.Object`. Overall, the following types are allowed for each field in your object: -->

* String => `String`
* Number => `Number`
* Bool => `bool`
* Array => `JSON Array`
* Object => `JSON Object`
* Date => `Date`
* File => `Parse.File`
* Pointer => other `Parse.Object`
* Relation => `Parse.Relation`
* Null => `null`

一些例子：
<!-- Some examples: -->

<pre><code class="javascript">
var number = 42;
var bool = false;
var string = "the number is " + number;
var date = new Date();
var array = [string, number];
var object = { number: number, string: string };
var pointer = MyClassName.createWithoutData(objectId);

var BigObject = Parse.Object.extend("BigObject");
var bigObject = new BigObject();
bigObject.set("myNumber", number);
bigObject.set("myBool", bool);
bigObject.set("myString", string);
bigObject.set("myDate", date);
bigObject.set("myArray", array);
bigObject.set("myObject", object);
bigObject.set("anyKey", null); // this value can only be saved to an existing key
bigObject.set("myPointerKey", pointer); // shows up as Pointer &lt;MyClassName&gt; in the Data Browser
bigObject.save();
</code></pre>

我们不建议在`Parse.Object`上存储大型二进制数据，如图像或文档。 `Parse.Object`s的大小不应超过128字节。 我们建议您使用`Parse.File`s来存储图像，文档和其他类型的文件。 您可以通过实例化一个`Parse.File`对象并将其设置在字段上来实现。 有关详细信息，请参阅[文件](#files)。
<!-- We do not recommend storing large pieces of binary data like images or documents on `Parse.Object`. `Parse.Object`s should not exceed 128 kilobytes in size. We recommend you use `Parse.File`s to store images, documents, and other types of files. You can do so by instantiating a `Parse.File` object and setting it on a field. See [Files](#files) for more details. -->


有关Parse如何处理数据的更多信息，请查看我们关于[数据](#data)的文档。
<!-- For more information about how Parse handles data, check out our documentation on [Data](#data). -->
