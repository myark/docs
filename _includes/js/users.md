<h1 id="users">用户</h1>

许多应用的核心部分，都有一个用户账号的概念，以便用户安全地访问他们都信息。
我们为此提供了一个`Parse.User`的用户管理类，它能自动完成用户账号管理中所需要的大部分功能。

使用此类，您可以在应用中添加用户账号管理功能。

`Parse.User`是`Parse.Object`的子类，并且具有所有相同的功能，比如灵活的架构、自动持久性和key-value设计接口。 
`Parse.Object`中的所有方法也存在于`Parse.User`中。 区别是`Parse.User`有一些特殊的附加方法来管理用户帐户。

<h2 id="users-property">属性</h2>

`Parse.User`有几个值将它从`Parse.Object`中分离出来：

*   username: 用户的用户名（必填）。
*   password: 用户的密码（注册时需要）。
*   email: 用户的电子邮件地址（可选）。

我们在这里将会详细介绍每一个用户方法的各种用例。


<h2 id="users-signup">注册</h2>

你的程序会做的第一件事可能是要求用户进行注册。 
以下代码是典型的注册用例：

``` javascript
var user = new Parse.User();
user.set("username", "my name");
user.set("password", "my pass");
user.set("email", "email@example.com");

// other fields can be set just like with Parse.Object
// 其他字段可以像Parse.Object一样进行设置
user.set("phone", "415-392-0202");

user.signUp(null, {
  success: function(user) {
    // Hooray! Let them use the app now.
    // 注册成功！让用户开始使用应用吧！
  },
  error: function(user, error) {
    // Show the error message somewhere and let the user try again.
    // 如果发生错误，在某处显示错误信息并让用户重新尝试注册
    alert("Error: " + error.code + " " + error.message);
  }
});
```

这个操作将会在你的Parse应用中异步创建一个新用户。
在此之前，它还会进行检查以确保用户名和邮件地址是唯一的。
此外，它在云服务器中使用了`bcrypt`进行密码的安全加密。
我们从不使用明文密码进行存储，也不会将明文密码发送到客户端。


注意，这里我们使用`signUp`而不是`save`方法。
新的`Parse.User`应该始终使用`signUp`方法创建。
用户在后边更新数据的话可以使用`save`方法来进行更新。


如果注册失败，您应该读取返回的错误信息对象。
最可能的情况是用户名或邮件地址已被注册过。
您应该向用户明确告知该情况，并要求他们尝试使用不同的用户名或邮件进行注册。


您可以使用电子邮件地址作为用户名。
只需要您的用户输入他们的邮件地址，但请把该地址填写到`username`属性中，这样`Parse.User`才会正常工作。
我们将在重置密码部分中讨论如何进行处理。



<h2 id="users-login">登录</h2>

当然，在您允许用户注册后，您需要让他们在以后进行登录。 要完成这个操作，你可以使用类方法`logIn`。

``` javascript
Parse.User.logIn("myname", "mypass", {
  success: function(user) {
    // Do stuff after successful login.
    // 登录成功后的回调函数
  },
  error: function(user, error) {
    // The login failed. Check error to see why.
    // 登录失败，可以进行失败信息展示
  }
});
```


## 验证邮箱

在应用程式的设定中启用电子邮件验证功能，可让应用程式为已确认电子邮件地址的使用者预留部分体验。 电子邮件验证将`emailVerified`键添加到`Parse.User`对象。 当一个`Parse.User`的`email`被设置或修改时，`emailVerified`被设置为`false`。 解析然后通过电子邮件发送给用户一个链接，该链接将`emailVerified`设置为`true`。
<!-- Enabling email verification in an application's settings allows the application to reserve part of its experience for users with confirmed email addresses. Email verification adds the `emailVerified` key to the `Parse.User` object. When a `Parse.User`'s `email` is set or modified, `emailVerified` is set to `false`. Parse then emails the user a link which will set `emailVerified` to `true`. -->

有三个`emailVerified`状态要考虑：
<!-- There are three `emailVerified` states to consider: -->

1.  `true` - 用户通过点击链接Parse通过电子邮件确认他或她的电子邮件地址。 `Parse.Users`在第一次创建用户帐户时永远不能有一个`true`值。
2.  `false` - 在上次刷新`Parse.User`对象时，用户没有确认他或她的电子邮件地址。 如果`emailVerified`是`false`，请考虑在`Parse.User`上调用`fetch`。
3.  _missing_ - `Parse.User`是在电子邮件验证关闭或`Parse.User`没有`email`时创建的。

