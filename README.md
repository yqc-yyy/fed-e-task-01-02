//lodash 代码题--------------------------

1.
 let isLastInStock=fp.flowRight(fp.prop("in_stock"),fp.last);

2.
 let isLastInStock=fp.flowRight(fp.prop("name"),fp.first);

3.
 let averageDollarValue=fp.flowRight(_average,fp.map(fp.prop("dollar_value")))

4.
 let sanitizeNames=fp.flowRight(_underscore,fp.toLower);


//函子---------------------------
1.
 let ex1=x=>{
    return fp.map(f=>{
      return fp.add(f,x)
    },maybe._value)
}

2.
 let ex2=()=>{
    return fp.first(xs._value)
}

3.
 let ex3=()=>{
    return fp.first(safeProp("name",user)._value)
}

4.
let ex4=function(n){
   return Maybe.of(n)
    .map(x=>{
        return parseInt(x)
    })._value
}


//手写promise代码
/*
   1.promise 是一个类，在执行的时候需要传递进去一个执行器，执行器会立即执行
   2.promise 中有三种状态，分别是成功(fulfilled)、失败(rejected)、等待(pending)
     pending-->fulfilled/rejected 且状态一旦确定就不可更改
   3.resolve和reject函数是用来更改状态的
     resolve：fulfilled
     reject：rejected
   4.then方法内部做的事情就是判断状态，如果状态是成功，调用成功回调，如果状态是失败，调用失败回调
     因为每个promise对象都有then方法，所以then方法是被定义在原型对象中的
   5.then成功回调有一个参数，代表成功之后的值， 
     then失败回调有一个参数，代表失败之后的原因
     存在同步和异步情况
   6.同一个promise对象下的then方法是可以被多次调用的
   7.then方法是可以被链式调用的，后面then方法的回调函数拿到的值是上一个then方法的回调函数的返回值
     1>.实现then方法的链式调用，必须返回一个promise对象
     2>.需要调用resolve方法把上一个then的返回值传递给下一个then方法的回调函数，
   8.then方法的返回值可以是一个普通值或者是一个promise对象,如果是普通值则调用resolve方法，
     如果是promise对象，则根据promise对象的返回结果决定调用resolve还是reject
   9.then方法链式调用如何识别promise对象自返回
  10.当promsie对象执行器中的代码在执行过程中发生错误的时候，把promsie的状态变成失败
     当then方法的回调函数在执行过程中发生错误的时候，这个错误要在下一个then方法中捕获到
  11.then方法的参数是可选的
  12.all方法用来解决异步并发问题，按照异步代码调用的顺序得到异步代码执行的结果
     接收一个数组作为参数，数组里元素的顺序则是执行的顺序,并返回一个promise对象
     如果all方法中所有的promise对象的状态都是成功，则返回的promsie对象的状态也是成功的
     如果all方法中所有的promise对象的状态有一个是失败的，则返回的promsie对象的状态就是失败的
  13.resolve是将给定的值转化成promsie对象且包裹给定的值
  14.finally方法无论当前promise的最终状态是成功或者失败，finally中的回调函数始终会被执行一次
     在finally方法的后边可以链式调用then方法拿到当前这个peomise对象的返回结果
  15.catch方法是处理当前promise对象的最终状态为失败的情况
  16.race方法接收一个数组作为参数，数组里元素的顺序则是执行的顺序,并返回一个promise对象
     当数组中有一个对象（最早改变状态）成功或失败时，就改变自身的状态(成功或失败)，并执行相应的回调函数。
*/

//把状态定义成常量是为了复用
const PENDING = "opending" // 等待
const FULFILLED = "fulfilled" //成功
const REJECTED = "rejected" //失败

class MyPromise {
    constructor(executor) {
        try {
            executor(this.resolve, this.reject)
        } catch (e) {
            //当promsie对象执行器中的代码在执行过程中发生错误的时候，把promsie的状态变成失败
            this.reject(e)
        }

    }
    //promise状态,默认为PENDING
    status = PENDING;
    //成功之后的值
    value = undefined
    //失败后的原因
    reason = undefined

