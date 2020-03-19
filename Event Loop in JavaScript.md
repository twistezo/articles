# Asynchronous and single-threaded JavaScript? Meet the Event Loop.

JavaScript is a single-threaded language but at the same time is also non-blocking, asynchronous and concurrent. This article will explain to you how it is happen.

## Runtime

JavaScript is an interpreted language, not a compiled. This means that it needs an interpreter which convert JS code to machine code. There are several interpreters that we call engines. The most popular browser engines are V8 (Chrome), Quantum (Firefox) and WebKit (Safari). It's worth to note that V8 is also used in popular non browser runtime - Node.js.

Each engine contains memory heap, call stack, event loop, callback queue and WebAPI which contains HTTP requests, timers, events etc. implemented in own way for faster and safer interpretation of JS code.

<p align="center">
    <img src="./img/runtime-architecture.png" width="450"/>
</p>
<p align="center">
    <em>Basic JS runtime architecture. Author: Alex Zlatkov</em>
</p>

## Single thread

Single-thread language menas that it has one call stack and one memory heap. This in turn means running one thing at a time.

`Stack` is a continuous region of memory, allocating local context for each executing function.

`Heap` is a much larger region, storing everything allocated dynamically.

`Call Stack` is a data structure which records basically where in the program we are.

### Call Stack

Let's write a simple code and track what's happening on the call stack.

<p align="center">
    <img src="./img/stack.gif" width="500"/>
</p>

As you can see, tasks are added to the stack, executed and later deleted. It works in the LIFO way - Last In, First Out. Each entry in the call stack is called `Stack Frame`.

Knowledge of call stack is useful for reading errors stack traces. Generally the exact reason of error is on the top at first line but the order of code execution is from the bottom to top.

Sometimes you can deal with popular error `Maximum call stack size exceeded`. It is easy to get this using recursion:

```
function foo() {
    foo()
}

foo()
```

and our browser or terminal freezes. Different browsers and even their different versions have different call stack size limits. In the vast majority of cases they are sufficient and the problem should be looked elsewhere.

### Blocked Call Stack

Here is example of blocking JS thread. Let's try to read file `foo` and `bar` using Node.js synchronous function `readFileSync`.

<p align="center">
    <img src="./img/blocking.gif" width="500"/>
</p>

This is looped GIF. As you see JS engine waits until the first call `readFileSync` is completed. But this will not happen because there is no `foo` file. The second function will never be called.

## Asynchronous behavior

It the other way JS can be non-blocking and may behave as if it were multi-threaded. It means that it doesn't wait for the response of an API call, I/O events etc. and can continue the code executing. It is possible thanks to the JS engines which use under the hood real multi-threading languages like C++ (Chrome) or Rust (Firefox). They provide us with the Web API under the browsers hoods or ex. I/O API under Node.js.

<p align="center">
    <img src="./img/callback-queue.gif" width="800"/>
</p>

In the GIF above we can see that first function is pushed to the Call Stack and executes immediately `Hi` in the console.

Then we call `setTimeout` function provided to us by browser WebAPI. It goes to the Call Stack and its asynchronous callback function `foo` goes to the WebApi's queue where waiting for the call, set after 3 seconds.

In the meantime, the program continues the code and in the console we see `Hi. I am not blocked`.

Each function in the WebAPI queue after its invoke, goes to the `Callback Queue`. It is queue where functions wait until Call Stack will be empty. When becomes empty the one by one goes to its.

So when our `setTimeout` timer finishes the countdown, our `foo` function goes to `Callback Queue`, waits until Call Stack becomes, goes there, executes and we see `Hi from asynchronous callback` in the console.

### Event Loop

The question is how does the runtime knows that the call stack is empty and how is the events in the callback queue invoked? Please meet the `Event Loop`. It is a part of JS engine. It is a process that constantly keeps on checking if the call stack is empty and if it is, it checks whether there is an event in the callback queue waiting to be invoked.

That's all the magic behind the scene!

## Wraping up theory

### Concurrency & Parallelism

`Concurrency` means executing multiple tasks at the same time but not simultaneously. Ex. two tasks works in overlapping time periods.

`Parallelism` means performing two or more tasks simultaneously. Ex. performing multiple calculations at the same time.

### Threads & Processes

`Threads` are a sequence of code execution which can be executed independently of one another.

`Process` is an instance of a running program. A program can have multiple processes.

### Synchronous & Asynchronous

In `synchronous` programming, tasks are executed one after another. Each task waits for any previous task to complete and then gets executed.

In `asynchronous` programming, when one task gets executed, you could switch to a different task without waiting for the previous to get completed.

### Synchronous and Asynchronous in a single and multi-threaded environment

`Synchronous with single thread`: Each task gets executed one after another. Each task waits for its previous task to get executed.

`Synchronous with multi-threads`: Tasks get executed in different threads but wait for any other executing tasks on any other thread.

`Asynchronous with single thread`: Tasks start executing without waiting for a different task to finish. At a given time a single task gets executed.

`Asynchronous with multi-threads`: Tasks get executed in different threads without waiting for any tasks and independently finish off their executions.

## JavaScript classification

Accordingly, if we consider how JS engines works under the hood, we can classify JS as asynchronous with single-threaded interpreted language. "Interpreted" word is very important. Because it is interpreted, it will always be runtime dependent and it will never be as fast as compiled languages that have multi-threading built-in.

It is noteworthy that in Node.js we can achieve real multithreading. With the proviso that each thread must be started as a separate process. There are libraries for this but Node.js has built in feature called [Worker Threads](https://nodejs.org/api/worker_threads.html#worker_threads_worker_threads).

All event loop GIFs come from the [Loupe](https://github.com/latentflip/loupe) application created by Philip Roberts. You can test there your asynchronous scenarios.
