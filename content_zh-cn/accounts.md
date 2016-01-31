---
title: 用户和账号
order: 5
description: 如何在流星应用中构建用户登录功能。让你的用户可以通过密码，Facebook，Google，Github，或其他（第三方）进行登录。
---

通过阅读本文，你可以知道：

1. 什么是流星激活用户账号的核心功能
1. 如何使用 accounts-ui 进行快速原型化
1. 如何使用 useraccounts 系列的包来构建你的登录UI
1. 如何构建完整功能的密码登录的经验
1. 如何使用通过OAuth提供者（如，Facebook）实现登录
1. 如何向流星的users集合添加自定义数据
1. 如何管理用户角色和权限

<h2 id="core-meteor">流星核心功能</h2>

在我们深入到不同面向用户账号功能之前，让我们先过一下，构建在流星DDP协议上的 `account-base` 包的一些功能。这些部分是你绝对需要知道的，如果你的应用中具有用户账号的话；多数的功能是可选的并且是通过添加／删除包来实现的。

<h3 id="userid-ddp">DDP中的userId</h3>

你可以通过阅读 [数据装载](data-loading.html) 和 [方法](methods.html)中的文章来了解如何使用它。除了，数据装载和方法调用的概念外，DDP还有一种内置功能 － 对于某个链接之上的 `userId` 字段概念。这是为了能够追踪登录状态，不管你使用何种账号UI包或者登录服务。

该内置功能意味着你总是可以在方法和发布中得到 `this.userId` ， 并且能够在客户端存取该用户的ID。这对于构建你自己的账号系统是一个很好的开始，而多数的开发人员不必担心相应的机制，相应的，你大多数时候只需要和 `account-base` 包打交道就可以了。

<h3 id="accounts-base">`accounts-base`</h3>

该包是流星面向开发者的用户账号功能的核心。包括：

