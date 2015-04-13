## Step 8 - 用户账户，认证与权限

用户系统是Meteor功能最强大的包之一。现在，我们的应用把parties全部的数据发布到了所有的客户端上，并且客户端可以任意更改parties的内容，同时这些更改也会自动的响应到其他客户端。

这个包功能强大使用简单，但是安全性怎么样呢？我们不想让任何人都能修改数据。

首先我们需要移除Meteor创建应用时自动添加的包"insecure"。"insecure"包使Meteor的数据库集合默认开放所有权限。通过移除这个包可以改变权限为拒绝所有。

所以首先执行命令：
```
meteor remove insecure
```

现在我们尝试更改parties或某个party的内容，会发现更改并没有生效。我们需要对有关Mongo集合的每一项操作都定义明确的安全规则。

所以接下来我们添加一个Meteor包"accounts-password"，这也是一个很强大的包，它提供了几乎所有有关账号认证的功能，比如登录、注册、修改密码、找回密码、验证邮箱等等。
```
meteor add accounts-password
```

我们还需要"accounts-ui"这个包，它包含了所有操作用户需要用到的表单的HTML/CSS，我们会在之后的教程中介绍如何用自定义Angular的表单替换它。
```
meteor add accounts-ui
```

安装完以上两个包后，我们在index.html中引用模板 {{> loginButtons }}，注意，如果你要在AngularJS的模板(*.ng.html)中引用loginButtons，不能再直接写{{> loginButtons }}，因为这是Meteor的模板语言，AngularJS中是不能解析的，angular-meteor为我们提供了一个指令"meteor-include"。

在index.html中添加登录按钮：
```
<meteor-include src="loginButtons"></meteor-include>
```
**index.html**
```
  <head>
    <base href="/">
  </head>
  <body>

  <div ng-app="socially">
    <meteor-include src="loginButtons"></meteor-include>
    <h1>
      <a href="/parties">Home</a>
    </h1>
    <div ui-view></div>
  </div>

  </body>
```

运行程序，现在帐号系统就可以用了。

接下来我们需要为parties设置权限，更改model目录下的parties.js文件如下:
**model/parties.js**
```
Parties = new Mongo.Collection("parties");

Parties.allow({
  insert: function (userId, party) {
    return userId && party.owner === userId;
  },
  update: function (userId, party, fields, modifier) {
    if (userId !== party.owner)
      return false;

    return true;
  },
  remove: function (userId, party) {
    if (userId !== party.owner)
      return false;

    return true;
  }
});
```

collection.allow函数定义了客户端对集合的增删改权限。如果允许操作就在操作类型的回调函数中返回true，否则返回false或者undefined。回调函数有：
- insert(userId, doc)
  
