# Promise原理

## Promise

### Promise解决了什么问题
- 回调地狱，代码难以维护， 常常第一个的函数的输出是第二个函数的输入这种现象
- promise可以支持多个并发的请求，获取并发请求中的数据

### reject细节
- 情况一：由于2已经捕获了错误，所以4不会执行，并且3和5可以正常执行
    ```js
    new Promise((resolve, reject) => {
        setTimeout(() => {
            reject(1)
        }, 0)
    }).then((value) => {
        console.log('success', value) // 1
    }, (error) => {
        console.log('error', error)   // 2
        return error
    }).then(value => {
        console.log('success', value) // 3
        return value
    }).catch(error => {
        console.log('catch error', error) // 4
    }).then(value => {
        console.log('success', value) // 5
         return value
    })
    ```
- 情况二：reject会被catch捕获，最终只有4、5会执行
    ```js
    new Promise((resolve, reject) => {
        setTimeout(() => {
            reject(1)
        }, 0)
    }).then((value) => {
        console.log('success', value) // 1
    }).then(value => {
        console.log('success', value) // 3
        return value
    }).catch(error => {
        console.log('catch error', error) // 4
    }).then(value => {
        console.log('success', value) // 5
         return value
    })
    ```
- 情况三：catch**不能**不会捕获到错误，，异常将会被视为 “Uncaught (in promise)” 被抛出到全局去。但使用await是可以捕获到的
    ```js
    try {
        new Promise((resolve, reject) => {
            setTimeout(() => {
                reject(1)
            }, 0)
        }).then((value) => {
            console.log('success', value)
        })
    } catch(error) {
        console.log(error)
    }

    // 使用await可以捕获到
    function test(id) {
        return new MyPromise(((resolve, reject) => {
            setTimeout(() => {
                reject({ test: id })
            }, 5000)
        }))
    }

    async function main() {
        try {
            await test()
        } catch(error) {
            console.log('error', error)
        }
    }
    main()
    ```

### Promise.resolve()
- 说明
    - 参数： 将被Promise对象解析的参数，也可以是一个Promise对象，或者是一个thenable（即带有"then" 方法）。
    - 返回值： 返回一个带着给定值解析过的**Promise对象**，如果参数本身就是一个Promise对象，则直接返回这个Promise对象。
- 使用

    ```js
    const promise1 = Promise.resolve(123);
    promise1.then((value) => {
    console.log(value);
    // expected output: 123
    });

    // 输出：123
    ```
    **不要在解析为自身的thenable 上调用Promise.resolve。这将导致无限递归**
    ```js
    let thenable = {
    then: (resolve, reject) => {
        resolve(thenable)
    }
    }

    Promise.resolve(thenable)  //这会造成一个死循环
    ```

### Promise.reject()
- 返回一个带有拒绝原因的Promise对象。
- 使用
    ```js
    function resolved(result) {
        console.log('Resolved');
    }

    function rejected(result) {
        console.error(result);
    }

    Promise.reject(new Error('fail')).then(resolved, rejected);
    // 输出: Error: fail

    ```
## Promise.race & Promise.all
- **Promise.race**
    - 说明
        - race 函数返回一个 Promise，一旦迭代器中的某个promise解决或拒绝，返回的 promise就会解决或拒绝。（只要有一个Promise有结果就会返回，其他Promise将被忽略）
        - 如果传的迭代是空的，则返回的 promise 将永远等待。
        - 如果迭代包含一个或多个`非Promise/Resolve Promise/Reject Promise`，则 Promise.race 将解析为迭代中找到的**第一个值**。(也就是说如果存在非Promise值如一个数字2，将会返回数字2，因为数字2不需要等待结果)
    - 使用
        ```js
        const promise1 = new Promise((resolve, reject) => {
        setTimeout(resolve, 500, 'one');
        });

        const promise2 = new Promise((resolve, reject) => {
        setTimeout(resolve, 100, 'two');
        });

        Promise.race([promise1, promise2]).then((value) => {
        console.log(value);
        // Both resolve, but promise2 is faster
        });
        // expected output: "two"
        ```
- **Promise.all**
    - 说明
        - 参数： 接收一个promise的iterable类型（注：Array，Map，Set都属于ES6的iterable类型）
        - 返回值：
            - 完成状态
                - 如果传入的可迭代对象为空，Promise.all 会**同步**地返回一个已完成（resolved）状态的promise。
                - 如果**所有**传入的 promise 都变为完成状态，或者传入的可迭代对象内没有 promise，Promise.all 返回的 promise 异步地变为完成。
                - 在任何情况下，Promise.all 返回的 promise 的完成状态的结果都是一个数组，它包含所有的传入迭代参数对象的值（也**包括非 promise 值**）。并且返回的数组顺序跟传入promise**顺序是一样的**
            - 失败
                - 如果传入的 promise 中有一个失败（rejected），Promise.all 异步地将失败的那个结果给失败状态的回调函数，而不管其它 promise 是否完成。
        - 总结：
            - **所有Promise都成功或者有一个失败才会返回**
            - promise.all中的任务是**并行执行**的，但结果是**按顺序返回**的
            - Promise.all 当且仅当传入的可迭代对象为空时为同步，其他情况为异步
            - 如果参数中包含非 promise 值，这些值将被忽略，但仍然会被放在返回数组中（如果 promise 完成的话）
    - 使用：
        ```js
        const promise1 = Promise.resolve(3);
        const promise2 = 42;
        const promise3 = new Promise((resolve, reject) => {
        setTimeout(resolve, 100, 'foo');
        });

        Promise.all([promise1, promise2, promise3]).then((values) => {
        console.log(values);
        });
        // expected output: Array [3, 42, "foo"]
        ```

