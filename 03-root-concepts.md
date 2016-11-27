# Root concepts
基本概念

Deadbolt is centered around a single idea - constraining access to a resource to a specific group of users.  I've had several e-mails from developers who have thought that Deadbolt had a "restrict from" approach, and therefore could not understand why the authorization was failing so spectacularly; to forestall future questions about this, I want to make it completely clear - Deadbolt uses "restrict to".  For example, a controller action annotated with `@Restrict(@Group("foo"))` would only allow users with the "foo" role to access the method.
Deadbolt围绕一个基本想法－对于特定用户访问资源的限制。我收到几个来自开发者的邮件，他们都认为Deadbolt有个"restrict from"的方法，因此根本不能理解为什么认证会如此惊人的失败，在更多的问题出现之前，我觉得应该把这件事情弄明白一点－Deadbolt用"restrict to"。例如，一个被"@Restrict(@Group("foo"))"标注的控制器行为只会允许拥有"foo"角色的用户访问这个方法。

Two mechanisms are provided to declare these constraints - one at the template level and another at the controller level.  In each case, there are differences between how these are applied in Java and Scala applications, so specific details will be provided in later chapters.  The capabilities of each version are roughly similar, taking into account the idiosyncrasies of each language.
这里有两种技巧来实现这些约束－一种是模版层面的，另一种是控制器层面的。不同的情况下会有不同的Java和Scala的实现方法，以后的章节中会提到更多的细节。对于这两种语言来说，各自实现出来的效果基本近似。



## Template-level constraints
For a Play application that uses server-side rendering, Deadbolt provides several template tags that will include or remove template content at the point of rendering.
## 模版层面的约束
对于一个使用服务端渲染的Play应用，Deadbolt在渲染时提供了好几种用来包含或者移除模版内容的模版标签（tags）.

A couple of basic use cases are
几种基本的用法是
* Only displayed a "Log in" link if there is no user present
* Even if a user is logged in, only display an "Administration" link if the user has administrative privileges
* 如果用户没有登录，只显示登录的链接
* 如果用户登录了，只有当用户具有管理员权限的时候才显示管理链接

However, it is **extremely** important to note that using these tags will only give you a cleaner UI, one that is better tailored to the user's privileges.  It will **not** secure your server-side code in any way except - possibly - obscurity.  Server-side routes can be invoked from outside of templates by changing the URL in the browser's address bar, using command-line tools such as cURL and many other ways.
然后，需要**特别**注意的是，这些标签只能为你提供一个根据用户权限裁剪过的干净的UI，它并不能以任何方式保护你的服务端代码。服务端的routes可能会被在模板以外的任何地方调用，比如用户在浏览器中改变URL路径或者是类似使用cURL命令行工具等.

If you have seen the original Dawn Of The Dead (Romero, 1978), you may remember the protagonists concealing the entrance to their living quarters using a panel of painted hardboard.  There are no additional defensive layers behind this concealment.  When a zombified protagonist breaks through the hardboard, knowing there is something he wants on the other side, all security is gone.  Minutes later, there's blood everywhere and the survivors have to flee.  If you haven't seen it, apologies for the spoiler.
如果你看过1978年Romero的原版的活死人黎明，你可能还记得主角们用一块彩绘的硬纸板遮住他们住所的入口。这种隐藏之后没有额外的防御层。blah...blah...（叙事省略，其实没懂什么意思）.

Template security is like painted hardboard - the features it offers are certainly nice to have, but a further level of defensive depth is required.  For this, you need controller action security - otherwise, the zombies will get you.
模板安全策略就像彩绘硬纸板－所提供的特性固然好，当时更深层次的防御还是需要的。所以，你需要控制器行为的安全策略，不然，僵尸们就会袭击你.

## Controller-level restrictions
The controller layer is most vulnerable part of your application to external attack, because that is the part that is visible to whichever networks it is on.  Attack in this sense may be a conscious attack on your system, or inadvertant damage caused by unauthorized users who are otherwise authenticated in your system.  Deadbolt can help with both of these scenarios in the same way, by limiting the capabilities of any given user at the application level.
## 控制器层面的限制
控制器层是对于应用的外在攻击来说最脆弱的一个环节，因为无论在哪种网络状态下他都是可见的部分。这种场景下的攻击可能是有意识的攻击，也可能是未认证用户以其他认证方式所造成的损坏。Deadbolt可以通过限制用户的权限在应用层面对这些情况提供帮助.

