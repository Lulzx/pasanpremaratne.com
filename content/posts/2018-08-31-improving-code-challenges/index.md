---
path: "/2018/08/31/improving-treehouse-code-challenges"
date: "2018-08-31"
title: "Improving Swift Code Challenges at Treehouse"
description: ""
---

At Treehouse we test student's code in the browser using a custom solution we call (as does everyone else) code challenges. The implementation details vary slightly depending on the language being tested but from the start testing Swift code has been somewhat painful.

All code challenges except for the iOS stack ran on Linux but Objective-C  and until recently Swift, were built using a macOS toolchain because, well, there wasn't any Linux support. From the inability to access the iOS SDK outside of a simulator, to the limitation of being able to only run a single instance of a simulator at once, there were a lot of hurdles to providing a good coding environment for students. But that's not what I want to focus on because some of those problems have been solved to some degree.

My current issues are less about tooling and more about effectively validating student code. To provide some context let's take a look at a really simple challenge. In an early Swift course, I ask a student to create a stored property named `title` on a struct named `Book` with an initial vaule of `"Animal Farm"`.

To check if the student did as asked, we use a custom tool along with the `XCTest` framework to validate the code. To test a task like this I can write a simple assert 

```swift
XCTAssertTrue(Book().title == "Animal Farm", "Failure message that's piped to students")
```

This seems fairly straightforward but there's one big problem. What if the student makes a simple mistake and declares a variable named `titl`. When `swift test` is run, the compiler is going to raise an error saying that there isn't a property named `title` to begin with. None of my test logic is even run on the student's code and worse, the student is going to see compiler errors that are raised in the test suite. Beginners to programming have enough to worry about without needing to see implementation details of our testing engine. One approach is to intercept these compiler errors and provide some feedback to the student directing them to the issue but in reality we want students to get familiar with the compiler. It'll be their best frenemy one day and we don't want hide it.

The workaround we've employed so far is to validate some of the student's code in a precompile step by parsing the AST generated by the student's code. This gets the job done but it is super painful. In earlier versions of the tool this had to be done in Ruby. Here's an example of an AST check for a different task:

```ruby
unless ast =~ /\(red: Double, green: Double, blue: Double, alpha: Double\) -> RGBColor/
  fail("Don't use the memberwise initializer Swift creates. Add your own custom initializer as specified in the directions")
end
```

Entire challenges were built on stringly typed code and extremely brittle regexes and as challenges got more complex it got out of hand. There's also the mental overhead of having to write tests for Swift code in a different language which introduces lots of room for mistakes. There was little that could be done about this however, largely due to the fact that we were really resource constrained and front end topics are far more important for Treehouse's business.

Over the past year though we've taken the steps to really improve our code challenge tooling and authoring experience. For Swift this meant moving to Linux based implementation, using the Swift package manager to build tiny executables and using the XCTest framework as mentioned earlier.

While the authoring experience is much better I still need to validate student code in much the same way. Which brings me to this post.

I've been wanting to learn about compilers and interpreters for a while now and even though I've had some [great](https://www.craftinginterpreters.com) [resources](https://en.wikipedia.org/wiki/Compilers:_Principles,_Techniques,_and_Tools) available I just haven't made the time. This problem provides a good reason however and recently I started working my way through Crafting Intepreters, written by [Robert Nystrom](https://twitter.com/munificentbob). As an aside I must say, as someone who writes educational content about programming and computer science for a living, he's done an excellent job. If you want to dive in and build an interpreter, this book is the place to start.

Working through the book you write an interepreter, in both C and Java, for Nystrom's language `Lox`. To bring this back to the original point, I don't need to write a full Swift compiler to solve my problem but a custom AST parser would solve the job nicely and AST parsing is a subset of what the book teaches.

My ideal scenario looks something like this. Prior to running `swift test` I'd like to generate and parse the AST generated by the student's code. Unlike most AST parser projects out there, my goals are to assert the existence of certain identifiers, validate method signatures and check types. I'd like to do something like this

```swift
let ast = SwiftASTInspector(source: file)
XCTAssertTrue(ast.contains(.keyword, named: .var), "Make sure you declare a variable and not a constant")
XCTAssertTrue(ast.contains(.keyword, named: .var).withIdentifier(named: "title"), "Make sure you declare a variable named title")
```

I should add that I haven't thought beyond the intial idea here so this interface isn't decided by any means. Some high level goals:

- The tool needs to be in Swift so that any Swift developer feels comfortable using it
- Design a fluent interface. I really like how [Quick](https://github.com/Quick/Quick) and [Chai](http://www.chaijs.com) read. 
- Cover simple cases first. No need to optimize for speed or generate the entire tree. Focus on testing simple challenges and work my way up.

In the next post, let's get started by writing a scanner