    //成功回调---默认值为数组是为了解决同一个promise对象下的then方法被多次调用的情况
    successCallback = []
    //失败回调
    failCallback = [];


    //使用箭头函数是为了让this指向promis对象
    resolve = value => {
        //判断状态是否为PENDING,如果状态不是OPENDING,组织程序向下执行
        if (this.status !== PENDING) return;
        //将状态更改为成功
        this.status = FULFILLED;
        //保存成功之后的值
        this.value = value;
        //判断成功回调是否存在，如果存在则调用
        //  this.successCallback && this.successCallback(this.value)
        while (this.successCallback.length) this.successCallback.shift()()
    }

    reject = reason => {
        //判断状态是否为PENDING,如果状态不是OPENDING,组织程序向下执行
        if (this.status !== PENDING) return;
        //将状态更改为失败
        this.status = REJECTED;
        //保存失败后的原因
        this.reason = reason;
        //判断失败回调是否存在，如果存在则调用
        //  this.failCallback && this.failCallback(this.reason)
        while (this.failCallback.length) this.failCallback.shift()()
    }

    then(successCallback, failCallback) {

        //判断successCallback是否存在
        successCallback = successCallback ? successCallback : value => value;
        //判断failCallback是否存在
        failCallback = failCallback ? failCallback : reason => { throw reason };

        //创建promise对象，
        let promise2 = new MyPromise((resolve, reject) => {//执行器立即执行
            //判断状态
            if (this.status === FULFILLED) {
                setTimeout(() => {
                    try {
                        let x = successCallback(this.value);
                        // resolve(x);//通过调用promise的resolve方法把值传递给下一个then方法
                        //判断x的值是普通值还是promise对象
                        //如果是普通值，直接调用resolve
                        //如果是promise对象则查看promise对象的返回结果
                        //再根据promise对象返回的结果，决定调用resolve还是调用rejected
                        //只有异步才能获取到promise2
                        resolvePromise(promise2, x, resolve, reject)
                    } catch (e) {
                        //当then方法的回调函数在执行过程中发生错误的时候，这个错误要在下一个then方法中捕获到
                        reject(e)//下一个then方法的reject方法
                    }

                }, 0)
            } else if (this.status === REJECTED) {
                setTimeout(() => {
                    try {
                        let x = failCallback(this.reason);
                        //判断x的值是普通值还是promise对象
                        //如果是普通值，直接调用resolve
                        //如果是promise对象则查看promise对象的返回结果
                        //再根据promise对象返回的结果，决定调用resolve还是调用rejected
                        //只有异步才能获取到promise2
                        resolvePromise(promise2, x, resolve, reject)
                    } catch (e) {
                        //当then方法的回调函数在执行过程中发生错误的时候，这个错误要在下一个then方法中捕获到
                        reject(e)//下一个then方法的reject方法
                    }

                }, 0)
            } else {//处理异步调用情况
                //等待
                //将成功回调和失败回调存储起来
                this.successCallback.push(() => {
                    setTimeout(() => {
                        try {
                            let x = successCallback(this.value);
                            // resolve(x);//通过调用promise的resolve方法把值传递给下一个then方法
                            //判断x的值是普通值还是promise对象
                            //如果是普通值，直接调用resolve
                            //如果是promise对象则查看promise对象的返回结果
                            //再根据promise对象返回的结果，决定调用resolve还是调用rejected
                            //只有异步才能获取到promise2
                            resolvePromise(promise2, x, resolve, reject)
                        } catch (e) {
                            //当then方法的回调函数在执行过程中发生错误的时候，这个错误要在下一个then方法中捕获到
                            reject(e)//下一个then方法的reject方法
                        }

                    }, 0)
                });
                this.failCallback.push(() => {
                    setTimeout(() => {
                        try {
                            let x = failCallback(this.reason);
                            //判断x的值是普通值还是promise对象
                            //如果是普通值，直接调用resolve
                            //如果是promise对象则查看promise对象的返回结果
                            //再根据promise对象返回的结果，决定调用resolve还是调用rejected
                            //只有异步才能获取到promise2
                            resolvePromise(promise2, x, resolve, reject)
                        } catch (e) {
                            //当then方法的回调函数在执行过程中发生错误的时候，这个错误要在下一个then方法中捕获到
                            reject(e)//下一个then方法的reject方法
                        }

                    }, 0)
                });
            }
        });


        return promise2;//返回一个promise对象，用来实现then方法的链式调用
    }

