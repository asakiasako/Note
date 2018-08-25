# JavaScript 异步调用的方法汇总

* callback function
* Promise
* async ... await ...

## callback function

``` javascript
client.invoke('a', 'b', 'c', (err, res, more) => {
    if (err) {
        // ...
    } else {
        // ...
    }
})
```

## Promise

### 包装成 Promise 对象

``` javascript
// encapsule into Promise

function promiseInvoke(...args) {
    return new Promise(function (resolve, reject) {
        client.invoke(...args, (err, res, more) => {
            if (err) {
                reject(err)
            } else {
                resolve(res, more)
            }
        })
    })
}

// 如果在 then 中传入两个回调函数，第一个会传递给 resolve， 第二个会传递给 reject
promiseInvoke('a', 'b', 'c').then((res, more) => {
    // ...
}, (err) => {
    // ...
})

// 用 then 传递 resolve， 用 catch 传递 reject
promiseInvoke('a', 'b', 'c').then((res, more) => {
    // ...
}).catch((err) => {
    // ...
})
```

### Promise 的方法

* Promise.prototype.then(resolve, reject)

    **resolve**

    当Promise变成接受状态（fulfillment）时，该参数作为回调函数被调用（参考： Function）。该函数有一个参数，即接受的最终结果（the fulfillment  value）。如果传入的 onFulfilled 参数类型不是函数，则会在内部被替换为(x) => x ，即原样返回 promise 最终结果的函数

    **reject**

    当Promise变成拒绝状态（rejection ）时，该参数作为回调函数被调用（参考： Function）。该函数有一个参数,，即拒绝的原因（the rejection reason）。

    注意：如果忽略针对某个状态的回调函数参数，或者提供非函数 (non-function) 参数，那么 then 方法将会丢失关于该状态的回调函数信息，但是并不会产生错误。如果调用 then 的 Promise 的状态（fulfillment 或 rejection）发生改变，但是 then 中并没有关于这种状态的回调函数，那么 then 将创建一个没有经过回调函数处理的新 Promise 对象，这个新 Promise 只是简单地接受调用这个 then 的原 Promise 的终态作为它的终态。

* Promise.prototype.catch(reject)

    **reject**

    当Promise 被rejected时,被调用的一个Function。 该函数拥有一个参数：

    `reason` rejection 的原因。

    如果 onRejected 抛出一个错误或返回一个本身失败的 Promise ，  通过 catch() 返回的Promise 被rejected；否则，它将显示为成功（resolved）。 

* Promise.prototype.finally(finally)

    由于无法知道promise的最终状态，所以finally的回调函数中不接收任何参数，它仅用于无论最终结果如何都要执行的情况。

* Promise.resolve(value)

    返回一个解析过带着给定值的 Promise 对象，如果返回值是一个 Promise 对象，则直接返回这个 Promise 对象。

    ``` javascript
    let promise1 = Promise.resolve(value)

    promise1.then((value) => {
        console.log(value)
    })
    ```

* Promise.reject(reason)

    静态函数 Promise.reject 返回一个被拒绝的Promise对象。

    ``` javascript
    let promise2 = Promise.reject(reason)

    promise2.then((reason) => {
        // 不执行
    }).catch((reason) => {
        console.log(reason)
    })
    ```

* Promise.all(iterable)

    如果传入的参数是一个空的可迭代对象，则返回一个已完成（already resolved）状态的 Promise。

    如果传入的参数不包含任何 promise，则返回一个异步完成（asynchronously resolved） Promise。注意：Google Chrome 58 在这种情况下返回一个已完成（already resolved）状态的 Promise。

    其它情况下返回一个处理中（pending）的Promise。这个返回的 promise 之后会在所有的 promise 都完成或有一个 promise 失败时异步地变为完成或失败。 见下方关于“Promise.all 的异步或同步”示例。返回值将会按照参数内的 promise 顺序排列，而不是由调用 promise 的完成顺序决定。

    ``` javascript
    var promise1 = Promise.resolve(3);
    var promise2 = 42;
    var promise3 = new Promise(function(resolve, reject) {
    setTimeout(resolve, 100, 'foo');
    });

    Promise.all([promise1, promise2, promise3]).then(function(values) {
    console.log(values);
    });
    // expected output: Array [3, 42, "foo"]
    ```

* Promise.race(iterable)

    race 函数返回一个 Promise，它将与第一个传递的 promise 相同的完成方式被完成。它可以是完成（ resolves），也可以是失败（rejects），这要取决于第一个完成的方式是两个中的哪个。

    如果传的迭代是空的，则返回的 promise 将永远等待。

    如果迭代包含一个或多个非承诺值和/或已解决/拒绝的承诺，则 Promise.race 将解析为迭代中找到的第一个值。

    ``` javascript
    var promise1 = new Promise(function(resolve, reject) {
        setTimeout(resolve, 500, 'one');
    });

    var promise2 = new Promise(function(resolve, reject) {
        setTimeout(resolve, 100, 'two');
    });

    Promise.race([promise1, promise2]).then(function(value) {
    console.log(value);
    // Both resolve, but promise2 is faster
    });
    // expected output: "two"
    ```

## async...await...

### 用 async 函数包装 异步调用

``` javascript
async asyncInvock(...args) {
    client.invoke(...args, (err, res, more) => {
        if (err) throw err
        else return (res, more)
    })
}
```

### async function

`async function name([param[, param[, ... param]]]) { statements }`

当调用一个 async 函数时，会返回一个 Promise 对象。当这个 async 函数返回一个值时，Promise 的 resolve 方法会负责传递这个值；当 async 函数抛出异常时，Promise 的 reject 方法也会传递这个异常值。

async 函数中可能会有 await 表达式，这会使 async 函数暂停执行，等待表达式中的 Promise 解析完成后继续执行 async 函数并返回解决结果。

注意， await 关键字仅仅在 async function中有效。如果在 async function函数体外使用 await ，你只会得到一个语法错误（SyntaxError）。

如果一个 async 函数中有多个 await 语句，那么它会在上一个 await 执行结束后再执行下一个 await，这点和 Promise.all() 是不同的。

``` javascript
function resolveAfter2Seconds() {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve('resolved');
    }, 2000);
  });
}

async function asyncCall() {
  console.log('calling');
  var result = await resolveAfter2Seconds();
  console.log(result);
  // expected output: 'resolved'
}

asyncCall();
```

# await

`[return_value] = await expression;`

await 表达式会暂停当前 async function 的执行，等待 Promise 处理完成。若 Promise 正常处理(fulfilled)，其回调的resolve函数参数作为 await 表达式的值，继续执行 async function。

若 Promise 处理异常(rejected)，await 表达式会把 Promise 的异常原因抛出。

另外，如果 await 操作符后的表达式的值不是一个 Promise，那么该值将被转换为一个已正常处理的 Promise。


