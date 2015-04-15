Title: Angular-meteor教程翻译#第8节-用户账户, 认证与权限
Date: 2015-04-15
Tags: Web前端, meteor, AngularJS

## [Step 8 - 用户账户，认证与权限](http://angularjs.meteor.com/tutorial/step_08)

用户系统(account)是Meteor功能最强大的包之一。目前我们的应用把parties全部的数据发布到了所有的客户端，客户端可以任意修改parties的内容，同时这些更改也会自动的响应到其他客户端。

这个包功能强大使用简单，但是安全性怎么样呢？毕竟我们不希望谁都能随便修改数据。

首先我们需要移除Meteor创建应用时自动添加的包"insecure"。"insecure"包使Meteor的数据库集合默认开放所有权限。通过移除这个包可以改变权限为拒绝所有。

执行命令一处"insecure"包：
```
meteor remove insecure
```

现在我们尝试更改parties或某个party的内容，会发现更改并没有生效，因为我们需要对有关Mongo集合的每一项操作都定义明确的安全规则。

接下来我们添加一个Meteor包"accounts-password"，这也是一个很强大的包，它提供了几乎所有有关账号认证的功能，比如登录、注册、修改密码、找回密码、验证邮箱等等。
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
  - userId是执行插入操作的用户的id，如果允许插入就返回true.
  - doc是
- update(userId, doc, fieldNames, modifier)
  - userId同上
  - fieldNames是一个数组，它的值是客户端要修改的文档的字段名, 如['name', 'score']
  - modifier是客户端需要执行的Mongo的原生修改器，如 {$set: {'name.first': "Admin"}, $inc: {score: 1}}，这个参数只支持Mongo的修改器，类似的操作如$set和$push，如果想替换整个文档请使用$-modifiers.
- remove(userId, doc)
  - userId同上，允许操作时返回true

在我们上面的代码示例中:
- insert - 只允许party所有者插入记录
- update - 只允许party所有者更新记录
- remove - 只允许party所有者删除记录

好了，到目前为止所有parties都没有所有者，所以我们没有权限更改任何一个party。我们要在创建party的时候定义party的所有者，只需要在创建party的时候把当前用户的id作为party所有者的id就行了。angular-meteor在$rootScope中提供了一个curentUser变量，如果用户已登录，$rootScope.currentUser的值就是包含当前用户信息的对象，否则为null。

$rootScope是应用的顶级作用域，每个应用都有唯一的根作用域，应用中其他的作用域都是根作用域的子域。

在控制器中访问$rootScope需要在依赖注入中添加$rootScope，在模板中访问$rootScope只需要写'$root.'和属性名就可以了。

把parties-list.ng.html中的添加按钮的代码修改为如下形式：
```
<button ng-click="newParty.owner=$root.currentUser._id; parties.push(newParty)">Add</button>
```

接下来像以前一样用不同的用户创建新的party存入数据库中，现在创建的party包含了创建者的id，接着打开两个浏览器，用不同的用户登录，试着编辑或删除party，看看有什么变化。

## 社会化登录
我们想让用户可以使用Facebook或者Twitter账户的登录我们的应用，要实现社会化登录，只需要添加对应的包就行了，在命令行输入:
```
meteor add accounts-facebook
meteor add accounts-twitter
```

运行应用，当第一次点击社会化登录按钮时,meteor会帮助你设置app信息，你也可以跳过帮助手动配置应用信息，关于手动设置请参考 [http://docs.meteor.com/#meteor_loginwithexternalservice](http://docs.meteor.com/#meteor_loginwithexternalservice)

你还可以使用以下的社会化登录服务:
- Facebook
- Github
- Google
- Meetup
- Twitter
- Weibo
- Meteor developer account

## 路由认证
现在我们已经阻止了用户操作不属于自己的party的信息，未登录也不允许查看party的详细信息。我们可以通过设置路由来组织他们访问party详情页面。angular-meteor给我们提供了两个方法在路由部分设置权限:
- waitForUser - 返回一个promise，如果用户已经登录会包含用户信息，否则返回null
- requireUser - 用户已登录的情况下和waitForUser相同，未登录时会处理而不是返回null

所以我们只需要在partyDetails路由部分添加requireUser即可阻止未登录用户访问。在ui-router和ngRoute中添加resolve对象：
**client/routes.js**
```
.state('partyDetails', {
  url: '/parties/:partyId',
  templateUrl: 'client/parties/views/party-details.ng.html',
  controller: 'PartyDetailsCtrl',
  resolve: {
    "currentUser": ["$meteor", function($meteor){
      return $meteor.requireUser();
    }]
  }
});
```

我们希望在用户未登录访问时将其重定向到应用主页，在路由上面部分添加一下代码:
**client/routes.js**
```
angular.module("socially").run(["$rootScope", "$state", function($rootScope, $state) {
  $rootScope.$on("$stateChangeError", function(event, toState, toParams, fromState, fromParams, error) {
    // We can catch the error thrown when the $requireUser promise is rejected
    // and redirect the user back to the main page
    if (error === "AUTH_REQUIRED") {
      $state.go('parties');
    }
  });
}]);
```

##总结
下一章我们谈一谈隐私，如何隐藏不允许用户查看的party。