    finally(callback) {
        return this.then(value => {
            return MyPromise.resolve(callback().then(() => value))
        }, reason => {
            return MyPromise.resolve(callback().then(() => { throw reason }))
        })
    }

    catch(failCallback) {
        return this.then(undefined, failCallback);
    }

    static all(array) {
        //判断参数是不是数组
        if (!Array.isArray(array)) {
            throw new TypeError('You must pass array')
        }

        //声明一个存放结果的数组
        let result = [];
        let index = 0;

        return new MyPromise((resolve, reject) => {
            //定义一个将结果放到数组中的方法
            function addData(key, value) {
                result[key] = value;
                index++;
                //如果index等于arry的长度，则说明数组里的方法全部执行完毕
                if (index === array.length) {
                    resolve(result)
                }
            }
            for (let i = 0, len = arry.length; i < len; i++) {
                let current = array[i];
                if (current instanceof MyPromise) {
                    //promsie对象
                    current.then(value => addData(i, value), reason => reject(reason))
                } else {
                    //普通值
                    addData(i, array[i])
                }
            }

        })
    }

    static race(array) {
        //判断参数是不是数组
        if (!Array.isArray(array)) {
            throw new TypeError('You must pass array')
        }

        return new MyPromise((resolve, reject) => {
            for (let i = 0, len = arry.length; i < len; i++) {
                arry[i].then(resolver, reject)
            }
        })

    }

    static resolve(value) {
        if (value instanceof MyPromise) return value;
        return new MyPromise(resolve => resolve(value));
    }
}

function resolvePromise(promise2, x, resolve, reject) {
    //判断返回的promise对象是不是当前then方法的promise对象
    if (promise2 === x) {
        return reject(new TypeError("Chaining cycle detected for promise #<Promsie>"))
    }
    if (x instanceof MyPromise) {
        //promise对象
        // x.then(value=>resolve(value),reason=>reject(reason))
        x.then(resolve, reject)
    } else {
        //普通值
        resolve(x)
    }
}



1.引用计数的原理
---设置引用数，判断当前引用数是否为0
---引用计数器
---引用关系改变时修改引用数字  （引用关系增加时引用数字加1，引用关系减少时引用数字减1）
---引用数字为0时立即回收

----优点
--1、可以立即回收垃圾对象
--2、减少程序卡顿时间

---缺点
--1、无法回收被循环引用的对象
--2、资源消耗大（需要修改引用计数）

2.标记整理的工作流程
--1、可以看做是标记清除的增强
--2、遍历所有对象找标记活动对象
--3、清除阶段会先执行整理，移动对象位置(在地址上产生连续)

----优点
--1、减少碎片化空间
----缺点
--1、不能立即回收垃圾对象

3.V8新生代垃圾回收
--1、内存空间一分为二
--2、小空间用于存储新生代对象（64位32M/32位16M）
--3、新生代指的是存活时间较短的对象


------回收实现
---1、回收过程采用复制算法+标记整理
---2、新生代内存区分为两个等大的小空间
---3、使用空间为From，空闲空间为To
---4、活动对象存储于From空间
---5、标记整理后将活动对象拷贝至To空间
---6、From和To交换空间完成释放


------回收细节
---1、拷贝过程中可能出现晋升，（一个对象同时存在于新生代和老生代空间就是导致对象晋升）
---2、晋升就是将新生代对象移至老生代空间
---3、经过一轮GC还存活的新生代对象将晋升至老生代空间
---4、To空间的使用率超过25%
（3和4会导致晋升）

4.增量标记算法在何时使用及工作原理
---在执行老年代垃圾回收的时候触发增量标记算法
---原理
--1、将当前一整段的垃圾回收操作拆分成多个小部分组合着去完整当前整个的垃圾回收，实现垃圾回收和程序执行交替实现，减少时间消耗
