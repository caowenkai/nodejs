# nodejs
浅谈promise与异步编程
前言
在说promise之前，不得不先介绍一下异步编程的知识；

javascript作为一门web而生的语言，他可以响应用户的的交互，点击等各种按键操作。Node.js的出现让javascript在的异步操作运用的更加流行起来，但是程序使用异步编程开发以后，javascript的事件和回调函数就开始无法满足更多复杂的事情（后面会说到）。而这也为后面的promise的出现做了很好的铺垫。

异步编程背景
按照es6里面的原话说，JavaScript引擎是基于单线程事件循环的概念构建的。相反的是java，c++这些多线程的语言来讲，当多个代码块同时访问并改变状态时，程序是很难维护并保证状态不会出错。

而JavaScript引擎的执行在同一时刻只会执行一个代码块。所以在准备执行的时候，每段代码都会被放到任务列表中去，每当引擎中的代码执行完以后事件环(event loop)就会执行下一个任务。然后，队列里面的任务会依次从第一个执行到最后一个。

事件模型和回调函数
事件模型：
普通的一个点击事件的执行如下

let button = document.getElementById("my-btn");

button.onclick = function(event){

console.log('Clicked');

};

在这段普通的代码中单击以后就会执行console.log的内容，这个事件也会同上面的原理一样被加入到队列中去，但是一旦点击事件onclick没有绑定到button上面的话，就不会发生任何事情。所以button必须要绑定onclick，所以尽管事件模型适用于响应用户的交互和完成这些低频的功能，但是对于比较复杂的需求来说就太死板了。

回调函数
如下是一个简单的回调模型：

readFile("example.txt",function(err, contents){

if(err){

throw err;

}

console.log(contents);

});

console.log("Hi")

上面的例子输出结果是先输出Hi，然后再读取example.txt

解释一下为什么会是这样：由于使用了回调模式，readFile函数会立即执行，但是读磁盘上面的文件的时候就会暂停，而是立即去执行console.log('Hi')；只有当readFile执行结束的时候，才会向任务队列结尾添加添加新的任务，该任务包含回调函数以及相应的参数，当队列的所有任务完成以后才会执行该任务，并且最终执行console.log(contents);

所以回调模式相对于上面的传统事件模型相比较更加灵活，因为它在readFile里面额已添加多个回调。类似if(err)可以写多个；

但是：回调函数在这里就出现了问题。看下面的代码

readFile("example.txt",function(err, contents){

if(err){

throw err;

}

writeFile("example.txt",function(){

if(err){

throw err;

}

console.log("written");

})

});

这里执行的顺序就是readFile函数执行完成以后，任务队列会多出任务，然后再执行writeFile函数，当wrieFile执行完成以后，就会向队列中添加一个任务。你会发现如果再在writeFile里面嵌套回到函数，就会进入传说中的回调地狱中去。这就导致js执行起来大部分时间浪费在回调的请求上面，而这些promise能够很好的解决这个问题。

promise的基本知识
什么是promise？看下面最简单的一个例子。

let promise = readFile("example.txt");

//readFile承诺将在未来的某一个时刻完成

这段代码的写法，就是根本不会去关心如何处理结果，不会去立马读取example.txt文件而是在执行完成readFile()函数以后会返回一个Promise对象，这个对象会有自己的生命周期，而success函数或者fail函数也是由这个promise对象的生命周期来决定。

借用廖雪峰的那句话说就是——古人云：“君子一诺千金”，这种“承诺将来会执行”的对象在JavaScript中称为Promise对象。

promise的生命周期的几个状态
进行中：pending（也是unsettled）

异步操作结束：settled

操作结束以后进入两个状态：

Fulfilled异步操作完成

Rejected未能成功完成



内部属性：
其中内部属性[[PromiseState]]表示promise的三种状态：pending，fulfilled，rejected

调用方法：
而这三个方法只能通过then()的方法来采取特定的行动，

所有的promise都有then方法，他接受两个参数，

1.fulfiled状态下要调用的函数（完成函数fulfilled function）

2.rejected状态下要调用的函数（拒绝函数rejection function）

所有成功失败的附加的数据都会传给这两个函数

（then未完待续，最近没时间，等空闲的时候再补上）

Promise.all方法
Promise.all只会接受一个参数并返回一个promise，而且该参数是一个含有多个受监视器Promise的可迭代对象（比如一个数组），只有当上面所有promise都被解决以后返回的promise才会被解决，只有当可迭代的对象中所有的promise被完成以后返回的promise才会被完成。如果下的例子

let p1 =new Promise(function (resolve,reject) {

resolve(42);

});

let p2 =new Promise(function (resolve,reject) {

resolve(43);

});

let p3 =new Promise(function (resolve,reject) {

resolve(44);

});

let p4 =Promise.all(p1,p2,p3);

p4.then(function (value) {

console.log(Array.isArray(value)); //true

console.log(value[0]); //42

console.log(value[1]); //42

console.log(value[2]); //42

})

上面代码中只有当promise p1,p2,p3都处于完成状态以后，p4才会被完成。所有传入promise.all()方法的promise只要有一个被拒绝，那么返回的promise没等所有promise都完成就立马被拒绝了。



promise.race()方法
和all方法切好相反，Promise.race()方法不用像all一样必须满足每一个都返回promise，这里只需要有一个Promise被解决返回的Promise就被解决

let p1 =new Promise.resolve(42);

let p2 =new Promise(function (resolve,reject) {

resolve(43);

});

let p3 =new Promise(function (resolve,reject) {

resolve(44);

});

let p4 =Promise.all(p1,p2,p3);

p4.then(function (value){

console.log(value); //42

})

小结
promise还有then()一集resolve，reject没有详细介绍，后面会慢慢补上，最近有点忙。加油吧

