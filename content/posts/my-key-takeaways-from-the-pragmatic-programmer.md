---
title: My key takeaways from The Pragmatic Programmer
date: 2022-03-20
summary: In this blog post, I list things that resonate with me most after reading this book.
showToc: true
TocSide: 'left'
TocOpen: true
cover:
    image: "craft.jpg"
---

## Introduction

In this post, I'd like to write a quick summary of key ideas presented in the book *The Pragmatic Programmer* written by David Thomas and Andrew Hunt. It will serve me (and you, hopefully) as a reference I can look up later to remind myself of these timeless concepts.

Obviously, it's not possible to cover everything here. I highly recommend reading this book to get a full picture. Here, I focus on things that are in my opinion most important in regards to our everyday work and advancing our careers as programmers.

This book is not strictly about code. It's not examining a specific language, tool, framework, or pattern. It's about a philosophy of being pragmatic in our craft. It's aimed at people who want to become more effective and productive programmers. 

The word *pragmatic* comes from the Latin *pragmaticus*, which means “skilled in business”. Let's see what the authors think it means to be a pragmatic programmer.

## Takeaways

The ideas listed below are presented in the same order as they appear in the book. The order doesn't imply importance. This means that the last idea is not the least important or useful.

### Craftsmanship

Programming is a craft and it's a difficult job. No doubt about it. Every day, we try to transform vague user requirements into a language that computers can understand.

The authors compare programmers to craftsmen who employ a set of good-quality tools and can choose the best one for the job at hand. We ought to do the same thing. It's not enough to learn one language and stick with it throughout our entire career. We should aspire to be polyglots that can adapt to the current situation by utilizing the best possible tools to solve the problem. 

> If all you have is a hammer, everything looks like a nail.

In an attempt to become pragmatic, we must also ask questions and be curious. *How does this library work?* *Why did you decide to do it this way?* *Is it the best way to solve this?* This will deepen our understanding, broaden our perspective and lead to a feeling of mastery and continuous improvement. 

### Be responsible and don't complain

Pragmatic programmers aren't afraid to admit ignorance or error. We are humans. A lot of things can go wrong even if we do our best with testing, documentation, or automation. The question is not *if* it happens, but *when*. It's unavoidable.

The best we can do then is to acknowledge the problem, take responsibility for it, and try to offer options to fix it. Unfortunately, what we usually do instead is complain. We blame someone or something else - coworkers, programming languages, tools, etc.

Imagine that it wasn't our code that introduced the error, but someone else's. **It doesn't matter**. We all work towards the same goal. If we help our teammates, they will do the same thing for us later. This builds a healthy relationship in the team. Others will also remember that you can be trusted.

> Suggest options. Don't make excuses.

Before I approach anyone and tell them why something can't be done or that it's not my fault, I stop and listen to myself. How does this excuse sound? What will they think of me?

I try to position myself in their shoes. How do I react when someone (such as an auto mechanic) comes to me with a lame excuse? What do I think of them or their company after that?

### Don't accept any broken window

The authors describe a phenomenon of cities where some buildings are in perfectly well shape while others have gone really bad. Researchers discovered that there exists a trigger mechanism that can quickly turn a beautiful and clean building into an abandoned derelict.

What is that trigger? **A broken window**.

Here is what they found:

>  One broken window, left unrepaired for any substantial length of time, instills in the inhabitants of the building a sense of abandonment—a sense that the powers don’t care about the building. So another window gets broken. People start littering. Graffiti appears. Serious structural damage begins. [...]

In my opinion, this analogy suits software quite well. I'm sure we've all seen codebases that were doing just fine only to find later that they are slowly descending to a ruin.

A broken window could be any shortcut that we've taken to make things easier for us at the moment. A broken design or architecture, wrong decision, poor or dirty code.

If we allow a single broken window in our codebase, we can be sure that there will be more to come.

### Your growth is your responsibility

We all know we should stay on top of technologies and the latest trends and constantly keep learning. I think no one has to be convinced about this anymore.

