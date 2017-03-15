# async-demo
关于一些处理异步请求的方法
### 1、常见的JavaScript场景
* 1.setInterval setTimeout这种计时器接口
* 2.http请求例如vue-resource、fetch、axios。等请求方式。

### 2、常见的现象
* 1、不能按照要求得到结果
* 2、想要得到结果，需要运用传统的回调函数，表示非常的繁琐。
```
function A（）{
       setTimeout(B(),200);
        C();
 
}
A()
执行顺序A->B->C

var c="";
Vue.http.jsonp('http://ipinfo.io/json').then(function(data) {
    self.weatherData.city = data.data.city;
    self.weatherData.country = data.data.country;
    c= data.data.city;
    });
c=? 
```
### 3、主要的方法
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

### 4、实战项目
* 1、Promise
```
var promise = null;//定义全局变量
//new 一个Promise对象
promise = new Promise(function(resolve, reject) {
 Vue.http.jsonp('http://ipinfo.io/json').then(function(data)
      {
            if (data != null) {
                resolve(data);
            } else {
                reject("fail");
            }
        self.weatherData.city = data.data.city;
        self.weatherData.country = data.data.country;
        });
    })
//利用promise进行链式传递
 promise.then(function(data) {
                        var city = data.data.city+data.data.country;
                        Vue.http.jsonp(api + city + units + appid).then(function(data) {
                            that.weatherShow(data);
                        });
                    })
```

#### 总结

* 1、promise主要解决回调地狱问题
* 2、使得原本的多层级的嵌套代码，变成了链式调用
* 3、让代码更清晰，减少嵌套数
* 缺点：Promise 的最大问题是代码冗余，原来的任务被Promise 包装了一下，不管什么操作，一眼看去都是一堆 then，原来的语义变得很不清楚。
[demo]()

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2604175-d2bdf103d7746341.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 2、generator和yield
```
getLoc: function() {
                    var that = this;
                    Vue.http.jsonp('http://ipinfo.io/json').then(function(data) {
                        self.weatherData.city = data.data.city;
                        self.weatherData.country = data.data.country;
                        it.next(data.data);
                        //重点4
                    });
                },
                getCurrent: function(data) {
                    var that = this;
                    var api = "http://api.openweathermap.org/data/2.5/weather?q=";
                    var units = "&units=metric";
                    var appid = "&APPID=061f24cf3cde2f60644a8240302983f2"
                    var cb = "&callback=JSON_CALLBACK";
                    var city = data.city + ',' + data.country;
                    Vue.http.jsonp(api + city + units + appid).then(function(data) {
                        it.next(data);//重点4
                    });
//重点1
function* generator() {
                var data = yield object.getLoc();
                data = yield object.getCurrent(data);
                yield object.weatherShow(data);
            }
            var it = generator();//重点2
            it.next();//重点3
```

#### 总结

* 1、利用迭代器原理，解决异步问题
* 2、让代码更清晰，减少嵌套数。
* 缺点：虽然 Generator 函数将异步操作表示得很简洁，但是流程管理却不方便
[demo]()

* 3、async和wait
```
getLoc: function() {
                    var that = this;
                    return new Promise(function(resolve, reject) {
                        Vue.http.jsonp('http://ipinfo.io/json').then(function(data) {
                            self.weatherData.city = data.data.city;
                            self.weatherData.country = data.data.country;
                            resolve(data.data);
                        });
                    })
                },
                getCurrent: function(data) {
                    var that = this;
                    var api = "http://api.openweathermap.org/data/2.5/weather?q=";
                    var units = "&units=metric";
                    var appid = "&APPID=061f24cf3cde2f60644a8240302983f2"
                    var cb = "&callback=JSON_CALLBACK";
                    var city = data.city + ',' + data.country;
                    return new Promise(function(resolve, reject) {
                        Vue.http.jsonp(api + city + units + appid).then(function(data) {
                            resolve(data);
                            // that.weatherShow(data);
                        });
                    });

                },
//重点1
 async function test() {
                const v1 = await object.getLoc();
                const v2 = await object.getCurrent(v1);
                const v3 = await object.weatherShow(v2);
            }
            test();
```

#### 总结

* 1、解决异步问题
* 2、await后面不能再跟thunk函数，而必须跟一个Promise对象（因此，Promise才是异步的终极解决方案和未来）。跟其他类型的数据也OK，但是会直接同步执行，而不是异步。
* 3、让代码更清晰，减少嵌套数。
* 4、代码的易读性来将，async-await更加易读简介，也更加符合代码的语意。而且还不用引用第三方库，也无需学习Generator那一堆东西，使用成本非常低。
[demo]()





