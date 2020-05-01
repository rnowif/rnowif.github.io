---
layout: post
title: "The Blind Golden Master"
date: 2020-05-01 17:46:38 -0400
comments: true
categories: [refactoring, test]
---

As part of my work, I regularly have to refactor/rewrite some part of our system that has little test coverage, if any. Most of the time, the code is too big or complex to write a test suite comprehensive enough to give me the confidence I need in a timely manner. Consequently, I have come to use a characterisation test technique that I called the “Blind Golden Master”.

<!-- more -->

## Characterisation Tests

If you know what a characterisation test is, you can skip this section. If not, bear with me for a moment. A characterisation test aims to check that an already existing piece of code always keeps the same behaviour throughout the changes you make. This behaviour does not need to be correct - if there is a bug in the code, these tests will check that it is still here at the end! Characterisation tests are then extremely useful during refactoring efforts since it is not necessary to understand (or even read) the code to write them.

One of the most usual characterisation test techniques is the Golden Master. The idea is to exercise the code and record its output somehow (e.g. in a file). Then, the test will exercise the code with the same input and check the output against the previously recorded one - the golden master. See [here](https://rnowif.github.io/blog/2017/04/10/trivia-refactoring/) for an example of this technique.

## The Blind Golden Master

Sometimes, it is impractical to store the output of the code - or we simply couldn’t be bothered to do it. Sometimes, the new implementation and the legacy one have to co-exist (and evolve) for a while and the only goal is to check that they always both behave the same.

With this technique, the test simply calls both the new and the legacy code and assert that their output is identical. This approach is extremely effective when the method has a limited set of parameters and no side effect.

For instance, I used it to change the template engine for our codebase (C#).

```csharp
string newResult = _newEngine.Render(templateName, model);
string legacyResult = _legacyEngine.Render(templateName, model);
Minify(newResult).Should().BeEquivalentTo(Minify(legacyResult));
```

I also used it to change an algorithm that determine the file extension of a document (see the Java codebase [here](https://github.com/rnowif/refactoring-with-tests/tree/refacto/socrates)).

```java
FileExtension legacyValue = application.getExtension(country, region);
FileExtension newValue = application.getExtensionV2(country, region);
assertEquals(legacyValue, newValue);
```

If the algorithm is crucial to the application and there are a lot of potential combinations to test, it is even possible to run the test in production: every time the code is called, it can invoke both the algorithms, compare their output and raise an error if they don’t match, and always return the legacy result. When there is no more error, it is time to switch to the new implementation. This [article](https://github.blog/2015-12-15-move-fast/) explains how GitHub rewrote its merge algorithm using this technique.

## Conclusion

The Blind Golden Master is an extremely simple technique that provides confidence during refactoring without having to spend hours writing tests for the legacy code. It is a powerful enabler for improvement that does not require any specific tooling or overhead. Since I started using it a few years ago, it has proven a precious addition to my testing toolbox.
