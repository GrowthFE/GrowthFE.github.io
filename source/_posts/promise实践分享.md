---
title: Promise实践
---

## promise学习笔记

**一 JS异步：**

1、 回调函数

2、事件发布/订阅模式（观察者模式）

3、使用Promise对象

4、使用Generator函数

5、使用async函数

 …等等

Promise 是异步编程的一种解决方案。解决了代码缩进的金字塔问题

**二  实现方式：**

用 new Promise 方法创建promise对象

用.then 或 .catch 添加promise对象的处理函数

基本写法：

    
    
    const promise =newPromise(function(resolve, reject){
    // ... some code
    if(/* 异步操作成功 */){
            resolve(value);
    }else{
            reject(error);
    }
    });
    promise.then(res =>{}).catch(err=>{})
    

**三 三种状态**

![enter image description here](http://liubin.org/promises-book/Ch1_WhatsPromises/img/promise-states.png)

从*Pending*转换为*Fulfilled*或*Rejected*之后，  这个promise对象的状态就不会再发生任何变化

    
    var promise =newPromise(function(resolve){
        resolve(42);
        reject(30)
    });
    
    promise.then(function(value){
        console.log(value);
    }).catch(err =>{
        console.log(err)
    });
    

**四 用return 替代*resolve**

- 无法改变状态
- 
无法链式调用

    
    const testReturn =(a)=>{
        returnnewPromise((resolve,reject)=>{
            if(a){
                return'this is return';
                //resolve('true');
            }else{
               reject('false');
            }
        })
    }
    testReturn(true).then(res =>{console.log(res)})
    
拿不到return结果，且promise状态不会变，依然是pending状态。

**五 用throw** 代替 **reject**

promise对象也变为Rejected状态，但是throw Error不会被catch的

    
    const testReturn =(a)=>{
        returnnewPromise((resolve,reject)=>{
            if(a){
                thrownewError('error');
                //reject(('error')
            }
        })
    }
    testReturn(true).catch(res =>{console.log(res)})
    

**六 代码执行顺序问题**
new Promise是同步的，会马上执行function参数中的代码。等function参数执行完，new Promise才返回一个promise实例对象。这时候再调用then，其实是已经fullfill了

简单例子：

    
    
    var promise =newPromise(function(resolve){
        console.log("inner promise");// 1
        resolve(42);
    });
    promise.then(function(value){
        console.log(value);// 3
    });
    console.log("outer promise");// 2
    
    
    
    newPromise(resolve =>{
        console.log(1);
        resolve(3);
    Promise.resolve().then(()=> console.log(4))
    }).then(num =>{
        console.log(num)
    });
    console.log(2);
    

当promise遇到setTimeout, **resolve**会将**Promise.then**放在微任务队列中。而setTimeout 属于 macrotask 宏任务。

浏览器的一个事件循环中只有一个macrotask任务，可以有一个或多个microtask任务。

promise队列中任务执行完毕，再执行setTimeout的任务队列。

首先，setTimeout 被推进到 macrotask 队列(将在下一个macrotask中执行)中 ->先执行 macrotask 中的第一个任务（整个script中的同步代和promise 构造函数 ）->promise.then 回调被推进到 microtask 队列中 (此时，已经执行完了第一个 macrotask) , ->顺序执行所有的 microtask, (也就是 promise.then 的回调函数)。 此时，microtask 队列中的任务已经执行完毕，执行剩下的 macrotask 队列中的任务setTimeout.

    
    setTimeout(function(){
        console.log(4)
    },0);
    newPromise(function(resolve){
         console.log(1)
    for(var i=0; i<10000; i++){
             i==9999&& resolve(5)
    }
        console.log(2)
    })
    .then(function(res){
         console.log(res)
    });
     console.log(3);
    


另一种情况: resolve在setTimeout中 1->2->4(宏任务结束)->3(微任务结束)

    
    
    const testReturn =(a)=>{
    returnnewPromise((resolve,reject)=>{
        setTimeout(()=>{
             if(a){
                 resolve(3);
                 console.log(2);
            }else{
                reject('false');
             }
        })
            console.log(1);
        })
    }
    testReturn(true).then(str=>{
        console.log(str);
    // console.log(testReturn)
    }).catch(err=>{
        console.log('err: ',err);
    })
    


当有**Error**的时候，**Error**后面的代码不会被执行，但是**Promise**的结果依旧是**fulfilled**

    
    const testReturn =(a)=>{
        returnnewPromise((resolve,reject)=>{
            if(a){
                resolve('exec true');
                console.log('this will be exec');
                thrownewError('error');
                console.log('this will not be exec')
            }else{
                reject('false');
            }
         })
    }
    testReturn(true).then(str=>{
        console.log(str);
    // console.log(testReturn)
    }).catch(err=>{
        console.log('err: ',err);
    })
    

**七 cacth()  和 then(null, ..)**

1. 使用promise.then(onFulfilled, onRejected) :  在 onFulfilled 中发生异常的话，在 onRejected 中捕获不到这个异常。

2. 使用 promise.then(onFulfilled).catch(onRejected) 的情况下: then 中产生的异常能在 .catch 中捕获

所以，cacth()  和 then(null, ..)并不完全相同

    
    
    function throwError(value){
    // 抛出异常
        thrownewError(value);
    }
    // <1> onRejected不会被调用
    function badMain(onRejected){
        returnPromise.resolve(42).then(throwError, onRejected);
    }
    // <2> 有异常发生时onRejected会被调用
    function goodMain(onRejected){
        returnPromise.resolve(42).then(throwError).catch(onRejected);
    }
    // 运行示例
    badMain(function(){
        console.log("BAD");
    });
    goodMain(function(){
        console.log("GOOD");
    });
    

**八 chain 链式调用**

在 then 方法内部，我们可以做三件事：

1. return 一个 promise 对象
2. return 一个同步的值或者是 undefined
3. 同步的 throw 一个错误

传给每个then 方法的value 的值都是前一个promise对象通过return 返回的值。实现依赖于 then 方法每次都会创建并返回一个新的promise对象

![](http://imweb-io-1251594266.file.myqcloud.com/Fm7j8HLbuQlAZEwUG8G7imBLRJrh)

    
    var aPromise =newPromise(function(resolve){
        resolve(100);
    });
    aPromise.then(function(value){
        return value *2;
    });
    aPromise.then(function(value){
        return value *2;
    });
    aPromise.then(function(value){
        console.log("1: "+ value);// => 100
    })
    
    var bPromise =newPromise(function(resolve){
        resolve(100);
    });
    bPromise.then(function(value){
        return value *2;
    }).then(function(value){
        return value *2;
    }).then(function(value){
        console.log("2: "+ value);// => 100 * 2 * 2
    });
    

then的错误写法：promise.then 中产生的异常不会被外部捕获，此外，即使 then有返回值，也拿不到返回值。
    
    function badAsyncCall(){
    var promise =Promise.resolve('111');
        promise.then(function(){
    // 任意处理
        return'222';
    });
        return promise;
    }
    badAsyncCall().then(res =>{console.log(res)})
    
    function anAsyncCall(){
        var promise =Promise.resolve('111');
        return promise.then(function(){
    // 任意处理
         return'222';
    });
    }
    anAsyncCall().then(res =>{console.log(res)})
    

**九 then同时处理多个异步请求:**

Promise.all 在接收到的所有的对象promise都变为 FulFilled 或者 Rejected 状态之后才会继续进行后面的处理，

Promise.all()  以一个 promise 对象组成的数组为输入，返回另一个 promise 对象。这个对象的状态只会在数组中所有的 promise 对象的状态都变为 resolved 的时候才会变成 resolved。可以将其理解为异步的 for 循环。

Promise.race() 只要有一个promise对象进入 FulFilled 或者 Rejected 状态的话，就会继续进行后面的处理，那个率先改变的Promise 实例的返回值，就传递给回调函数。romise.race 在第一个promise对象变为Fulfilled之后，并不会取消其他promise对象的执行

    
    var winnerPromise =newPromise(function(resolve){
            setTimeout(function(){
                console.log('this is winner');
                resolve('this is winner');
    },4);
    });
    var loserPromise =newPromise(function(resolve){
            setTimeout(function(){
                console.log('this is loser');
                resolve('this is loser');
    },1000);
    });
    // 第一个promise变为resolve后程序停止
    Promise.race([winnerPromise, loserPromise]).then(function(value){
        console.log(value);// => 'this is winner'
    });
    
    // promise.all 解决 forEach问题
    var winnerPromise =newPromise(function(resolve){
          resolve([1,2,3,4,5,6])
    });
    //错误
    winnerPromise.then(function(result){
        result.forEach(function(row){
    return result
    });
    }).then(function(res){
        console.log(res)
        // I naively believe all docs have been removed() now!
    });
    //正确
    winnerPromise.then(function(result){
        returnPromise.all(result.map(function(row){
         return result
    }));
    }).then(function(res){
        console.log(res)
    // All docs have really been removed() now!
    })
    

**十 new Promise**的快捷方式
    
    function p (num){
        if(num >=0){
            returnPromise.resolve(num)
        }else{
            returnPromise.reject(newError('参数不能小于 0'))
        }
    }
    p.then(res=>{console.log(res)})
