<h1 id="queries">查询</h1>

我们已经看到一个`Parse.Query`与`get`可以从Parse中检索一个`Parse.Object`。 
还有很多其他方法来使用`Parse.Query`检索数据 - 你可以一次检索多个对象，在你想要检索的对象上放置条件，等等。
<!-- We've already seen how a `Parse.Query` with `get` can retrieve a single `Parse.Object` from Parse. There are many other ways to retrieve data with `Parse.Query` - you can retrieve many objects at once, put conditions on the objects you wish to retrieve, and more. -->

<h2 id="basic-queries">基本查询</h2>

在许多情况下，`get`不够强大，无法指定要检索的对象。 `Parse.Query`提供了不同的方法来检索对象列表，而不仅仅是一个对象。
<!-- In many cases, `get` isn't powerful enough to specify which objects you want to retrieve. `Parse.Query` offers different ways to retrieve a list of objects rather than just a single object. -->

一般的模式是创建一个`Parse.Query`，放置条件，然后使用`find`检索一个`Array`匹配`Parse.Object`s。 例如，要检索具有特定`playerName`的分数，请使用`equalTo`方法来约束键的值。
<!-- The general pattern is to create a `Parse.Query`, put conditions on it, and then retrieve an `Array` of matching `Parse.Object`s using `find`. For example, to retrieve the scores that have a particular `playerName`, use the `equalTo` method to constrain the value for a key. -->

<pre><code class="javascript">
var GameScore = Parse.Object.extend("GameScore");
var query = new Parse.Query(GameScore);
query.equalTo("playerName", "Dan Stemkoski");
query.find({
  success: function(results) {
    alert("Successfully retrieved " + results.length + " scores.");
    // Do something with the returned Parse.Object values
    for (var i = 0; i < results.length; i++) {
      var object = results[i];
      alert(object.id + ' - ' + object.get('playerName'));
    }
  },
  error: function(error) {
    alert("Error: " + error.code + " " + error.message);
  }
});
</code></pre>

<h2 id="query-constraints">查询约束</h2>

有几种方法可以对`Parse.Query`找到的对象设置约束。 您可以使用`notEqualTo`过滤掉带有特定键值对的对象：
<!-- There are several ways to put constraints on the objects found by a `Parse.Query`. You can filter out objects with a particular key-value pair with `notEqualTo`: -->

<pre><code class="javascript">
query.notEqualTo("playerName", "Michael Yabuti");
</code></pre>

您可以给出多个约束，并且只有在对象匹配所有约束时，对象才会在结果中。 换句话说，它就像一个约束的AND。
<!-- You can give multiple constraints, and objects will only be in the results if they match all of the constraints.  In other words, it's like an AND of constraints. -->

<pre><code class="javascript">
query.notEqualTo("playerName", "Michael Yabuti");
query.greaterThan("playerAge", 18);
</code></pre>

你可以通过设置`limit`来限制结果的数量。 默认情况下，结果限制为100，但1到1000之间的任何值都是有效的限制：
<!-- You can limit the number of results by setting `limit`. By default, results are limited to 100, but anything from 1 to 1000 is a valid limit: -->

<pre><code class="javascript">
query.limit(10); // limit to at most 10 results
</code></pre>

如果你想要一个结果，一个更方便的替代方法可能是使用`first`而不是使用`find`。
<!-- If you want exactly one result, a more convenient alternative may be to use `first` instead of using `find`. -->

<pre><code class="javascript">
var GameScore = Parse.Object.extend("GameScore");
var query = new Parse.Query(GameScore);
query.equalTo("playerEmail", "dstemkoski@example.com");
query.first({
  success: function(object) {
    // Successfully retrieved the object.
  },
  error: function(error) {
    alert("Error: " + error.code + " " + error.message);
  }
});
</code></pre>

你可以通过设置`skip`来跳过第一个结果。 这对分页很有用：
<!-- You can skip the first results by setting `skip`. This can be useful for pagination: -->

<pre><code class="javascript">
query.skip(10); // skip the first 10 results
</code></pre>

对于可排序类型（如数字和字符串），您可以控制返回结果的顺序：
<!-- For sortable types like numbers and strings, you can control the order in which results are returned: -->

<pre><code class="javascript">
// Sorts the results in ascending order by the score field
query.ascending("score");

// Sorts the results in descending order by the score field
query.descending("score");
</code></pre>

对于可排序类型，您还可以在查询中使用比较：
<!-- For sortable types, you can also use comparisons in queries: -->

<pre><code class="javascript">
// Restricts to wins < 50
query.lessThan("wins", 50);

// Restricts to wins <= 50
query.lessThanOrEqualTo("wins", 50);

// Restricts to wins > 50
query.greaterThan("wins", 50);