-  **Promise.allSettled**
    - 该Promise.allSettled()方法返回一个在所有给定的promise都已经fulfilled或rejected后的promise，并带有一个对象数组，每个对象表示对应的promise结果。

## 手写Promise
- Promise遵循[【翻译】Promises/A+规范](https://www.ituring.com.cn/article/66566)
- Promise 必须为以下三种状态之一：等待态（Pending）、执行态（Fulfilled）和拒绝态（Rejected）。
- 一旦Promise 被 resolve 或 reject，不能再迁移至其他任何状态（即状态 immutable）。
- 规范规定了 x（resolve 传入的值）不能跟 不能与 promise2（then生成的promise） 相等，这样会发生循环引用的问题
- 不论 promise1 被 reject 还是被 resolve 时 promise2 都会被 resolve，只有出现异常时才会被 rejected。
- then 方法可以被同一个 promise 调用多次，then 方法必须返回一个 promise 对象

```js
const PENDING = 'pending' // 等待态 Pending
const RESOLVED = 'resolved' // 执行态 Fulfilled
const REJECTED = 'rejected' // 拒绝态 Rejected

/**
 * 基于 PromiseA+ 规范的 Promise 模型
 * [【翻译】Promises/A+规范](https://www.ituring.com.cn/article/66566)
 * @param {*} fn
 */
function MyPromise(fn) {
	const that = this // 码可能会异步执行，用于获取正确的 this 对象
	that.state = PENDING
	that.value = null // 用于保存 resolve 或者 reject 中传入的值
	that.resolvedCallbacks = [] // 用于保存 then 中的回调，因为当执行完 Promise 时状态可能还是等待中，这时候应该把 then 中的回调保存起来用于状态改变时使用
	that.rejectedCallbacks = []

	/**
	 * resolve函数
	 * @param {*} value
	 */
	function resolve(value) {
		// 对于 resolve 函数来说，首先需要判断传入的值是否为 Promise 类型
		if (value instanceof MyPromise) {
			return value.then(resolve, reject)
		}
		// 使用setTimeout保证执行顺序
		setTimeout(() => {
			if (that.state === PENDING) {
				console.log('resolve:', value, 'resolvedCallbacks len:', that.resolvedCallbacks.length)
				that.state = RESOLVED
				that.value = value
				that.resolvedCallbacks.map(cb => cb(that.value))
			}
		}, 0)
	}

	/**
	 * reject函数
	 * @param {*} value
	 */
	function reject(value) {
		setTimeout(() => {
			if (that.state === PENDING) {
				console.log('reject', value)
				that.state = REJECTED
				that.value = value
				that.rejectedCallbacks.map(cb => cb(that.value))
			}
		}, 0)
	}

	/**
	 * 执行 Promise 中传入的函数
	 */
	try {
		fn(resolve, reject)
	} catch (e) {
		reject(e)
	}
}

/**
 * 兼容多种 Promise 的 resolutionProcedure 函数
 * @param {*} promise2 - 新的promise
 * @param {*} x - 终值
 * @param {*} resolve
 * @param {*} reject
 * @returns
 */
function resolutionProcedure(promise2, x, resolve, reject) {
	// 规范规定了 x 不能与 promise2 相等，这样会发生循环引用的问题
	// 如果 promise 和 x 指向同一对象，以 TypeError 为据因拒绝执行 promise
	if (promise2 === x) {
		return reject(new TypeError('Error'))
	}
	// 如果 x 为 Promise ，则使 promise 接受 x 的状态:
	// 1. 如果 x 处于等待态，Promise 需保持为等待态直至 x 被执行或拒绝
	// 2. 如果 x 处于执行态，用相同的值执行 promise
	// 3. 如果 x 处于拒绝态，用相同的据因拒绝 promise
	// 当然以上这些是规范需要我们判断的情况，实际上我们不判断状态也是可行的。
	if (x instanceof MyPromise) {
		console.log('isPromise')
		x.then(function (value) {
			resolutionProcedure(promise2, value, resolve, reject)
		}, reject)
		return
	}
	/**
	 * 接下来我们继续按照规范来实现"x 为对象或函数"的代码:
	 * - 首先创建一个变量 `called` 用于判断是否已经调用过函数
	 * - 然后判断 `x` 是否为对象或者函数，如果都不是的话，将 `x` 传入 `resolve` 中
	 * - 如果 `x` 是对象或者函数的话，先把 `x.then` 赋值给 `then`，然后判断 `then` 的类型，如果不是函数类型的话，就将 `x` 传入 `resolve` 中
	 * - 如果 `then` 是函数类型的话，就将 `x` 作为函数的作用域 `this` 调用之，并且传递两个回调函数作为参数，第一个参数叫做 `resolvePromise` ，第二个参数叫做 `rejectPromise`，两个回调函数都需要判断是否已经执行过函数，然后进行相应的逻辑
	 * - 以上代码在执行的过程中如果抛错了，将错误传入 `reject` 函数中
	 */
	let called = false
	if (x !== null && (typeof x === 'object' || typeof x === 'function')) {
		try {
			let then = x.then
			if (typeof then === 'function') {
				console.log('isThenable')
				then.call(
					x,
					y => {
						// resolvePromise
						if (called) return
						called = true
						resolutionProcedure(promise2, y, resolve, reject)
					},
					e => {
						// rejectPromise
						if (called) return
						called = true
						reject(e)
					}
				)
			} else {
				resolve(x)
			}
		} catch (e) {
			if (called) return
			called = true
			reject(e)
		}
	} else {
		resolve(x)
	}
}

/**
 * Then方法
 * 一个 promise 必须提供一个 then 方法以访问其当前值、终值和据因。
 * onFulfilled 和 onRejected 必须被作为函数调用（即没有 this 值）
 * then 方法可以被同一个 promise 调用多次，then 方法必须返回一个 promise 对象
 * 	- 当 promise 成功执行时，所有 onFulfilled 需按照其注册顺序依次回调
 * 	- 当 promise 被拒绝执行时，所有的 onRejected 需按照其注册顺序依次回调
 *
 * 注意：不论 promise1 被 reject 还是被 resolve 时 promise2 都会被 resolve，只有出现异常时才会被 rejected。
 * @param {*} onFulfilled 可选，如果 onFulfilled 不是函数，其必须被忽略
 * @param {*} onRejected 可选，如果 onRejected 不是函数，其必须被忽略
 * @returns
 */
MyPromise.prototype.then = function (onFulfilled, onRejected) {
	const that = this
	console.log('then', that.state)
	onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : v => v
	onRejected = typeof onRejected === 'function' ? onRejected : r => {
		throw r
	}
	if (that.state === PENDING) {
		// 每个 then 函数都需要返回一个新的 Promise 对象，该变量用于保存新的返回对象
		return (promise2 = new MyPromise((resolve, reject) => {
			that.resolvedCallbacks.push(() => {
				try {
					const x = onFulfilled(that.value) // onFulfilled第一个参数为 promise 的终值
					resolutionProcedure(promise2, x, resolve, reject)
				} catch (r) {
					reject(r)
				}
			})

			that.rejectedCallbacks.push(() => {
				try {
					const x = onRejected(that.value) // onRejected第一个参数为 promise 的据因
					resolutionProcedure(promise2, x, resolve, reject)
				} catch (r) {
					reject(r)
				}
			})
		}))
	}
	if (that.state === RESOLVED) {
		// 这段代码和判断等待态的逻辑基本一致，无非是传入的函数的函数体需要异步执行，这也是规范规定的
		return (promise2 = new MyPromise((resolve, reject) => {
			setTimeout(() => {
				try {
					const x = onFulfilled(that.value)
					resolutionProcedure(promise2, x, resolve, reject)
				} catch (reason) {
					reject(reason)
				}
			})
		}))
	}
	if (that.state === REJECTED) {
		return (promise2 = new MyPromise((resolve, reject) => {
			setTimeout(() => {
				try {
					const x = onRejected(that.value)
					resolutionProcedure(promise2, x, resolve, reject)
				} catch (reason) {
					reject(reason)
				}
			})
		}))
	}
}

function test(id) {
	return new MyPromise(((resolve, reject) => {
		setTimeout(() => {
			resolve({ test: id })
		}, 5000)
	}))
}

try {
	new MyPromise((resolve, reject) => {
		setTimeout(() => {
			// resolve(1)
			reject(1)
		}, 0)
	}).then(value => {
		console.log('success', value)
		return value
	}, error => {
		console.log('error', error)
		return 999
	}).then(value => {
		console.log('success2', value)
		return test(value ? ++value : 99)
	}).then(value => {
		console.log('success3', value)
		return value.test ? value.test + 1 : 0
	}).then(value => {
		console.log('success4', value)
	})
} catch (error) {
	console.log(error)
}
```


# 参考
- [【翻译】Promises/A+规范](https://www.ituring.com.cn/article/66566)
- [这一次，彻底弄懂 Promise 原理](https://juejin.cn/post/6844904063570542599)