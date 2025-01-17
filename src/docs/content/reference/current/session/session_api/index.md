---
title: "Session API"
description: "Concepts of the Gatling Session API"
lead: ""
date: 2021-04-20T18:30:56+02:00
lastmod: 2021-04-20T18:30:56+02:00
weight: 4010
---

## Concept

### Going Stateful

In most load testing use cases, it's important that the virtual users don't play the same data.
Otherwise, you might end up not testing your application but your caches.

Moreover, if you're running an application on a Java Virtual Machine, the Just In Time compiler (JIT) will make dramatic optimizations, and your system will behave very differently from your actual one.

Though, **you have to make your scenario steps dynamic, based on virtual user specific data**.

### Session

Session is a virtual user's state.

Basically, it's a `Map[String, Any]`: a map with key Strings.
In Gatling, entries in this map are called **Session attributes**.

{{< alert tip >}}
A Gatling scenario is a workflow where every step is an `Action`.
`Session`s are the messages that are passed along a scenario workflow.
{{< /alert >}}

### Injecting Data

The first step is to inject state into the virtual users.

There's 3 ways of doing that:

* using [Feeders]({{< ref "../feeder" >}})
* extracting data from responses and saving them, e.g. with [HTTP Check's saveAs]({{< ref "../../http/check#saving" >}})
* manually with the Session API

### Fetching Data

Once you have injected data into your virtual users, you'll naturally want to retrieve and use it.

There are 2 ways of implementing this:

* using Gatling's [Expression Language]({{< ref "../expression_el" >}})
* manually with the Session API

{{< alert tip >}}
If Gatling complains that an attribute could not be found, check that:

* you don't have a typo in a feeder file header
* you don't have a typo in a Gatling EL expression
* your feed action is properly called (e.g. could be properly chained with other action because a dot is missing)
* the check that should have saved it actually failed
{{< /alert >}}

## Session API

### Setting Attributes

Session has the following methods:

* `set(key: String, value: Any): Session`: add or replace an attribute
* `setAll(newAttributes: (String, Any)*): Session`: bulk add or replace attributes
* `setAll(newAttributes: Iterable[(String, Any)]): Session`: same as above but takes an Iterable instead of a varags
* `reset`: reset all attributes but loop counters, timestamps and Gatling internals (baseUrl, caches, etc)

{{< alert warning >}}
`Session` instances are immutable!

Why is that so? Because Sessions are messages that are dealt with in a multi-threaded concurrent way,
so immutability is the best way to deal with state without relying on synchronization and blocking.

A very common pitfall is to forget that `set` and `setAll` actually return new instances.
{{< /alert >}}

{{< include-code "SessionSample.scala#sessions-are-immutable" scala >}}

### Getting Attributes

Let's say a Session instance variable named session contains a String attribute named "foo".

{{< include-code "SessionSample.scala#session" scala >}}

Then:

{{< include-code "SessionSample.scala#session-attribute" scala >}}

{{< alert warning >}}
`session("foo")` doesn't return the value, but a wrapper.
{{< /alert >}}

You can then access methods to retrieve the actual value in several ways:

`session("foo").as[Int]`:

* returns a `Int`,
* throws a `NoSuchElementException` if the *foo* attribute is undefined,
* throws a `NumberFormatException` if the value is a String and can't be parsed into an Int,
* throws a `ClassCastException` otherwise.

`session("foo").asOption[Int]`:

* returns an `Option[Int]`,
* which is `None` if the *foo* attribute is undefined,
* which is `Some(value)` otherwise and *value* is an Int, or is a String that can be parsed into an Int,
* throws a `NumberFormatException` if the value is a String and can't be parsed into an Int,
* throws a `ClassCastException` otherwise.

`session("foo").validate[Int]`:

* returns a `Validation[Int]`,
* which is `Success(value)` if the *foo* attribute is defined and *value* is an Int or is a String that can be parsed into an Int,
* which is `Failure(errorMessage)` otherwise.

{{< alert tip >}}
Trying to get a `[String]` actually performs a `toString` conversion and thus, always works as long as the entry is defined.
{{< /alert >}}

{{< alert tip >}}
If the value is a `[String]`, Gatling will try to parse it into a value of the expected type.
{{< /alert >}}

{{< alert tip >}}
Using `as` will probably be easier for most users.
It will work fine, but the downside is that they might generate lots of expensive exceptions once things starts going wrong under load.

We advise considering `validate` once accustomed to functional logic as it deals with unexpected results in a more efficient manner.
{{< /alert >}}