// Restricts to wins >= 50
query.greaterThanOrEqualTo("wins", 50);
</code></pre>

如果要检索与值列表中的任何值匹配的对象，可以使用`containedIn`，提供可接受值的数组。
这通常用于用单个查询替换多个查询。 例如，如果您要检索特定列表中任何播放器所做的评分：
<!-- If you want to retrieve objects matching any of the values in a list of values, you can use `containedIn`, providing an array of acceptable values. This is often useful to replace multiple queries with a single query. For example, if you want to retrieve scores made by any player in a particular list: -->

<pre><code class="javascript">
// Finds scores from any of Jonathan, Dario, or Shawn
query.containedIn("playerName",
                  ["Jonathan Walsh", "Dario Wunsch", "Shawn Simon"]);
</code></pre>

如果要检索不匹配任何几个值的对象，可以使用`notContainedIn`，提供一个可接受值的数组。 
例如，如果您要从列表中除了那些播放器之外的播放器检索分数：
<!-- If you want to retrieve objects that do not match any of  several values you can use `notContainedIn`, providing an array of acceptable values.  For example if you want to retrieve scores from players besides those in a list: -->

<pre><code class="javascript">
// Finds scores from anyone who is neither Jonathan, Dario, nor Shawn
query.notContainedIn("playerName",
                     ["Jonathan Walsh", "Dario Wunsch", "Shawn Simon"]);
</code></pre>

如果要检索具有特定键集的对象，可以使用`exists`。 相反，如果你想检索没有特定键集合的对象，你可以使用`doesNotExist`。
<!-- If you want to retrieve objects that have a particular key set, you can use `exists`. Conversely, if you want to retrieve objects without a particular key set, you can use `doesNotExist`. -->

<pre><code class="javascript">
// Finds objects that have the score set
query.exists("score");

// Finds objects that don't have the score set
query.doesNotExist("score");
</code></pre>

您可以使用`matchesKeyInQuery`方法获取对象，其中键与来自另一个查询的一组对象中的键的值匹配。 例如，如果您有一个包含运动队的类，并且您在用户类中存储了用户的家乡，则可以发出一个查询以查找其家乡球队有获胜记录的用户列表。 查询将如下所示：
<!-- You can use the `matchesKeyInQuery` method to get objects where a key matches the value of a key in a set of objects resulting from another query.  For example, if you have a class containing sports teams and you store a user's hometown in the user class, you can issue one query to find the list of users whose hometown teams have winning records.  The query would look like: -->

<pre><code class="javascript">
var Team = Parse.Object.extend("Team");
var teamQuery = new Parse.Query(Team);
teamQuery.greaterThan("winPct", 0.5);
var userQuery = new Parse.Query(Parse.User);
userQuery.matchesKeyInQuery("hometown", "city", teamQuery);
userQuery.find({
  success: function(results) {
    // results has the list of users with a hometown team with a winning record
  }
});
</code></pre>

相反，要获取对象，其中的键不匹配从另一个查询产生的一组对象中的键的值，请使用`doesNotMatchKeyInQuery`。 
例如，要查找其家乡团队失去记录的用户：
<!-- Conversely, to get objects where a key does not match the value of a key in a set of objects resulting from another query, use `doesNotMatchKeyInQuery`. For example, to find users whose hometown teams have losing records: -->

<pre><code class="javascript">
var losingUserQuery = new Parse.Query(Parse.User);
losingUserQuery.doesNotMatchKeyInQuery("hometown", "city", teamQuery);
losingUserQuery.find({
  success: function(results) {
    // results has the list of users with a hometown team with a losing record
  }
});
</code></pre>