What I like about this book is that it gives us some very specific guidelines for building our knowledge portfolio that we can take inspiration from: 

* **Learn at least one new language every year**. You'll learn several different approaches to solving various problems. It'll broaden your thinking.
* **Read a technical book each month**. A book will give you the depth that you won't find in podcasts, blog posts, or online videos.
* **Take classes or workshops**. Look for occasions to expand your knowledge.
* **Participate in local user groups or meetups**. Don't just go and listen. Participate actively.
* **Experiment with different environments**. For example, if you've only worked with IDE, consider spending some time with a text editor, and vice versa.
* **Stay current**. Follow interesting people in your field, read blogs, make side-projects.

### Easier To Change

Programmers often like to argue and discuss which framework, language, or design pattern is the best. 

When we learn to program, we are taught about different rules or principles that we are supposed to follow. [YAGNI](https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it) (You Aren't Gonna Need It), [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) (Don't Repeat Yourself), [KISS](https://en.wikipedia.org/wiki/KISS_principle) (Keep It Simple, Stupid) the [Law of Demeter](https://en.wikipedia.org/wiki/Law_of_Demeter), and a whole bunch of others. 

Authors argue that at the end of the day, what really matters is that the system we are working on is **Easier To Change (ETC)**. 

Why should we favor composition over inheritance? Because it makes things easier to change.

Why is decoupling useful? Because by isolating concerns we make them easier to change.

Why do we always make sure to follow the Single Responsibility rule? Because if there is a change in the requirements, we only have to touch a single module. ETC.

ETC is a value, not a rule, that should help you make decisions. By asking whether the approach I'm taking will make it relatively easy to change it later when needed, you will usually know which path to choose.

### Prototypes and Tracer Bullets

One of the ideas behind prototyping is that you will throw away every piece of code you wrote to explore or try the concept.

For example, to prove that your idea is doable, you could start with a more forgiving language like Python. At this point, you don't need any user interface, specific architecture, or design. None of the code will end up in the codebase anyway.

When you verify that your vision is indeed possible to implement, you ditch everything and recode it properly using the lessons you've learned and tools that work on your target's platform.

On the other spectrum, we have tracer bullets. Authors use the term tracer bullet development to visually illustrate the need for immediate feedback under actual conditions with a moving goal.

It's an important distinction. With the tracer bullets approach, you don't throw away any code when you finish. The code you write will end up in the final product. The only difference is that you focus on the key aspects of the system first.

For instance, imagine building a mobile application where users fill out a form. That submission hits our server to be processed later. A finished app requires a pleasant user interface with animations, validation on both the client and server-side, well-formatted responses, etc. 

With the tracer bullet development approach, we would first focus on the most important things. We would start with a very basic form without any validation, and the submission would be printed to the console or saved in logs on the server-side without hitting the database.

At this point, we have a solid foundation. We've established the communication between the mobile app and our server. The code is covered by tests. Now, we have something to show to our clients or teammates. Later, we can expand and add more functionality with confidence.

### Estimating time

Managers or clients often ask us to evaluate how long something will take. Estimating time is extremely difficult, especially when people expect us to provide a single number, like 12 days.

In the real world, people don't estimate with single numbers. They use a range of scenarios.

There is a methodology that adopts this style of assessment. It's called *Program Evaluation Review Technique*, or [**PERT**](https://en.wikipedia.org/wiki/Program_evaluation_and_review_technique).

Every PERT task has an optimistic, most likely, and pessimistic estimate. The tasks are arranged into a dependency network, and then you use some simple statistics to identify likely best and worst times for the overall project. 

With this approach, you don't specify one fixed number. It helps to account for different potential problems that we might encounter along the way.

Of course, a single method won't suddenly make estimating easy. That's why it's important to record our estimates. When we finish a project or an iteration, we can see how close we were. If the gap was significant, we should stop and think about a reason. With time and practice, we will give more accurate estimations.


### Be fluent with your tools

If you recall yourself trying to learn how to drive a car, you might remember that you had to think about every action you took. Later, with more experience, controlling the car became automatic and instinctive.

We should aim for the same automation mechanisms when using our tools. If we don't have to think about all the keyboard shortcuts and options while programming, our minds will have more capacity to think about the problem at hand.

The authors give us some tips about what it means to be fluent with our tools. You are fluent when you can:

* When editing text, move and make selections by character, word, line, and paragraph.
* When editing code, move by various syntactic units (matching delimiters, functions, modules, etc.).
* Reindent code following changes.
* Comment and uncomment blocks of code with a single command.
* Split the editor window into multiple panels and navigate between them.
* Navigate to a particular line number.
* Search for both strings and regular expressions.
* Temporarily create multiple cursors based on a selection or a pattern match, and edit the text at each in parallel.
* Run the current project’s tests.

Ideally, you can do all of this without using a mouse or a trackpad.

### Debug with confidence

Debugging and finding errors is not always fun. There is no point in making it more problematic than it has to be. The authors give us some general steps that we can follow to make this process at least a bit more effortless for us.

First, before you start debugging, you should adopt the right mindset. Turn off all defensive mechanisms that protect your ego. Also, don't waste time thinking that this couldn't happen or it was impossible. You are looking at the stack trace right now. It clearly did happen.

Then, always begin with making a bug reproducible. If you can't do it, you will never know if it's fixed. Ideally, try to make it possible to reproduce the error with a single command or action.

The root of the problem may lie in the OS, the compiler, or a third-party library. But that should never be your first thought. Always assume that the bug exists in your code and start from there. Because even if it does lie in the library, you would still have to trace the problem in your code before submitting the bug report to the maintainers.

Lastly, make sure that whatever was the problem, you’ll know if it occurs again. If your tests pass with the bug in the code, you can't trust them with catching the bug next time.

Write a test, see it fail, fix the bug, and sleep well knowing that you are covered.

### Engineering daybook

Have you ever found yourself trying to remember how you solved a particular problem? You probably wished at that moment that you had written the solution and explanation somewhere. It would save you a lot of time and energy.

That's why Andy and Dave recommend keeping an engineering daybook when working. It's a kind of journal in which you record what you do, things you learned, sketches of ideas, notes from meetings, variable values when debugging, etc.
  
The daybook has many benefits:

* It's more reliable than memory.
* It acts as a kind of rubber duck. It forces you to think about a problem before writing it down.
* It serves as a database of documented memories. You can look at it and think about the people you collaborated with, projects you worked on, and problems you faced.

The authors suggest sticking to a plain old pen and paper. However, I think it might make sense to use a digital version for one reason. It's searchable, and you can categorize your notes more easily.

### Dead programs don't lie

Programmers tend to take a defensive approach when programming. We try to catch or rescue all possible exceptions, check for nullability before using variables, verify that the lists passed as arguments are not empty, etc. We avoid crashes at all costs.

The problem with this approach is that the app or system we are working on might stop working **silently** on production without us having any way of detecting it. 

Sure, we checked that the list we wanted to display on the screen wasn't empty or null. There will be no exception thrown when we try to access it. But what will the user see? Most likely a blank screen. Our app will be in a state that it wasn't designed for. At this point, it's hard to tell why the list was empty. Did we forget to populate it? Did we receive corrupted data from the backend?

If our app gets in a weird state, it would be nice to know it. That's why unhandled exceptions are not always such a bad thing. They are collected by your crash reporting tool and will help you find the root of the problem.

Additionally, allowing our program to continue after entering an inappropriate state might be disastrous. Anything it does from this point forward can introduce strange problems like inconsistent or corrupted data sent to our production database.

> A dead program does a lot less damage than a crippled one. 

I also recommend reading [this](https://jeroenmols.com/blog/2017/03/08/appcrash/) article (and its comment section for contrasting opinions) about crashes in mobile apps.

### Take small steps

It's usually better to take small and deliberate steps. You then check for feedback and adjust before proceeding. Feedback can take many forms, for example, an MVP that you show to your client or unit tests that verify the correctness of your last code change.

We should avoid steps that are too big. They require an attempt to predict the future. No one can do that. You might want to think twice when you find yourself doing one of those things:

* Estimate completion dates months in advance.
* Guess users' future needs and preferences.
* Predict future tech trends and tools.

Around two decades ago, debates raged in online forums over questions like "Who would win the desktop GUI wars, Motif or OpenLook?". Chances are, you've never heard of these technologies. Remember this story next time you see yourself fortune-telling.

### Decoupling

We want our code to be as flexible as possible. Coupled code is hard to change. Modifications in one place can have secondary effects in other locations.

The authors present this snippet of code to demonstrate the problem:

```kotlin
private fun applyDiscount(customer: Customer, orderId: Int, discount: Discount) {
    customer
        .orders
        .find(orderId)
        .getTotals()
        .applyDiscount(discount)
}
```

What we can see here is the so-called *Train Wreck*. This code is traversing five levels of abstraction, from customer to total amounts. Our top-level code has to know about each level's internal implementation. It knows that the `customer` object exposes orders, that the `orders` have a find method, and that the order object has a `totals` object which contains information about discounts and grand totals.

There is a lot of coupling here. Imagine that someone decides that no order can have a discount of more than 50%. Where should we implement this change? You might think that it belongs to `applyDiscount()`. It's true, but the problem is that there might be other places that modify the `totals` object besides this function. We would have to find all of them and adjust accordingly. That's potentially a lot of work.

So, what's the solution? Andy and Dave have a principle that they call:

> Tell, Don't Ask.

We shouldn't ask about an object's internal state and then modify it based on the information we receive. We should tell it to do what we need. 

We could refactor the code above to something like this:

```kotlin
private fun applyDiscount(customer: Customer, orderId: Int, discount: Discount) {
    customer
        .findOrder(orderId)
        .applyDiscount(discount)
}
```

With this change, our top-level code doesn't have to know that the order class uses a separate object to store its totals and discounts. We also don't fetch a list of orders from the customer and search for a specific one. We get the order that we want directly from the customer.

Right now, we have only one place that manages discounts and all rules can be encapsulated there.

### Fear of the blank page

I'm sure we've all experienced this. We start working on a new feature or a completely new project. The cursor is blinking. The screen is empty and ready to be filled with code. For some reason, the experience is very intimidating. We put off making the initial commitment of starting.

The authors say the most likely reason we feel this way is that we are afraid of making a mistake. We fear the architecture we are implementing will not be flexible enough. We worry that the code will not be readable by other developers. We stress about introducing new bugs or problems. Potentially, we may think that the task is beyond our skills. We can't see our way through to the end.

Andy and Dave claim that they found a brain hack that seems to work in this kind of situation:

> Pretend that you are prototyping.

Remember that prototypes get thrown away, even if they don't fail. If your mind knows this, you are less likely to feel anxious. If anything goes wrong, you will stash all of it away. No problem. But what's usually happening is this. After some time, you find that the code flies from your brain into the editor. The initial stress is gone. You start to see the bigger picture. You know how to proceed.

To make it even more clear to you, the authors suggest writing "I'm prototyping" on a sticky note and attaching it on the side of your screen.

I tried this trick a couple of times and it worked for me. It helps with one of the hardest things - forcing yourself to start writing.

### Don't program by coincidence

When you use a technology you don't understand, you program by coincidence. If you're unsure why something works, you won't know why it fails. Programming by coincidence is the opposite of programming deliberately.

"It doesn't work without it" is not a good reason to keep maintaining a specific portion of code. Unnecessary calls slow the program down or introduce bugs that are difficult to spot. What's worse is that other developers won't usually bother trying to mess with it to improve it. "If it works now, it's better to leave it alone..."

We all want to work with error-free code and catch all bugs as early in the development cycle as possible. To do that, we should always program deliberately. Make sure you have at least a basic understanding of all the libraries and tools you use in your application or system. Asks yourself questions - would I be able to explain this piece of code to a more junior programmer? If not, perhaps you are relying on coincidence.

Next time something seems to work, but you don’t know why, make sure it isn’t just a coincidence. Don't assume anything. Prove it.

### Code is a garden

> A tourist visiting England’s Eton College asked the gardener how he got the lawns so perfect. “That’s easy,” he replied, “You just brush off the dew every morning, mow them every other day, and roll them once a week.” 
> 
> “Is that all?” asked the tourist. “Absolutely,” replied the gardener. “Do that for 500 years and you’ll have a nice lawn, too.” 

Code is not static. It constantly evolves. Inaccurately, some people associate software development with building construction. It implies that there is an architect that draws the initial blueprints. Then, contractors dig the foundation, build the superstructure and apply final touches. At this point, there is no easy way to change anything. The project is considered finished and the tenants move in.

The software doesn't quite work that way. It's more like a garden when things change all the time. You plant different things according to the initial plan and environment. After observing the typical weather conditions, you may move plantings to other places. Overgrown plants get split. You pull weeds and fertilize plantings. The health of the garden needs constant monitoring and adjustments.

The gardening metaphor is much closer to the realities of software development. Laws of physics don't constrain code as they do with buildings. 

This philosophy led to a discipline called *Refactoring*. Martin Fowler defines it as a "disciplined technique for restructuring an existing body of code, altering its internal structure without changing its external behavior." This technique inlines with the idea that code requires constant improvement and refinement.

### Tests aren't only about finding errors

When we ask developers why they write tests, most of them would say "to make sure the code works". It's a perfectly valid reason. But Andy and Dave argue that there is much more to tests than that.

They believe that the sole act of **thinking** about tests while coding is the main benefit of tests.

Consider the following example. You are working on a class that is supposed to fetch tomorrow's lottery numbers. That's the code you have written so far:

```kotlin
class FetchTomorrowsLotteryNumbersUseCase {
    private val service = LotteryNumbersService()
    
    operator fun invoke() = service.fetch()
}
```

The above code works completely fine when you run it. Now what's left is to wait for tomorrow and hope that this service is reliable.

But because you are constantly thinking about tests, you start wondering how to write tests for the given code. It would be reasonable to use some test data and not rely on the remote service. The current implementation makes it problematic because we don't control how this class interacts with data. The easiest way to make it possible during tests is to pass the service as a parameter. The code could now look like this:

```kotlin
class FetchTomorrowsLotteryNumbersUseCase(
    private val service: LotteryNumbersService
) {     
    operator fun invoke() = service.fetch()
}
```

Thinking about testing made us reduce coupling in the code (by passing in a service that we can control from the outside) and increase flexibility. When we think about writing tests for the code we are working on, we imagine ourselves as clients of the code, not its authors.

### Be proud of your work

The authors say that pragmatic programmers should be proud to sign their work just like artisans of an earlier age. When we are accountable for a piece of code, we aspire to do a job we can be proud of. 

The opposite of it is anonymity. It provides room for mistakes, sloppiness, and bad code, especially on large projects. It creates an environment where no one feels any responsibility.

We should aim to associate our signatures with an indication of high quality. When other developers see our code, they should expect it to be solid, well-written, thoroughly tested, and documented.

## Summary

Andy and Dave end the book by concluding that we have a lot of power (and responsibility) as programmers. Systems and devices that we build can shape the lives of millions. How often do we stop and think about this?

We should do our best to protect the users from any potential harm. If we are involved in a project that requires us to do something against our beliefs or values, we shouldn't be afraid to say "no". Would I enjoy using this software as a user? Would I be comfortable with my sensitive data being shared with this company? Do I know something important that users don't?

We are building the future for ourselves and our descendants. Let's envision the future everyone would like to live in and have the courage to work on it every day.