# Promise

> author: huiyu

```
const p1 = new Promise((resolve, reject) => {
  // ... some code
  resolve('p1成功了');
})
​
const p2 = new Promise((resolve, reject) => {
  setTimeout(() => reject(new Error('request timeout')), 5000)
})
​
Promise.race([p1, p2])
.then(result => console.log(result))
.catch(e => console.log(e));
// p1成功了
```
​
### Promise.allSettled() 
Promise.allSettled()方法接受一组 Promise 实例作为参数，包装成一个新的 Promise 实例。只有等到所有这些参数实例都返回结果，不管是fulfilled还是rejected，包装实例才会结束。该方法由 ES2020 引入。
​
```
const p1 = Promise.resolve(42);
const p2 = Promise.reject(-1);
​
Promise.allSettled([p1, p2])
.then(results => console.log(results));
// [
//    { status: 'fulfilled', value: 42 },
//    { status: 'rejected', reason: -1 }
// ]
```
​
### Promise.any()
Promise.any()方法接受一组 Promise 实例作为参数，包装成一个新的 Promise 实例。只要参数实例有一个变成fulfilled状态，包装实例就会变成fulfilled状态；如果所有参数实例都变成rejected状态，包装实例就会变成rejected状态。该方法目前是一个第三阶段的提案 。
​
```
const p1 = Promise.resolve(0);
const p2 = Promise.reject(-1);
const p3 = Promise.reject(-2);
​
Promise.any([p1, p2, p3])
.then(result => console.log(result))
.catch(e => console.log(e));
// 0
​
Promise.any([p2, p3])
.then(result => console.log(result))
.catch(e => console.log(e));
// AggregateError: All promises were rejected
​
```
### Promise.try()
不知道或者不想区分，函数f是同步函数还是异步操作，但是想用 Promise 来处理它。因为这样就可以不管f是否包含异步操作，都用then方法指定下一步流程，用catch方法处理f抛出的错误。
```
const f = () => console.log('now');
Promise.resolve().then(f);
console.log('next');
// next
// now
​
const f = () => console.log('now');
Promise.try(f);
console.log('next');
// now
// next
​
```
​