您可以通过使用键列表调用`select`来限制返回的字段。 
要检索只包含`score'和`playerName`字段（以及特殊的内置字段，如`objectId`，`createdAt`和`updatedAt`）的文档：
<!-- You can restrict the fields returned by calling `select` with a list of keys. To retrieve documents that contain only the `score` and `playerName` fields (and also special built-in fields such as `objectId`, `createdAt`, and `updatedAt`): -->

<pre><code class="javascript">
var GameScore = Parse.Object.extend("GameScore");
var query = new Parse.Query(GameScore);
query.select("score", "playerName");
query.find().then(function(results) {
  // each of results will only have the selected fields available.
});
</code></pre>

剩余的字段可以稍后通过在返回的对象上调用`fetch`来获取：
<!-- The remaining fields can be fetched later by calling `fetch` on the returned objects: -->

<pre><code class="javascript">
query.first().then(function(result) {
  // only the selected fields of the object will now be available here.
  return result.fetch();
}).then(function(result) {
  // all fields of the object will now be available here.
});
</code></pre>

<h2 id="queries-on-array-values">数组值查询</h2>

对于数组类型的键，可以通过以下方式查找键的数组值包含2的对象：
<!-- For keys with an array type, you can find objects where the key's array value contains 2 by: -->

<pre><code class="javascript">
// Find objects where the array in arrayKey contains 2.
query.equalTo("arrayKey", 2);
</code></pre>

您还可以找到对象的键的数组值包含每个元素2，3和4的对象，其中包含以下内容：
<!-- You can also find objects where the key's array value contains each of the elements 2, 3, and 4 with the following: -->

<pre><code class="javascript">
// Find objects where the array in arrayKey contains all of the elements 2, 3, and 4.
query.containsAll("arrayKey", [2, 3, 4]);
</code></pre>

<h2 id="queries-on-string-values">对字符串值的查询</h2>

<div class='tip info'><div>
 如果您尝试实施通用搜索功能，我们建议您查看此博文：<a href="http://blog.parse.com/learn/engineering/implementing-scalable-search-on-a-nosql-backend/">Implementing Scalable Search on a NoSQL Backend</a>.
</div></div>

使用`startsWith`限制以特定字符串开头的字符串值。 类似于MySQL LIKE操作符，它是索引的，因此对于大型数据集是有效的：
<!-- Use `startsWith` to restrict to string values that start with a particular string. Similar to a MySQL LIKE operator, this is indexed so it is efficient for large datasets: -->

<pre><code class="javascript">
// Finds barbecue sauces that start with "Big Daddy's".
var query = new Parse.Query(BarbecueSauce);
query.startsWith("name", "Big Daddy's");
</code></pre>

上面的例子将匹配任何`BarbecueSauce`对象，其中`name`字符串键中的值以`Big Daddy's`开头。 
例如，"Big Daddy's"和"Big Daddy's BBQ" 将匹配，但"big daddy's"或"BBQ Sauce: Big Daddy's"不会。
<!-- The above example will match any `BarbecueSauce` objects where the value in the "name" String key starts with "Big Daddy's". For example, both "Big Daddy's" and "Big Daddy's BBQ" will match, but "big daddy's" or "BBQ Sauce: Big Daddy's" will not. -->

有正则表达式约束的查询非常昂贵，尤其是对于具有超过100,000个记录的类。 
解析限制在任何给定时间在特定应用程序上可以运行多少此类操作。
<!-- Queries that have regular expression constraints are very expensive, especially for classes with over 100,000 records. Parse restricts how many such operations can be run on a particular app at any given time. -->


<h2 id="relational-queries">关系查询</h2>

有几种方法可以为关系数据发出查询。 如果要检索字段与特定`Parse.Object`匹配的对象，您可以使用`equalTo`，就像其他数据类型一样。 例如，如果每个`Comment`在它的`post`字段中有一个`Post`对象，你可以获取一个特定的`Post`的注释：
<!-- There are several ways to issue queries for relational data. If you want to retrieve objects where a field matches a particular `Parse.Object`, you can use `equalTo` just like for other data types. For example, if each `Comment` has a `Post` object in its `post` field, you can fetch comments for a particular `Post`: -->

<pre><code class="javascript">
// Assume Parse.Object myPost was previously created.
var query = new Parse.Query(Comment);
query.equalTo("post", myPost);
query.find({
  success: function(comments) {
    // comments now contains the comments for myPost
  }
});
</code></pre>

如果要检索字段包含与另一个查询匹配的`Parse.Object`的对象，可以使用`matchesQuery`。 请注意，默认限制100和最大限制1000也适用于内部查询，因此对于大型数据集，您可能需要仔细构造查询以获取所需的行为。 
为了找到包含图片的帖子的评论，您可以：
<!-- If you want to retrieve objects where a field contains a `Parse.Object` that matches a different query, you can use `matchesQuery`. Note that the default limit of 100 and maximum limit of 1000 apply to the inner query as well, so with large data sets you may need to construct queries carefully to get the desired behavior. In order to find comments for posts containing images, you can do: -->

<pre><code class="javascript">
var Post = Parse.Object.extend("Post");
var Comment = Parse.Object.extend("Comment");
var innerQuery = new Parse.Query(Post);
innerQuery.exists("image");
var query = new Parse.Query(Comment);
query.matchesQuery("post", innerQuery);
query.find({
  success: function(comments) {
    // comments now contains the comments for posts with images.
  }
});
</code></pre>

如果你想检索一个字段包含一个不匹配不同查询的`Parse.Object`的对象，你可以使用`doesNotMatchQuery`。 
为了找到没有图片的帖子的评论，您可以：
<!-- If you want to retrieve objects where a field contains a `Parse.Object` that does not match a different query, you can use `doesNotMatchQuery`.  In order to find comments for posts without images, you can do: -->