1. 一个具有标准架构的用户集合，通过 [`Meteor.users`](http://docs.meteor.com/#/full/meteor_users)，客户端的单例 [`Meteor.userId()`](http://docs.meteor.com/#/full/meteor_userid)和 [`Meteor.user()`](http://docs.meteor.com/#/full/meteor_user)，代表客户端登录状态 来进行存取。
2. 一组有用的通用方法集用来追踪登录状态、登出，验证用户等。请访问[文档的账号章节](http://docs.meteor.com/#full/accounts_api)，可以找到完整的列表。
3. 一个注册新登录的处理器API，可以用来其他账号包集成到账号系统中。该API没有任何的官方文档，但你可以[阅读更多关于它的信息在MeteorHacks博客上](https://meteorhacks.com/extending-meteor-accounts)。

通常，你不需要显式包含 `accounts-base` 因为，如果你使用 `accounts-password` 或其他类似的，它会被自动添加，但，理解它是什么总是一件好事。 

<h2 id="accounts-ui">通过 `accounts-ui` 快速原型化</h2>

通常，当你开始一个新的应用时，构建一个复杂的账号系统不是你想要做的第一件事情，所以，有一些能让你通过放置快速开始，是很有用的。这是 `accounts-ui` 存在的意义 － 它只需要一行代码被放置在你的应用中，就可以获得一个账号系统。要添加它： 

```js
meteor add accounts-ui
```

接着，只需要在任意某个Blaze模版中包含下列代码：

```html
{{> loginButtons}}
```

接着，只需要挑选一个登录提供器；它们会自动与 `accounts-ui` 进行集成：

```sh
# 从以下挑选一个或者多个
meteor add accounts-password
meteor add accounts-facebook
meteor add accounts-google
meteor add accounts-github
meteor add accounts-twitter
meteor add accounts-meetup
meteor add accounts-meteor-developer
```

现在只需要打开你的应用，遵循配置步骤，那就好了 － 如果你已经完成了 [流星教程](https://www.meteor.com/tutorials/blaze/adding-user-accounts)，那么你已经可以看到效果。当然，在生产环境中的应用，你可能需要某个更定制化的用户界面和一些逻辑来完成某些裁剪过的用户体验，但这正是为什么我们会有接下来的章节。

这里有一些关于 `accounts-ui`的截图，应该是：

<img src="images/accounts-ui.png">

<h2 id="useraccounts">定制化用户界面: useraccounts</h2>

一旦你已经有了初始化的原型并且通过 `accounts-ui` 运行起来了，你可以向更具有威力、可配置的方向去发展，以至于你能够更好地和你的程序集成登录流程。该 [`useraccounts`系列包](http://useraccounts.meteor.com/)是非常有力的、为流星准备的一组账号管理用户界面控件。如果你需要进一步的定制化，你可以发布你自己的系统，但请首先尝试 `useraccounts` 是否值得。

<h3 id="useraccounts-flexibility">使用任意的router或者用户界面框架</h3>

	理解 `useraccounts` 的第一件事情是核心账号管理逻辑是独立于HTML模版和路由包的。这意味着你可以使用 [`useraccounts:core`](https://atmospherejs.com/useraccounts/core)来构建你自己的一组登录模版。总的来说，你会想要选择一个登录模版包和一个登录路由包。模版选项包含：

- [`useraccounts:unstyled`](https://atmospherejs.com/useraccounts/unstyled) 可以让你代入你自己的CSS；该包被使用在Todos示例应用中，使得登录用户界面无缝地融合在该应用中。
- 为了 [Bootstrap, Semantic UI, Materialize, 以及其他](http://useraccounts.meteor.com) 的预制模版。这些模版没有实际的CSS框架，所以你可以选择你喜欢的Bootstrap包，举个例子。

当它是可选的并且基础的功能不需要它也能工作，选择一个路由来集成是个好主意：

- [Flow Router](https://atmospherejs.com/useraccounts/flow-routing), 该路由 [本指南推荐](routing.html).
- [Iron Router](https://atmospherejs.com/useraccounts/iron-routing), 另一个流行在流星社区的路由。

在该示例程序中，我们使用Flow路由集成得非常成功。稍候章节将会谈到如何定制化该路由和模版以更好的适应你的应用。

<h3 id="useraccounts-drop-in">放置用户界面不使用路由</h3>

如果你不想要为你的登录流程配置路由，你可以仅仅放置一个自我管理的账号屏幕。无论你想要在何处渲染账号用户界面模版，只需要包含 `atForm` 模版，如下所示：

```html
{{> atForm}}
```

一旦你根据 [以下章节](#useraccounts-customizing-routes)配置了路由，你将会需要移除该包含。

<h3 id="useraccounts-customizing-templates">自定义模版</h3>

对于有些应用，不同的 `useraccounts` 用户界面包提供现成的模版工作得还行，但多数应用会要自定义一些表现形式。有个简单的方法可以实现，那就是使用 `aldeed:template-extension` 包的模版替换功能。

首先，通过查看源代码弄清楚哪个模版是你想要替换的。例如，在 `useraccounts:unstyled` 包中，所有的模版被列在 [Github的目录中](https://github.com/meteor-useraccounts/unstyled/tree/master/lib)。通过查看文件名和一些HTML字符串，我们能够清除，我们也许希望替换 `atPwdFormBtn` 模版。让我们来看看原始的模版：

```html
<template name="atPwdFormBtn">
  <button type="submit" class="at-btn submit {{submitDisabled}}" id="at-btn">
    {{buttonText}}
  </button>
</template>
```

一旦你认定了哪个模版是你要替换的，定义一个新模版。在本例中，我们需要修改按钮的（CSS）类以便于程序可以和CSS工作在一起。当覆写某个模版时，有些事情请牢记：

1. 和之前的模版一样渲染所有的帮助器。在本例中我们使用 `buttonText`。
2. 保留任何的 `id` 标签，如 `at-btn`，因为这些会在事件处理中被使用到。

这里是我们新覆写的模版，如下：

```html
<template name="override-atPwdFormBtn">
  <button type="submit" class="btn-primary" id="at-btn">
    {{buttonText}}
  </button>
</template>
```

Then, use the `replaces` function on the template to override the existing template from `useraccounts`:接着，使用 `replaces` 函数在模版上覆写已经存在于 `useraccounts` 的模版，如下：

```js
Template['override-atPwdFormBtn'].replaces('atPwdFormBtn');
```

<h3 id="useraccounts-customizing-routes">自定义路由</h3>

除了控制模版外，你或许也想要能控制路由和 `useraccounts` 提供对于不同视图的URLs。既然Flow路由是流星官方推荐的路由选项，我们将特别对其进行说明。

首先，我们需要配置我们想要使用的布局，当渲染账号模版时：

```js
AccountsTemplates.configure({
  defaultTemplate: 'Auth_page',
  defaultLayout: 'App_body',
  defaultContentRegion: 'main',
  defaultLayoutRegions: {}
});
```

本例中，我们会使用 `App_body` 布局模版应用在所有的账号相关的页面上。这个模版具有一个内容区域被称为 `main`。现在，让我们来配置一些路由：

```js
// 定义这些路由在某个客户端和服务器端均可存取的文件中
AccountsTemplates.configureRoute('signIn', {
  name: 'signin',
  path: '/signin'
});

AccountsTemplates.configureRoute('signUp', {
  name: 'join',
  path: '/join'
});

AccountsTemplates.configureRoute('forgotPwd');

AccountsTemplates.configureRoute('resetPwd', {
  name: 'resetPwd',
  path: '/reset-password'
});
```

现在我们可以很容易在我们的登录页面上渲染链接，如下：

```html
<div class="btns-group">
  <a href="{{pathFor 'signin'}}" class="btn-secondary">Sign In</a>
  <a href="{{pathFor 'join'}}" class="btn-secondary">Join</a>
</div>
```

注意我们特别指定了一个密码重置的路由。通常，我们会不得不配置流星账号系统以发送该路由到密码重置电邮中，但 `useraccounts:flow-routing` 已经为我们做好了这一切。[阅读更多关于配置电邮流程如下。](#email-flows)

You can find a complete list of different available routes in the [documentation the `useraccounts:flow-routing`](https://github.com/meteor-useraccounts/flow-routing#routes).在 [ `useraccounts:flow-routing` 文档](https://github.com/meteor-useraccounts/flow-routing#routes)中，我们可以找到完整的，不同的可用的路由，的列表。

<h3 id="useraccounts-further-customization">进一步的自定义</h3>

`useraccounts` 提供很多其他自定义的选项超越了模版和路由。阅读 [`useraccounts` 指南](https://github.com/meteor-useraccounts/core/blob/master/Guide.md)来学习其它所有的选项。

<h2 id="accounts-password">密码登录</h2>

流星具有一个安全、功能完整的、开箱即用的登录系统。要使用它，添加包：

```sh
meteor add accounts-password
```

要查看有哪些选项，请阅读[流星文档中的`accounts-password` API](http://docs.meteor.com/#/full/accounts_passwords)的完整描述。

<h3 id="requiring-username-email">需要用户名或电邮</h3>

> 注意：如果你使用 `useraccounts` ，那么你并不需要这么做。它禁用了常规的额流星客户端账号创建功能，并且做了自定义的验证。

默认的， `Accounts.createUser` 函数是由`accounts-password` 提供的，用来通过用户名，电邮或者两者来创建一个账号。多数应用期待某个特定的两者之间的组合，所以，你将会当然地想要验证新用户的创建：

```js
// 确保每个用户都有一个电子邮件地址，这应该在服务器端的代码中
Accounts.validateNewUser((user) => {
  new SimpleSchema({
    _id: { type: String },
    emails: { type: Array },
    'emails.$': { type: Object },
    'emails.$.address': { type: String },
    'emails.$.verified': { type: Boolean },
    createdAt: { type: Date },
    services: { type: Object, blackbox: true }
  }).validate(user);

  // 返回true以允许用户创建进行
  return true;
});
```

<h3 id="multiple-emails">多个电子邮件地址</h3>

经常性地，用户可能想要关联多个电子邮件地址到同一个账号。 `accounts-password` 通过存储电子邮件地址作为user集合中的一个数组。还提供一些有用的API方法来进行处理[adding](http://docs.meteor.com/#/full/Accounts-addEmail), [removing](http://docs.meteor.com/#/full/Accounts-removeEmail), 和 [verifying](http://docs.meteor.com/#/full/accounts_verifyemail)电子邮件。

对于你的应用，一个“主”电子邮件地址概念被添加，是一件有用的事情。这种方式，如果用户要添加多个电子邮件，你会知道向哪里发送确认邮件。

<h3 id="case-sensitivity">大小写敏感</h3>

流星1.2之前，所有存储在数据库中的电子邮件和用户名是大小写敏感的。这意味着，如果你用 `AdaLovelace@example.com` 注册了一个账号，你可能看到一个错误用来指定已经有用户使用这个电邮。当然，这是很令人疑惑，所以我们决定在流星1.2中做出改进。但情况并不是如所见的那么简单；既然MongoDB没有大小写敏感索引的概念，要保证数据库层面的电子邮件唯一性是不可能的。因为这个原因，我们有了一些为查询和更新用户而特定的API，用来在应用级别上管理大小写敏感的问题。

<h4 id="case-sensitivity-in-my-app">这对于我的应用意味着什么？</h4>

只需遵循一条简单的原则：不要通过数据库直接查询 `username` 或 `email`。请使用由流星提供的[`Accounts.findUserByUsername`](http://docs.meteor.com/#/full/Accounts-findUserByUsername) 和 [`Accounts.findUserByEmail`](http://docs.meteor.com/#/full/Accounts-findUserByEmail) 方法作为替代。 这会替你运行一条大小写敏感的查询，你就总能找到你想要找到的用户了。

<h3 id="email-flows">电子邮件流程</h3>

当你需要基于用户电子邮件来为你的应用制作一个登录系统，这就可能用到基于电邮账号的流程。在所有这些工作流中有一个共性就是都包含发送一个特定的链接到用户的电子邮件地址，而且，当用户点击时会做一些特定的事情。让我们来看一些通用的例子，这是由流星的`accounts-password` 开箱就支持的。

1. **重置密码。** 当用户在他们的邮件中点击链接，他们会被定向到某个页面，在那里他们可以为他们的账号输入新密码。
1. **用户注册。** 当某个新用户被管理员创建出来，但却没有设置密码。当用户在他们的邮件中点击链接时，他们会被定向到某个页面，在那里，他们可以设置新的密码。和重置密码很类似。
1. **电子邮件验证。** 当用户在他们的邮件中点击链接时，应用会记录该电子邮件是否属于正确的用户。

这里，我们探讨一下如何从头至尾手动管理整个流程。

<h4 id="default-email-flow">电子邮件和账号用户界面包协同工作</h4>

如果你想要一些开箱即用的功能，你可以使用 `accounts-ui` 或 `useraccounts`，基本上覆盖了所有的需求。只有当你相当确定地想要建立自己的电子邮件流程全部时，才需要遵循以下指示。

<h4 id="sending-email">发送电子邮件</h4>

`accounts-password` 具有一些有用的方法，使你能够从服务器端调用发送一个电子邮件。它们的行为就和它们的命名一样：

1. [`Accounts.sendResetPasswordEmail`](http://docs.meteor.com/#/full/accounts_sendresetpasswordemail)
2. [`Accounts.sendEnrollmentEmail`](http://docs.meteor.com/#/full/accounts_sendenrollmentemail)
3. [`Accounts.sendVerificationEmail`](http://docs.meteor.com/#/full/accounts_sendverificationemail)

The email is generated using the email templates from [Accounts.emailTemplates](http://docs.meteor.com/#/full/accounts_emailtemplates), and include links generated with `Accounts.urls`. We'll go into more detail about customizing the email content and URL later.电子邮件的产生是通过从[Accounts.emailTemplates](http://docs.meteor.com/#/full/accounts_emailtemplates) 指定的电子邮件模版，其包含通过 `Accounts.urls` 产生的链接。我们稍候深入关于自定义电子邮件内容和链接的细节。

<h4 id="identifying-link-click">甄别何时链接会被点击</h4>

当用户收到电子邮件并且点击其中的链接，他们的浏览器将会把用户带到你的应用。现在，你需要能够合适地甄别这些特殊的链接和行为。如果你不能够自定义这些链接URL，那么你可以使用一些内置的回调来甄别，当该应用处于电子邮件流程中的时候。

通常地，当流星客户端链接到服务器，第一件事情就是传递 _login resume token_ 到重连接的上次登录。然而，当这些回调从电子邮件流被触发时， resume token 不会被发送，直到你的代码发送信号，它已经完成处理该请求，通过调用 `done` 函数（该函数被传递到注册过的回调）来完成。这意味着如果你先前作为用户A进行过登录，接着你为用户B点击了重置密码的链接，接着你通过调用`done()` 来取消重置密码的流程，客户端将再次登录为A。

1. [`Accounts.onResetPasswordLink`](http://docs.meteor.com/#/full/Accounts-onResetPasswordLink)
2. [`Accounts.onEnrollmentLink`](http://docs.meteor.com/#/full/Accounts-onEnrollmentLink)
3. [`Accounts.onEmailVerificationLink`](http://docs.meteor.com/#/full/Accounts-onEmailVerificationLink)

以下是如何使用其中某个方法：

```js
Accounts.onResetPasswordLink((token, done) => {
  // 显示密码重置用户界面，得到新的密码...

  Accounts.resetPassword(token, newPassword, (err) => {
    if (err) {
      // 显示错误
    } else {
      // 继续正常流程
      done();
    }
  });
})
```

如果为了你的密码重置页面，需要一个不同的URL，你需要通过 `Accounts.urls` 选项来定制它：

```js
Accounts.urls.resetPassword = (token) => {
  return Meteor.absoluteUrl(`reset-password/${token}`);
};
```

如果你定制了这个URL，你会要添加一个新的路由到你的路由器中以处理你所指定的URL，并且默认的 `Accounts.onResetPasswordLink` 和他的朋友将不再工作。

<h4 id="completing-email-flow">显示一个合适的用户界面和完成该流程</h4>

现在，你知道哪些用户试图重置它们的密码，设置一个初始化的密码，货验证他们的电子邮件，你应该显示一个合适的用户界面以允许他们这么做。例如，你可能想要显示一个页面，具有一个表单，让用户可以输入他们的新密码。

当用户提交该表单时，你需要调用相应的函数并将他们的改变提交到数据库中。每个函数都具有新值和token（从上一步的事件中获得）。

1. [`Accounts.resetPassword`](http://docs.meteor.com/#/full/accounts_resetpassword) － 这应该被用来作为重置密码，和注册一个新用户；它会接受两种令牌。
2. [`Accounts.verifyEmail`](http://docs.meteor.com/#/full/accounts_verifyemail)

在你调用了上述两个函数中的一个或用户取消了该过程，调用了 `done` 函数（在链接的回调中获得）。这会告诉流星离开它进入的特殊的状态（这种特殊的状态是指，你正在出入电子邮件账号流程中的某一步时）。

<h3 id="customizing-emails">自定义账号电子邮件</h3>

你可能想要定制化`accounts-password` 发送出的电子邮件，根据你的要求。这能够很容易地通过[`Accounts.emailTemplates` API](http://docs.meteor.com/#/full/accounts_emailtemplates)来实现。以下是从Todos应用中来的代码示例：

```js
Accounts.emailTemplates.siteName = "Meteor Guide Todos Example";
Accounts.emailTemplates.from = "Meteor Todos Accounts <accounts@example.com>";

Accounts.emailTemplates.resetPassword = {
  subject(user) {
    return "Reset your password on Meteor Todos";
  },
  text(user, url) {
    return `Hello!
Click the link below to reset your password on Meteor Todos.
${url}
If you didn't request this email, please ignore it.
Thanks,
The Meteor Todos team
`
  },
  html(user, url) {
    // This is where HTML email content would go.
    // See the section about html emails below.
  }
};
```

你可能注意到，我们可以使用ES2015模版字符串功能来产生多行字符串用来包含密码重置的URL。我们也能设置一个自定义的  `from` 地址以及电子邮件标题。

<h4 id="html-emails">HTML 电子邮件</h4>

如果你曾经需要处理，从应用中，发送漂亮的HTML电子邮件，你可能知道这会很快变成一场噩梦。流行的电子邮件客户端基本HTML功能（如CSS）的兼容性是众所周知的千奇百怪，所以要写一些能够在所有客户端都工作的，尤其困难。从一个[responsive email template](https://github.com/leemunroe/responsive-html-email-template) 或者 [framework](http://foundation.zurb.com/emails/email-templates.html)开始, 然后使用一个工具来转换你的电子邮件内容到能够适应所有的那些客户端。[Mailgun的博客文章覆盖了HTML电子邮件的主要问题](http://blog.mailgun.com/transactional-html-email-templates/) 理论上，一个社区包应该可以扩展流星的构建系统来为你做到电子邮件编译，但在写作的时候，我们还没有发现这样的包。

<h2 id="oauth">OAuth 登录</h2>

很久之前，要让谷歌或Facebook的登录与你的应用结合工作会是一个头疼的问题。谢天谢地，多数流行的登录提供者都已经围绕某版[OAuth](https://en.wikipedia.org/wiki/OAuth)标准化了，而且流行支持一些最流行的登录服务开箱即用。

<h3 id="supported-login-services">Facebook, Google, 以及更多</h3>

这里是登录提供者完整的列表，这些提供器是由流星积极维护的核心包：

1. Facebook 通过 `accounts-facebook`
2. Google 通过 `accounts-google`
3. GitHub 通过 `accounts-github`
4. Twitter 通过 `accounts-twitter`
5. Meetup 通过 `accounts-meetup`
6. Meteor Developer Accounts 通过 `accounts-meteor-developer`

有个新浪微博的登录包，但已经很久没有积极地维护了。

<h3 id="oauth-logging-in">登录</h3>

如果你使用现成的登录用户界面，如 `accounts-ui` 或 `useraccounts`，在添加完上述相应的包后，你甚至不需要写任何代码。如果你正在从头构建登录体验，你可以编程地登录使用[`Meteor.loginWith<Service>`](http://docs.meteor.com/#/full/meteor_loginwithexternalservice)函数。它看上去如下所示：

```js
Meteor.loginWithFacebook({
  requestPermissions: ['user_friends', 'public_profile', 'email']
}, (err) => {
  if (err) {
    // 处理错误
  } else {
    // 成功登录
  }
});
```

<h3 id="oauth-configuration">配置OAuth</h3>

在配置OAuth登录需要知道的几点：

1. **客户 ID和秘密.** It's best to keep your OAuth secret keys outside of your source code, and pass them in through Meteor.settings. Read how in the [Security article](security.html#api-keys-oauth).最好不要在你的源代啊中保留OAuth秘钥，而是通过Meteor.settings来传递。参见[安全性 文章](security.html#api-keys-oauth)。
2. **重定向URL.** 对于OAuth提供着方面，你需要指定一个_redirect URL_（重定向URL）。该URL会看上去像：`https://www.example.com/_oauth/facebook`。用你使用的服务替换掉 `facebook` 。注意你需要配置两个URLs － 一个你为了你的生产环境，另一个是为了你的开发环境，这个URL可能是 `http://localhost:3000/_oauth/facebook`。
3. **权限.** 每个登录服务提供者应该有文档描述哪些可用的权限。例如， [这是Facebook那页](https://developers.facebook.com/docs/facebook-login/permissions)。如果你想要额外的用户数据的权限，当他们登录时，可以传递一些字符串在 `requestPermissions` 选项中，到 `Meteor.loginWithFacebook` 或 [`Accounts.ui.config`](http://docs.meteor.com/#/full/accounts_ui_config)。下一节中，我们将会讨论如何获取这些数据。

<h3 id="oauth-calling-api">为了更多的数据调用服务API</h3>

如果你的应用支持或甚至需要外部服务登录，如Facebook，那么使用那服务API来获取额外关于该用户的数据也是很自然的事情。例如，那你可能想要一组某个Facebook用户的照片。

First, you'll need to request the relevant permissions when logging in the user. See the [section above](#oauth-configuration) for how to pass those options.首先，你需要取得相关的权限，当该用户登录的时候。参见[上面的章节](#oauth-configuration) 来了解如何传递这些选项。

接着，你需要获得用户的准入令牌。你可以在`Meteor.users` 集合的 `services` 字段中找到该令牌。例如，如果你需要获得某个特定用户的Facebook准入令牌：

```js
// 给定某个userId，获得用户的Facebook准入令牌
const user = Meteor.users.findOne(userId);
const fbAccessToken = user.services.facebook.accessToken;
```

更多关于数据存储在用户数据库中的细节，参见以下关于存取用户数据的章节。

现在你有了准入令牌，你需要对适当的API发起一个请求。这里你呦两个选择：

1. 使用[`http` 包](http://docs.meteor.com/#/full/http) 来直接存取服务的API。你可能需要传递准入令牌在http的头中。细节你可以查询该服务的API文档。
2. 使用从Atmosphere 或者 NPM而来的某个包，包装了该API到某个好用的Javascript接口。例如，如果你试图载入从Facebook而来的数据，你可以使用[fbgraph](https://www.npmjs.com/package/fbgraph) NPM 包。阅读更多关于如何从你的应用中使用NPM，在[构建系统文章](build-tool.html#npm)。

<h2 id="displaying-user-data">载入和显示用户数据</h2>

流星的账号系统，在 `accounts-base` 中实现，包含了一个数据库集合和萎了获得用户数据的通用方法集。

<h3 id="current-user">当前登录的用户</h3>

一旦某个用户通过上述方法中的某个，登录到你的应用中，能甄别到哪个用户已经登录，并且获得在注册过程中提供的数据，是会很有用的。

<h4 id="current-user-client">在客户端: Meteor.userId()</h4>

对于客户端运行的代码，全局具有 `Meteor.userId()` 响应式的方法会给予你当前登录用户的ID。

额外的，对于该核心API，有些有用的帮助器：`Meteor.user()`, 相当于调用`Meteor.users.findOne(Meteor.userId())`，以及`{% raw %}{{currentUser}}{% endraw %}` Blaze帮助器，用来返回 `Meteor.user()` 的值。

Note that there is a benefit to restricting the places you access the current user to make your UI more testable and modular. Read more about this in the [UI article](ui-ux.html#global-stores).注意通过限制你存取当前用户的地点，可以使得你的用户界面更具有可测试性和模块化。阅读更多相关信息 [用户界面文章](ui-ux.html#global-stores)。

<h4 id="current-user-server">服务器端: this.userId</h4>

在服务器端，每个连接具有不同的登录用户，所以没有全局的登录用户状态被定义。既然流星为每个流星调用追踪环境，你仍然可以使用全局的 `Meteor.userId()` ，其可能返回不同的值，依赖于哪个方法你从中调用，但你可能遇到极端的情况，当需要处理异步代码时。还有， `Meteor.userId()` 不会在发布中工作。

我们建议使用 `this.userId` 属性在方法和发布上下文中，并且以传递函数参数形式，到任何你想要它的地方。

```js
// 在发布中获得 this.userId
Meteor.publish('Lists.private', function() {
  if (!this.userId) {
    return this.ready();
  }

  return Lists.find({
    userId: this.userId
  }, {
    fields: Lists.publicFields
  });
});
```

```js
// 在方法中获得 this.userId
Meteor.methods({
  'Todos.methods.updateText'({ todoId, newText }) {
    new SimpleSchema({
      todoId: { type: String },
      newText: { type: String }
    }).validate({ todoId, newText }),

    const todo = Todos.findOne(todoId);

    if (!todo.editableBy(this.userId)) {
      throw new Meteor.Error('Todos.methods.updateText.unauthorized',
        'Cannot edit todos in a private list that is not yours');
    }

    Todos.update(todoId, {
      $set: { text: newText }
    });
  }
});
```

<h3 id="meteor-users-collection">Meteor.users 集合</h3>

流星具有对于用户数据的默认MongoDB集合。它被存储在数据库中位于 `users` 命名下，你的代码可以通过 `Meteor.users` 进行存取。该集合中用户文档的架构会依据被使用的登录服务用来创建账号。这里有一个使用 `accounts-password` 创建账号的用户的示例：

```js
{
  "_id": "DQnDpEag2kPevSdJY",
  "createdAt": "2015-12-10T22:34:17.610Z",
  "services": {
    "password": {
      "bcrypt": "XXX"
    },
    "resume": {
      "loginTokens": [
        {
          "when": "2015-12-10T22:34:17.615Z",
          "hashedToken": "XXX"
        }
      ]
    }
  },
  "emails": [
    {
      "address": "ada@lovelace.com",
      "verified": false
    }
  ]
}
```

以下是同一个用户如果他们使用Facebook进行登录：

```js
{
  "_id": "Ap85ac4r6Xe3paeAh",
  "createdAt": "2015-12-10T22:29:46.854Z",
  "services": {
    "facebook": {
      "accessToken": "XXX",
      "expiresAt": 1454970581716,
      "id": "XXX",
      "email": "ada@lovelace.com",
      "name": "Ada Lovelace",
      "first_name": "Ada",
      "last_name": "Lovelace",
      "link": "https://www.facebook.com/app_scoped_user_id/XXX/",
      "gender": "female",
      "locale": "en_US",
      "age_range": {
        "min": 21
      }
    },
    "resume": {
      "loginTokens": [
        {
          "when": "2015-12-10T22:29:46.858Z",
          "hashedToken": "XXX"
        }
      ]
    }
  },
  "profile": {
    "name": "Sashko Stubailo"
  }
}
```

注意，当用户通过不同的登录服务进行注册时，架构是不同的。当处理该集合时，需要清楚几件事情：

1. 数据库中的用户文档具有秘钥数据，如准入钥匙和hashed密码。当 [发布用户数据到客户端](#publish-custom-data)，特别注意不要包含那些客户端不应该看到的数据。
2. DDP，流星的数据发布协议，只知道如何解决顶层字段的冲突。这意味着你不能有一个发布用来发送 `services.facebook.first_name` 同时 另一个发送 `services.facebook.locale` - 两者只取其一，而且其中只有一个会在客户端中可用。修复这种情况的最好办法是冗余化数据到自定义的顶层字段，描述于[自定义用户数据](#custom-user-data).
3. OAuth登录服务包会刷新 `profile.name` 。我们不推荐使用这个字段，但是，如果你计划着么做，请确保否定客户端写入 `profile`。参见关于[用户`profile`字段](dont-use-profile)。
4. When finding users by email or username, make sure to use the case-insensitive functions provided by `accounts-password`. See the [section about case-sensitivity](#case-sensitivity) for more details.当通过电子邮件或用户名查找用户时，确保使用由 `accounts-password` 提供的大小写不敏感的方法。参见[关于大小写敏感一节](#case-sensitivity)以获取更多详细信息。

<h2 id="custom-user-data">用户数据自定义</h2>

随着你的应用的越来越复杂，你还是一如既往地需要存储个人用户的数据，而最自然的地方用来放置该数据就是 `Meteor.users` 集合中的额外字段。子啊某个多数非冗余的数据情况下，保持流星的用户数据和你的在不同表中是个好主意，但既然MongoDB不擅长应对数据关联，那么把它们放在一个集合中也说得过去。

<h3 id="top-level-fields">在用户文档中添加顶层字段</h3>

最好的方式来存储你自定义的数据，就是在 `Meteor.users` 集合中添加一个新的唯一命名的顶层字段。例如，如果你想要添加为用户添加一个邮寄的地址，你需要做的可能如下所示：

```js
// Using address schema from schema.org
// https://schema.org/PostalAddress
const newMailingAddress = {
  addressCountry: 'US',
  addressLocality: 'Seattle',
  addressRegion: 'WA',
  postalCode: '98052',
  streetAddress: "20341 Whitworth Institute 405 N. Whitworth"
};

Meteor.users.update(userId, {
  $set: {
    mailingAddress: newMailingAddress
  }
});
```

<h3 id="adding-fields-on-registration">用户注册时添加字段</h3>

上述代码，可能只是服务器端的某个流星的方法，用来设置某人的邮寄地址。有时，你需要当用户第一次创建他们的账号时设置一个字段，例如，初始化某个默认值或者计算一些他们的社交数据。你可以通过[`Accounts.onCreateUser`](http://docs.meteor.com/#/full/accounts_oncreateuser)来实现。

```js
// Generate user initials after Facebook login
Accounts.onCreateUser((options, user) => {
  if (! user.services.facebook) {
    throw new Error('Expected login with Facebook only.');
  }

  const { first_name, last_name } = user.services.facebook;
  user.initials = first_name[0].toUpperCase() + last_name[0].toUpperCase();

  // 最后不要忘记返回新用户对象！
  return user;
});
```

注意那个提供的 `user` 对象 中还不具有 `_id` 字段。如果你需要新用户的ID来做一些事情，有用的技巧就是可以自己产生一个ID：

```js
// Generate a todo list for each new user
Accounts.onCreateUser((options, user) => {
  // Generate a user ID ourselves
  user._id = Random.id(); // 需要添加 `random` 包

  // Use the user ID we generated
  Lists.createListForUser(user._id);

  // Don't forget to return the new user object at the end!
  return user;
});
```

<h3 id="dont-use-profile">不要使用profile</h3>

有个具有诱惑力的字段叫做 `profile` ，其默认会在新用户注册时被添加。该字段是历史上的原因试图被用作用户特定数据的草图 － 也许是他们的图片头像，名字，介绍文字等等。由此  **`profile` 字段 对于每个用户而言，从客户端都是可读可写的**。而且它也是对于特定用户自动被发布到客户端的。

事实证明默认具有这么个可写字段绝对不是什么好主意。很多新流星开发者在其中存入诸如`isAdmin`...然后某个恶意的用户能够轻易地将其设置为true，在任何时候都可以将他们变成管理员。即使你对此不感冒，对于让恶意用户在你的数据库中存储大量含糊的数据总不是个好主意。

与其处理该字段的特定性，倒不如完全忽略该存在。你可以安全的否定从客户端而来的所有对该字段的写入。

```js
// Deny all client-side updates to user documents
Meteor.users.deny({
  update() { return true; }
});
```

即使忽略 `profile` 的安全隐患，将应用中自定义的数据放在一个字段中也不是个好主意，就如在[集合文章中](collections.html#schema-design)讨论的，流星的数据传输协议不能够深度分辨嵌套的字段，所以在文档中，将你的对象扁平化成许多顶层字段，是个比较好的主意。

<h3 id="publish-custom-data">发布自定义数据</h3>

如果你想要存取你对于 `Meteor.users` 集合添加的自定义数据， 在你的用户界面中，你需要将其发布到客户端。多数情况下，只需要遵守在[数据载入](data-loading.html#publications) 和 [安全](security.html#publications) 文章中的建议即可。

最重要的是记住，用户文档时你的用户的隐私数据。特定情况下，用户文档会包含外部API的hashed密码数据和准入钥匙。这意味着对于发送到任意客户端的用户文档，进行[筛选字段](http://guide.meteor.com/security.html#fields)显得极其重要。

注意在流星的发布和订阅系统中，多次发布相同的文档，但具有不同的字段 － 它们会自动合并并且客户端会看到一致的、具有所有字段在一起的文档，是没有问题的。所以，如果你只是添加一个自定义字段，你应该只写一个发布用来发布该字段。让我们来看一个例子，如何发布上述的 `初始化` 字段 ：

```js
Meteor.publish('Meteor.users.initials', function ({ userIds }) {
  // Validate the arguments to be what we expect
  new SimpleSchema({
    userIds: { type: [String] }
  }).validate({ userIds });

  // Select only the users that match the array of IDs passed in
  const selector = {
    _id: { $in: userIds }
  };

  // Only return one field, `initials`
  const options = {
    fields: { initials: 1 }
  };

  return Meteor.users.find(selector, options);
});
```

This publication will let the client pass an array of user IDs it's interested in, and get the initials for all of those users.该发布从客户端传递用户ID的数组，它所关心的是获得所有这些用户的初始值。

<h2 id="roles-and-permissions">角色和权限</h2>

你想要添加登录系统到你应用的一个重要原因可能是对于数据存取需要权限。例如，如果你运营一个论坛，你可能需要管理员或者版主能够删除任意贴子，但普通用户只能删除他们自己的。这包含了两种类型的权限：

1. 基于角色的权限
2. 每－文档的权限

<h3 id="alanning-roles">alanning:roles</h3>

在流星中最流行的基于角色的权限包是 [`alanning:roles`](https://atmospherejs.com/alanning/roles)。例如，以下是你如何将某个用户添加为管理员，或某个版主：

```js
// Give Alice the 'admin' role
Roles.addUsersToRoles(aliceUserId, 'admin', Roles.GLOBAL_GROUP);

// Give Bob the 'moderator' role for a particular category
Roles.addUsersToRoles(bobsUserId, 'moderator', categoryId);
```

现在，假设你想要检测某人是否对于某个论坛帖子具有删除的权限：

```js
const forumPost = Posts.findOne(postId);

const canDelete = Roles.userIsInRole(userId,
  ['admin', 'moderator'], forumPost.categoryId);

if (! canDelete) {
  throw new Meteor.Error('unauthorized',
    'Only admins and moderators can delete posts.');
}

Posts.remove(postId);
```

注意我们可以一次性对于多个角色进行检测，且如果某人具有的角色在 `GLOBAL_GROUP` 中，他们会被认为在其它组中也具有该角色。在本例中，组是由category ID担任的，你可以使用任意的唯一标示来创建一个组。

阅读更多[`alanning:roles` 包文档](https://atmospherejs.com/alanning/roles).

<h3 id="per-document-permissions">每－文档的权限</h3>

有时，抽象权限到组并不具有意义 － 你只是需要文档具有主人，就那么多。这种情况下，你可以使用集合帮助器实施简洁的策略。

```js
Lists.helpers({
  // ...
  editableBy(userId) {
    if (!this.userId) {
      return true;
    }

    return this.userId === userId;
  },
  // ...
});
```

现在你可以调用该简单的方法来决定某个特定的用户是否能够编辑该列表：

```js
const list = Lists.findOne(listId);

if (! list.editableBy(userId)) {
  throw new Meteor.Error('unauthorized',
    'Only list owners can edit private lists.');
}
```
学习更多关于如何使用集合帮助器，请参见 [集合文章](collections.html#collection-helpers).
