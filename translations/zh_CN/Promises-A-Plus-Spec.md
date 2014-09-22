# Promises/A+ 规范说明

_一个由实现者制定和使用的合理可操作的JavaScript Promise开放标准_

一个_promise_代表异步操作的最终结果。与promise互动的最主要方式是通过它的`then`方法，`then`方法注册回调函数来接受promise的最终结果或者不能被满足的原因。

这份规范详述了`then`方法的行为，并且为所有遵守Promises/A+规范的promise实现提供了一个可依赖，可操作的基础。就这一点来说，这份规范应该被认为是非常稳定的。虽然Promises/A+组织可能偶尔对这份规范进行向下兼容的修订来提出新发现的边界条件，对于较大的变更或者向下不兼容的内容，我们将在谨慎的考虑，讨论和测试后再做出整合。

从历史上来说，Promises/A+声明了早期的[Promises/A提案](http://wiki.commonjs.org/wiki/Promises/A)中的行为条款，并对Promises/A+进行了扩展，包括了实际行为，删去了部分有问题的或者有特定条件的部分。

最终，Promises/A+规范的核心并没有处理如何创建，满足或者拒绝promises，而是集中精力提供一个可操作的`then`方法。在之后相关规范的工作中将会涉及这些主题。

# 1. 术语

## 1.1 “promise”是包含一个行为符合本规范的`then`方法的对象或者函数。
## 1.2 “thenable”是一个定义了`then`方法的对象或者函数。
## 1.3 “value”是任意合法的JavaScript值（包括`undefined`, thenable 或者 promise）。
## 1.4 “exception”是一个用`throw`声明抛出的值。
## 1.5 “reason”是一个指出为什么promise被拒绝的值。

# 2. 要求

## 2.1 Promise 状态

一个promise必须在三个状态之中：pending（保持），fulfilled（被满足）或者 rejected（被拒绝）。

### 2.1.1 当处于pending（保持）状态时，promise：
#### 2.1.1.1 可能转换到fulfilled 或者 rejected 状态。
### 2.1.2 当处于fulfilled（被拒绝）状态时，promise：
#### 2.1.2.1 不能转换到任何其他状态。
#### 2.1.2.2 必须有一个值而且不能改变。
### 2.1.3 当处于 rejected （被拒绝）状态时，promise：
#### 2.1.3.1 不能转换到任何其他状态。
#### 2.1.3.2 必须有一个`reason`而且不能改变。

这里的“不能改变”指不变一致性（比如===）,并不是指深入不变性。

## 2.2 `then`方法

一个promise必须提供一个`then`方法来访问它当前值“value”，最终值“value”或者拒绝原因“reason”。

一个promise的`then`方法接受两个参数：

```javascript
promise.then(onFulfilled, onRejected)
```
### 2.2.1 `onFulfilled`和`onRejected`都是可选参数：
#### 2.2.1.1 如果`onFulfilled`不是一个函数，它必须被忽略。
#### 2.2.1.2 如果`onRejected`不是一个函数，它必须被忽略。

### 2.2.2 如果`onFulfilled`是一个函数：
#### 2.2.2.1 它必须在`promise`被满足后被调用，并且`promise`的值“value”作为第一个参数。
#### 2.2.2.2 在`promise`被满足前，它不能够被调用。
#### 2.2.2.3 它不能被多次调用。

### 2.2.3 如果`onRejected`是一个函数：
#### 2.2.3.1 它必须在`promise`被拒绝后被调用，并且`promise`的拒绝原因“reason”作为第一个参数。
#### 2.2.3.2 在`promise`被拒绝前，它不能够被调用。
#### 2.2.3.3 它不能被多次调用。

### 2.2.4 `onFulfilled`和`onRejected`直到[执行上下文](http://es5.github.io/#x10.3)栈只包含平台代码时才可以被调用。

### 2.2.5 `onFulfilled`和`onRejected`必须作为函数被调用（例如：不包含`this`值）［[3.2](http://promisesaplus.com/#notes)］

### 2.2.6 `then`可以在同一个promise上被调用多次。
#### 2.2.6.1 当`promise`被满足时，各个`onFulfilled`回调函数必须按照它们在`then`中出现的顺序被执行。
#### 2.2.6.2 当`promise`被拒绝时，各个`onRejected`回调函数必须按照它们在`then`中出现的顺序被执行。

### 2.2.7 `then`必须返回一个promise［[3.3](http://promisesaplus.com/#notes)］

```javascript
promise2 = promise1.then(onFulfilled, onRejected);
```
#### 2.2.7.1 如果`onFulfilled`或者`onRejected`返回了一个值`x`,运行Promise Resolution Procedure `[[Resolve]](promise2, x)` 
#### 2.2.7.2 如果`onFulfilled`或者`onRejected`抛出了异常`e`,`promise2`必须被拒绝，`e`作为拒绝原因。
#### 2.2.7.3 如果`onFulfilled`不是一个函数，而且`promise1`被满足了，`promise2`必须以`promise1`满足的值被满足。
#### 2.2.7.4 如果`onRejected`不是一个函数，而且`promise1`被拒绝了，`promise2`必须以`promise1`拒绝的原因被拒绝。

## 2.3 Promise Resolution Procedure
Promise resolution procedure是一个输入为一个promise和一个值“value”的抽象操作，我们表示为`[[Resolve]](promise, x)`。如果`x`是一个`thenable`，当在`x`的行为在某种程度是一个promise的假设下，它尝试去促使`promise`接受`x`的状态。

只要promise的各种实现暴露Promise/A+适用的`then`方法，那么这种对`thenable`的处理就允许promise的各种实现可以相互操作。它也允许Promise/A+实现去“同化”带有合理的`then`方法但是与本规范不一致的实现。

运行`[[Resolve]](promise, x)`, 经历以下几个步骤：

### 2.3.1 如果promise和x都涉及相同的对象，就用一个TypeError作为原因去拒绝promise。
### 2.3.2 如果`x`是一个promise，就接受它的状态［[3.4](http://promisesaplus.com/#notes)］:
#### 2.3.2.1 如果`x`在保持状态，promise必须维持在保持状态直到x被满足或者拒绝。
#### 2.3.2.2 如果`x`被满足了，用相同的值满足promise。
#### 2.3.2.3 如果`x`被拒绝了，用相同的原因拒绝promise。
### 2.3.3 否则，如果`x`是一个对象或函数，
#### 2.3.3.1 让`then`成为`x.then`。 ［[3.5](http://promisesaplus.com/#notes)］
#### 2.3.3.2 如果在一个抛出的异常`e`中得到了`x.then`的结果，用`e`作为原因“reason”拒绝promise。
#### 2.3.3.3 如果`then`是一个函数，将`x`作为`this`调用它，第一个参数`resolvePromise`，第二个参数`rejectPromise`：
##### 2.3.3.3.1 当`resolvePromise`用值“value”`y`调用的时候，运行`[[Resolve]](promise,y)`.
##### 2.3.3.3.2 当`rejectPromise`用原因“reason”`r`被调用时，用`r`拒绝promise
##### 2.3.3.3.3 如果`resolvePromise`和`rejectPromise`都被调用了，或者用同样的参数多次调用，优先第一次调用，忽略其他调用。
##### 2.3.3.3.4 如果在调用then的时候抛出了异常`e`，
###### 2.3.3.3.4.1 如果`resolvePromise`或者`rejectPromise`已经被调用了，则忽略。
###### 2.3.3.3.4.2 否则，用`e`作为原因“reason”拒绝promise。
### 2.3.4 如果x不是一个对象或者函数，用x满足promise。

如果一个promise被一个循环thenable链中的"thenable"解决了，例如[[Resolve]](promise, thenable)递归类型,最终导致`[[Resolve]](promise,thenable)`被再次调用，根据以上的算法将导致死循环。这一点鼓励去实现，但是不是必须的，检测这种循环并用一个消息TypeError作为原因拒绝promise。［[3.6](http://promisesaplus.com/#notes)］。

#3. Notes

## 3.1 这里的“platform code”指的时引擎，环境还有promise的实现代码。在实践中，这个要求保证在事件列表中的`then`用一个新的栈被调用了后，onFulfilled和onRejected异步执行。这个可以用一个宏任务“macro-task”机制比如`setTimeout`或者`setImmediate`实现，也可以用一个微任务“micro-task”机制，比如`MutationObserver`或者`process.nextTick`。因为promise实现被认为是平台代码，它可能自己包含一个处理函数在其中被调用的计划任务队列或者“trampoline”。

## 3.2 在严格模式（strict mode）下`this`为`undefined`；在松散模式下时全局对象。

## 3.3 具体实现可能会允许`promise2 === promise1`，并且所有的实现都符合规范要求。每一种实现都应该用文档说明是否允许`promise2 === promise1`，如果允许的话有什么条件需要满足。

## 3.4 一般来说，在当前实现中只需要知道`x`是一个promise。这个条款允许特殊实现的实现，这意味着采用已知promise的状态。

## 3.5 这个过程的第一步将存储一个到`x.then`的引用，然后测试那个引用并且调用哪个应用，避免多次防问`x.then`属性。这些前置条件对于保证一个可访问属性的一致性是非常重要的，因为它的值可能在两次读取的时候发生改变。

## 3.6 具体实现不应该在thenable链的深度上设置任意的限制，并且假设超出那个限制后将会陷入死循环。只有真的循环可以导致`TypeError`;如果一个单个thenable的无限长的链允许的华，那么无限循环是一个正确的行为。
