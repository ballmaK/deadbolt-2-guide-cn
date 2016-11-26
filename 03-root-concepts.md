# Root concepts
基本概念

Deadbolt is centered around a single idea - constraining access to a resource to a specific group of users.  I've had several e-mails from developers who have thought that Deadbolt had a "restrict from" approach, and therefore could not understand why the authorization was failing so spectacularly; to forestall future questions about this, I want to make it completely clear - Deadbolt uses "restrict to".  For example, a controller action annotated with `@Restrict(@Group("foo"))` would only allow users with the "foo" role to access the method.
Deadbolt围绕一个基本想法－对于特定用户访问资源的限制。我收到几个来自开发者的邮件，他们都认为Deadbolt有个"restrict from"的方法，因此根本不能理解为什么认证会如此惊人的失败，在更多的问题出现之前，我觉得应该把这件事情弄明白一点－Deadbolt用"restrict to"。例如，一个被"@Restrict(@Group("foo"))"标注的控制器行为只会允许拥有"foo"角色的用户访问这个方法。

Two mechanisms are provided to declare these constraints - one at the template level and another at the controller level.  In each case, there are differences between how these are applied in Java and Scala applications, so specific details will be provided in later chapters.  The capabilities of each version are roughly similar, taking into account the idiosyncrasies of each language.
这里有两种技巧来实现这些限制－一种是模版层面的，另一种是控制器层面的。不同的情况下会有不同的Java和Scala的实现方法，以后的章节中会提到更多的细节。对于这两种语言来说，各自实现出来的效果基本近似。



## Template-level constraints
For a Play application that uses server-side rendering, Deadbolt provides several template tags that will include or remove template content at the point of rendering.
## 模版层面的限制
对于一个使用服务端渲染的Play应用，Deadbolt在渲染时提供了好几种用来包含或者移除模版内容的模版标签（tags）。

A couple of basic use cases are
几种基本的用法是
* Only displayed a "Log in" link if there is no user present
* Even if a user is logged in, only display an "Administration" link if the user has administrative privileges
* 如果用户没有登录，只显示登录的链接
* 如果用户登录了，只有当用户具有管理员权限的时候才显示管理链接

However, it is **extremely** important to note that using these tags will only give you a cleaner UI, one that is better tailored to the user's privileges.  It will **not** secure your server-side code in any way except - possibly - obscurity.  Server-side routes can be invoked from outside of templates by changing the URL in the browser's address bar, using command-line tools such as cURL and many other ways.
然后，需要特别注意的是，这些标签只能为你提供一个根据用户权限裁剪过的干净的UI，它并不能以任何方式保护你的服务端代码。服务端的routes可能会被在模板以外的任何地方调用，比如用户在浏览器中改变URL路径或者是类似使用cURL命令行工具等。

If you have seen the original Dawn Of The Dead (Romero, 1978), you may remember the protagonists concealing the entrance to their living quarters using a panel of painted hardboard.  There are no additional defensive layers behind this concealment.  When a zombified protagonist breaks through the hardboard, knowing there is something he wants on the other side, all security is gone.  Minutes later, there's blood everywhere and the survivors have to flee.  If you haven't seen it, apologies for the spoiler.
如果你看过1978年Romero的原版的活死人黎明，你可能还记得主角们用一块彩绘的硬纸板遮住他们住所的入口。这种隐藏之后没有额外的防御层。blah...blah...（叙事省略，其实没懂什么意思）

Template security is like painted hardboard - the features it offers are certainly nice to have, but a further level of defensive depth is required.  For this, you need controller action security - otherwise, the zombies will get you.
模板安全策略就像彩绘硬纸板－所提供的特性固然好，当时更深层次的防御还是需要的。所以，你需要控制器行为的安全策略，不然，僵尸们就会袭击你。

## Controller-level restrictions
The controller layer is most vulnerable part of your application to external attack, because that is the part that is visible to whichever networks it is on.  Attack in this sense may be a conscious attack on your system, or inadvertant damage caused by unauthorized users who are otherwise authenticated in your system.  Deadbolt can help with both of these scenarios in the same way, by limiting the capabilities of any given user at the application level.


