# Introduction
At the date of writing this, the small code experiment that turned into Deadbolt is nearly five years old.  In that time, it has grown in capabilities, power and - hopefully - usefulness.  It has certainly reached the point where a few badly-written lines of text can adequately cover its features and use.  Once I factored in that it now has support for Java and Scala, and that its new architecture allows for easy extension with other languages, it became clear that a short booklet (or booklet-like document) was required.


I hope this document (booklet-like or otherwise) will prove useful as you integrate Deadbolt 2 into your application.


## History
Back in September 2010, I was embarking on my first project using the Play! Framework (for fans of unnecessary detail, it was version 1.03.2, back when it still had an exclamation mark) and discovering the Secure module it shipped with was unsuitable for the required authorization.  As a result, Deadbolt 1.0 was written to provide AND/OR/NOT support for roles.  Sometime later, dynamic rule support was added and other new features would be released as use cases and bug reports cropped up.


The user guide for Deadbolt 1 - which I can still highly recommend if you need authorization support in your Play 1 apps - starts with this:


> Deadbolt is an authorization mechanism for defining access rights to certain controller methods or parts of a view using a simple AND/OR/NOT syntax. It is based on the 
> original Secure module that comes with the Play! framework.
> 
> Note that Deadbolt doesn’t provide authentication! You can still use the existing Secure module alongside Deadbolt to provide authentication, and in cases where 
> authentication is handled outside your app you can just hook up the authorization mechanism to whatever auth system is used.


How much of this still holds true for Deadbolt 2?  More than 50% and less than 100%, give or take. 


* Deadbolt is still used for authorization
* It can control access to controllers
* It can control access to templates
* The capabilities have expanded beyond the original role-based static checks
* Deadbolt 2 is based on Deadbolt 1, so it's related to the old Secure module in spirit, if not in implementation
* You can (or should be able to) combine Deadbolt 2 with any authentication system


Deadbolt 2 v1.0 was released at roughly the same time as Play 2.0, and was essentially the logic of Deadbolt 1 exposed in the Play 2 style.  Nine months after that initial release - nine months, I should add, of woefully inadequate Scala support - I re-designed the architecture to a more modular approach, and made a few small changes to the API to remove anachronistic elements.


There was now a core module written in Java, and separate idiomatic modules for Java and Scala.  This is slightly different to the architecture of Play 2 itself, where the core and the Scala API are co-located.  By the time v2.5 was released, I had deprecated the core - I felt it forced Java classes into the Scala API, leading to the need for Scala-to-Java type conversions.  The Java and Scala versions of Deadbolt are now united in philosophy and capability, but no longer by any common code.


## Java versus Scala
I have tried my best, within the constraints of both languages and my knowledge of them, to give equal capabilities to each version of Deadbolt 2.  Scala is generally detailed after Java in this book for two reasons.


The first reason is the alphabet.


The second is that by writing the Scala section last, I have a chance to increase my knowledge of a language that is frequently beautiful and occasionally a little bit like an ice-cream headache.

## Versions

The examples in this book are for Play 2.5, and use the following versions of Deadbolt

- `"be.objectify" %% "deadbolt-java" % "2.5.0"`
- `"be.objectify" %% "deadbolt-scala" % "2.5.1"`


## Target audience

This book works on the following assumptions: 

- you have a working knowledge of Play
- you know Java or Scala (or both)

If you want to learn Play, the [Play website](https://playframework.com/documentation/2.5.x/Home) is the best place to start.

There are also several books that you might want to check out.

- [Play For Scala](https://www.manning.com/books/play-for-scala)
- [Play For Java](https://www.manning.com/books/play-for-java)
- [Learning Play Framework 2](https://www.packtpub.com/web-development/learning-play-framework-2)
- [Reactive Web Applications](https://www.manning.com/books/reactive-web-applications) covers more advanced usage of Play

## Acknowledgements

Nothing which attempts to be useful can gestate in a vacuum.  Many people donated their time, knowledge and critical capabilities to help make this book more useful.

- Peter Hilton ([@PeterHilton](https://twitter.com/PeterHilton)) - software developer, speaker, writer, author of "Play for Scala" and beer lover.
- Francis De Brabandere ([@somatik](https://twitter.com/somatik)) - software developer, technology enthusiast, start-up founder and beer lover.
- Daryl Greensill ([@BigClumsyOaf](https://twitter.com/BigClumsyOaf)) - todo: check one-line write-up

## Feedback

Any and all feedback of this is greatly appreciated.  You can send feedback using...

- [The book's repository](https://github.com/schaloner/deadbolt-2-guide) on GithHub
- By emailing [deadbolt-book@objectify.be](mailto:deadbolt-book@objectify.be)
