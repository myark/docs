# 文件

## 创建一个 Parse.File

`Parse.File`允许您将应用程序文件存储在云中，否则这些应用程序文件将太大或难以适应常规的“Parse.Object”。 最常见的用例是存储图像，但也可以将其用于文档，视频，音乐和任何其他二进制数据（最多10 MB）。
<!-- `Parse.File` lets you store application files in the cloud that would otherwise be too large or cumbersome to fit into a regular `Parse.Object`. The most common use case is storing images, but you can also use it for documents, videos, music, and any other binary data (up to 10 megabytes). -->

开始使用`Parse.File`是很容易的。 有几种方法来创建文件。 第一个是使用base64编码的字符串。
<!-- Getting started with `Parse.File` is easy. There are a couple of ways to create a file. The first is with a base64-encoded String. -->

<pre><code class="javascript">
var base64 = "V29ya2luZyBhdCBQYXJzZSBpcyBncmVhdCE=";
var file = new Parse.File("myfile.txt", { base64: base64 });
</code></pre>

或者，您可以从字节值数组创建文件：
<!-- Alternatively, you can create a file from an array of byte values: -->

<pre><code class="javascript">
var bytes = [ 0xBE, 0xEF, 0xCA, 0xFE ];
var file = new Parse.File("myfile.txt", bytes);
</code></pre>

Parse将根据文件扩展名自动检测您正在上传的文件的类型，但您可以使用第三个参数指定`Content-Type`：
<!-- Parse will auto-detect the type of file you are uploading based on the file extension, but you can specify the `Content-Type` with a third parameter: -->

<pre><code class="javascript">
var file = new Parse.File("myfile.zzz", fileData, "image/png");
</code></pre>

但最常见的HTML5应用程序，你会想要使用带有文件上传控件的html表单。 在现代浏览器上，这很容易。 创建文件输入标记，允许用户从其本地驱动器中选择要上传的文件：
<!-- But most commonly for HTML5 apps, you'll want to use an html form with a file upload control. On modern browsers, this is easy. Create a file input tag which allows the user to pick a file from their local drive to upload: -->

<pre><code>
&lt;input type="file" id="profilePhotoFileUpload"&gt;
</code></pre>

然后，在点击处理程序或其他函数中，获取对该文件的引用：
<!-- Then, in a click handler or other function, get a reference to that file: -->

<pre><code class="javascript">
var fileUploadControl = $("#profilePhotoFileUpload")[0];
if (fileUploadControl.files.length > 0) {
  var file = fileUploadControl.files[0];
  var name = "photo.jpg";

  var parseFile = new Parse.File(name, file);
}
</code></pre>

注意在这个例子中，我们给文件名'photo.jpg`。 这里有两件事要注意：
<!-- Notice in this example that we give the file a name of `photo.jpg`. There's two things to note here: -->

* 你不需要担心文件名冲突。 每次上传都有一个唯一的标识符，因此上传多个名为`photo.jpg`的文件没有问题。    
* 重要的是给一个具有文件扩展名的文件命名。 这让Parse找出文件类型并相应地处理它。 因此，如果您要存储PNG图像，请确保您的文件名以`.png`结尾。
<!-- *   You don't need to worry about filename collisions. Each upload gets a unique identifier so there's no problem with uploading multiple files named `photo.jpg`.
*   It's important that you give a name to the file that has a file extension. This lets Parse figure out the file type and handle it accordingly. So, if you're storing PNG images, make sure your filename ends with `.png`. -->

接下来，您需要将文件保存到云端。 与`Parse.Object`一样，根据什么类型的回调和错误处理，您可以使用`save`方法的许多变体。
<!-- Next you'll want to save the file up to the cloud. As with `Parse.Object`, there are many variants of the `save` method you can use depending on what sort of callback and error handling suits you. -->

<pre><code class="javascript">
parseFile.save().then(function() {
  // The file has been saved to Parse.
}, function(error) {
  // The file either could not be read, or could not be saved to Parse.
});
</code></pre>

最后，保存完成后，您可以将`Parse.File`与`Parse.Object`相关联，就像其他任何数据一样：
<!-- Finally, after the save completes, you can associate a `Parse.File` with a `Parse.Object` just like any other piece of data: -->

<pre><code class="javascript">
var jobApplication = new Parse.Object("JobApplication");
jobApplication.set("applicantName", "Joe Smith");
jobApplication.set("applicantResumeFile", parseFile);
jobApplication.save();
</code></pre>

## 检索文件内容

如何最佳地检索文件内容取决于您的应用程序的上下文。 由于跨域请求问题，最好是让浏览器为您完成工作。 通常，这意味着将文件的URL转换为DOM。 这里我们使用jQuery在页面上呈现上传的个人资料照片：
<!-- How to best retrieve the file contents back depends on the context of your application. Because of cross-domain request issues, it's best if you can make the browser do the work for you. Typically, that means rendering the file's URL into the DOM. Here we render an uploaded profile photo on a page with jQuery: -->

<pre><code class="javascript">
var profilePhoto = profile.get("photoFile");
$("profileImg")[0].src = profilePhoto.url();
</code></pre>

如果您要在Cloud Code中处理文件的数据，可以使用我们的http网络库检索该文件：
<!-- If you want to process a File's data in Cloud Code, you can retrieve the file using our http networking libraries: -->

<pre><code class="javascript">
Parse.Cloud.httpRequest({ url: profilePhoto.url() }).then(function(response) {
  // The file contents are in response.buffer.
});
</code></pre>

您可以使用[REST API](/parse-doc-zh/rest＃files-deleting-files)删除对象引用的文件。 您将需要提供主密钥，以便允许删除文件。
<!-- You can delete files that are referenced by objects using the [REST API](/docs/rest#files-deleting-files). You will need to provide the master key in order to be allowed to delete a file. -->

如果您的文件未被应用程序中的任何对象引用，则无法通过REST API删除它们。 您可以在应用程式的[设定]页面中要求清除未使用的档案。 请记住，这样做可能会破坏依赖于通过其网址属性访问未引用文件的功能。 当前与某个对象关联的文件将不受影响。
<!-- If your files are not referenced by any object in your app, it is not possible to delete them through the REST API. You may request a cleanup of unused files in your app's Settings page. Keep in mind that doing so may break functionality which depended on accessing unreferenced files through their URL property. Files that are currently associated with an object will not be affected. -->