Controller authorization blocks or allows access to an action.  Whereas template restrictions are essentially a boolean evaluation - "if user satisfies these conditions, then...", controller authorization is quite a bit more powerful.  Specifically, while an authorized result is generated from your application code, unauthorized results can be customised as required; you can return any status code you like along with any content you like.  If you're feeling particularly nasty, why not send a 302 redirect to a not-suitable-for-work website?  If you want to, the option is there.

## Core entities
Deadbolt has three interfaces which can be used to represent authorization entities in your application - `Subject`, `Role` and `Permission`.


### Subject
A subject represents an authorizable entity - in other words, a user or account.  A subject embodies four pieces of information - the fact a user is (or isn't) authenticated, that user's identity, and that user's roles and permissions.

Depending on the application you're building, you may always have a subject present - call it Guest, for example.  This subject may have a very restricted set of roles and/or permissions.  Alternatively, you may require explicit authentication for some or all of the application, in which case it's possible no subject is present. 

Fun fact - `Subject` was originally known as `RoleHolder`, but this swiftly became an anacronysm as Deadbolt gained capabilities beyond checking roles.  As of Deadbolt 2.0, `RoleHolder` became ´Subject´.


### Role
A `Role` is essentially a wrapper around a string.  It is the primary entity for the `Restrict` and `Restrictions` constraints.  Role A is equal to Role B, even if they are different objects, if they have **exactly** the same name.  Role names should be case-aware, so "Admin" is not the same as "admin".

As `Role` is an interface/trait, it can be implemented as a class or an enum.

If you do not require roles in your application, you do not need to implement this interface - just return an empty list from `Subject#getRoles`.

### Permission
A `Permission`, just like a `Role`, is essentially a wrapper around a string.  It is the primary entity for `Pattern` constraints, and has different interpretations depending on the type of pattern constraint that is being applied to it.  For example, a `PatternType.EQUALITY` test would perform a case-sensitive comparison between a user's permissions and the test value.  A `PatternType.REGEX` would assess a user's permissions in the context of regular expressions, and so on.

As `Permission` is an interface/trait, it can be implemented as a class or an enum.

If you do not require permissions in your application, you do not need to implement this interface - just return an empty list from `Subject#getPermissions`.


## Hooks
There are two hooks that can be used to integrate Deadbolt into your application - `DeadboltHandler` and `DynamicResourceHandler`.  In the Java version, these are represented by interfaces; in the Scala version, they are traits.  There are some small differences between them caused by design differences with the Java and Scala APIs themselves, so exact breakdowns of these types will be covered in the language-specific sections.  For now, it's enough to remind ourselves of where we are working in terms of a HTTP request.

A HTTP request has a life cycle.  At a high level, it is

* Sent
* Received
* Processed
* Answered

The *processed* point is where our web applications live.  In a sense, this high-level life cycle is repeated here, as the request is sent from the container into the application, received by the app, processed and answered.  Controller constraints occur at the point where the container (the Play server, in this case) hands the request over to the application;  templates work during the processing phase as a response body is rendered.  Both places are where any `DeadboltHandler` and `DynamicResourceHandler` instances are active.

## Static and dynamic constraints
Deadbolt supports two categories of constraint - static and dynamic.  This is a little like saying two types of animal exist - cats and not-cats - so read on.  This bit is important.

A static constraint is one that requires no further effort on the part of the developer, because the necessary information can be determined from the `DeadboltHandler` and the `Subject`.  For example, the "is a subject present?" constraint can be answered by requesting the subject from the handler; similarly, group membership can be determined by requesting a subject's roles and testing them for the required values.  Simply put, a static constraint is one in which the developer specifies *what* they want and Deadbolt handles the *how* based on existing information.

Dynamic constraints, on the other hand, require work by the developer.  They embody arbitrary logic, and have total freedom as long as they eventually return a result.  For example, you may have a business rule that a user on subscription plan x can only make y calls to an API within z amount of time - this could be implemented as a dynamic constraint.Dynamic constraints are exposed via the `DynamicResourceHandler` (in retrospect, I probably should have called it `DynamicConstraintHandler`...) and will be discussed in detail in the language-specific chapters. 

One small but pertinent fact regarding dynamic constraints is they don't *necessarily* require subjects; if you write a constraint that does not need a subject, the presence or absence of a subject is irrelevent.  If your boss, in a bizarre act of caprice, decides that no-one can have access to the accounting system on a Tuesday, the constraint would essentially be `if (day == tuesday)...`; the subject is not needed and so not used.