## 当前用户

如果用户每次打开您的应用程序时都必须登录，这将是麻烦的。 你可以通过使用缓存的当前`Parse.User`对象来避免这种情况。

每当您使用任何注册或登录方法时，用户都将缓存在localStorage中。 您可以将此缓存视为会话，并自动假定用户已登录：
<!-- It would be bothersome if the user had to log in every time they open your app. You can avoid this by using the cached current `Parse.User` object.

Whenever you use any signup or login methods, the user is cached in localStorage. You can treat this cache as a session, and automatically assume the user is logged in:
 -->
<pre><code class="javascript">
var currentUser = Parse.User.current();
if (currentUser) {
    // do stuff with the user
} else {
    // show the signup or login page
}
</code></pre>
您可以通过注销当前用户来清除：
<!-- 
You can clear the current user by logging them out: -->

<pre><code class="javascript">
Parse.User.logOut().then(() => {
  var currentUser = Parse.User.current();  // this will now be null
});
</code></pre>

## 设置当前用户
如果您已经创建了自己的身份验证例程，或以其他方式登录到服务器端的用户，您现在可以将会话令牌传递给客户端并使用`become`方法。 
此方法将确保会话令牌在设置当前用户之前有效。
<!-- 
If you’ve created your own authentication routines, or otherwise logged in a user on the server side, you can now pass the session token to the client and use the `become` method. This method will ensure the session token is valid before setting the current user. -->

<pre><code class="javascript">
Parse.User.become("session-token-here").then(function (user) {
  // The current user is now set to user.
}, function (error) {
  // The token could not be validated.
});
</code></pre>

## 用户对象的安全性

`Parse.User`类在默认情况下是安全的。 `Parse.User`中存储的数据只能由该用户修改。 默认情况下，数据仍然可以由任何客户端读取。 因此，一些`Parse.User`对象被认证并且可以被修改，而其他是只读的。

具体来说，你不能调用任何`save`或`delete`方法，除非`Parse.User`是使用认证方法获得的，例如`logIn`或`signUp`。 这确保只有用户可以更改自己的数据。

以下说明此安全策略：
<!-- The `Parse.User` class is secured by default. Data stored in a `Parse.User` can only be modified by that user. By default, the data can still be read by any client. Thus, some `Parse.User` objects are authenticated and can be modified, whereas others are read-only.

Specifically, you are not able to invoke any of the `save` or `delete` methods unless the `Parse.User` was obtained using an authenticated method, like `logIn` or `signUp`. This ensures that only the user can alter their own data.

The following illustrates this security policy: -->

<pre><code class="javascript">
var user = Parse.User.logIn("my_username", "my_password", {
  success: function(user) {
    user.set("username", "my_new_username");  // attempt to change username
    user.save(null, {
      success: function(user) {
        // This succeeds, since the user was authenticated on the device

        // Get the user from a non-authenticated method
        var query = new Parse.Query(Parse.User);
        query.get(user.objectId, {
          success: function(userAgain) {
            userAgain.set("username", "another_username");
            userAgain.save(null, {
              error: function(userAgain, error) {
                // This will error, since the Parse.User is not authenticated
              }
            });
          }
        });
      }
    });
  }
});

</code></pre>

从`Parse.User.current()`获取的`Parse.User`将始终被认证。

如果你需要检查一个`Parse.User`是否被认证，你可以调用`authenticated`方法。 你不需要检查`authenticated`与通过认证方法获得的`Parse.User`对象。
<!-- The `Parse.User` obtained from `Parse.User.current()` will always be authenticated.

If you need to check if a `Parse.User` is authenticated, you can invoke the `authenticated` method. You do not need to check `authenticated` with `Parse.User` objects that are obtained via an authenticated method. -->

## 其他对象的安全