Controller authorization blocks or allows access to an action.  Whereas template restrictions are essentially a boolean evaluation - "if user satisfies these conditions, then...", controller authorization is quite a bit more powerful.  Specifically, while an authorized result is generated from your application code, unauthorized results can be customised as required; you can return any status code you like along with any content you like.  If you're feeling particularly nasty, why not send a 302 redirect to a not-suitable-for-work website?  If you want to, the option is there.
控制器认证屏蔽或者允许一个对于行为的访问，然后模板限制本质上只是一种布尔评估－“如果用户满足这些情况，就...”，控制器认证相比之下就更强力了。特别要说明的是，当一个通过认证的结果被您的应用代码生成过后，未认证结果也可以按需被定制；你可以回返任何你喜欢的状态码。如果你感觉特别恶心，为什么不未一个不适合工作的网站返回一个302重定向的状态码呢？只要你愿意，你就可以做这样的设置.

## Core entities
Deadbolt has three interfaces which can be used to represent authorization entities in your application - `Subject`, `Role` and `Permission`.
## 核心实体
Deadbolt有三种相当于认证实体的接口－`Subject`, `Role`, `Permission`.

### Subject
A subject represents an authorizable entity - in other words, a user or account.  A subject embodies four pieces of information - the fact a user is (or isn't) authenticated, that user's identity, and that user's roles and permissions.
### Subject
Subject相当于一种可认证的实体－换句话说，用户或者是账户。Subject实体包括了4种信息－用户是否认证，用户的身份信息，用户的角色和用户的权限.

Depending on the application you're building, you may always have a subject present - call it Guest, for example.  This subject may have a very restricted set of roles and/or permissions.  Alternatively, you may require explicit authentication for some or all of the application, in which case it's possible no subject is present. 
对于你正在构建的应用来说，你可能会常常有一种subject－比如说游客（Guest）。这种Subject可能只具有很少一部分角色或者权限。或者，您可能需要对某些或所有应用程序进行显式身份验证，在这种情况下可能没有主题存在.

Fun fact - `Subject` was originally known as `RoleHolder`, but this swiftly became an anacronysm as Deadbolt gained capabilities beyond checking roles.  As of Deadbolt 2.0, `RoleHolder` became ´Subject´.
有趣的是－`Subject`通常是被叫做`RoleHolder`，但随着Deadbolt获得了超越检查角色的能力，这一切迅速成为了一个难题。所以在Deadbolt2.0中，`RoleHolder`成为了`Subject`.

### Role
A `Role` is essentially a wrapper around a string.  It is the primary entity for the `Restrict` and `Restrictions` constraints.  Role A is equal to Role B, even if they are different objects, if they have **exactly** the same name.  Role names should be case-aware, so "Admin" is not the same as "admin".
“Role”本质上是一个字符串的包装。它是`Restrict`he`Restrictions`两种约束的主要实体。如果角色A和角色B有**确实**相同的名字，虽然她们是不同的对象，它们也是相同的。角色名称是大小写敏感的.

As `Role` is an interface/trait, it can be implemented as a class or an enum.
`Role`可以是接口，也可以是特征，可以被实现为一个类或者一个枚举.

If you do not require roles in your application, you do not need to implement this interface - just return an empty list from `Subject#getRoles`.
如果在你的应用中不需要角色，你完全可以不用去实现这个接口－只需要让`Subject#getRoles`返回一个空的列表.

### Permission
A `Permission`, just like a `Role`, is essentially a wrapper around a string.  It is the primary entity for `Pattern` constraints, and has different interpretations depending on the type of pattern constraint that is being applied to it.  For example, a `PatternType.EQUALITY` test would perform a case-sensitive comparison between a user's permissions and the test value.  A `PatternType.REGEX` would assess a user's permissions in the context of regular expressions, and so on.
### Permission
`Permission`就像`Role`，也是个字符串的封装。它是`Pattern`限制的主要实体，根据应用于它的不同的正则式，可以有不同的理解。例如，`PatternType.EQUALITY`测试可以用做用户权限和值间区分大小写的比较。`PatternType.REGEX`将用正则表达式来评估用户处于上下文中的权限。

As `Permission` is an interface/trait, it can be implemented as a class or an enum.
`Permission`可以是接口，也可以是特征，可以被实现为一个类或者一个枚举.

If you do not require permissions in your application, you do not need to implement this interface - just return an empty list from `Subject#getPermissions`.
如果在你的应用中不需要权限，你完全可以不用去实现这个接口－只需要让`Subject#getPermissions`返回一个空的列表.

## Hooks
There are two hooks that can be used to integrate Deadbolt into your application - `DeadboltHandler` and `DynamicResourceHandler`.  In the Java version, these are represented by interfaces; in the Scala version, they are traits.  There are some small differences between them caused by design differences with the Java and Scala APIs themselves, so exact breakdowns of these types will be covered in the language-specific sections.  For now, it's enough to remind ourselves of where we are working in terms of a HTTP request.
## Hooks
有两种钩子可以将Deadbolt引入你的应用－`DeadboltHandler`和`DynamicResourceHandler`。在Java版本中，它们相当于接口，在Scala版本中，它们是特征。由于Java和Scala自身API设计的原因，它们在此会有一些小差别，因此对于这些的详细分析会用不同的语言段落来解释。现在有必要提醒我们自己就HTTP请求而言处于什么阶段。

A HTTP request has a life cycle.  At a high level, it is
一个HTTP请求具有生命周期。从高级一点的层面来讲：

* Sent
* Received
* Processed
* Answered
* 发送
* 接受
* 处理
* 响应

The *processed* point is where our web applications live.  In a sense, this high-level life cycle is repeated here, as the request is sent from the container into the application, received by the app, processed and answered.  Controller constraints occur at the point where the container (the Play server, in this case) hands the request over to the application;  templates work during the processing phase as a response body is rendered.  Both places are where any `DeadboltHandler` and `DynamicResourceHandler` instances are active.
*处理*点是我们的Web应用程序存在的地方。在某种意义上，随着请求从容器发送到应用程序，由应用程序接收，处理和回答，生命周期在这里重复。控制器约束出现在容器（这目前状况下是Play 服务器）处理应用发送过来的请求的时候；模板约束为了呈现响应主体在处理阶段期间工作。`DeadboltHandler`和`DynamicResourceHandler`在这些情况下都处于激活状态。


## Static and dynamic constraints
Deadbolt supports two categories of constraint - static and dynamic.  This is a little like saying two types of animal exist - cats and not-cats - so read on.  This bit is important.
## 静态和动态约束
Deadbolt支持两种约束－静态约束和动态约束。这有点像说有两种动物－一种像猫，一种不像猫－让我们继续，这还是有必要说的。

A static constraint is one that requires no further effort on the part of the developer, because the necessary information can be determined from the `DeadboltHandler` and the `Subject`.  For example, the "is a subject present?" constraint can be answered by requesting the subject from the handler; similarly, group membership can be determined by requesting a subject's roles and testing them for the required values.  Simply put, a static constraint is one in which the developer specifies *what* they want and Deadbolt handles the *how* based on existing information.
静态约束对于开发者来说可能更轻松，因为必要的信息仅仅被`DeadboltHandler`和`Subject`所决定。用户是否在线可以通过请求该用户的处理者得到答案；同样的，组关系也可通通过请求用户的角色和测试他们所需要的值决定。开发者可以根据自己所想的，通过简单的置入一个静态的约束，让Deadbolt去处理。

Dynamic constraints, on the other hand, require work by the developer.  They embody arbitrary logic, and have total freedom as long as they eventually return a result.  For example, you may have a business rule that a user on subscription plan x can only make y calls to an API within z amount of time - this could be implemented as a dynamic constraint.Dynamic constraints are exposed via the `DynamicResourceHandler` (in retrospect, I probably should have called it `DynamicConstraintHandler`...) and will be discussed in detail in the language-specific chapters. 
另一方面，动态约束需要开发者做一些工作。

One small but pertinent fact regarding dynamic constraints is they don't *necessarily* require subjects; if you write a constraint that does not need a subject, the presence or absence of a subject is irrelevent.  If your boss, in a bizarre act of caprice, decides that no-one can have access to the accounting system on a Tuesday, the constraint would essentially be `if (day == tuesday)...`; the subject is not needed and so not used.