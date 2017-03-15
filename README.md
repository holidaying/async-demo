# async-demo
关于一些处理异步请求的方法
### 常见的JavaScript场景
* 1.setInterval setTimeout这种计时器接口
* 2.http请求例如vue-resource、fetch、axios。等请求方式。

### 常见的现象
* 1、不能按照要求得到结果
* 2、想要得到结果，需要运用传统的回调函数，表示非常的繁琐。
```
function A（）{
       setTimeout(B(),200);
        C();
 
}
A()
执行顺序A->B->C

```
### 主要的方法
* 1、回调函数
```
f1();
f2();

function f1(callback){
　　　　setTimeout(function () {
　　　　　　// f1的任务代码
　　　　　　callback();
　　　　}, 1000);
　　}

f1(f2);
```
* 2、事件监听
```
f1.on('done', f2);
function f1(){
　　　　setTimeout(function () {
　　　　　　// f1的任务代码
　　　　　　f1.trigger('done');
　　　　}, 1000);
　}
f1();
```
* 3、发布/订阅
```
jQuery.subscribe("done", f2);
function f1(){
　　　　setTimeout(function () {
　　　　　　// f1的任务代码
　　　　　　jQuery.publish("done");
　　　　}, 1000);
}
f1();
jQuery.unsubscribe("done", f2);
```
* 4、Promises对象
```
f1().then(f2);
function f1(){
　　　　var dfd = $.Deferred();
　　　　setTimeout(function () {
　　　　　　// f1的任务代码
　　　　　　dfd.resolve();
　　　　}, 500);
　　　　return dfd.promise;
　}
f1();
```
> 前面四种都是比较传统，同时还依赖jQuery，用的时候扩展性和方便都不带劲[具体代码请查看这里](http://www.ruanyifeng.com/blog/2012/12/asynchronous%EF%BC%BFjavascript.html)，建议使用以下3中方式。
* 5、ES6 promise
* 6、ES6 generator
* 7、ES7 async和wait