应用于`Parse.User`的相同的安全模型可以应用于其他对象。 对于任何对象，您可以指定允许哪些用户读取对象，以及允许哪些用户修改对象。 为了支持这种类型的安全性，每个对象具有由[`Parse.ACL`类实现的访问控制列表](http://en.wikipedia.org/wiki/Access_control_list)。

使用`Parse.ACL`的最简单的方法是指定对象只能由单个用户读取或写入。 这是通过使用`Parse.User`初始化`Parse.ACL`来实现的：`new Parse.ACL(user)`生成一个`Parse.ACL`，限制对该用户的访问。 与其他任何属性一样，对象的ACL将在保存对象时更新。 因此，要创建只能由当前用户访问的私人备注：
<!-- The same security model that applies to the `Parse.User` can be applied to other objects. For any object, you can specify which users are allowed to read the object, and which users are allowed to modify an object. To support this type of security, each object has an [access control list](http://en.wikipedia.org/wiki/Access_control_list), implemented by the `Parse.ACL` class.

The simplest way to use a `Parse.ACL` is to specify that an object may only be read or written by a single user. This is done by initializing a Parse.ACL with a `Parse.User`: `new Parse.ACL(user)` generates a `Parse.ACL` that limits access to that user. An object's ACL is updated when the object is saved, like any other property. Thus, to create a private note that can only be accessed by the current user: -->

<pre><code class="javascript">
var Note = Parse.Object.extend("Note");
var privateNote = new Note();
privateNote.set("content", "This note is private!");
privateNote.setACL(new Parse.ACL(Parse.User.current()));
privateNote.save();
</code></pre>

然后，只有当前用户可以访问此注释，但该用户所登录的任何设备都可以访问此注释。此功能对于您希望在多个设备上启用对用户数据访问权限的应用程序（例如个人待办事项 列表。

也可以在每个用户的基础上授予权限。 您可以使用`setReadAccess`和`setWriteAccess`将权限单独添加到`Parse.ACL`。 例如，假设您有一封邮件将发送给一组几个用户，其中每个用户都有权读取和删除该邮件：
<!-- This note will then only be accessible to the current user, although it will be accessible to any device where that user is signed in. This functionality is useful for applications where you want to enable access to user data across multiple devices, like a personal todo list.

Permissions can also be granted on a per-user basis. You can add permissions individually to a `Parse.ACL` using `setReadAccess` and `setWriteAccess`. For example, let's say you have a message that will be sent to a group of several users, where each of them have the rights to read and delete that message:
 -->
<pre><code class="javascript">
var Message = Parse.Object.extend("Message");
var groupMessage = new Message();
var groupACL = new Parse.ACL();

// userList is an array with the users we are sending this message to.
for (var i = 0; i < userList.length; i++) {
  groupACL.setReadAccess(userList[i], true);
  groupACL.setWriteAccess(userList[i], true);
}

groupMessage.setACL(groupACL);
groupMessage.save();
</code></pre>
您也可以使用`setPublicReadAccess`和`setPublicWriteAccess`向所有用户授予权限。 
这允许在留言板上发布评论的模式。 例如，要创建只能由其作者编辑但可由任何人阅读的帖子：
<!-- You can also grant permissions to all users at once using `setPublicReadAccess` and `setPublicWriteAccess`. This allows patterns like posting comments on a message board. For example, to create a post that can only be edited by its author, but can be read by anyone: -->

<pre><code class="javascript">
var publicPost = new Post();
var postACL = new Parse.ACL(Parse.User.current());
postACL.setPublicReadAccess(true);
publicPost.setACL(postACL);
publicPost.save();
</code></pre>

禁止的操作（例如删除您没有写入权限的对象）会导致`Parse.Error.OBJECT_NOT_FOUND`错误代码。 
出于安全目的，这防止客户端区分哪些对象ID存在但是是安全的，而哪些对象ID根本不存在。
<!-- Operations that are forbidden, such as deleting an object that you do not have write access to, result in a `Parse.Error.OBJECT_NOT_FOUND` error code. For security purposes, this prevents clients from distinguishing which object ids exist but are secured, versus which object ids do not exist at all.
 -->
## 重置密码

这是一个事实，一旦你在系统中引入密码，用户将忘记他们。 在这种情况下，我们的库提供了一种方法让他们安全地重置其密码。

要启动密码重置流程，请向用户提供其电子邮件地址，然后致电：
<!-- It's a fact that as soon as you introduce passwords into a system, users will forget them. In such cases, our library provides a way to let them securely reset their password.

To kick off the password reset flow, ask the user for their email address, and call: -->

<pre><code class="javascript">
Parse.User.requestPasswordReset("email@example.com", {
  success: function() {
  // Password reset request was sent successfully
  },
  error: function(error) {
    // Show the error message somewhere
    alert("Error: " + error.code + " " + error.message);
  }
});
</code></pre>

这将尝试匹配给定的电子邮件与用户的电子邮件或用户名字段，并将向他们发送密码重置电子邮件。 通过这样做，您可以选择让用户使用他们的电子邮件作为其用户名，或者您可以单独收集并将其存储在电子邮件字段中。

密码重置的流程如下：
<!-- This will attempt to match the given email with the user's email or username field, and will send them a password reset email. By doing this, you can opt to have users use their email as their username, or you can collect it separately and store it in the email field.

The flow for password reset is as follows: -->

1.用户通过输入他们的电子邮件请求重置他们的密码。
2.解析向他们的地址发送电子邮件，并使用特殊的密码重置链接。
3.用户点击重置链接，并被定向到一个特殊的Parse页面，允许他们键入一个新的密码。
4.用户键入新密码。 他们的密码现在已重置为他们指定的值。
<!-- 1.  User requests that their password be reset by typing in their email.
2.  Parse sends an email to their address, with a special password reset link.
3.  User clicks on the reset link, and is directed to a special Parse page that will allow them type in a new password.
4.  User types in a new password. Their password has now been reset to a value they specify. -->

请注意，此流中的消息传递将以您在Parse上创建此应用程序时指定的名称引用您的应用程序。
<!-- Note that the messaging in this flow will reference your app by the name that you specified when you created this app on Parse.
 -->

## 查询
要查询用户，您可以简单地为`Parse.User`s创建一个新的`Parse.Query`：
<!-- 
To query for users, you can simple create a new `Parse.Query` for `Parse.User`s: -->

<pre><code class="javascript">
var query = new Parse.Query(Parse.User);
query.equalTo("gender", "female");  // find all the women
query.find({
  success: function(women) {
    // Do stuff
  }
});
</code></pre>

## 协会

涉及`Parse.User`的关联在框的右边工作。 例如，假设您正在制作博客应用程序。 要为用户存储新帖子并检索其所有帖子：
<!-- Associations involving a `Parse.User` work right of the box. For example, let's say you're making a blogging app. To store a new post for a user and retrieve all their posts: -->

<pre><code class="javascript">
var user = Parse.User.current();

// Make a new post
var Post = Parse.Object.extend("Post");
var post = new Post();
post.set("title", "My New Post");
post.set("body", "This is some great content.");
post.set("user", user);
post.save(null, {
  success: function(post) {
    // Find all posts by the current user
    var query = new Parse.Query(Post);
    query.equalTo("user", user);
    query.find({
      success: function(usersPosts) {
        // userPosts contains all of the posts by the current user.
      }
    });
  }
});
</code></pre>

## Facebook用户

Parse提供了一种将Facebook与您的应用程序集成的简单方法。 `Parse.FacebookUtils`类集成了`Parse.User`和Facebook Javascript SDK，以便将用户与他们的Facebook身份联系起来很容易。

使用我们的Facebook集成，您可以将已认证的Facebook用户与`Parse.User`关联。 只需几行代码，您就可以在应用程序中提供`使用Facebook登录`选项，并且能够将其数据保存到Parse。
<!-- Parse provides an easy way to integrate Facebook with your application. The `Parse.FacebookUtils` class integrates `Parse.User` and the Facebook Javascript SDK to make linking your users to their Facebook identities easy.

Using our Facebook integration, you can associate an authenticated Facebook user with a `Parse.User`. With just a few lines of code, you'll be able to provide a "log in with Facebook" option in your app, and be able to save their data to Parse.
 -->

### 建立

要开始使用Facebook与Parse，您需要：

1. [设置Facebook应用](https://developers.facebook.com/apps)，如果你还没有。 在"选择您的应用与Facebook的集成方式"下选择"使用Facebook登录的网站"选项，然后输入您的网站的网址。
2.在Parse应用程序的设置页面上添加应用程序的Facebook应用程序ID。
3.按照[这些说明](https://developers.facebook.com/docs/javascript/quickstart/)将Facebook JavaScript SDK加载到应用程序中。
4.用对`Parse.FacebookUtils.init()`的调用替换对`FB.init()`的调用。 例如，如果您异步加载Facebook JavaScript SDK，您的`fbAsyncInit`函数将如下所示：
<!-- To start using Facebook with Parse, you need to:

1.  [Set up a Facebook app](https://developers.facebook.com/apps), if you haven't already.  Choose the "Website with Facebook Login" option under "Select how your app integrates with Facebook" and enter your site's URL.
2.  Add your application's Facebook Application ID on your Parse application's settings page.
3.  Follow [these instructions](https://developers.facebook.com/docs/javascript/quickstart/) for loading the Facebook JavaScript SDK into your application.
4.  Replace your call to `FB.init()` with a call to `Parse.FacebookUtils.init()`. For example, if you load the Facebook JavaScript SDK asynchronously, your `fbAsyncInit` function will look like this: -->

<pre><code class="javascript">
  // Initialize Parse
  Parse.initialize("$PARSE_APPLICATION_ID", "$PARSE_JAVASCRIPT_KEY");

      window.fbAsyncInit = function() {
    Parse.FacebookUtils.init({ // this line replaces FB.init({
      appId      : '{facebook-app-id}', // Facebook App ID
      status     : true,  // check Facebook Login status
      cookie     : true,  // enable cookies to allow Parse to access the session
      xfbml      : true,  // initialize Facebook social plugins on the page
      version    : 'v2.3' // point to the latest Facebook Graph API version
    });

        // Run code after the Facebook SDK is loaded.
  };

      (function(d, s, id){
    var js, fjs = d.getElementsByTagName(s)[0];
    if (d.getElementById(id)) {return;}
    js = d.createElement(s); js.id = id;
    js.src = "//connect.facebook.net/en_US/sdk.js";
    fjs.parentNode.insertBefore(js, fjs);
  }(document, 'script', 'facebook-jssdk'));
</code></pre>

一旦Facebook JavaScript SDK完成加载，就会运行分配给`fbAsyncInit`的函数。 您要在加载Facebook JavaScript SDK之后运行的任何代码都应放在此函数中，并在调用`Parse.FacebookUtils.init()`之后。

如果你遇到任何与Facebook相关的问题，一个好的资源是[从Facebook官方入门指南](https://developers.facebook.com/docs/reference/javascript/)。

如果您遇到类似于从Parse服务器返回的问题，请尝试从应用程序的设置页面中删除您的Facebook应用程序的应用程序密钥。

与您的Parse用户使用Facebook有两个主要方法：
（1）作为Facebook用户登录并创建一个`Parse.User`，或
（2）将Facebook链接到现有的`Parse.User`。
<!-- The function assigned to `fbAsyncInit` is run as soon as the Facebook JavaScript SDK has completed loading. Any code that you want to run after the Facebook JavaScript SDK is loaded should be placed within this function and after the call to `Parse.FacebookUtils.init()`.

If you encounter any issues that are Facebook-related, a good resource is the [official getting started guide from Facebook](https://developers.facebook.com/docs/reference/javascript/).

If you encounter issues that look like they're being returned from Parse's servers, try removing your Facebook application's App Secret from your app's settings page.

There are two main ways to use Facebook with your Parse users: (1) logging in as a Facebook user and creating a `Parse.User`, or (2) linking Facebook to an existing `Parse.User`. -->

### 登陆注册

`Parse.FacebookUtils`提供了一种方法来允许你的`Parse.User`s登录或通过Facebook注册。 这是使用`logIn()`方法实现的：
<!-- `Parse.FacebookUtils` provides a way to allow your `Parse.User`s to log in or sign up through Facebook. This is accomplished using the `logIn()` method:
 -->
<pre><code class="javascript">
Parse.FacebookUtils.logIn(null, {
  success: function(user) {
    if (!user.existed()) {
      alert("User signed up and logged in through Facebook!");
    } else {
      alert("User logged in through Facebook!");
    }
  },
  error: function(user, error) {
    alert("User cancelled the Facebook login or did not fully authorize.");
  }
});
</code></pre>

运行此代码时，会发生以下情况：

1.用户显示Facebook登录对话框。
2.用户通过Facebook验证，您的应用程序接收回调。
我们的SDK接收Facebook数据并将其保存到`Parse.User`。 如果它是基于Facebook ID的新用户，则创建该用户。
你的`success`回调函数是由用户调用的。

您可以选择提供逗号分隔的字符串，指定您的应用程序从Facebook用户需要的[permissions](https://developers.facebook.com/docs/authentication/permissions/)。 例如：
<!-- When this code is run, the following happens:

1.  The user is shown the Facebook login dialog.
2.  The user authenticates via Facebook, and your app receives a callback.
3.  Our SDK receives the Facebook data and saves it to a `Parse.User`. If it's a new user based on the Facebook ID, then that user is created.
4.  Your `success` callback is called with the user.

You may optionally provide a comma-delimited string that specifies what [permissions](https://developers.facebook.com/docs/authentication/permissions/) your app requires from the Facebook user.  For example:
 -->
<pre><code class="javascript">
Parse.FacebookUtils.logIn("user_likes,email", {
  success: function(user) {
    // Handle successful login
  },
  error: function(user, error) {
    // Handle errors and cancellation
  }
});
</code></pre>

`Parse.User`集成不需要任何权限就可以开箱即用（即`null`或指定没有权限是完全可以接受的）。 [详细了解Facebook开发人员指南的权限。](https://developers.facebook.com/docs/reference/api/permissions/)
<!-- `Parse.User` integration doesn't require any permissions to work out of the box (ie. `null` or specifying no permissions is perfectly acceptable). [Read more about permissions on Facebook's developer guide.](https://developers.facebook.com/docs/reference/api/permissions/)
 -->
<div class='tip info'><div>
  它是由你来记录您需要从Facebook用户的任何数据，他们验证后。 要实现这一点，您需要使用Facebook SDK执行图形查询。
</div></div>


### 链接
如果要将现有的`Parse.User`关联到Facebook帐户，可以将其链接如下：
<!-- 
If you want to associate an existing `Parse.User` to a Facebook account, you can link it like so:
 -->
<pre><code class="javascript">
if (!Parse.FacebookUtils.isLinked(user)) {
  Parse.FacebookUtils.link(user, null, {
    success: function(user) {
      alert("Woohoo, user logged in with Facebook!");
    },
    error: function(user, error) {
      alert("User cancelled the Facebook login or did not fully authorize.");
    }
  });
}
</code></pre>

链接时发生的步骤与登录非常相似。不同的是，在成功登录后，现有的`Parse.User`将使用Facebook信息进行更新。 以后通过Facebook登录现在将用户登录到他们现有的帐户。

如果您要取消Facebook与用户的链接，只需执行以下操作：
<!-- The steps that happen when linking are very similar to log in. The difference is that on successful login, the existing `Parse.User` is updated with the Facebook information. Future logins via Facebook will now log the user into their existing account.

If you want to unlink Facebook from a user, simply do this: -->

<pre><code class="javascript">
Parse.FacebookUtils.unlink(user, {
  success: function(user) {
    alert("The user is no longer associated with their Facebook account.");
  }
});
</code></pre>

### Facebook SDK and Parse

Facebook Javascript SDK提供了一个主要的`FB`对象，这是许多与Facebook的API的交互的起点。 [您可以在这里阅读更多关于他们的SDK](https://developers.facebook.com/docs/reference/javascript/)。

使用Parse SDK的Facebook登录要求在调用`Parse.FacebookUtils.init()`之前已经加载了Facebook SDK。

我们的库为你管理`FB`对象。 `FB`单例默认与当前用户同步，因此任何调用它的方法都将作用于与当前`Parse.User`相关联的Facebook用户。 显式调用`FB.login()`或`FB.logOut()`会导致`Parse.User`和`FB`对象不同步，不推荐使用。
<!-- The Facebook Javascript SDK provides a main `FB` object that is the starting point for many of the interactions with Facebook's API. [You can read more about their SDK here](https://developers.facebook.com/docs/reference/javascript/).

Facebook login using the Parse SDK requires that the Facebook SDK already be loaded before calling `Parse.FacebookUtils.init()`.

Our library manages the `FB` object for you. The `FB` singleton is synchronized with the current user by default, so any methods you call on it will be acting on the Facebook user associated with the current `Parse.User`. Calling `FB.login()` or `FB.logOut()` explicitly will cause the `Parse.User` and `FB` object to fall out of synchronization, and is not recommended. -->