<pre><code class="javascript">
var Post = Parse.Object.extend("Post");
var Comment = Parse.Object.extend("Comment");
var innerQuery = new Parse.Query(Post);
innerQuery.exists("image");
var query = new Parse.Query(Comment);
query.doesNotMatchQuery("post", innerQuery);
query.find({
  success: function(comments) {
    // comments now contains the comments for posts without images.
  }
});
</code></pre>

你也可以通过`objectId`做关系查询：
<!-- You can also do relational queries by `objectId`: -->

<pre><code class="javascript">
var post = new Post();
post.id = "1zEcyElZ80";
query.equalTo("post", post);
</code></pre>

在某些情况下，您希望在一个查询中返回多种类型的相关对象。 你可以使用`include`方法。 
例如，假设您要检索最后十条评论，并且想要同时检索其相关帖子：
<!-- In some situations, you want to return multiple types of related objects in one query. You can do this with the `include` method. For example, let's say you are retrieving the last ten comments, and you want to retrieve their related posts at the same time: -->

<pre><code class="javascript">
var query = new Parse.Query(Comment);

// Retrieve the most recent ones
query.descending("createdAt");

// Only retrieve the last ten
query.limit(10);

// Include the post data with each comment
query.include("post");

query.find({
  success: function(comments) {
    // Comments now contains the last ten comments, and the "post" field
    // has been populated. For example:
    for (var i = 0; i < comments.length; i++) {
      // This does not require a network access.
      var post = comments[i].get("post");
    }
  }
});
</code></pre>

你也可以做多级包括使用点符号。 如果你想包含评论的帖子和帖子的作者，你可以做：
<!-- You can also do multi level includes using dot notation.  If you wanted to include the post for a comment and the post's author as well you can do: -->

<pre><code class="javascript">
query.include(["post.author"]);
</code></pre>

您可以通过多次调用`include`来发出包含多个字段的查询。 
这个功能也可以与`Parse.Query`帮助函数，如`first`和`get`。
<!-- You can issue a query with multiple fields included by calling `include` multiple times. This functionality also works with Parse.Query helpers like `first` and `get`. -->

<h2 id="counting-objects">计数对象</h2>

警告：计数查询的速率限制为每分钟最多160个请求。 
它们还可以为具有超过1,000个对象的类返回不准确的结果。 
因此，最好构建应用程序，以避免这种计数操作（例如使用计数器）。
<!-- Caveat: Count queries are rate limited to a maximum of 160 requests per minute.  They can also return inaccurate results for classes with more than 1,000 objects.  Thus, it is preferable to architect your application to avoid this sort of count operation (by using counters, for example.) -->

如果你只需要计算有多少个对象匹配一个查询，但你不需要检索所有匹配的对象，你可以使用`count`而不是`find`。 
例如，计算特定玩家已经玩了多少游戏：
<!-- If you just need to count how many objects match a query, but you do not need to retrieve all the objects that match, you can use `count` instead of `find`. For example, to count how many games have been played by a particular player: -->

<pre><code class="javascript">
var GameScore = Parse.Object.extend("GameScore");
var query = new Parse.Query(GameScore);
query.equalTo("playerName", "Sean Plott");
query.count({
  success: function(count) {
    // The count request succeeded. Show the count
    alert("Sean has played " + count + " games");
  },
  error: function(error) {
    // The request failed
  }
});
</code></pre>

<h2 id="compound-queries">复合查询</h2>

如果要查找与几个查询中的一个匹配的对象，可以使用`Parse.Query.or`方法来构造一个查询，该查询是传入的查询的OR。
例如，如果您想查找具有 很多的胜利或几场胜利，你可以做：
<!--  If you want to find objects that match one of several queries, you can use `Parse.Query.or` method to construct a query that is an OR of the queries passed in.  For instance if you want to find players who either have a lot of wins or a few wins, you can do: -->

<pre><code class="javascript">
var lotsOfWins = new Parse.Query("Player");
lotsOfWins.greaterThan("wins", 150);

var fewWins = new Parse.Query("Player");
fewWins.lessThan("wins", 5);

var mainQuery = Parse.Query.or(lotsOfWins, fewWins);
mainQuery.find({
  success: function(results) {
     // results contains a list of players that either have won a lot of games or won only a few games.
  },
  error: function(error) {
    // There was an error.
  }
});
</code></pre>

您可以向新创建的`Parse.Query`添加附加约束，作为`AND`运算符。
<!-- You can add additional constraints to the newly created `Parse.Query` that act as an 'and' operator. -->

注意，我们不支持GeoPoint或非过滤约束（例如`near'，`withinGeoBox`，`limit`，`skip`，`ascending`/`descending`，`include`） 复合查询。
<!-- Note that we do not, however, support GeoPoint or non-filtering constraints (e.g. `near`, `withinGeoBox`, `limit`, `skip`, `ascending`/`descending`, `include`) in the subqueries of the compound query. -->
